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

```bash
poetry install --all-extras -v
```

Start a Poetry shell.

Create a minimal project to test a hook. This hook just echos a message.

(Note: there's a bug in `put`. I wanted to read until the null byte, but as written it reads until the first `0` character. To read until the first null byte, I think I need to use `read -d ''`. I read that it's not [explicitly documented](https://unix.stackexchange.com/questions/174016/how-do-i-use-null-bytes-in-bash), but as it takes the first character of the delimiter then it becomes the null byte.)

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

Try to make the same calls using the Cmd hook interface.

It depends on Stack, which depends on a stack config (not sure about the type here but `dict` seems to work) and on valid session credentials (discover that by running without a default profile and get `sceptre.exceptions.InvalidAWSCredentialsError: Session credentials were not found. Profile: None. Region: region1.` or by running with an expired session and getting `botocore.exceptions.UnauthorizedSSOTokenError: The SSO session associated with this profile has expired or is otherwise invalid. To refresh this SSO session run aws sso login with the corresponding profile.`).

Minimal setup to run the hook in Python.

```python
from sceptre.hooks.cmd import Cmd
from sceptre.stack import Stack
stack = Stack("stack1", "project1", "region1", template_handler_config={"template": "path.yaml"})
cmd = Cmd("echo 'Hello, world!'", stack)
cmd.run()
```

```text
Hello, world!
```

---

Try to use a shell other than `/bin/sh` via `check_call`.

`check_call` delegates to [`Popen`](https://docs.python.org/3/library/subprocess.html#subprocess.Popen).

> On POSIX with `shell=True`, the shell defaults to `/bin/sh.`

>  If `shell=True`, on POSIX the _executable_ argument specifies a replacement shell for the default `/bin/sh`.

`check_call`'s signature includes `**other_popen_kwargs`. It supports the `executable` argument directly.

```python
subprocess.check_call('echo "Hello, world!"', shell=True, executable='/bin/bash')
```

```text
Hello, world!
0
```

In this way I can use the `[[` syntax that prompted the GitHub issue.

```python
subprocess.check_call('if [[ 1 > 0 ]]; then echo "Hello"; fi', shell=True, executable='/bin/bash')
```

```text
Hello
0
```

Without setting executable, I get the same error from `/bin/sh`. The return code is still 0, though.

```python
subprocess.check_call('if [[ 1 > 0 ]]; then echo "Hello"; fi', shell=True)
```

```text
/bin/sh: 1: [[: not found
0
```

What if I want to set arguments to the shell executable? This doesn't work.

```python
subprocess.check_call('if [[ 1 > 0 ]]; then echo "Hello"; fi', shell=True, executable="/bin/sh -e")
```

```text
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python3.8/subprocess.py", line 359, in check_call
    retcode = call(*popenargs, **kwargs)
  File "/usr/lib/python3.8/subprocess.py", line 340, in call
    with Popen(*popenargs, **kwargs) as p:
  File "/usr/lib/python3.8/subprocess.py", line 858, in __init__
    self._execute_child(args, executable, preexec_fn, close_fds,
  File "/usr/lib/python3.8/subprocess.py", line 1704, in _execute_child
    raise child_exception_type(errno_num, err_msg, err_filename)
FileNotFoundError: [Errno 2] No such file or directory: '/bin/sh -e'
```

In this case it looks like the easier thing to do is just ignore shell mode and build the command string directly. Se this [Stack Overflow](https://stackoverflow.com/a/15782232/111424) answer for inspiration.

```python
shell = "bash -e"
cmd = 'echo "Hello, world!"'
args = shlex.split(shell) + ["-c", cmd]
subprocess.check_call(args)
```

I'm not sure whether it's okay to hard-code the `"-c"` part. It's part of `sh''s [Posix standard](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html), so maybe it is.

---

Now I'm ready to implement the new version.

Rewrite the Cmd hook to handle the dict parameters. The call becomes:

```python
subprocess.check_call(
    self.argument["args"],
    shell=True,
    env=envs,
    executable=self.argument["executable"]
)
```

When I run sceptre with the new parameters in the hook YAML, it works.

```text
     3	[2023-08-27 17:46:07] - test - Launching Stack
     4	[2023-08-27 17:46:09] - test - Stack is in the CREATE_COMPLETE state
     5	Hello, world!
     6	[2023-08-27 17:46:09] - test - Updating Stack
     7	[2023-08-27 17:46:09] - test - No updates to perform.
```

Add another stack config that uses the original syntax.

```bash
put "$tmp/config/test2.yaml" <<"EOF"
template:
  type: file
  path: test.yaml
hooks:
    before_update:
        - !cmd 'echo "Hello, world!"'
EOF
```

The two configs are now side-by-side.

```bash
tree "$tmp"
```

```text
/tmp/tmp.XXT21f8f4F
├── config
│   ├── config.yaml
│   ├── test2.yaml
│   └── test.yaml
└── templates
    └── test.yaml
```

test2 does create on the first launch because the hook triggers only before update.

On the second launch, I get an error:

```text
     3	[2023-08-27 18:14:16] - test - Launching Stack
     4	[2023-08-27 18:14:16] - test2 - Launching Stack
     5	[2023-08-27 18:14:17] - test - Stack is in the CREATE_COMPLETE state
     6	Hello, world!
     7	[2023-08-27 18:14:17] - test2 - Stack is in the CREATE_COMPLETE state
     8	[2023-08-27 18:14:17] - test - Updating Stack
     9	[2023-08-27 18:14:17] - test - No updates to perform.
    10	"The argument \"echo \"Hello, world!\"\" is the wrong type - cmd hooks require arguments of type string."
```

I expected it to fail, but the error message doesn't make much sense.

InvalidHookArgumentTypeError is a SceptreException. Sceptre appears to just print the message instead of the whole stack trace for this exception type.

Definition in `sceptre/exceptions.py`:

```python
class InvalidHookArgumentTypeError(SceptreException):
    """
    Error raised if a hook's argument type is invalid.
    """

    pass
```

If I comment out the original code and just raise the TypeError, what happens?

```python
try:
    ...
except TypeError:
    # raise InvalidHookArgumentTypeError(
    #     'The argument "{0}" is the wrong type - cmd hooks require '
    #     "arguments of type string.".format(self.argument)
    # )
    raise
```

I get a traceback:

```text
    10	Traceback (most recent call last):
    ...
    59	TypeError: string indices must be integers
```

The special handling for SceptreException is in `cli/helpers.py`. The `catch_exceptions` function returns its input function wrapped in an error handler. It catches `SceptreException`, `BotoCoreError`, `ClientError`, `Boto3Error`, and `TemplateError`. It calls `write` on the error (I suppose this is what prints out the exception message) and then calls `sys.exit(1)`. So all of these errors are fatal errors in the Sceptre CLI. I don't know how the other types are defined. I can look them up later.

If I add this part before the try-catch:

```python
if isinstance(self.argument, str):
     args = self.argument
     executable = None
elif isinstance(self.argument, dict):
     args = self.argument["args"]
     executable = self.argument["executable"]
```

And use this subprocess call:

```python
subprocess.check_call(args, shell=True, env=envs, executable=executable)
```

Then the hook handles both input types.

```text
     3	[2023-08-27 18:43:03] - test - Launching Stack
     4	[2023-08-27 18:43:03] - test2 - Launching Stack
     5	[2023-08-27 18:43:03] - test - Stack is in the CREATE_COMPLETE state
     6	Hello, world!
     7	[2023-08-27 18:43:03] - test - Updating Stack
     8	[2023-08-27 18:43:03] - test2 - Stack is in the CREATE_COMPLETE state
     9	Hello, world!
    10	[2023-08-27 18:43:03] - test2 - Updating Stack
    11	[2023-08-27 18:43:04] - test2 - No updates to perform.
    12	[2023-08-27 18:43:04] - test - No updates to perform.
```

In which circumstances would a type error be raised? Maybe if the `!cmd` directive took an int or a list.

I'll share this proof of concept with the other developers before diving deeper into error handling.

One more test: does it support a hook with Bash syntax?

```bash
put "$tmp/config/test3.yaml" <<"EOF"
template:
  type: file
  path: test.yaml
hooks:
    before_update:
        - !cmd
            args: 'if [[ true ]]; then echo "Supports Bash syntax."; else "Not Bash? Impossible!"; fi'
            executable: '/bin/bash'
EOF
```

Yes, it supports Bash syntax.

```text
     2	  from pkg_resources import iter_entry_points
     3	[2023-08-27 19:41:46] - test - Launching Stack
     4	[2023-08-27 19:41:46] - test3 - Launching Stack
     5	[2023-08-27 19:41:46] - test2 - Launching Stack
     6	[2023-08-27 19:41:47] - test - Stack is in the CREATE_COMPLETE state
     7	Hello, world!
     8	[2023-08-27 19:41:47] - test3 - Stack is in the CREATE_COMPLETE state
     9	[2023-08-27 19:41:47] - test - Updating Stack
    10	Supports Bash syntax.
    11	[2023-08-27 19:41:47] - test3 - Updating Stack
    12	[2023-08-27 19:41:47] - test2 - Stack is in the CREATE_COMPLETE state
    13	Hello, world!
    14	[2023-08-27 19:41:47] - test2 - Updating Stack
    15	[2023-08-27 19:41:47] - test2 - No updates to perform.
    16	[2023-08-27 19:41:47] - test - No updates to perform.
    17	[2023-08-27 19:41:47] - test3 - No updates to perform.
```

---

Now I prepare the draft PR to get feedback from the other developers.
