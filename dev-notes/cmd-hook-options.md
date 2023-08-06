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
