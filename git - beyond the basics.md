# `git` Beyond The Basics

## What You Need To Know To REALLY Fuck Things Up

### References

#### Single

- `^` - First parent **branch** of a **reference** - travel by breadth
- `^{Nth}` - Nth parent **branch** of a **reference**
- `~` - Also first parent of a **reference**, but should really always be previous commit. HOWEVER...
- `~{X} - ...steps back X "parents" i.e. commits - travel by depth

```sh
git show HEAD^1  # First parent
git show HEAD^2  # Second parent **branch**
git show HEAD~1  # Previous commit. Maybe it's a parent in the loosest sense, maybe not
git show HEAD~X  # "X" commits back, on the first parent branch

# Can be combined
git show HEAD^2~2  # Second previous *commit* on parent branch #2

```

##### Ranges

###### Double-dot

```sh
# all commits in {branch_name} *not* in {other_branch_name}
git log {other_branch_name}..{branch_name}

# inverse; all commits in {other_branch_name} *not* in {branch_name}
git log {branch_name}..{other_branch_name}

# (git substitutes HEAD if there's no reference)
git log {other_branch_name}..
# or
git log ..{other_branch_name}

# NICE: show any commits in the current branch which aren’t in {other_branch_name} on {remote}.
git log {remote}/{other_branch_name}..HEAD
```

###### Triple-dot

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

# See log of where HEAD and other branch refs have pointed (recently, for some values of "recently"), not unlike bash history
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
# (copy content from another branch, but NO HISTORY IS COPIED, so *nothing* like `reset`)
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
- `f` "flatten"/"fold" (technically `fixup`, which makes no sense to me) - "Fold" commit, i.e. apply on top of previous, discarding commit message
- `s` "squash" - Same as `f`, but *keep* the message and append to previous message
- `d` "drop", although it's easier to just delete the commit's line

Less common:

- `r` "reword" - Basically "--amend"
- `e` "edit" - Stop to allow for editing working copy. Like being in a pocket universe: changes must be `add`-ed and `commit`-ed, and rebase `continue`-d

### Fixes

#### Can't `push --force`? Don't want a merge commit?

```sh
git rebase -i {remote_name}/{branch_name}
# and then push
```

### Before submitting an MR

```sh
git checkout master
git fetch --all
git merge --ff-only upstream/master
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

##### Rename

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

## Nice aliases (for .gitconfig)

```config
patch = "!git diff --no-pager --no-color #"
dsf = "!git [ -z \"$GIT_PREFIX\" ] || cd \"$GIT_PREFIX\" && git diff --color \"$@\" | diff-so-fancy  | less --tabs=4 -RFX #"
mr = "!git fetch \"$1\" merge-requests/\"$2\"/head:mr-\"$1\"-\"$2\" && git checkout mr-\"$1\"-\"$2\" #"
con = config --global -l
co = checkout
s = status -s -b
pu = push
fa = fetch --all
bc = log --oneline --abbrev-commit master..
l = "!git log --pretty=format:\"%C(auto)%h %s %C(#555555)/ %aN @ %ar\" #"
st = stash
sl = stash list
cm = commit -am
br = checkout -b
pl = "!git fa && git merge upstream/master --ff-only #"
rei = "![ x$# != x1 ] && echo \"commit-ish required\" >&2 || git rebase -i HEAD~\"$1\" #"
```

`dsf` requires [diff-so-fancy](https://github.com/so-fancy/diff-so-fancy)
