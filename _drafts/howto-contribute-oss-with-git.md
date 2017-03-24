---
title: Basics of git for open-source projects
layout: post
author: eric
cover: /images/developer-headache.jpg
tags:
- tools
- git
---

In this article...

---

# Summary
{:.no_toc}

* TOC
{:toc}

# Preamble

So you're going to contribute to a big open-source project, huh?  

**Well, this is your new leitmotiv**  
git checkout master  
git fetch upstream master  
git rebase upstream/master  
git push -f  
git checkout feature/awesome-new-feature  
git rebase -i {some commit hash}
git rebase origin/master
git push -f

**Yummy** :grin:.
Let's set the stage!

(But first take a deep breath and [**relax**](https://www.youtube.com/watch?v=glb88Sv9xpo))


# Setting the stage: Clone the git-tutorial repo

For this tutorial, we're going to use a repo that I've set up on our helyo github.  
[**https://github.com/helyo-world/git-tutorial**](https://github.com/helyo-world/git-tutorial)

Let's say this repo is a big open-source project we'd like to contribute to.  
First of all, assuming you already have a github account, log-in and fork the git-tutorial repo.

![fork the git-tutorial project on github](/images/posts/git-fork.png)

Back in your own github account, you should now see your own copy of the git-tutorial repo in the **Repositories** tab

![your own copy of the git-tutorial repo](/images/posts/forked-repo.png)

Now in your favorite shell, clone this repo
```
git clone https://github.com/ebarault/git-tutorial.git
```

**Note**: use `git clone git@github.com:ebarault/git-tutorial.git` if you're using ssh rather than https (which you should!), but let's stay focused for now :blush:


# Add a new remote pointing to the main repo

`cd` to the **git-tutorial** directory and execute the following command:

``` bash
$ git remote -vvv
origin	git@github.com:ebarault/git-tutorial.git (fetch)
origin	git@github.com:ebarault/git-tutorial.git (push)
```

For now, our local git repo is configured with a single remote leading to your forked instance of the git-tutorial repo.
We need to add another remote so we can easily fetch any modifications on the main repo.
For this we use the `git remote add upstream` command.

``` bash
$ git remote add upstream git@github.com:helyo-world/git-tutorial.git

$ git remote -vvv
origin	git@github.com:ebarault/git-tutorial.git (fetch)
origin	git@github.com:ebarault/git-tutorial.git (push)
upstream	git@github.com:helyo-world/git-tutorial.git (fetch)
upstream	git@github.com:helyo-world/git-tutorial.git (push)
```

You can see now that the repo is configured with a new remote leading to the main **git-tutorial** repo.

**Note**: I usually choose the name _upstream_ to refer to the main repo, but feel free to customize this according to your own tastes and habits.


# Create a feature branch and start the heavy lifting

Now we decide to start implementing some awesome new feature we'd like to ultimately backport to the main repo.
The best practice for this in general is to create a feature branch in your own forked repo, work in that branch until you consider the work ready to be reviewed, and then create a pull request in the main repo. Let's see how it's done.

First let's create a new branch for our new feature and check it out:

``` bash
$ git branch feature/my-awesome-new-feature
$ git checkout feature/my-awesome-new-feature
Switched to branch 'feature/my-awesome-new-feature'

$ git branch -vvv
* feature/my-awesome-new-feature
  master                         3fcdf26 [origin/master] first commit
```

**Note**: it's usually the best practice to prefix feature branches with **feature/**, it will help keep the work organized. But it's important that you follow the rules put in place in the main repo. Look for such rules in the **README.md** file or ask one of the repo maintainers.

Now let's implement our tremendous new feature, commit and push it to our forked repo.
And by tremendous I mean things like editing a Readme file, yes, this kind of heaving lifting! :muscle:

So with your favorite text editor, insert a few lines in the **README.md** file.
In fact, to simulate a day-to-day workflow, we're going to make several commits.

First I add the following line to the Readme.
> 20170314: ebarault was here and said `helyo world!`.

First commit:
``` bash
$ git add .
$ git commit -m 'first commit on forked repo'
```

Now I edit once again the Readme and fix my previous addition:
> 20170314: ebarault was here and said `helyo world!`. Isn't this a wonderful day?

Second commit:
``` bash
$ git add .
$ git commit -m 'second commit on forked repo'
```

Now we push our work to our own repo.
At first it won't work because our feature branch has no remote yet. 

```
$ git push
fatal: The current branch feature/my-awesome-new-feature has no upstream branch
```

We need to tell `git` where the content should be pushed: to which remote and to which remote branch. We fix this using the `--set-upstream` option.
We tell git to use the `origin` remote (which points to our repo), and use a remote branch named similarly to our local branch.

```
$ git push --set-upstream origin feature/my-awesome-new-feature
Total 0 (delta 0), reused 0 (delta 0)
To github.com:ebarault/git-tutorial.git
 * [new branch]      feature/my-awesome-new-feature -> feature/my-awesome-new-feature
Branch feature/my-awesome-new-feature set up to track remote branch feature/my-awesome-new-feature from origin.

~/helyo/git-tutorial$ git branch -vvv
* feature/my-awesome-new-feature 3fcdf26 [origin/feature/my-awesome-new-feature] second commit on forked repo
  master                         3fcdf26 [origin/master] first commit on main repo
```


# Share the work back with the community

## Act.1: Rebasing against the main master branch

:+1: Good, it looks we're ready to share this work with the main repo maintainers. For this we're going to create a pull request in the main repo.
Now the thing is, some ongoing work may have been committed to the master branch in the main repo while we were busy with all this Readme editing thing... So the commit tree of our feature branch in our forked repo is not aligned with the tree of the master branch of the main repo. A merge of some sort will be required in order to realign them all together.

![Async master branches](/images/posts/async-masters.png)

If we were to create the pull request now, the people in charge of reviewing your code would not be able to test it against the latest modifications they have merged in the master branch.

### Sync the masters

First we need to fetch the changes on the main repo's master and rebase our work on top of the changes committed while we were working on our awesome new feature.

```
$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.

$ git fetch upstream master
..
From github.com:helyo-world/git-tutorial
 * branch            master     -> FETCH_HEAD
   f64e8cb..b095079  master     -> upstream/master

$ git rebase upstream/master
First, rewinding head to replay your work on top of it...
Fast-forwarded master to upstream/master.
```

So let's review this:
1. We checked out our master branch.
2. Using `git fetch`, we fetched the work in the main master branch, asking for the **master** branch from the `upstream` remote.
3. Using `git rebase`, we rebased the content of the upstream/master branch on top of the currently selected branch which is our own master.

**Note**: Although it's not absolutely required to keep a carbon-copy of the main master in your forked repo, I find it more handy and less error-prone to always rebase against a forked master rather than the main one.

Let's check the git history:

```
~/helyo/git-tutorial$ git log
commit f64e8cb947190f08a52545321ee206f83b2d3cf0
Author: Eric Barault <eric.barault@gmail.com>
Date:   Tue Mar 14 12:09:43 2017 +0100

    First update on README.md

commit c70831ece6336394f3e4f1ecb47a1bca840c133c
Author: Eric Barault <eric.barault@gmail.com>
Date:   Tue Mar 14 12:08:45 2017 +0100

    Second update on README.md

commit 3fcdf26d867c7014a2e50da35b7d4fa4cd8b3d4c
Author: ebarault <eric.barault@gmail.com>
Date:   Tue Mar 14 09:39:52 2017 +0100

    first commit on main repo
```

Ok, it looks like 2 new commits were added on the main repo's master since we did the fork.
Our local master branch is now in sync with the main master branch, let's push the changes to our remote forked repo.

```
$ git push -f
...
To github.com:ebarault/git-tutorial.git
   3fcdf26..f64e8cb  master -> master
```

**Note**: The -f option is a shortcut for the `force` option. Now, you will find a lot of articles claiming that you should not force push, furthermore combined with rebasing, that you'd rewrite history, summon godzilla, or cause all sorts of mayhem, but here it's totally safe to do this as we just want to carbon-copy the main master in our own foked repo.

Good, the masters are in sync now! Let's move on.
![Synced master branches](/images/posts/sync-masters.png)

### Rebase the feature branch against the master

Now we're going to apply the latest changes committed on the master branch into our awesome feature branch.

```
$ git checkout feature/awesome-new-feature
$ git rebase origin/master
First, rewinding head to replay your work on top of it...
Applying: ebarault was here
Using index info to reconstruct a base tree...
M	README.md
Falling back to patching base and 3-way merge...
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
error: Failed to merge in the changes.
..
```

Arff... some changes applied into the master branch on the Readme are now conflicting with the changes we committed on the Readme in our feature branch (In other words, git is not smart enough to merge the changes and requires us to do it). Let's edit the **README.md** file to set things right.

README.md (before):
```
<<<<<<< HEAD
A sample repo to test git tricks
**Disclaimer**: Pull Requests are welcome !
=======
A sample repo to test git tricks

20170314: ebarault was here and said `helyo world!`
>>>>>>> ebarault was here
```

README.md (after):
```
A sample repo to test git tricks
**Disclaimer**: Pull Requests are welcome !

20170314: ebarault was here and said `helyo world!	 
```

**Note**: You will note that the Readme.md does not contain the content of our second commit, it misses the "_Isn't this a wonderful day?_" addition. It's because git has rewinded back our branch back to point where it is aligned with the master branch, and is now applying back our commits one by one. A conflict occured while applying the first of our two commits, hence the content of the **README.md** file at this stage.


All good, now save the file, `git add` it, and finish the `rebase` thanks to the `--continue` option:
```
$git add .
$git rebase --continue
```

Now let's verify the git history:
```
$git log
commit 431c42599821a2d1e2dc4cc48e53d87ad8f27662
Author: ebarault <eric.barault@gmail.com>
Date:   Tue Mar 14 12:13:26 2017 +0100

    second commit on forked repo

commit 2d51466c85e187025a76387416bfc386f10d2539
Author: ebarault <eric.barault@gmail.com>
Date:   Tue Mar 14 11:53:26 2017 +0100

    first commit on forked repo

commit f64e8cb947190f08a52545321ee206f83b2d3cf0
Author: Eric Barault <eric.barault@gmail.com>
Date:   Tue Mar 14 12:09:43 2017 +0100

    Update README.md

commit c70831ece6336394f3e4f1ecb47a1bca840c133c
Author: Eric Barault <eric.barault@gmail.com>
Date:   Tue Mar 14 12:08:45 2017 +0100

    Update README.md

commit 3fcdf26d867c7014a2e50da35b7d4fa4cd8b3d4c
Author: ebarault <eric.barault@gmail.com>
Date:   Tue Mar 14 09:39:52 2017 +0100

    first commit on main repo
```

It looks like everything is here: the new commits from master, our new commits from our feature branch.  
Let's check the **README.md** file.

`$ vim README.md`
```
A sample repo to test git tricks
**Disclaimer**: Pull Requests are welcome !

20170314: ebarault was here and said `helyo world!`. Isn't this a wonderful day?
```

Highest of fives?... Niiice!  
![Highest of fives](http://www.reactiongifs.com/r/5s.gif)


Let's force push the result of the rebasing to our repo.  

Why force push, you ask? Well the rule of the game here is to have the commit trees of the main master and our feature branch in compatible states. So if we were to go through a classic pull > merge > push process, we'd pollute the commit tree with our merge commits. The trees would end in the same position but not using the same track. We just don't want that so we force push, yes we rewrite history as if we'd been working with these two new commits from the main master from the beginning. This is what we do:

```
$ git push -f
```

## Act.2: Create a pull request

We are now ready to fire our pull request. Go back to your forked git-tutorial repo and go on with the shortcut proposed by github:

![Create a pull request](/images/posts/create-pull-request1.png)
Or create the pull request directly from the **branch** tab
![Create a pull request](/images/posts/create-pull-request2.png)

Go through the pull request creation steps and then go back to the main git-tutorial repo.
Go in the **pull request** tab: your pull request should show there. Click on it to see the details.

### Allow edits from maintainers

There's one very important thing here you should pay attention to, at least once: this little ticked option down the page, on the right, labelled "**Allow edits from maintainers**".
<div style="text-align: right">
  <img src="/images/posts/create-pull-request3.png" alt="Allow edits from maintainers">
</div>

What this means is: by default, if you don't uncheck this option, you are allowing the owners of the main repo to commit changes on the branch you have created a pull request from, in your own forked repo. This is meant to make the project maintainers' lives easier when interacting with your code, aiming at pushing the final tweaks required before merging your branch.

You can find more information regarding this option at the [following link](https://help.github.com/articles/allowing-changes-to-a-pull-request-branch-created-from-a-fork/#enabling-repository-maintainer-permissions-on-existing-pull-requests).

## Act.3: Keep the project's master commit tree clean

Now you're closing the discussion with the project's maintainers, ending with more new commits, possibly more rebasing against the master branch, and at this stage keeping all those commits was good to ease the discussion and permit rollbacks or cherrypicks to particular commits (more on this in a further article).

As good maintainers, usually the project owners will ask you to:
* **squash** all the commits related to this new feature into one
* **reword** the title of this single commit to reflect the main idea
* and provide a description of the feature inside the commit

This can easily be done using git's **interactive rebase** feature. For this you'll need the hash of the commit that was pushed just before your first feature commit.

```
git rebase -i f64e8cb947190f08a52545321ee206f83b2d3cf0
```

Following this command, git opens your favorite text editor and shows something similar to this:

```
pick 2d51466 first commit on forked repo
pick 431c425 second commit on forked repo

# Rebase f64e8cb..431c425 onto f64e8cb (2 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
```

For now the rebase is set up to pick (use) your two commits. What we want is to meld the second into the first and change the title of the resulting commit. To achieve this, simply edit the first 2 lines as:
```
reword 2d51466 ebarault was here
fixup 431c425 adding more wonderful content
```

**Note**: you can use either fixup or squash. It is entirely up to you, depending on whether you want to keep some info from the melded commits in the resulting one.

Now save the file and quit. Git will now reopen the editor, this time to let you reword the commit message:

```
An awesome new feature

In a first commit we added one line in the Readme.
Then in a second commit we modified this line.
Then we melded the 2 commits into one and came up
to this description.
```

Now save and quit the editor, the rebasing process ends:

```
[detached HEAD 14a3dde] ebarault was here at first content was added in 2 commits, then the second commit was merged in the first commit using fixup
 Date: Tue Mar 14 11:53:26 2017 +0100
 1 file changed, 2 insertions(+)
[detached HEAD c5ed990] ebarault was here at first content was added in 2 commits, then the second commit was merged in the first commit using fixup
 Date: Tue Mar 14 11:53:26 2017 +0100
 1 file changed, 2 insertions(+)
Successfully rebased and updated refs/heads/feature/my-awesome-new-feature.
```

Let's verify the git history:

```
$ git log
commit c5ed9900e6acfd86144d79359134f28fab1153ed
Author: ebarault <eric.barault@gmail.com>
Date:   Tue Mar 14 11:53:26 2017 +0100

    An awesome new feature

commit f64e8cb947190f08a52545321ee206f83b2d3cf0allo
Author: Eric Barault <eric.barault@gmail.com>
Date:   Tue Mar 14 12:09:43 2017 +0100

    Update README.md
...
```

Ok, our former two interim commits are gone, replaced by our new commit description.
Now we just have to force push the state of our branch to our forked repo. Again using force push since you deliberately want to rewrite history to make the commit tree cleaner now. Of course you can create a backup branch at any time before starting the rebasing process by doing nothing more than `git branch backup/my-awesome-new-feature` which would keep all your interim commits safe in there.

```
$ git push -f
```

The new branch content is now pushed to your feature branch, which is instantly reflected in the pull request you've created on the main repo:


# Happy ending

![Create a pull request](/images/posts/rebase-ok.png)

:+1: Now the project owners can safely merge your branch into the master branch, resulting in only two additional commits logged in the commit tree: the first one for your feature, the second for the merge.

![Create a pull request](/images/posts/pr-merged.png)

We are done. Congratulations, you've learned a few more tricks to contribute more efficently in open-source projects. :clap:  :tada: :rocket:
