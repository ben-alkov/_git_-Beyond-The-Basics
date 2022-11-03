# `git` Beyond The Basics

## What You Need To Know To REALLY Fuck Things Up

### Easy CLI cherry-pick of a merge commit on a dev trunk

```sh
# `co`, `refresh`* and `new-branch` are aliases
git co main
git refresh
git co dev_main
git refresh-dev_main
git co main
git new-branch cherry-pick-foo
git cherry-pick -m 1 SHAHASHASHAHASHASHAHASHASHAHASHA
git add FOO
git commit
git push
# Now go open a PR
```

### References

#### Single

- `^` - First parent **branch** of a **reference** - travel by breadth
- `^{Nth}` - Nth parent **branch** of a **reference**
- `~` - Also first parent of a **reference**, but should really always be
        previous commit. HOWEVER...
- `~{X} - ...steps back X "parents" i.e. commits - travel by depth

```sh
git show HEAD^1  # First parent
git show HEAD^2  # Second parent **branch**
git show HEAD~1  # Previous commit. Maybe it's a parent in the loosest sense, maybe not
git show HEAD~X  # "X" commits back, on the first parent branch

# Can be combined
git show HEAD^2~2  # Second previous *commit* on parent branch #2

```

#### Ranges

##### Double-dot

```sh
# all commits in {branch_name} *not* in {other_branch_name}
git log {other_branch_name}..{branch_name}

# inverse; all commits in {other_branch_name} *not* in {branch_name}
git log {branch_name}..{other_branch_name}

# (git substitutes HEAD if there's no reference)
git log {other_branch_name}..
# or
git log ..{other_branch_name}

# NICE: show any commits in the current branch which aren’t in
# {other_branch_name} on {remote}.
git log {remote}/{other_branch_name}..HEAD
```

##### Triple-dot

```sh
# Show commits on either branch but *not* on both.
git log {other_branch_name}...{branch_name}

# NICE: Prints '<' or '>' to show which branch a commit comes from
git log --left-right {other_branch_name}...{branch_name}
```

### Misc

```sh
# set remote:
git remote add {remote_name}

# Uses HEAD for {branch_name}
git show {branch_name}

# See hash for {branch_name}'s HEAD
git rev-parse {branch_name}

# See log of where HEAD and other branch refs have pointed (recently, for some
# values of "recently"), not unlike bash history
git reflog
"
fe06ab (HEAD -> odcs-omits-composeid-in-autorebuild, origin/master, origin/HEAD , master) HEAD@{0}: checkout: moving from odcs-downgrade-intent to odcs-omits-composeid-in-autorebuild
8a70bb9 (odcs-downgrade-intent) HEAD@{1}: rebase -i (finish): returning to refs/heads/odcs-downgrade-intent
.
.
.
"

# Now use this reference type, e.g. 5 commits back
git show HEAD@{5}

# Moar magics
git show {branch_name}@{yesterday}  # date magics need literal '{}'

# Overwrite something in this branch using content from another branch...
# (copy content from another branch, but NO HISTORY IS COPIED,
# so *nothing* like `reset`)
git checkout {remote_name}/{branch_name} {file_or_dir}

# ...or even a specific commit
git checkout {commit_hash} {relative_path_to_file_or_dir}

# Hard reset a specific file
git checkout HEAD -- {file}

# diff anything
git diff {remote_name}/{branch_name}:path/to/foo.bar {other_remote_name}/{other_branch_name}:path/to/deeper/baz.bar
```

### Commit magic

```sh
git rebase -i HEAD~{num_commits_to_go_back}
```

Commits can be re-ordered

- `p` "pick" - Use a commit unchanged
- `f` "flatten"/"fold" (technically `fixup`, which makes no sense to me) -
      "Fold" commit, i.e. apply on top of previous, discarding commit message
- `s` "squash" - Same as `f`, but *keep* the message and append to previous message
- `d` "drop", although it's easier to just delete the commit's line

Less common:

- `r` "reword" - Basically "--amend"
- `e` "edit" - Stop to allow for editing working copy. Like being in a pocket
      universe: changes must be `add`-ed and `commit`-ed, and rebase `continue`-d

### Fixes

#### Interactive Rebase for fun and profit

- Scenario: Content from commit 'B' *should* be in commit 'C'

    `git rebase -i` - note the "edit" entries below

    ```text
    pick 73ff1bb Commit A, creating file1.txt
    edit 34b066b Commit B, creating file2.txt + editing file1.txt
    edit 97e9d62 Commit C, editing file2.txt
    ```

    ```text
    Stopped at 34b066b... Commit B, creating file2.txt + editing file1.txt
    You can amend the commit now, with

        git commit --amend

    Once you are satisfied with your changes, run

        git rebase --continue
    ```

    ```sh
    vi file1.txt    # edit file1 to delete content from commit 'B'
    git commit --amend file1.txt
    git rebase --continue
    ```

    git continues rebasing and stops on the Commit C to edit it

    ```sh
    vi file1.txt   # edit file1 to insert content into commit 'C'
    git commit --amend file1.txt
    git rebase --continue
    ```

    Note that it's a very good idea to put the content which is moving between
    commits into a temp file...

#### Remove files from a commit

Say you have a doc file in a commit which should be all unit tests...

NOTE: this is a destructive operation for any changes made to the "unwanted"
files. You should stash or otherwise back up the changes you mistakenly
committed.

It's a question of moving the mistakenly committed files back to the staging
area, from the previous commit. This can be done like

`git reset --soft HEAD^`

or

`git reset --soft HEAD~1  # or some other number of commits back`

