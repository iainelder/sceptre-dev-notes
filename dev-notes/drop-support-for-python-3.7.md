# Drop support for Python 3.7

[Slack thread](https://og-aws.slack.com/archives/C01JNN8RGBB/p1694624952882979).

[GitHub issue 1381](https://github.com/Sceptre/sceptre/issues/1381).

Ideas:

* Search for `version_info` from `sys`
* Search for `3.7`
* Search for `python_version` from `platform`
* Search for `python_version_tuple` from `platform` (learned via [nkmk](https://note.nkmk.me/en/python-sys-platform-version-info/)'s blog).

Search code for references to Python versions. A 3 followed by a number between 0 and 19 inclusive separated by some non-digits.

```bash
ack --ignore-dir=test-reports -- '\b3\b\D+\b([0-9]|[12][0-9])\b' \
| sed -E 's/^([^:]+):([^:]+):/\1\x00\2\x00/' \
| jq -R -c 'split("\u0000") | {"path": "`\(.[0])`", "line": .[1], "text": "`\(.[2])`"}' \
| jtbl -m
```

TODO: Ask Kelly Jon Brazil to add an ack parser to jc.

| path                                                       | line | text                                                                                                              |
| ---------------------------------------------------------- | ---- | ----------------------------------------------------------------------------------------------------------------- |
| `integration-tests/features/dependency-resolution.feature` | 9    | `    And that stack "3/A" was created before "3/B"`                                                               |
| `integration-tests/features/dependency-resolution.feature` | 10   | `    And that stack "3/B" was created before "3/C"`                                                               |
| `integration-tests/features/delete-stack.feature`          | 21   | `    and stack "3/A" depends on stack "4/C"`                                                                      |
| `sceptre/connection_manager.py`                            | 133  | `            "profile='{1}', stack_name='{2}', sceptre_role='{3}', sceptre_role_session_duration='{4}')".format(` |
| `sceptre/__init__.py`                                      | 7    | `if sys.version_info < (3, 8):`                                                                                   |
| `sceptre/__init__.py`                                      | 18   | `# http://docs.python.org/3.3/howto/logging.html#configuring-logging-for-a-library`                               |
| `sceptre/config/reader.py`                                 | 144  | `        if sys.version_info < (3, 10):`                                                                          |
| `sceptre/config/__init__.py`                               | 11   | `# http://docs.python.org/3.3/howto/logging.html#configuring-logging-for-a-library`                               |
| `sceptre/template.py`                                      | 242  | `        if sys.version_info < (3, 10):`                                                                          |
| `sceptre/plan/__init__.py`                                 | 11   | `# http://docs.python.org/3.3/howto/logging.html#configuring-logging-for-a-library`                               |
| `CHANGELOG.md`                                             | 96   | `## 3.3.0 (2023.02.06)`                                                                                           |
| `CHANGELOG.md`                                             | 139  | `## 3.2.0 (2022.09.20)`                                                                                           |
| `CHANGELOG.md`                                             | 142  | ` - [Resolve #1225] Added Python 3.10 support (#1227)`                                                            |
| `CHANGELOG.md`                                             | 149  | ` - [Resolve #1225] Updating `troposphere` version for python 3.10 compatibility (#1226)`                         |
| `CHANGELOG.md`                                             | 150  | ` - [Resolve #1225] Updating Sceptre to use importlib on Python 3.10 (#1240)`                                     |
| `CHANGELOG.md`                                             | 163  | ` - Updated sceptre-circleci docker image for Python 3.10 support (#1230)`                                        |
| `CHANGELOG.md`                                             | 166  | `## 3.1.0 (2022.04.13)`                                                                                           |
| `CHANGELOG.md`                                             | 174  | `## 3.0.0 (2022.02.22)`                                                                                           |
| `CHANGELOG.md`                                             | 177  | ` - Python 3.6 support has been removed due to that version reaching end-of-life status`                          |
| `CHANGELOG.md`                                             | 275  | `- Update docker container to use Python 3.7`                                                                     |
| `CHANGELOG.md`                                             | 295  | `- Added support for python 3.8 & 3.9`                                                                            |
| `CHANGELOG.md`                                             | 302  | `- Removed support for python 2.7 & 3.5`                                                                          |
| `CHANGELOG.md`                                             | 333  | `## 2.3.0 (2020.02.03)`                                                                                           |
| `CHANGELOG.md`                                             | 359  | `- Keep version `1.3.4` and `1.4.2` active on github pages`                                                       |
| `CHANGELOG.md`                                             | 361  | `- Upgrade Dockerfile to Alpine 3.10`                                                                             |
| `CHANGELOG.md`                                             | 370  | `- `typing` install dependency for Python version < 3.5`                                                          |
| `CHANGELOG.md`                                             | 428  | `- Officially support Python 3.7`                                                                                 |
| `CHANGELOG.md`                                             | 646  | `## 1.3.4 (2018-02-19)`                                                                                           |
| `CHANGELOG.md`                                             | 651  | `##1.3.3 (2018-02-19)`                                                                                            |
| `CHANGELOG.md`                                             | 653  | `- Released in Error. Contained breaking changes from v2. Fixed in 1.3.4.`                                        |
| `CHANGELOG.md`                                             | 655  | `## 1.3.2 (2017.11.28)`                                                                                           |
| `CHANGELOG.md`                                             | 663  | `## 1.3.1 (2017.10.23)`                                                                                           |
| `CHANGELOG.md`                                             | 669  | `## 1.3.0 (2017.10.16)`                                                                                           |
| `CHANGELOG.md`                                             | 713  | `## 1.1.0 (2017.3.3)`                                                                                             |
| `CHANGELOG.md`                                             | 1032 | `- Bumping boto3 dependency version to 1.3.1.`                                                                    |
| `CHANGELOG.md`                                             | 1155 | `## 0.3.2 (2016.03.10)`                                                                                           |
| `CHANGELOG.md`                                             | 1159 | `## 0.3.1 (2016.03.10)`                                                                                           |
| `CHANGELOG.md`                                             | 1163 | `## 0.3.0 (2016.03.09)`                                                                                           |
| `Dockerfile`                                               | 1    | `FROM python:3.10-alpine`                                                                                         |
| `pyproject.toml`                                           | 18   | `  "Programming Language :: Python :: 3.7",`                                                                      |
| `pyproject.toml`                                           | 19   | `  "Programming Language :: Python :: 3.8",`                                                                      |
| `pyproject.toml`                                           | 20   | `  "Programming Language :: Python :: 3.9",`                                                                      |
| `pyproject.toml`                                           | 21   | `  "Programming Language :: Python :: 3.10",`                                                                     |
| `pyproject.toml`                                           | 22   | `  "Programming Language :: Python :: 3.11"`                                                                      |
| `pyproject.toml`                                           | 50   | `python = ">=3.7,<3.12"`                                                                                          |
| `pyproject.toml`                                           | 56   | `jinja2 = "^3.0"`                                                                                                 |
| `pyproject.toml`                                           | 57   | `jsonschema = "~3.2"`                                                                                             |
| `pyproject.toml`                                           | 84   | `freezegun = ">=0.3.12,<0.4.0"`                                                                                   |
| `pyproject.toml`                                           | 91   | `tox = "^3.23.0"`                                                                                                 |
| `tests/test_template_handlers/test_helper.py`              | 75   | `@pytest.mark.skipif(sys.version_info < (3, 8), reason="requires Python >= 3.8")`                                 |
| `tests/test_hooks/test_cmd.py`                             | 93   | `    if platform.python_version().startswith("3.7."):`                                                            |
| `tests/test_cli/test_prune.py`                             | 158  | `        self.all_stacks[3].dependencies.append(self.all_stacks[1])`                                              |
| `tests/test_cli/test_prune.py`                             | 179  | `        self.all_stacks[3].dependencies.append(self.all_stacks[1])`                                              |
| `tests/test_cli/test_prune.py`                             | 193  | `        self.all_stacks[3].dependencies.append(self.all_stacks[1])`                                              |
| `tests/test_helpers.py`                                    | 193  | `        arg = [(a, 1), (a, 3), (b, 0), (b, 2)]`                                                                  |
| `tests/test_config_reader.py`                              | 333  | `                {"A/3", "A/2", "A/1"},`                                                                          |
| `tests/test_config_reader.py`                              | 334  | `                {"A/3", "A/2", "A/1"},`                                                                          |
| `tests/test_actions.py`                                    | 955  | `        if sys.version_info < (3, 8):`                                                                           |
| `tests/test_actions.py`                                    | 995  | `                            2016, 3, 15, 14, 1, 0, 0, tzinfo=tzutc()`                                            |
| `tests/test_actions.py`                                    | 1004 | `                            2016, 3, 15, 14, 2, 0, 0, tzinfo=tzutc()`                                            |
| `tests/test_actions.py`                                    | 1013 | `                datetime.datetime(2016, 3, 15, 14, 0, 0, 0, tzinfo=tzutc())`                                     |
| `tests/test_actions.py`                                    | 1042 | `                            2023, 8, 15, 14, 3, 0, 0, tzinfo=tzutc()`                                            |
| `tests/test_resolvers/test_resolver.py`                    | 461  | `        expected = {"a": 4, "c": 3, "f": 5}`                                                                     |
| `tests/test_resolvers/test_resolver.py`                    | 527  | `        expected = [1, 3, 6]`                                                                                    |
| `.circleci/documentation-versions.py`                      | 38   | `KEEP_VERSIONS = ["1.5.0", "1.4.2", "1.3.4"]  # these versions won't be removed`                                  |

Put it all together (gives same results).

```bash
ack --ignore-dir=test-reports -- '(\b3\b\D+\b([0-9]|[12][0-9])\b|version_info|python_version|python_version_tuple)' \
| sed -E 's/^([^:]+):([^:]+):/\1\x00\2\x00/' \
| jq -R -c 'split("\u0000") | {"path": "`\(.[0])`", "line": .[1], "text": "`\(.[2])`"}' \
| jtbl -m
```

The changelog shows that Sceptre has already added and dropped support for old Python versions.

Since the changelog is prose, I can search it with a more selective regex to get a summary of Python version changes. I search in VS Code with pattern `(?<!\.)\b[23]\.\d+(?!\.\d+|rc)`. Or with a simpler pattern `[Pp][Yy].*[23]\.`.

Matching changelog entries:

```markdown
## 3.2.0 (2022.09.20)

### Added
 - [Resolve #1225] Added Python 3.10 support (#1227)

### Changed
 - [Resolve #1225] Updating `troposphere` version for python 3.10 compatibility (#1226)
 - [Resolve #1225] Updating Sceptre to use importlib on Python 3.10 (#1240)

### Nonfunctional
 - Updated sceptre-circleci docker image for Python 3.10 support (#1230)

## 3.0.0 (2022.02.22)

### Breaking Changes
 - Python 3.6 support has been removed due to that version reaching end-of-life status

### Removed
 - [Resolves #1201] Remove Py3.6 support (#1206)

## 2.6.0 (2021.07.29)

### Added

- Update docker container to use Python 3.7

## 2.5.0 (2021.05.01)

### Added

- Added support for python 3.8 & 3.9

### Removed

- Removed support for python 2.7 & 3.5

## 2.4.0 (2020.10.03)

- Remove documented support for Python 2.7

## 2.2.1 (2019.08.19)

### Fixed

- `typing` install dependency for Python version < 3.5

## 2.1.4 (2019.06.26)

### Nonfunctional

- Officially support Python 3.7

## 0.1.1 (2016.03.08)

- Updating tox to only support Python 2.6 versions > 2.6.9.
```

The `git tag` command shows that tags are missing for v0.1.1` and v2.4.0. The repo's commit history stops at 2017, so missing v0.1.1 makes sense. But missing v2.4.0 looks like a mistake.

Q: What happened to tag `v2.4.0`?

Find the commits in the changelog extract and commits that refer to a Python version.

```bash
tmp="$(mktempdir)"
cat > "$tmp/commits.txt" \
<(git log --format="%H" --grep='[Pp][Yy].*[23]\.') \
<(git tag --format="%(objectname)" --list v3.2.0 v3.0.0 v2.6.0 v2.5.0 v2.4.0 v2.2.1 v2.1.4 v0.1.1)
```

Summarize the commits.

```bash
git log --no-walk=sorted --format="%cs%x00%h%x00%(describe:tags=true)%x00%s" $(<"$tmp/commits.txt") \
| jq -R -c 'split("\u0000") | {"commit date": .[0], "commit hash": .[1], "descriptor": .[2], "subject": .[3]}' \
| jtbl -m
```

<!-- vale off -->
| commit date | commit hash | descriptor         | subject                                                                     |
| ----------- | ----------- | ------------------ | --------------------------------------------------------------------------- |
| 2023-08-13  | 5d574b7     | v4.2.2-1-g5d574b7  | [Resolve #1370] Use Python 3.11 for Black (#1371)                           |
| 2023-07-18  | 6f6773a     | v4.2.1-1-g6f6773a  | [Resolve #1358] Updating PyYaml to ^6.0 (#1359)                             |
| 2022-09-23  | 7d6da28     | v3.2.0             | creating v3.2.0 release (#1254)                                             |
| 2022-06-17  | d932f27     | v3.1.0-10-gd932f27 | [Resolves #1225] Support python 3.10 (#1227)                                |
| 2022-06-16  | a0fc6ff     | v3.1.0-9-ga0fc6ff  | Update sceptre to use importlib (#1240)                                     |
| 2022-06-10  | f994eb7     | v3.1.0-7-gf994eb7  | update sceptre-circleci docker image (#1230)                                |
| 2022-05-26  | f2b9118     | v3.1.0-4-gf2b9118  | Update troposphere version (#1226)                                          |
| 2022-02-22  | e7f03b4     | v3.0.0             | Sceptre v3.0.0 Release details (#1207)                                      |
| 2022-02-22  | 27f0b9f     | v2.7.1-11-g27f0b9f | [Resolves #1201] Fix dependency conflict (#1206)                            |
| 2021-10-31  | 0953d4b     | v2.6.3-30-g0953d4b | Remove .python-version file (#1146)                                         |
| 2021-09-15  | 400b488     | v2.6.3-5-g400b488  | [Resolves #582] update imp to importlib (#1092)                             |
| 2021-07-29  | 8a1649a     | v2.6.0             | bump version 2.5.0 -> 2.6.0 and update changelog                            |
| 2021-06-04  | fce128a     | v2.5.0-11-gfce128a | Update Docker to Python 3.7 (#1054)                                         |
| 2021-05-01  | c84abac     | v2.5.0             | bump version v2.4.0 -> v2.5.0                                               |
| 2021-04-29  | 19d0a4f     | v2.3.0-51-g19d0a4f | [Resolves #1005] Testing uplift (#1022)                                     |
| 2021-04-23  | 339780d     | v2.3.0-46-g339780d | Set up a basic pyproject.toml for PEP-518 builds (#1009)                    |
| 2021-04-15  | beafbb6     | v2.3.0-36-gbeafbb6 | remove python 3.5 from supported list (#1011)                               |
| 2021-04-15  | 5ee9cde     | v2.3.0-35-g5ee9cde | [Resolves #978] Only install typing on Python<3.5 (#979)                    |
| 2021-04-13  | 52730d8     | v2.3.0-32-g52730d8 | [Resolved #992] Use the same version of alpine as the circleci tests (#999) |
| 2020-12-23  | bd3ebd4     | v2.3.0-20-gbd3ebd4 | [Resolves #921 and #956] Remove Python 2.7 support and fix build (#959)     |
| 2020-08-13  | 20579c4     | v2.3.0-10-g20579c4 | [Resolves #921] Drop python version 2.7 support (#922)                      |
| 2019-08-19  | b7585fd     | v2.2.1             | Bump version: 2.2.0 → 2.2.1                                                 |
| 2019-06-27  | 9e4e2f3     | v2.1.4             | Allow empty docs commits in CI build                                        |
| 2019-06-26  | d4db470     | v2.1.1-13-gd4db470 | [Resolve #622] Add official Python 3.7 support                              |
| 2019-01-10  | 0370c41     | v2.0.0-27-g0370c41 | Update PyPi classifiers                                                     |
| 2017-05-05  | 64bc5cc     | v1.1.1-42-g64bc5cc | Merge pull request #98 from cloudreach/add-python36-to-tox-testing          |
| 2017-04-21  | 1472f02     | v1.1.1-36-g1472f02 | add python 3.6 to tox                                                       |
<!-- vale on-->

Read the message body and the patch for each commit above and pick out the relevant affected files.

```bash
git log --no-walk=sorted --patch $(<"$tmp/commits.txt")
```

A table entry with `-` in the file column mentions a Python version in the commit message but doesn't change a version in the code.

| commit date | commit hash | file                       | key                                 | value                                                                                 |
| ----------- | ----------- | -------------------------- | ----------------------------------- | ------------------------------------------------------------------------------------- |
| 2023-08-14  | 5d574b7     | `.pre-commit-config.yaml`  | `language_version`                  | `python3.11`                                                                          |
| 2023-07-18  | 6f6773a     | -                          | -                                   | -                                                                                     |
| 2022-09-23  | 7d6da28     | `CHANGELOG.md`             | `3.2.0`                             | `[Resolve #1225] Added Python 3.10 support (#1227)`                                   |
| 2022-06-17  | d932f27     | `setup.py`                 | `classifiers`                       | `Programming Language :: Python :: 3.10`                                              |
| 2022-06-17  | d932f27     | `tox.ini`                  | `envlist`                           | `py{37,38,39,310},flake8`                                                             |
| 2022-06-16  | a0fc6ff     | `sceptre/config/reader.py` | `_iterate_entry_points`             | `if sys.version_info < (3, 10)`                                                       |
| 2022-06-16  | a0fc6ff     | `sceptre/template.py`      | `_iterate_entry_points`             | `if sys.version_info < (3, 10)`                                                       |
| 2022-06-10  | f994eb7     | `.circleci/config.yml`     | `docker.image`                      | `cloudreach/sceptre-circleci:1.0.0`                                                   |
| 2022-05-26  | f2b9118     | `.circleci/config.yml`     | `jobs.build.steps.save_cache.paths` | `.pyenv/versions/3.9.4/envs/venv`                                                     |
| 2022-02-22  | e7f03b4     | `CHANGELOG.md`             | `3.0.0`                             | `Python 3.6 support has been removed due to that version reaching end-of-life status` |
| 2022-02-22  | 27f0b9f     | `tox.ini`                  | `envlist`                           | `envlist = py{37,38,39},flake8`                                                       |
| 2021-10-31  | 0953d4b     | `.python-version`          | -                                   | `3.6.9`                                                                               |
| 2021-09-15  | 400b488     | -                          | -                                   | -                                                                                     |
| 2021-07-29  | 8a1649a     | `CHANGELOG.md`             | `2.6.0`                             | `Update docker container to use Python 3.7`                                           |
| 2021-06-04  | fce128a     | `Dockerfile`               | `FROM`                              | `python:3.7-alpine`                                                                   |
| 2021-05-01  | c84abac     | `CHANGELOG.md`             | `2.5.0 Added`                       | `Added support for python 3.8 & 3.9`                                                  |
| 2021-05-01  | c84abac     | `CHANGELOG.md`             | `2.5.0 Removed`                     | `Removed support for python 2.7 & 3.5`                                                |
| 2021-04-29  | 19d0a4f     | `setup.py`                 | `classifiers`                       | `Programming Language :: Python :: 3.7`                                               |
| 2021-04-29  | 19d0a4f     | `tox.ini`                  | `envlist`                           | `py{36,37,38,39},flake8`                                                              |
| 2021-04-23  | 339780d     | `.circleci/config.yml`     | `docker.image`                      | `cloudreach/sceptre-circleci:0.8.0`                                                   |
| 2021-04-15  | 339780d     | `.python-version`          | -                                   | `3.6-dev`, `3.7-dev`                                                                  |
| 2021-04-15  | 339780d     | `requirements/prod.txt`    | -                                   | `typing>=3.7,<3.8`                                                                    |
| 2021-04-15  | beafbb6     | `setup.py`                 | `classifiers`                       | `Programming Language :: Python :: 3.5`                                               |
| 2021-04-15  | 5ee9cde     | `setup.py`                 | `install_requirements`              | `typing>=3.7,<3.8`                                                                    |
| 2021-04-14  | 52730d8     | `Dockerfile`               | `FROM`                              | `python:3.6.13-alpine3.13`                                                            |
| 2020-12-23  | bd3ebd4     | `.circleci/config.yml`     | `docker.image`                      | `cloudreach/sceptre-circleci:0.6.0`                                                   |
| 2020-08-13  | 20579c4     | `.python-version`          | -                                   | `2.7-dev`                                                                             |
| 2020-08-13  | 20579c4     | `CONTRIBUTING.md`          | -                                   | `Run unit tests and coverage using tox for Python 3.6 and 3.7:`                       |
| 2020-08-13  | 20579c4     | `setup.py`                 | `classifiers`                       | `Programming Language :: Python :: 2.7`                                               |
| 2020-08-13  | 20579c4     | `tox.ini`                  | `envlist`                           | `py36,py37`                                                                           |
| 2019-08-19  | b7585fd     | -                          | -                                   | -                                                                                     |
| 2019-06-27  | 9e4e2f3     | -                          | -                                   | -                                                                                     |
| 2019-06-26  | d4db470     | `circleci/Dockerfile`      | `FROM`                              | `alpine:3.7`                                                                          |
| 2019-06-26  | d4db470     | `circleci/Dockerfile`      | -                                   | `pyenv install 2.7-dev`, `pyenv install 3.6-dev`, `pyenv install 3.7-dev`             |
| 2019-06-26  | d4db470     | `.circleci/config.yaml`    | `docker.image`                      | `cloudreach/sceptre-circleci:0.4`                                                     |
| 2019-06-26  | d4db470     | `.python-version`          | -                                   | `2.7-dev`, `3.6-dev`, `3.7-dev`                                                       |
| 2019-06-26  | d4db470     | `setup.py`                 | `classifiers`                       | `Programming Language :: Python :: 3.7`                                               |
| 2019-06-26  | d4db470     | `tox.ini`                  | `envlist`                           | `py27,py36,py37`                                                                      |
| 2019-01-10  | 0370c41     | `setup.py`                 | `classifiers`                       | `Programming Language :: Python :: 2.6`                                               |
| 2017-05-05  | 64bc5cc     | -                          | -                                   | -                                                                                     |
| 2017-04-21  | 1472f02     | `tox.ini`                  | `envlist`                           | `py269, py27, py352, py361`                                                           |


Notes:

* d932f27: The current version uses Poetry. Infer `classifiers` in `pyproject.toml`
* a0fc6ff: Looks like a bug. See Q below.
* f2b9118: The current version doesn't put a Python version at `jobs.build.steps.save_cache.paths`
* 0953d4b: Removed file for good
* 400b488: Matched message `The imp.py library has been deprecated since ver 3.4`, a fluke.

The regular expression doesn't extract the full matched message. What I have is good enough. If I need to go over this again consider searching for phrases such as `ver X.Y`.

Q: Commit `a0fc6ff` branches on the Python version to use either `importlib` or `pkg_resources`. It says Python 3.7 needs `pkg_resources` and Python 3.8 and up can use `importlib`. But the branch uses `pkg_resources` for Python versions less than 3.10. What's the intended behavior? (See aside on `pkg_resources` deprecation.)

## An aside on `pkg_resources` deprecation

The `pkg_resources` module is part of the `setuptools` package. The [package changelog](https://setuptools.pypa.io/en/latest/history.html#v67-5-0) says that version v67.5.0 officially deprecates the module. Jason R. Coombs created [PR #3843](https://github.com/pypa/setuptools/pull/3843) on 2023-03-05 to emit the deprecation message. This change is so recent that not all my installations of Python show this behavior.

My virtualenv for Sceptre shows the behavior. Its version of `setuptools` is v68.0.0.

```console
$ poetry run python -c 'import pkg_resources'
<string>:1: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html

$ poetry run pip freeze --all | grep setuptools
setuptools==68.0.0
```

(`pip` doesn't list `setuptools` without the `--all` option.)

My system Python doesn't show the behavior. The `import` command produces no output. Its version of `setuptools` is v62.3.1.

```console
$ python3.8 -c 'import pkg_resources'

$ python3.8 -m pip freeze --all | grep setuptools
setuptools==62.3.1
```

My pipx-installed version of Sceptre shows the behavior. Its version of `setuptools` is v68.2.2.

```console
$ ~/.local/pipx/venvs/sceptre/bin/python -c 'import pkg_resources'
<string>:1: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html

$ ~/.local/pipx/venvs/sceptre/bin/python -m pip freeze --all | grep setuptools
setuptools==68.2.2
```

## Feedback from Khai Do

Via Slack:

> FYI, we’ve dropped support for python 3.6 pretty recently and here’s the PR for that https://github.com/Sceptre/sceptre/pull/1206
>
> we should also remove it from https://github.com/Sceptre/sceptre-circleci  which is the docker container that we use for testing sceptre on circle-ci
>
> and from the included sceptre plugins as well.. https://github.com/Sceptre/sceptre/blob/master/pyproject.toml#L61-L62

I inspected PR 1206 via commit 27f0b9f.

## Create GitHub issue

Create [issue 1381](https://github.com/Sceptre/sceptre/issues/1381) to track the work across all the repos.

---

Remove support for Python 3.7 from the following repos:

* [ ] [Main Sceptre repo](https://github.com/Sceptre/sceptre)
* [ ] [Docker image for CircleCI builds](https://github.com/Sceptre/sceptre-circleci)
* [ ] [`!rcmd` shell command resolver](https://github.com/Sceptre/sceptre-resolver-cmd)
* [ ] [`!file` file content resolver](https://github.com/Sceptre/sceptre-file-resolver)

Python 3.7 reached the [end of its support lifecycle on 2023-06-27](https://devguide.python.org/versions/).

I track my own progress in [my dev notes for this task](https://github.com/iainelder/sceptre-dev-notes/blob/main/dev-notes/drop-support-for-python-3.7.md).

## Make changes to main Sceptre repo

2023-11-12.

Made changes in [draft PR 1382](https://github.com/Sceptre/sceptre/pull/1382).

Shared with Khai for feedback.

## Note failed CircleCI execution

It looks like CircleCI is failing to run Sceptre builds. On recent PRs GitHub shows

> CircleCI Pipeline—Could not find a usable config.yml, you may have revoked the CircleCI OAuth app.

It [links to a CircleCI page](https://app.circleci.com/pipelines/github/Sceptre/sceptre/1977) that shows

> Could not find a usable config.yml, you may have revoked the CircleCI OAuth app. Please sign out of CircleCI and log back in with your VCS before triggering a new pipeline.

What do we need to do to fix this?

## Note merge of main repo changes

2023-11-22.

Khai Do approved my PR on 2023-11-13 and merged it to main on 2023-11-14.

## Review Khai's PR for CircleCI repo

2023-11-22.

Khai submitted [PR 19](https://github.com/Sceptre/sceptre-circleci/pull/19/files) for the CircleCI repo.

Clone the CircleCI repo.

```bash
git clone https://github.com/Sceptre/sceptre-circleci.git ~/Repos/sceptre/circleci
```

Build the Docker image using the new Docker build syntax.

```bash
docker buildx build .
```

The build completes. I'm not sure what to do with it next. I think it's only used by CircleCI.

The `FROM cimg/python:3.12.0-node` line refers to the name on Docker Hub of the [CircleCI Convenience Image for Python](https://hub.docker.com/r/cimg/python).

> Each tag contains a complete Python version via pyenv. pip, pipenv, and poetry are pre-installed, and any binaries and tools that are required for builds to complete successfully in a CircleCI environment.

I ask some questions to Khai on his PR to understand what each Python version means.

## Learn about pyenv versions

2023-11-25.

I'll use the term "feature release" to describe a Python version like "3.8" and "bugfix release" to describe a Python version like "3.8.10". I can't find a plain description of that in the official documentation, but it's how I and this [Real Python author](https://realpython.com/python-bugfix-version/) think about Python versions.

Khai first wanted to use these Python bugfix versions in the sceptre-circlici Dockerfile:

```text
3.8.16
3.9.16
3.10.10
3.11.2
```

Because some of those were unavailable in the CircliCI environment, he rolled them back to these versions:

```text
3.8.16
3.9.16
3.10.9
3.11.1
```

Khai installed pyenv 2.3.25.

My pyenv version is 2.3.32.

```text
$ pyenv --version
pyenv 2.3.32
```

This command uses the `pyenv latest --known` command to gives the latest bugfix versions for all Python feature release versions that Sceptre supports or plans to support.

```bash
echo 3.7 3.8 3.9 3.10 3.11 | xargs -n1 pyenv latest --known
```

```text
3.7.17
3.8.18
3.9.18
3.10.13
3.11.6
```

Pyenv installs itself by cloning a Git repo. It updates itself by pulling changes from that repo. A git tag marks each pyenv version. The repo stores a list of all known Python version. Newer pyenv versions extend the Python version list. GitHub hosts the pyenv repo, so I can use the GitHub API to check what the latest Python bugfix versions would be for a given pyenv version.

Read this [Stack Overflow question](https://stackoverflow.com/questions/25370881/github-api-get-contents-of-tag-instead-of-master) to learn how to list GitHub repo contents at a tagged version.

Write this function to show the latest Python versions listed by a given pyenv version. It shows just the Python feature versions that Sceptre supports. It ignores development versions.

```bash
function list_latest_pythons_known_to_pyenv_version() {
  pyenv_version="$1"
  curl -Ss "https://api.github.com/repos/pyenv/pyenv/contents/plugins/python-build/share/python-build?ref=v$pyenv_version" \
  | jq 'map(.name | capture("^(?<feature>3\\.(7|8|9|10|11))\\.(?<bugfix>[01234567890]+)$"))' \
  | jq 'group_by(.feature) | map(max_by(.bugfix | tonumber))' \
  | jq -r '.[] | "\(.feature).\(.bugfix)"' \
  | sort -V
}
```

Use this function to check what pyenv 2.3.32 would return (the version I installed).

```bash
list_latest_pythons_known_to_pyenv_version "2.3.32"
```

The output matches that from my installed pyenv version.

```text
3.7.17
3.8.18
3.9.18
3.10.13
3.11.6
```

Use the function to check what pyenv 2.3.25 would return (the version Khai installed).

```bash
list_latest_pythons_known_to_pyenv_version "2.3.25"
```

```text
3.7.17
3.8.18
3.9.18
3.10.13
3.11.5
```

pyenv 2.3.25 shows an earlier bugfix version for feature version 3.11.

In general an earlier pyenv version will have the same or earlier bugfix version for a given feature version.

Q: Khai says for 3.10 the latest bugfix version is 3.10.10. I can't explain why my function gives 3.10.13 and he gets 3.10.10. Maybe there's something I'm missing about how pyenv works.

I suspect that the CircliCI image may use an ever earlier version of pyenv.

I already built the image from the master branch so I can run it to check. I built it without any tags so I need to run it using the image ID.

```bash
docker run -it --rm 04b13bde0c0b bash
```

I know it's the right image because the prompt says "`circleci`".

```text
circleci@5cf07ee38878:~/project$$
```

The image's pyenv version is 2.3.8.

```console
$ pyenv --version
pyenv 2.3.8
```

The image base is `cimg/python:3.11.1-node`.

Pull that image and check it has the same version.

The pull is fast because all the layers already belong to the CircleCI image. All Docker needs to add is the metadata.

```console
$ docker pull cimg/python:3.11.1-node
3.11.1-node: Pulling from cimg/python
301a8b74f71f: Already exists
3a115dde664b: Already exists
560f8b954700: Already exists
07a263cdc41b: Already exists
45547161523e: Already exists
9d5a29a12b14: Already exists
573e5075c506: Already exists
4f4fb700ef54: Already exists
6345f6d63563: Already exists
547dac75ac99: Already exists
031ea5de6478: Already exists
39315802cfd3: Already exists
a5fa0027729b: Already exists
dff807b318bc: Already exists
Digest: sha256:fac71c651914675c788807a3024d3730d897d3c898c47fa207268082fcba41c8
Status: Downloaded newer image for cimg/python:3.11.1-node
docker.io/cimg/python:3.11.1-node
```

The base image's pyenv version is also 2.3.8.

```console
$ docker run --rm cimg/python:3.11.1-node pyenv --version
pyenv 2.3.8
```

List the latest Python versions known to pyenv 2.3.8.

```console
$ docker run --rm cimg/python:3.11.1-node bash -c 'echo 3.7 3.8 3.9 3.10 3.11 | xargs -n1 pyenv latest --known'
3.7.16
3.8.16
3.9.16
3.10.9
3.11.1
```

My function gives the same output.

```bash
list_latest_pythons_known_to_pyenv_version "2.3.8"
```

Pull the new base image that Khai wants to use.

```bash
docker pull cimg/python:3.12.0-node
```

The new base image uses pyenv 2.3.29.

```console
$ docker run --rm cimg/python:3.12.0-node pyenv --version
pyenv 2.3.29
```

List the latest Python versions known to pyenv 2.3.29.

```console
$ docker run --rm cimg/python:3.12.0-node bash -c 'echo 3.7 3.8 3.9 3.10 3.11 | xargs -n1 pyenv latest --known'
3.7.17
3.8.18
3.9.18
3.10.13
3.11.6
```

My function gives the same output.

```bash
list_latest_pythons_known_to_pyenv_version "2.3.29"
```

The CircleCI base images' own Dockerfiles are in the GitHub repo [CircleCI-Public/cimg-python](https://github.com/CircleCI-Public/cimg-python/). The maintainers update each Dockerfile for each new bugfix version of the Python feature version. For example, [the history of the Dockerfile for Python 3.11](https://github.com/CircleCI-Public/cimg-python/commits/main/3.11/Dockerfile) shows changes for Python versions 3.11.0, 3.11.1, 3.11.2, and all bugfix versions up to 3.11.6.

## Answer Khai's question about pyenv versions

2023-11-25.

This time I don't think the operating system matters, but the pyenv version certainly does.

My pyenv version is 2.3.32. That's the latest version. I ran `pyenv update` just before I checked this.

The pyenv version in the Docker image is what matters most.

Base image `cimg/python:3.11-node` has pyenv version 2.3.8.

Base image `cimg/python:3.12-node` has pyenv version 2.3.29.

This table shows, for each Python feature version that Sceptre supports, the latest bugfix version that each pyenv version can install.

| Feature | Pyenv 2.3.8 latest | Pyenv 2.3.29 latest |
|:--------|-------------------:|--------------------:|
| 3.7     |                 16 |                  17 |
| 3.8     |                 16 |                  18 |
| 3.9     |                 16 |                  18 |
| 3.10    |                  9 |                  13 |
| 3.11    |                  1 |                   6 |

## Propose Dockerfile changes for simpler maintenance

2023-11-25.

If we don't need the absolute latest bugfix version, then we can just use CircleCI's chosen pyenv version.

Unless we care about the exact bugfix version number, we don't have to write them in the Dockerfile. We can use compound pyenv commands to install whatever latest version it knows about.

So the Dockerfile would look like this:

```Dockerfile
RUN pyenv install $(pyenv latest --known 3.8)
RUN pyenv install $(pyenv latest --known 3.9)
RUN pyenv install $(pyenv latest --known 3.10)
RUN pyenv install $(pyenv latest --known 3.11)
RUN pyenv global system $(pyenv versions --bare)
```

I built it locally and reported the Python versions available to pyenv:

```console
$ docker run -it --rm 1fb7c2f153d2 pyenv versions
* system (set by /home/circleci/.pyenv/version)
* 3.8.18 (set by /home/circleci/.pyenv/version)
* 3.9.18 (set by /home/circleci/.pyenv/version)
* 3.10.13 (set by /home/circleci/.pyenv/version)
* 3.11.6 (set by /home/circleci/.pyenv/version)
* 3.12.0 (set by /home/circleci/.pyenv/version)
```

I believe writing the Dockerfile this way would allow us to use the next bugfix version automatically whenever CircleCI updates their base image. Then we would never need to manage the bugfix versions again in this file. We would change this file only when Sceptre's supported feature versions change.

2023-11-28.

Khai accepted the suggestion and merged PR 19.

## Read sceptre-resolver-cmd repo for Python feature versions

2023-11-26.

Search for strings that suggest Python feature versions and tabulate the results. This is the same search I did in the main repo.

```bash
ack -- '(\b3\b\D+\b([0-9]|[12][0-9])\b|version_info|python_version|python_version_tuple)' \
| sed -E 's/^([^:]+):([^:]+):/\1\x00\2\x00/' \
| jq -R -c 'split("\u0000") | {"path": "`\(.[0])`", "line": .[1], "text": "`\(.[2])`"}' \
| jtbl -m
```

Only the programming language classifiers in `setup.py` look like a real match.

| path               | line | text                                               |
|--------------------|------|----------------------------------------------------|
| `setup.py`         | 32   | `    "pytest>=3.2",`                               |
| `setup.py`         | 65   | `        "Programming Language :: Python :: 3.7",` |
| `setup.py`         | 66   | `        "Programming Language :: Python :: 3.8",` |
| `setup.py`         | 67   | `        "Programming Language :: Python :: 3.9",` |
| `setup.py`         | 68   | `        "Programming Language :: Python :: 3.10"` |
| `requirements.txt` | 5    | `pytest-runner>=3.0.0,<3.1.0`                      |
| `requirements.txt` | 6    | `pytest>=3.2.0,<3.3.0`                             |
| `requirements.txt` | 10   | `tox>=2.9.1,<3.0.0`                                |

The repo has just 23 commits, so read it all to find any references to Python feature versions. "Programming Language" is the only one.

Use this command to see where "Programming Language" occurrences change.

```bash
git log \
    --no-walk=sorted \
    --format="%cs%x00%h%x00%(describe:tags=true)%x00%s" \
    $(git log --format="%H" -G "Programming Language") \
| jq -R -c 'split("\u0000") | {"commit date": .[0], "commit hash": .[1], "descriptor": .[2], "subject": .[3]}' \
| jtbl -m
```

| commit date | commit hash | descriptor        | subject                                |
|-------------|-------------|-------------------|----------------------------------------|
| 2023-02-13  | ace0fd4     | v1.2.1-1-gace0fd4 | Preparing for Sceptre v4 release (#12) |
| 2021-07-05  | 8e78bdd     |                   | [Resolves #3] Move to Sceptre Org      |
| 2020-08-21  | 9d32ad4     |                   | Initial commit                         |


## Review update to the main Sceptre CI configuration

2023-11-28.

Khai asked me to review [PR 1393](https://github.com/Sceptre/sceptre/pull/1393).

I left some questions as feedback to learn more about this part of the process.

2023-11-30.

Khai closed PR 1393 to replace it with another [PR 1394](https://github.com/Sceptre/sceptre/pull/1394). I'm getting confused with so many!

Khai has discovered that the project no longer needs a custom container to control multiple Python versions. The CirclCI base image already includes Poetry and somehow it can already switch between different Python feature versions. That means the work on sceptre-circleci PR 19 and that whole repo is obsolete.

I suggest we use the `3.12` tagged image instead of the `3.12.0` tagged image to get Python bugfix updates automatically.

Khai suggested to change the black language version from 3.11 to 3.11 because the [CircleCI build failed](https://app.circleci.com/pipelines/github/Sceptre/sceptre/2010/workflows/e64f2159-65b5-4b30-9a1d-0ed329fefaf3/jobs/10978).

The error message:

```text
An unexpected error has occurred: CalledProcessError: command: ('/home/circleci/.cache/pypoetry/virtualenvs/sceptre-3aSsmiER-py3.10/bin/python', '-mvirtualenv', '/home/circleci/.cache/pre-commit/repou3wgujmq/py_env-python3.11', '-p', 'python3.11')
return code: 1
stdout:
    RuntimeError: failed to find interpreter for Builtin discover of python_spec='python3.11'

stderr: (none)
Check the log at /home/circleci/.cache/pre-commit/pre-commit.log
```

The first output of pre-commit shows that, perhaps arbitrarily, pre-commit chose python3.10 because the project doesn't support the default python3.12.0.

```text
The currently activated Python version 3.12.0 is not supported by the project (>=3.8,<3.12).
Trying to find and use a compatible version.
Using python3.10 (3.10.6)
```

Rather than change the language version of black, we should probably pull in some of changes from #1393 that this PR replaces.

You can reproduce the black error locally like this.

(I copied the command from `.circleci/config.yaml`.

```bash
docker run -it cimg/python:3.12.0 bash -c '
git clone https://github.com/Sceptre/sceptre .
git fetch origin pull/1394/head:pr
git switch pr
sed -i "46s/language_version: python3.10/language_version: python3.11/" .pre-commit-config.yaml
poetry install --all-extras -v
poetry run pre-commit run black --all-files --show-diff-on-failure
'
```

[See GitHub documentation for how to check out pull requests locally](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/checking-out-pull-requests-locally).

## Debug missing Python interpreters

2023-12-01.

Khai pulled in changes from PR 1393 and PR 1390 (from Homebrew maintainer Branch Vincent) to make Sceptre work with Python 3.12.

It looks like the stock CircleCI container doesn't make available all the Python feature versions we need.

Maybe we do still need a custom image after all.

Use this command to check the reported version of each target Python feature version.

```bash
docker run -it cimg/python:3.12 bash -c 'for py in python3.{8,9,10,11,12}; do "$py" --version; done'
```

Python 3.10 and 3.12 report back. Python 3.8, 3.9, and 3.11 are missing.

```text
bash: line 1: python3.8: command not found
bash: line 1: python3.9: command not found
Python 3.10.6
bash: line 1: python3.11: command not found
Python 3.12.0
```

I discovered it running the unit tests as the CI pipeline would.

```bash
docker run -it cimg/python:3.12 bash -c '
git clone https://github.com/Sceptre/sceptre .
git fetch origin pull/1394/head:pr
git switch pr
poetry install --all-extras -v
poetry run tox
echo "Tox exit code: $?"
'
```

Tox passes `py310`. It skips `py38`, `py39`, and `py311` because it didn't find the interpreter.

```text
SKIPPED:  py38: InterpreterNotFound: python3.8
SKIPPED:  py39: InterpreterNotFound: python3.9
  py310: commands succeeded
SKIPPED:  py311: InterpreterNotFound: python3.11
  congratulations :)
Tox exit code: 0
```

The worst part is that Tox exits with code 0.

The 0 exit code means [CircleCI run 11003 for the build-and-unit-test workflow reported success](https://app.circleci.com/pipelines/github/Sceptre/sceptre/2017/workflows/1e9218f3-2354-4528-b1fb-40d010b09479/jobs/11003).

I would prefer that Tox fails when it can't find the target interpreter.

I can make that happen by deleting the `skip_missing_interpreter` line from its config.

https://github.com/Sceptre/sceptre/blob/1ad3f303f0bb1aa946c36ee2674553dc2e502031/tox.ini#L4

```bash
docker run -it cimg/python:3.12 bash -c '
git clone https://github.com/Sceptre/sceptre .
git fetch origin pull/1394/head:pr
git switch pr
sed -i "/skip_missing_interpreters/d" tox.ini
poetry install --all-extras -v
poetry run tox
echo "Tox exit code: $?"
'
```

Tox still runs all the tests it can, but now it fails because of missing interpreters.

```text
ERROR:  py38: InterpreterNotFound: python3.8
ERROR:  py39: InterpreterNotFound: python3.9
  py310: commands succeeded
ERROR:  py311: InterpreterNotFound: python3.11
Tox exit code: 1
```

From the [tox docs](https://tox.wiki/en/latest/config.html#skip_missing_interpreters):

> Setting this to `true` will force tox to return success even if some of the specified environments were missing. This is useful for some CI systems or when running on a developer box, where you might only have a subset of all your supported interpreters installed but don’t want to mark the build as failed because of it.

## Reopen PR 1393

2023-12-02

Khai closed PR 1394 and reopened PR 1393. He requested a review again.

This one doesn't include all the changes for Python 3.12. That wasn't the main point anyway. The point is to drop Python 3.7.

It uses the sceptre-circleci image that installs all the Python versions.

Here is the log of the image layers for version 2.1.0.

https://hub.docker.com/layers/sceptreorg/sceptre-circleci/2.1.0/images/sha256-1fbd22047c910e3709b77bf80aafd005ec9a84ce8ee69b7195f8eea83633bdf3?context=explore

```console
$ docker run -it --rm sceptreorg/sceptre-circleci:2.1.0 bash -c 'for py in python3.{7,8,9,10,11,12}; do "$py" --version; done'
bash: line 1: python3.7: command not found
Python 3.8.18
Python 3.9.18
Python 3.10.6
Python 3.11.6
Python 3.12.0
```

## Show how Poetry generates classifiers

2023-12-08.

[A response to Khai's question about the Python version constraint](https://github.com/Sceptre/sceptre-file-resolver/pull/12#discussion_r1420896311).

> @branchvincent mentioned that the supported python version classifier is automatically added based on the python requirement? don't we need the upper bound to restrict the documented support?

[The Poetry documentation](https://python-poetry.org/docs/pyproject#classifiers) doesn't give much detail on how this works.

Thanks @branchvincent for the demo. I see the same behavior here.

List available Python feature versions.

```bash
compgen -c | grep -P '^python3\.\d+$' | sort -V -u
```

I still have Python 3.7 installed, and I haven't installed Python 3.12 yet.

```text
python3.7
python3.8
python3.9
python3.10
python3.11
```

Build a Python package from Poetry's new project template.

```bash
cd "$(mktemp --dir)"
poetry new . --quiet
poetry build --quiet
```

Check the Python constraint.

```bash
cat pyproject.toml | grep -P "dependencies|python = "
```

Python 3.8 is the minimum Python version. All later Python 3 versions are allowed.

```toml
[tool.poetry.dependencies]
python = "^3.8"
```

Check the classifiers.

```bash
unzip -q -c dist/*.whl | grep "Programming Language"
```

Because the minimum Python version is 3.8 Poetry excludes the classifier for Python 3.7.

Because the environment doesn't have Python 3.12 Poetry excludes the classifier for Python 3.12.

```text
Classifier: Programming Language :: Python :: 3
Classifier: Programming Language :: Python :: 3.8
Classifier: Programming Language :: Python :: 3.9
Classifier: Programming Language :: Python :: 3.10
Classifier: Programming Language :: Python :: 3.11
```

## Collect and link all PRs

2023-12-12.

I updated the first comment in main PR to link to all the PRs I know of that address this work.

## Debug failed docs build

2023-12-16.

Branch Vincent submitted [PR 1395](https://github.com/Sceptre/sceptre/pull/1395) to use CircleCI's own Python image and other native features to simplify CI configuration.

[Khai noticed](https://github.com/Sceptre/sceptre/pull/1395/files#r1421777615) that the [`deploy-docs-branch` job failed](https://app.circleci.com/pipelines/github/Sceptre/sceptre/2034/workflows/885ac577-7398-406f-a33d-3542580761cd/jobs/11094).

```text
Creating virtualenv *******-3aSsmiER-py3.8 in /home/circleci/.cache/pypoetry/virtualenvs
# github.com:22 SSH-2.0-babeld-756a9a22
Cloning into '*******.github.io'...
remote: Enumerating objects: 9231, done.
remote: Counting objects: 100% (1110/1110), done.
remote: Compressing objects: 100% (181/181), done.
remote: Total 9231 (delta 930), reused 1098 (delta 924), pack-reused 8121
Receiving objects: 100% (9231/9231), 11.52 MiB | 47.36 MiB/s, done.
Resolving deltas: 100% (7459/7459), done.
Building docs in /home/circleci/docs/*******.github.io/dev
./.circleci/github-pages.sh: line 63: sphinx-apidoc: command not found

Exited with code exit status 127
```

The `sphinx-apidoc` command is part of the sphinx package or one of its companions. Some of those packages are optional extras.

Poetry's install command's `--all-extras` option controls whether to install optional extras.

The PR rewrites the `--all-extras` part. Is the setting now ignored? (See below for more analysis on the rewrite.)

The old CircleCI config uses a `run` step to execute the same command I would in my terminal.

```yaml
jobs:
  build:
    docker:
      - image: sceptreorg/sceptre-circleci:2.1.0
    steps:
      - ...
      - run:
          name: 'Installing Dependencies'
          command: poetry install --all-extras -v
```

The new CircleCI config replaces that with:

```yaml
jobs:
  build:
    executor: python/default
      - python/install-packages:
          pkg-manager: poetry
          args: --all-extras
```

Which package does `sphinx-apidoc` belong to?

How do you find the source package for an executable?

Read [JRD's answer](https://stackoverflow.com/questions/33483818/how-to-find-which-pip-package-owns-a-file/33484229#33484229) on Stack Overflow.

There's no simple command for that, but you can build a solution using `pip show`.

I'll use this once in an environment with all the optional extras and once with no extras.

```bash
poetry run bash <<"EOF"
find_package_for_file() {
  file="$1"
  dependencies=$(pip list | tail +3 | cut -d' ' -f1)

  for package in $dependencies; do
      package_info=$(pip show -f "$package")
      if [[ $package_info =~ $file ]]; then
        printf "%s\n" "$package_info"
      fi
  done
}

find_package_for_file "sphinx-apidoc"
EOF
```

Rebuild the virtualenv and use the `--all-extras` option.

```bash
bash <<"EOF"
poetry env remove 3.8
poetry env use 3.8
poetry install --verbose --all-extras
EOF
```

The package search prints the package info for `sphinx`:

```text
Name: Sphinx
Version: 5.1.1
Summary: Python documentation generator
Home-page: https://www.sphinx-doc.org/
Author: Georg Brandl
Author-email: georg@python.org
License: BSD
Location: /home/isme/.cache/pypoetry/virtualenvs/sceptre-ltLaG0f3-py3.8/lib/python3.8/site-packages
Requires: alabaster, babel, docutils, imagesize, importlib-metadata, Jinja2, packaging, Pygments, requests, snowballstemmer, sphinxcontrib-applehelp, sphinxcontrib-devhelp, sphinxcontrib-htmlhelp, sphinxcontrib-jsmath, sphinxcontrib-qthelp, sphinxcontrib-serializinghtml
Required-by: sphinx-autodoc-typehints, sphinx-click, sphinx-rtd-theme
Files:
  ../../../bin/sphinx-apidoc
...
```

The virtualenv has the `sphinx-apidoc` command.

```console
$ poetry run sphinx-apidoc
usage: sphinx-apidoc [OPTIONS] -o <OUTPUT_PATH> <MODULE_PATH> [EXCLUDE_PATTERN, ...]
sphinx-apidoc: error: the following arguments are required: module_path, exclude_pattern, -o/--output-dir
```

Rebuild the virtualenv and don't use the `--all-extras` option.

```bash
bash <<"EOF"
poetry env remove 3.8
poetry env use 3.8
poetry install --verbose
EOF
```

The `--verbose` option shows skipped packages.

```text
  • Installing sphinx (5.1.1): Skipped for the following reason: Not required
```

The package search prints no package info.

The virtualenv doesn't have the `sphinx-apidoc` command.

```console
$ poetry run sphinx-apidoc
Command not found: sphinx-apidoc
```

## Understand the all-extras rewrite

2023-12-16.

PR 1395 rewrites the step that installs the Poetry project with the `--all-extras` option.

Now the `deploy-docs-branch` job fails because the `sphinx-apidocs` command is missing.

The old CircleCI config uses a `run` step to execute the same command I would in my terminal.

```yaml
jobs:
  build:
    docker:
      - image: sceptreorg/sceptre-circleci:2.1.0
    steps:
      - ...
      - run:
          name: 'Installing Dependencies'
          command: poetry install --all-extras -v
```

The new CircleCI config replaces that with CircleCI orb syntax:

```yaml
orbs:
  python: circleci/python@2.1.1

jobs:
  build:
    executor: python/default
      - python/install-packages:
          pkg-manager: poetry
          args: --all-extras
```

Read the [Jobs and Steps documentation](https://circleci.com/docs/jobs-steps/).

On orbs:

> Orbs are packages or reusable configuration that you can use in your projects. Orbs usually contain commands that you can use in your jobs, as well as whole jobs that you can schedule in your workflows.

On executors:

> Jobs can be run in Docker containers, using the Docker executor, or in virtual machines using the `machine` executor, with Linux, macOS, or Windows images.

> When using the Docker executor, images listed under the `docker` key specify the containers you want to start for your job.

Read the [Execution environments overview](https://circleci.com/docs/executor-intro/).

CircleCI has a special key for the Docker executor (`docker`) and the machine executor (`machine`). Other executors, such as those provided by orbs, use the `executor` key.

Read the [circleci/python orb](https://circleci.com/developer/orbs/orb/circleci/python).

The `default` executor appears to be an alias for the docker image `cimg/python`. The version tag is parametrized and defaults to `3.8`.

The `install-packages` command sets up a Python environment and installs the packages.

> With poetry as pkg-manager, the command will assume `--no-ansi`.

If the install config looks like this:

```yaml
      - python/install-packages:
          pkg-manager: poetry
          args: --all-extras
```

The the command it should run is `poetry install --no-ansi --all-extras`.

The step that installs the dependencies is in a different job from the step that build the docs and fails.

* Project: Sceptre
* Branch: ci-matrix
* Workflow: build-test-and-deploy
* [Job: build (11085)](https://app.circleci.com/pipelines/github/Sceptre/sceptre/2034/workflows/885ac577-7398-406f-a33d-3542580761cd/jobs/11085)
* Step: Install dependencies with poetry using project pyproject.toml
* Command: `poetry install --no-ansi --all-extras`

Step output:

```text
Creating virtualenv sceptre-3aSsmiER-py3.8 in /home/circleci/.cache/pypoetry/virtualenvs
Installing dependencies from lock file

Package operations: 76 installs, 1 update, 0 removals

  • Installing six (1.16.0)
  • Installing certifi (2023.11.17)
  • Installing charset-normalizer (3.3.2)
  • Installing idna (3.6)
  • Installing jmespath (1.0.1)
  • Installing markupsafe (2.1.3)
  • Installing pyparsing (3.1.1)
  • Installing python-dateutil (2.8.2)
  • Installing pytz (2023.3.post1)
  • Installing urllib3 (1.26.18)
  • Installing zipp (3.17.0)
  • Installing alabaster (0.7.13)
  • Installing babel (2.13.1)
  • Installing botocore (1.33.6)
  • Installing distlib (0.3.7)
  • Installing docutils (0.16)
  • Installing exceptiongroup (1.2.0)
  • Installing click (8.1.7)
  • Installing importlib-metadata (6.9.0)
  • Installing jinja2 (3.1.2)
  • Installing iniconfig (2.0.0)
  • Installing pluggy (1.3.0)
  • Installing pyyaml (6.0.1)
  • Installing requests (2.31.0)
  • Installing filelock (3.13.1)
  • Installing pygments (2.17.2)
  • Installing parse (1.20.0)
  • Installing imagesize (1.4.1)
  • Updating setuptools (68.0.0 -> 69.0.2)
  • Installing packaging (21.3)
  • Installing sphinxcontrib-applehelp (1.0.4)
  • Installing sphinxcontrib-htmlhelp (2.0.1)
  • Installing sphinxcontrib-serializinghtml (1.1.5)
  • Installing tomli (2.0.1)
  • Installing sphinxcontrib-qthelp (1.0.3)
  • Installing platformdirs (4.0.0)
  • Installing webencodings (0.5.1)
  • Installing sphinxcontrib-jsmath (1.0.1)
  • Installing snowballstemmer (2.2.0)
  • Installing sphinxcontrib-devhelp (1.0.2)
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
  • Installing attrs (23.1.0)
  • Installing bleach (6.1.0)
  • Installing cfgv (3.4.0)
  • Installing cfn-flip (1.3.0)
  • Installing coverage (7.3.2)
  • Installing identify (2.5.32)
  • Installing nodeenv (1.8.0)
  • Installing ordered-set (4.1.0)
  • Installing parse-type (0.6.2)
  • Installing py (1.11.0)
  • Installing s3transfer (0.8.2)
  • Installing pytest (7.4.3)
  • Installing pyrsistent (0.20.0)
  • Installing termcolor (2.4.0)
  • Installing sphinx (5.1.1)
  • Installing virtualenv (20.25.0)
  • Installing toml (0.10.2)
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
  • Installing behave (1.2.6)
  • Installing boto3 (1.33.6)
  • Installing colorama (0.4.3)
  • Installing deepdiff (5.8.1)
  • Installing freezegun (0.3.15)
  • Installing jsonschema (3.2.0)
  • Installing pre-commit (2.21.0)
  • Installing readme-renderer (24.0)
  • Installing requests-mock (1.11.0)
  • Installing sceptre-file-resolver (1.0.6)
  • Installing sphinx-click (3.1.0)
  • Installing sphinx-rtd-theme (0.5.2)
  • Installing deprecation (2.1.0)
  • Installing sphinx-autodoc-typehints (1.19.2)
  • Installing troposphere (4.5.2)
  • Installing pytest-cov (2.12.1)
  • Installing tox (3.28.0)
  • Installing pytest-sugar (0.9.7)
  • Installing networkx (2.6.3)
  • Installing sceptre-cmd-resolver (2.0.0)
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10
Connection pool is full, discarding connection: pypi.org. Connection pool size: 10

Installing the current project: sceptre (4.3.0)
```

The step does appear to install sphinx, so my guess about it ignoring the `--all-extras` option is wrong.

## Next steps

Sceptre/sceptre:

Q: `Dockerfile` Python 3.10. Should it be Python 3.11?

Q: NullHandler in various `__init__` files. Use the built-in version?

Q: Review `__author__` and `__email__` details for all modules.

Q: Simplify `_iterate_entry_points` now that all target Pythons support some version of importlib.metadata?
