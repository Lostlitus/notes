# Git Note

## Check Changes

Git has various ways to check changes made, and the most basic one is `git
diff`.  Normally, we want to check these changes.

- Working directory and staging area.
- Staging area and any commit.
- Working directory and any commit.
- Two given commits.

### Compare working directory and staging area

`git diff` compares differences between working directory and staging area. If
all your modifications are staged, then `git diff` shows nothing.

### Compare staging area and any commit

`git diff --cached <commit>` compares differences between staging area and
given commit.

### Compare working directory and any commit

`git diff <commit>` comapres differences between working directory and given
commit.

### Compare two given commits

`git diff <commit> <commit>` compares differences between two given commits.

## Remove File

Remember that git has two areas in normal workflow, working tree directory and
staging area, respectively. Commands vary depending on from which part(s) we
want to remove file(s). Here we introduce three commands.

### Remove from working directory

`rm <file>...` removes given file(s) from working directory, and git prompts
removed file(s) in `Changes not staged for commit`.

### Remove from staging area

`git --cached rm <file>...` removes given file(s) from staging area, and git
prompts removed file(s) both in `Changes to be committed` and `Changes not
staged for commit`.

### Remove from both

`git rm <file>...` removes given file(s) from both working directory and
staging area, and git prompts removed file(s) in `Changes to be committed`.
When applying this command, git is quite cautious. That is, any modification of
the files you want to remove will stop the removal. Git then prompts you to
choose to either remove from staging area only with `git --cached rm <file>...`
or remove from both forcefully with `git -f rm <file>...`. This is a safety
feature to prevent accidental removal of data that has not yet been recorded in
a snapshot and that can not be recovered from Git.

## Remote Branch

Some implict rules come into play when git handles remote branch. To understand
these implict rules, we elaborate the following topics step by step.

1. How is branch implemented.
2. How does git synchronize remote and local branch.
3. What do commonly used git branch commands do.

### Branch in nutshell

Branch is a pointer in nutshell. It points to a commit(or nothing right after
initialization), and moves forward as commits added. We are always working on
one branch, and only this branch moves when we commit, while other branches
stay still. Git keeps a pointer of branch called HEAD to record where we are,
the whole relation looks like this.

```txt
HEAD -> branch -> commit
```

Add `--decorate` option to `git log` shows HEAD and branch information
explicitly.

### Synchronize remote and local

Git manipulates three branches to make remote and local branches work properly.

- Upstream branch
- Remote-tracking branch
- local branch

Upstream branch is the branch that stores in the remote. The local reference of
upstream branch is Remote-tracking branch, it takes the form
`<remote>/<branch>`.

Remote-tracking branch here acts as the bridge between local branch and
upstream branch, so that all work can be done locally without having to
contract remotes. When git fetches data from the remote, remote-tracking branch
synchronizes with the upstream branch automatically.

Local branch are created locally. It can track one upstream branch, and this
is manifested as tracking the corresponding remote-tracking branch.

### Git branch commands

We introduce git branch commands in the order of normal workflow.

1. `git clone <remote>` clones the remote repository. Now the local has all
   remote branches' information. Note that `git clone <remote>` only creates
   one local branch that trackes the remote master branch, other remote
   branches are not tracked(i.e. corresponding local branches are not created).
   So after `git clone <remote>`, we can see a local master branch and a
   remote-tracking `<remote>/master` branch which is corresponding to the
   remote master branch.
2. `git fetch <remote>` fetches data from the remote. Remote-tracking branches
   are moved forward, while local branches(including local branches that
   trackes remote-tracking branches) stay still.
3. `git remote add <remote>` followed by a `git fetch <remote>` adds a new
   remote and fetches data from it. Like `git clone <remote>`, it creates one
   remote-tracking branch that is corresponding to the remote master branch.
   But it does not create a local branch, because there is already one.
4. `git push <remote> <branch>` pushes the branch to the remote, then the
   remote adds this branch, which can be fetched later by others. But there
   exists one problem. `git push <remote> <branch>` by default does not set the
   local branch to track the remote one, so commands like `git pull` entered
   after pushing will not work. There are two commonly used workarounds. First
   one is pushing with `-u` option, which makes git to track the remote branch
   after pushing. Second one is `git branch -u <remote-branch>` after pushing,
   which sets the upstream branch explicitly.

## Filter Commit

Git provides lots of options to help filter commit that matches given pattern.
There are some frequently used options.

- Commit content matches pattern.
- Commit message matches pattern.
- Date in a given range.
- Modified files at a given path.

### Commit content