Then reset the unwanted files in order to leave them out of the commit

`git reset HEAD path/to/unwanted_file`

This *will* reset the files to their pre-commit status. You should stash or
otherwise back up the changes you mistakenly committed.

Now commit again (this will re-use the original commit message)

`git commit -c ORIG_HEAD`

#### Can't `push --force`? Don't want a merge commit?

```sh
git rebase -i {remote_name}/{branch_name}
# and then push
```

### Before submitting an MR

```sh
git checkout master
git fetch --all
git merge --ff-only {upstream}/master
git checkout {local_branch_name}
git rebase -i master
```

### Nice `git log` commands

```sh
# For short hashes: `--abbrev-commit`
git log --oneline -10 master.. # go back 10 commits

# everything on all branches forever, in glorious color ASCII
git log --graph --all --oneline
```

### Branch magic

```sh
# remove remote ref after del/rename
git branch --unset-upstream
```

#### Delete

```sh
# local
git branch -d {branch_name}

# local, ignoring merged status
git branch -D {branch_name}
```

```sh
# remote
git push {remote_name} -d {branch_name}
```

#### Delete branches without upstreams

##### For remotes

`git remote prune {remote_name}`

Deletes any local branch (e.g. "origin/hotfix-jenkins-jobs": note the remote
    before the '/') which refers to a non-existent branch in a remote. If this
    is confusing, remember that git tracks remote branches locally (after e.g.
    "fetch")

##### For locals

`git branch -vv | grep 'origin/.*: gone]' | awk '{print $1}' | xargs git branch -d`

1. `git branch -vv`:
    "hotfix-jenkins-jobs    39b04e8 [origin/hotfix-jenkins-jobs: gone] Remove stale comments from jenkins osbs-integration-tests.yaml"

    Note "gone" after the branch name
2. `git branch -vv | grep 'origin/.*: gone]' | awk '{print $1}'`: "hotfix-jenkins-jobs"

    Grab just the branch names
3. `git branch -vv | grep 'origin/.*: gone]' | awk '{print $1}' | xargs git branch -d`

    Pipe to `xargs` for deletion

#### Rename

```sh
# this branch...
git branch -m {new_name}
# ...or a different branch
git branch -m {branch_name} {new_name}

# delete existing and push new
git push {remote_name} :{branch_name} {new_name}

# fix remote tracking branch
git push {remote_name} -u {new_name}
```

### Nice aliases (for .gitconfig)

Note that `dsf` requires [diff-so-fancy]

```config
# **Some dragons follow. Read and understand before using.**
bc = log --oneline --abbrev-commit master..
cm = commit -m
co = checkout
con = config --global -l
dsf = "!git [ -z \"$GIT_PREFIX\" ] || cd \"$GIT_PREFIX\" && git diff --color \"$@\" | diff-so-fancy  | less --tabs=4 -RFX #"
fa = fetch --all
l = "!git log --pretty=format:\"%C(auto)%h %s %C(#555555)/ %aN @ %ar\" #"

# Create a new branch, set it to track remote "origin", and push to "origin" to
#     create the tracking branch
new-branch = "![ x$# != x1 ] && echo \"Usage -- git new-branch <branch_name>\" >&2 || git co -b \"$1\" --track; git push origin -u #"
prune-remote = "![ x$# != x1 ] && echo \"remote name required\" >&2 || git remote prune \"$1\" #"
patch = "!git diff --no-pager --no-color #"
pu = push

# Refresh *your* local fork with a rebase of remote "upstream"'s master, then
#     push to *your* remote
pul = "!git fa && git rebase upstream/master && git push #"

# From a topic branch, switch to & refresh master, then switch back and start
#     an interactive rebase. This assumes two remotes; "upstream"
#     (from 'git pul'), and your fork
refresh = "!git co master && git pul && git push && git co - && git rebase -i master"

# The workhorse. Interactively rebase your topic branch against current
#     master/HEAD. You **MUST** provide the correct number of commits to rebase.
#     This WILL screw you horribly if you get it wrong, so inspect the list of
#     commits to edit carefully; if anything looks squirrely, remove everything
#     from the commit list, save, exit, and try again
rei = "![ x$# != x1 ] && echo \"number of commits to rebase required\" >&2 || git rebase -i HEAD~\"$1\" #"
s = status -s -b
sl = stash list

# Always use this for stashing. Requires that you provide a name.
ss = "![ x$# != x1 ] && echo \"stash message required\" >&2 || git stash push -m \"$1\" #"
st = stash

# Shows everything which git is ignoring in this repo, based on *all* configs,
#     along with the the source of the rule
what-ignore = "!git check-ignore -v *; git check-ignore -v **/*"
```

### Appendix: Fun With GitLab MRs

If you're a collaborator on an MR, and you can see "Members who can merge are allowed
to add commits.", do the following to make adjustments **in the PR itself**

> Note: 'thedude' is the person who opened the PR, 'foo' is their fork.
>
> 'topic-branch' is the work branch in 'thedude's fork.

```bash
git fetch git@gitlab.com:thedude/foo.git topic-branch
git checkout -b thedude-foo-topic-branch FETCH_HEAD
```

Now you're on local branch 'thedude-foo-topic-branch'. Make/add/commit any
adjustments, as you would in any topic branch.

When done, push as follows (Note the ':' between local_branch:remote_branch)

```bash
git push git@gitlab.com:thedude/foo.git thedude-foo-topic-branch:topic-branch
```

your changes should now automagically show up in the MR.

[diff-so-fancy]: https://github.com/so-fancy/diff-so-fancy
