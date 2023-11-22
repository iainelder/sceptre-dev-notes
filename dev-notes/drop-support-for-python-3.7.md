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

## Next steps

Sceptre/sceptre:

Q: `Dockerfile` Python 3.10. Should it be Python 3.11?

Q: NullHandler in various `__init__` files. Use the built-in version?

Q: Review `__author__` and `__email__` details for all modules.

Q: Simplify `_iterate_entry_points` now that all target Pythons support some version of importlib.metadata?

TODO: Follow Khai's advice on Slack to set up my CircleCI account
