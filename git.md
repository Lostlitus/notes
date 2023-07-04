# Git Note

## Pull Request

### Usage

Ideally, if someone want to commit to a remote repository, what he/she need to
do is to clone the remote repository, make some commits in local repository,
and then push it to the remote one. However, this needs the remote repository
to grant permission to everyone that claims to commit. But no permission
management always leads to a mess. So the more reasonable choice is to grant
permission only a core set of people as maintainers, and others who what to
commit send the commit information to these maintainers to let them deal with
merge stuff. This turns the commit routine from `committer -> repository` to
`committer -> maintainer -> repository`. maintainers serve as a role of
reviewer.

Nowadays, one of the most popular way to send the commit information is Pull
Request(or Merge Request in gitlab). Committer who opens a Pull Request sends
the information of the branch he/she specified to the remote, and maintainers
deside if to merge this branch and how. Anyone can discuss about this Pull
Request and committer can take some actions based on others' reactions.  All
changes to the branch stay in sync with the opened Pull Request.

A basic Pull Request workflow looks like this.

1. Committer forks the remote to his/her own remote repository.
2. Committer clones the forked repository as origin and sets the remote as
   upstream.
3. Committer creates a new branch from the branch he/she want to commit.
4. Committer makes commits.
5. Committer pushes this branch to the origin.
6. Committer opens a Pull Request to the upstream.
7. People discuss about this Pull Request and Committer makes changes according
   to the conclusion.(optional)
8. Maintainers agrees with this Pull Request and merge it.
9. Committer pull the upstream to update its local repository.

### Best Practice

We now introduce some best practices to scenarios that appear frequently.
Detailed introduction can be found in [Pro Git
5.2](https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project)

a. One branch for each Pull Request

Lets say the remote branch we are going to work on is branch *dev*, then for
each feature or fix or something else, you should create one branch for each on
*dev*. Compare to working directively on the local *dev* or mixing edits on one
separate branch, an independent branch for each Pull Request  means
flexibility.

b. Rebase before Pull Request

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

c. New Pull Request if needed

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
