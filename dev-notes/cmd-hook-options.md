# cmd hook options

I volunteered to provide a solution for issue 1213.

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