`git log -S<string>` and `git log -G<regex>` are used to filter commits content
with given pattern. Two things to note.

1. `-S` accepts a string by default, `--pickaxe-regex` is needed to make it
   match an extended POSIX regex.
2. `-S` filters a commit when the number of `<string>`'s occurrences in the
   whole commit changes, while `-G` filters each removed/added line. This means
   if one commit removes a line containing `<string>` and then add a line
   containing `<string>`, `-S` will not output it, while `-G` will.

### Commit message

`git log --grep=<pattern>` filters commit message with given pattern. If there
are more than one grep option, git by default shows commits that match any of
the patterns.  Adding the `--all-match` option further limits the output to
just those commits that match all patterns. There are other options that change
the behaviour of this option, like `-i` which lets regex matching ignore
letters case.

### Date range

`git log --since=<date>` and `git log --until=<date>` filter commit in this
range. This command works with lots of formats, so we can specify the date both
absolutely and relatively.

### Given path

`git log -- <path>...` filters commit that modifies files under the given
paths.

## Pull Request

Ideally, if someone want to commit to a remote repository, what he/she need to
do is to clone the remote repository, make some commits in local repository,
and then push it to the remote one. However, this needs the remote repository
to grant permission to everyone that wants to commit. Abuse permission like
this always leads to a mess. So the more reasonable choice is to grant only a
core set of people as maintainers, and others who what to commit should send
the commit information to these maintainers to let them deal with merge stuff.
This turns the commit routine from `committer -> repository` to `committer ->
maintainer -> repository`. maintainers serve as a role of reviewer.

Nowadays, one of the most popular way to send the commit information is Pull
Request(or Merge Request in gitlab). Committer opens a Pull Request with the
content of the branch he/she specified, and maintainers deside if to merge this
branch and how. Anyone can discuss about this Pull Request and committer can
take some actions based on others' reactions.  All changes to the branch stay
in sync with the opened Pull Request.

A basic Pull Request workflow looks like this.

1. Committer forks the remote to his/her own remote repository.
2. Committer clones the forked repository as origin and sets the remote as
   upstream.
3. Committer creates a new branch from the branch he/she want to commit.
4. Committer makes commits.
5. Committer pushes this branch to the origin.
6. Committer opens a Pull Request from the origin to the upstream.
7. People discuss about this Pull Request and Committer makes changes according
   to the conclusion.(optional)
8. Maintainers agrees with this Pull Request and merge it.
9. Committer pull the upstream to update its local repository.

### Best Practice

We now introduce some best practices to scenarios that appear frequently.
Detailed introduction can be found in [Pro Git
5.2](https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project)

#### One branch for each Pull Request

Lets say the remote branch we are going to work on is branch *dev*, then for
each feature or fix or something else, you should create one branch for each on
*dev*. Compare to working directively on the local *dev* or mixing edits on one
separate branch, an independent branch for each Pull Request  means
flexibility.

#### Rebase before Pull Request

It is always good for a committer to handle conflicts before Pull Request.
While merge and rebase are two basic methods to deal with conflicts. Both have
their applicable scenarios. In the case of Pull Request, rebase is better.

Normally, your Pull Request is put into a pool and wait for maintainers to
merge. So you should always assume that others are merged before yours and then
new conflicts are introduced. This means maintainers almost always take effects
to handle conflicts. Now that your branch are always merged by maintainers, why
do you have to merge before Pull Request and leave a unclean commit log?  It is
important to remember that efficient cooperation is always the first objective
in the case of Pull Request. A clean commit log benifits everyone. So always
rebase before Pull Request.

#### New Pull Request if needed

There exists cases that your Pull Request needs additional modification. In this
scenario, you either add commits to current Pull Request, or create a new Pull
Request. The former is enough in most cases, but the latter is possible as well.
For example, your Pull Request is hard to merge, and maintainer hope you can deal
with conflicts first. So your mission is to duplicate your branch to the newest
commit and create a new Pull Request. Details are list below.

Lets say the remote branch we are going to work on is branch *dev*, and your
stale Pull Request is on branch *feature* under HEAD~1. To duplicate *feature*
to HEAD, here is what to do.

```sh
git checkout dev
git pull dev
git checkout -b featruev2
git merge --squash feature
```

Note that what `git merge --squash feature` do is duplicating commits from
branch *feature* to branch *featruev2*. After handling conflicts, branch
*featruev2* can be used to open a new Pull Request with a logical same and
conflicts handled commits log. Now you only need to close the old Pull Request
and refer to it in this new Pull Request.
