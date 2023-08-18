# cmd hook options

I volunteered to provide a solution for [issue 1213](https://github.com/Sceptre/sceptre/issues/1213).

See [Slack thread](https://og-aws.slack.com/archives/C01JNN8RGBB/p1689931787395429) for advice from Jon Falkenstein.

> So there are two places you should make this change, one is in the separate repo for the rcmd resolver, and the other is in the core sceptre repo for the cmd hook. I'd recommend starting in the core sceptre repo with the hook.
>
> Take a look at the cmd hook in sceptre/hooks. It defines a run method. If you evaluate self.argument, you can treat string arguments just like the current way it runs, but dict arguments can work like we've discussed.
>
> I'd recommend getting up a draft PR before you put a ton of work into it and we can discuss from there. Before we actually merge, you'll need to make sure there are unit tests and docs, but we can start discussion on effectively a prototype

## Create a basic hook setup

Clone my fork of the Sceptre repo. Install the dependencies with Poetry.

Start a Poetry shell.

Create a minimal project to test a hook. This hook just echos a message.

```bash
put() {
    local path="$1"
    local input
    read -d \0 input

    mkdir --parents "$(dirname "$path")"
    cat > "$path" <<< "$input"
}

tmp="$(mktemp --dir)"

put "$tmp/templates/test.yaml" <<"EOF"
Resources:
  Dummy:
    Type: AWS::CloudFormation::WaitConditionHandle
EOF

put "$tmp/config/test.yaml" <<"EOF"
template:
  type: file
  path: test.yaml
hooks:
    before_update:
        - !cmd 'echo "Hello, world!"'
EOF

put "$tmp/config/config.yaml" <<"EOF"
project_code: test
region: eu-west-1
EOF
```

Use my sandbox profile.

```console
$ .env aws-sandbox
$ use-profile
sandbox.Organization-Management-Account.480783779961.AdministratorAccess
```

Check that the project works with Sceptre.

```bash
(cd "$tmp"; sceptre launch --yes .)
```

The output shows that the hook executes.

```text
/home/isme/Repos/sceptre/sceptre/config/reader.py:145: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import iter_entry_points
[2023-08-06 21:33:58] - test - Launching Stack
[2023-08-06 21:33:58] - test - Stack is in the CREATE_COMPLETE state
Hello, world!
[2023-08-06 21:33:58] - test - Updating Stack
[2023-08-06 21:33:59] - test - No updates to perform.
```

## Run pre-commit checks

I try to commit this documentation.

But pre-commit fails on black.

```console
$ pre-commit run black --all-files
[INFO] Installing environment for https://github.com/psf/black.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
An unexpected error has occurred: CalledProcessError: command: ('/home/isme/.cache/pypoetry/virtualenvs/sceptre-b3JPbPox-py3.8/bin/python', '-mvirtualenv', '/home/isme/.cache/pre-commit/repow5d0hgf1/py_env-python3.10', '-p', 'python3.10')
return code: 1
stdout:
    RuntimeError: failed to find interpreter for Builtin discover of python_spec='python3.10'

stderr: (none)
Check the log at /home/isme/.cache/pre-commit/pre-commit.log
```

For now I commit with `--no-verify` to bypass pre-commit.

The hook has a line that requires Python version 3.10.

```yaml
  - repo: https://github.com/psf/black
    rev: 23.1.0
    hooks:
      - id: black
        # It is recommended to specify the latest version of Python
        # supported by your project here, or alternatively use
        # pre-commit's default_language_version, see
        # https://pre-commit.com/#top_level-default_language_version
        language_version: python3.10
```

When I comment out the line, the hook works.

```console
$ pre-commit run black --all-files
[INFO] Installing environment for https://github.com/psf/black.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
black....................................................................Passed
```

I thought that pre-commit ran hooks in Docker containers, but that must be wrong. It apparently uses a virtual environment for Python hooks.

I find the same problem with black reported in [the pre-commit project](https://github.com/pre-commit/pre-commit/issues/1761).

Anothony Sottile, the pre-commit maintainer, also uses Ubuntu and maintains the deadsnakes PPA for Python packages.

Use [the deadsnakes PPA](https://launchpad.net/~deadsnakes/+archive/ubuntu/ppa) to install a version of Python 3.10.

```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.10
```

Now restore the `language_version` line in the hook and try again.

The hook works as expected.

## Read the contributing guide

The [contributing guide](https://github.com/Sceptre/sceptre/blob/master/CONTRIBUTING.md) shows pre-commit and tox and various other tasks I should take care of as a developer.

## Run unit tests with tox

```bash
poetry run tox
```

To pass all the unit tests I need to install all the supported Python versions. Python 3.10 was alreay installed for black.

```bash
sudo apt install python3.7 python3.9 python3.11
```

Tox still fails on Python 3.7 because of a missing depdency.

```bash
poetry run tox -e py37
```

```console
ModuleNotFoundError: No module named 'distutils.cmd'
```

[Ask Ubuntu](https://askubuntu.com/a/1296996/143624) gives a solution to this exact problem. Install the missing distutils package from deadsnakes.

```bash
sudo apt install python3.7-distutils
```

Now the tests pass on Python 3.7.

```text
  py37: commands succeeded
```

## Ask question about version difference between black and tox

See [GitHub issue 1370](https://github.com/Sceptre/sceptre/issues/1370).

## Read code for cmd hook

The cmd hook appears to be implemented at `sceptre/hooks/cmd.py`. It's short.

The `Cmd` class is basically a wrapper for [`subprocess.check_call`](https://docs.python.org/3/library/subprocess.html#subprocess.check_call).

> Run command with arguments. Wait for command to complete. If the return code was zero then return, otherwise raise CalledProcessError.

The `run` method take no arguments, so the command string must be handled in the initializer.

`__init__` just delegates to the superclass `Hook`.

`Hook` has no `__init__` method.

`Hook`'s superclass is `CustomYamlTagBase`.

`CustomYamlTagBase`'s docstring starts "A base class for custom Yaml Elements (i.e. hooks and resolvers).".

`CustomYamlTagBase`'s `__init__` method takes two keyword arguments `argument` and `stack`. It assigns `argument` to `self._argument` and assigns `stack` to `self.stack`. The class defines an `argument` property that either just returns `self._argument` or calls `self._resolve_argument`.

`Cmd`'s `run` method passes `self.argument` to `check_call`.

`Cmd.run` also passes environment variables to `check_call` after retrieving them from `self.stack.connection_manager.create_session_environment_variables()`.

The `asg_scaling_processes` hook also takes a string argument.

Review Jon Falkenstein's notes in the issue about a possible solution. Now it makes more sense!

> the `cmd` hook simply executes `subprocess.check_call(self.argument, shell=True)`
>
> I can imagine a way for that hook to evaluate `self.argument` and decide:
>
> * If `self.argument` is a `str`, it would execute the command just like it does now
>
> * If `self.argument` is a dict, it would let you specify subprocess kwargs, like:
>   ```yaml
>       before_update:
>           - !cmd
>               args: "echo I'm using bash!"
>               executable: "/bin/bash"
>   ```

Next steps:

* Write a hook with dict args to see how it behaves (should fail)
* Attempt to modify the hook to handle `args` and `executable` arguments
* Put up a draft PR for discussion with Sceptre dev team

## Write a hook with dict args

There is already a lot of test material in the tests folder. I'll ignore it for now and set up just what I need for a proof of concept.

I will trim all the deprecation warnings and full stace traces from the outputs except when relevant.

Add args to the hook.

```bash
put "$tmp/config/test.yaml" <<"EOF"
template:
  type: file
  path: test.yaml
hooks:
    before_update:
        - !cmd
            args: 'echo "Hello, world!"'
            executable: "/bin/bash"
EOF
```

See how Sceptre behaves. Number the output lines for easier referencing.

```bash
(cd "$tmp"; sceptre launch --yes .) 2>&1 | cat -n
```

```text
     3	[2023-08-17 19:33:50] - test - Launching Stack
     4	[2023-08-17 19:33:50] - test - Stack is in the CREATE_COMPLETE state
     5	executable: 1: args: not found
     6	Traceback (most recent call last):
    57	subprocess.CalledProcessError: Command '{'args': 'echo "Hello, world!"', 'executable': '/bin/bash'}' returned non-zero exit status 127.
```

Line 5 is the shell output from the hook command.

Line 57 is the Python error that halts Sceptre.

It's unclear exactly what is being sent to the shell for execution.

The shell is `/bin/sh`, which on my system is `/bin/dash`.

Line 57 suggests it is a line like this (input escaped for `bash` before passing to `dash`):

```console
$ /bin/sh -c "{'args': 'echo \"Hello, world\!\"', 'executable': '/bin/bash'}"
/bin/sh: 1: {args:: not found
```

The output has the `1: ` and the `: not found` parts. That looks like a line number and an error message from dash. But the other parts, my input, the interpreter name and the command, are different.

This invocation gives the same error message on line 5.

```console
$ /bin/sh -c 'args' executable
executable: 1: args: not found
```

Check the Cmd hook code.

The object is just passed to `check_call`, so Sceptre does whatever it does.

```python
subprocess.check_call(self.argument, shell=True, env=envs)
```

Execute the same in a Python shell.

```python
import subprocess
subprocess.check_call({'args': 'echo \"Hello, world\!\"', 'executable': '/bin/bash'}, shell=True)
```

I get the same output from dash and the same Python exception.

(Also I get a strange error from requests. What's it doing here!?)

```text
executable: 1: args: not found
/usr/lib/python3/dist-packages/requests/__init__.py:89: RequestsDependencyWarning: urllib3 (1.26.16) or chardet (3.0.4) doesn't match a supported version!
  warnings.warn("urllib3 ({}) or chardet ({}) doesn't match a supported "
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python3.8/subprocess.py", line 364, in check_call
    raise CalledProcessError(retcode, cmd)
subprocess.CalledProcessError: Command '{'args': 'echo "Hello, world\\!"', 'executable': '/bin/bash'}' returned non-zero exit status 127.
```

If I pass a simple string I get the output from the command and then the return value of the `check_call` function, which is the exit code of dash.

```python
subprocess.check_call('echo "Hello, world!"', shell=True)
```

```text
Hello, world!
0
```

Next step: rewrite the Cmd hook to handle both of these types of input.
