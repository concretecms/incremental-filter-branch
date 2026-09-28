[![Tests](https://github.com/concretecms/incremental-filter-branch/actions/workflows/tests.yml/badge.svg)](https://github.com/concretecms/incremental-filter-branch/actions/workflows/tests.yml)

## Introduction

[`git filter-branch`](https://git-scm.com/docs/git-filter-branch) is a really nice git feature.
For instance, it allows fancy stuff like subtree-splitting.

Problems may arise when the repository contains a lot of commits: this operation can take a lot of time.

Luckily recent versions of git allow us to perform this operation in an incremental way:
the first time `filter-branch` still requires some time, but following calls can be very fast.


## Requirements

- git 2.16.0 or newer (on macOS: git 2.17.0 or newer)
- a POSIX shell
- common commands (`sed`, `grep`, `awk`, `cut`, `sort`, `tr`, `mktemp`, ...)
- `md5sum` (or `md5`)
- `flock` (optional: see the `--no-lock` option)

This script is tested on Linux and macOS, and it also works on Windows with the shell of [Git for Windows](https://gitforwindows.org/) (by using the `--no-lock` option).


## Installation

You can simply download the `incremental-git-filterbranch` script of one of the [released versions](https://github.com/concretecms/incremental-filter-branch/releases).
For example:

```sh
# Replace 1.2.3 with the version you want to use
VERSION=1.2.3
curl -sSLf -o incremental-git-filterbranch "https://raw.githubusercontent.com/concretecms/incremental-filter-branch/${VERSION}/bin/incremental-git-filterbranch"
chmod +x incremental-git-filterbranch
```

This script is also available as a [Composer](https://getcomposer.org/) package:

```sh
composer require concretecms/incremental-filter-branch
```


## Usage

Get the script and read the syntax using the `--help` option.

```sh
./bin/incremental-git-filterbranch --help
```

The script needs a working directory where it stores the data required by the following executions (see the `--workdir` option).
If you delete that directory, the next execution will process the whole source repository again.


### Filter

The filter is the list of the arguments to be passed to `git filter-branch`.
It's parsed with the syntax of the shell, so that you can use quotes to specify arguments containing spaces:

```sh
./bin/incremental-git-filterbranch \
    /path/to/source \
    '--prune-empty --msg-filter "sed s/foo/bar/"' \
    /path/to/destination
```

Please remark that the filter is evaluated by the shell: don't use filters coming from untrusted sources.


### Tags

You can control which tags are created in the destination repository with the `--tags-plan` option:

- `visited` (default): only the tags associated to the commits rewritten by the filter.
  For instance, when extracting a directory, the tags associated to commits that don't change that directory are skipped.
- `all`: all the tags.
  The tags associated to commits that are not rewritten by the filter are associated to the nearest rewritten commit.
  You can limit how far that commit can be with the `--tags-max-history-lookup` option
  (1: only the commit of the tag, 2: the commit of the tag and its parents, and so on).
- `none`: no tags at all.


## Examples

```sh
./bin/incremental-git-filterbranch \
    --branch-whitelist 'develop master rx:release\/.*' \
    --tag-blacklist 'rx:5\..*' \
    --tags-plan all --tags-max-history-lookup 10 \
    https://github.com/concretecms/concretecms.git \
    '--prune-empty --subdirectory-filter concrete' \
    git@github.com:concretecms/concretecms-core.git
```


## Legal stuff

Use at your own risk.
[MIT License](https://github.com/concretecms/incremental-filter-branch/blob/master/LICENSE).


## Credits

Special thanks to [Ian Campbell](https://github.com/ijc) for the implementation of the `--state-branch` option of git,
and his hints about how it can be used.
This script works only thanks to him (and if it doesn't work I'm the only person to blame).
