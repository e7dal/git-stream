# Git Stream

[![Build Status](https://travis-ci.org/mrkmg/git-stream.svg?branch=master)](https://travis-ci.org/mrkmg/git-stream)

Git Stream is a branching model inspired by [git-flow](https://github.com/nvie/gitflow). I really enjoy what git-flow
offers, but the multiple primary branches, duplicated functionality (feature/support/bugfix), and dealing with pull
requests always kind of bugged me. After reading this [blog](http://endoflineblog.com/gitflow-considered-harmful) I
decided I would build my own "git-flow".

The name "Git Stream" was chosen as this plugin should streamline some of the repetitious actions in git.


[TOC levels=2-4]: # "#### Table of Contents"
#### Table of Contents
- [Installation](#installation)
- [Usage](#usage)
    - [Commands](#commands)
        - [init](#init)
        - [hotfix](#hotfix)
        - [feature](#feature)
        - [release](#release)
    - [Hooks](#hooks)
- [Example](#example)
- [Recommendations](#recommendations)
- [Other Notes](#other-notes)
- [Roadmap](#roadmap)
- [License](#license)


## Installation

Installation should be easy, and compatible with any platform which has bash and git installed.

Basic Procedure

- Clone Repo Somewhere
- Checkout the latest version
- Test on your platform. If you encounter a failing test, please report it as an issue
- Install

Example Installation on Linux

    git clone https://github.com/mrkmg/git-stream.git /tmp/git-stream
    cd /tmp/git-stream
    git submodule update --init --recursive
    #checkout latest release (v0.8.1)
    git checkout $(git describe --abbrev=0 --tags)
    make test
    sudo make install
    # To install the bash and zsh completion
    sudo make install_completion

By default, Git Stream will be installed to `/usr/local/bin`. If you would prefer to install somewhere else, you can
change the prefix. for example:

    sudo make install PREFIX=/usr

## Usage

All Git Stream commands are issued as follows:

    git stream <command>

### Commands

#### init

Initialize Git Stream on the current project.

    -d, --defaults                  Initialize with all default options
    -f, --force                     Force Initialization
    --version-prefix {prefix}       Version Prefix []
    --feature-prefix {prefix}       Feature Branch Prefix [feature/]
    --hotfix-prefix {prefix}        Hotfix Branch Prefix [hotfix/]
    --release-prefix {prefix}       Release Branch Prefix [release/]
    --working-branch {branch}       Working Branch [master]
    --default-push-remote {remote}  Remote branch to push to. If empty, no automated pushes will occur [origin]

#### hotfix

Work with hotfixes. Hotfixes are used to fix a bug in a release.

    start {version} {hotfix-name}
    finish [-l -m {message} -n -p] {versioned-hotfix-name} {new-version}
    list

    -m, --message {message}   A message for the merge (Implies -n)
    -n, --no-ff               Force a non fast-forward merge
    -d, --no-merge            Do not merge the hotfix branch back into the working branch (Implies -l)
    -l, --leave               Do not remove the hotfix branch
    -p, --no-push             Do not push the changes to the remote

`-l/--leave` is useful when hot-fixing an LTS version which is no longer relevant to the working branch

#### feature

Work with features. Features are used to implement new functionality.

    start {feature-name}
    finish [-m {message} -n] {feature-name}
    list
    update {feature-name}

    -m, --message {message}   A message for the merge (Implies -n)
    -n, --no-ff               Force a non fast-forward merge
    -l, --leave               Do not remove the feature branch
    -p, --no-push             Do not push the changes to the remote

`update` will bring the {feature-name} branch up to date with master. This is useful if master has changed since the
feature branch was created.

#### release

Work with releases. Releases are used to mark specific versions.

    start {version}
    finish [-l -m {message} -n -d] {version}
    list
    update

    -m, --message {message}   A message for the merge (Implies -n)
    -n, --no-ff               Force a non fast-forward merge
    -l, --leave               Do not remove the release branch
    -d, --no-merge            Do not merge the release branch back into the working branch
    -p, --no-push             Do not push the changes to the remote

### Hooks

Git Stream will run any hooks in the .git/hooks directory. In order to see boilerplate scripts, look at the
support/hooks directory.

Any "pre" hook which returns a non-zero status will halt the operation. "post" hooks exit codes are not examined and
will not affect the action from running.

The available hooks are: `post-stream-feature-finish`, `post-stream-feature-start`, `post-stream-feature-update`
`post-stream-hotfix-finish`, `post-stream-hotfix-start`, `post-stream-release-finish`, `post-stream-release-start`,
`pre-stream-feature-finish`, `pre-stream-feature-start`, `pre-stream-feature-update`, `pre-stream-hotfix-finish`,
`pre-stream-hotfix-start`, `pre-stream-release-finish`, and `pre-stream-release-start`

## Example

The following is a contrived example of working on a library with Git Stream.

If you have not already, initialize Git on your project.

    git init
    git add .
    git commit -m "Initial Commit"

Next, initialize Git Stream. (the -d makes Git Stream use all the default options)

    git stream init -d

You should now be on the master branch. This branch is where changes are stored for the next release.

So lets say you want to implement a new feature. Run the following:

    git stream feature start new-feature

In the above command, "new-feature" is the name of the new feature. This will create a new branch named
"feature/new-feature" forked from master and put you into that branch. This is where you would implement
your new feature.

Feel free to switch back to master, or any other branch. Your branch will stay there and allow you to come back and work
on the new feature anytime.

After you finish writing the new feature, go ahead and finish up the feature.

    git stream feature finish new-feature

If this runs correctly, git stream will merge that feature into your master branch and push it up to the origin.

If you run into the message `Failed to merge feature/new-feature into master.`, that means you can not simply merge the
feature into master. This can often happen if multiple features are being finished. You can either use Git Stream to fix
the issue or fix it manually.

To allow Git Stream to fix the issue, run the following:

    git stream feature update new-feature

or you can fix it manually with:

    git checkout feature/new-feature
    git rebase master

Fix all the conflicts, finish the rebase, and finish the feature.

    git add -a
    git rebase --continue
    git stream feature finish new-feature

If you would prefer to merge, you can merge master into feature/new-feature as well.

Now, your new feature is integrated into master, so lets make a release with this new feature. Start a new release.

    git stream release start 1.1.0

A new branch will be made named "release/1.1.0". This branch will be used to make the release. Actions may include
updating version numbers in documentation and source files, producing compiled files, etc. After you have completed
making all the changes and committed them, finish the release.

    git stream release finish 1.1.0

This should not have any conflicts, but if it does you can rebase like you did with features. This will merge the
release back into master and create a tag for the version. It will also push both your master branch and the new tags
up to origin.

    git stream release update 1.1.0

The last feature is the hotfix. Lets say some time goes by, you have committed to master a few times, started a couple
new features, but then a bug is reported that needs an immediate fix. This is where Hot Fixes come into play. If back in
version 1.1.0 a critical bug was introduced, we would want the release a version 1.1.1 which addresses this bug.

To make that fix, you start the hotfix.

    git stream hotfix start 1.1.0 security-flaw-bug

This will create a branch named "hotfix/1.1.0-security-flaw-bug" forked from the tag 1.1.0. Fix the bug and then finish
the hotfix, and implement it in 1.1.1.

    git stream hotfix finish 1.1.0-security-flaw-bug 1.1.1

This will tag your current hotfix branch as 1.1.1 and then rebase master with the fix.

If the output is "Hot Fix security-flaw-bug failed to be merged into master. Please manually merge.", you will need to
manually merge the bugfix into master. This can be caused if work done on master conflicts with the hotfix. Run the
following while checked out of master:

    git merge 1.1.1

Make the proper corrections, stage them, then make a commit.

## Recommendations

*The following items are personal preference.*

- Rebase instead of merge when updating your working branch. Rebasing prevents commit messages like "Merged upstream"
  which provide no meaningful history.
- Even in small projects, use feature branches. While not "needed", it can really help when you have a ton of ideas you
  are trying to manage.
- Tell your friends about Git Stream, and submit a PR if something is not awesome.

## Other Notes

For an automated deployment, you may want to checkout the latest release. _If you only use tags for release, you can
easily accomplish this with:_

    git checkout $(git describe --abbrev=0 --tags)

If you are working on a long running feature, you may want to occasionally bring the feature branch up to date. This can
be done via a rebase or merge. If we want to merge, issue the following while on your feature branch:

    git merge master

Not all use cases are covered here, for example putting multiple hotfixes in one release. Git Stream is very minimal and
you can use it when you want. For small, single-developer projects, it may be fine to work on new features directly on
the master branch.

While some of the git functions such as merging, pulling, pushing, etc are handled by Git Stream, there are still many
cases where you will still be using the native git commands. This is intentional. The goal of Git Stream is not to
replace the usage of git, but to augment it with some simple helper scripts.

## Roadmap

Please see the ROADMAP file.

## License

The MIT License (MIT) Copyright (c) 2017 Kevin Gravier <kevin@mrkmg.com>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated
documentation files (the "Software"), to deal in the Software without restriction, including without limitation the
rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit
persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the
Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE
WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR
OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
