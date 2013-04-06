---
layout: post
comments: true
title: "Migrating Your Source from TFS to Git"
---
# Migrating from TFS to Git

## Background
Recently the team that I was working with were mulling over switching from TFS to Git. Not that there were any questions about whether or not Git was superior, but there were some concerns around losing some of the history. Having never done this before, I just wanted to share the experience I had with this.

## Maintaining History
Luckily, prior to making this switch I had already been using git for the project. It was an android project and I had already found TFS extremely frustrating to work with so I was seeking another alternative. This led me to [git-tf](http://gittf.codeplex.com/). Git-TF is a set of cross-platform command-line tools for working between Git and TFS repositories. Though I have a couple of beefs with using git-tf, for the purposes of migrating between the two it will suffice.

### Keeping All TFS Commits
Once you have installed git-tf, the first thing you will want to do is clone from TFS into git. By default, git-tf will not preserve the entire history...but if you ask it nicely, it will oblige. The trick is the `--deep` operator.

{% highlight bash %}
git tf clone http://tfs:8080/tfs $/Path/To/Your/Repo what-i-aliased-it-as --deep
{% endhighlight %}

After running this, you should have the entire commit history from TFS in the `what-i-aliased-it-as` directory. If you `git log` the directory, you will see something like this:

{% highlight ruby %}
commit 7e752232e0591e90299273f5d66ec2005dc234d1
Author: Dude Feller <SOMEDOMAIN\dfeller>
Date:   Fri Apr 5 09:09:38 2013 -0400

    Some commit from TFS

commit 50f4e4df101fbea574fe8120fb77a47f9cff8e01
Author: Levi Wilson <SOMEDOMAIN\lwilson>
Date:   Thu Apr 4 16:59:49 2013 -0400

    Committing after I probably had to do a force pull from TFS
    because it sucks

commit a9b2f9e83a7dfe2505af0ee8a781c287395ae1c2
Author: Levi Wilson <SOMEDOMAIN\lwilson>
Date:   Thu Apr 4 16:59:48 2013 -0400

    Test all the thingz
{% endhighlight %}

This will suffice as we are able to jump around to any commit using a hash that `git-tf` created when it pulled from TFS. However, if you push this to GitHub, it is not going to know who `SOMEDOMAIN\dfeller` is. But if `Dude Feller` creates a GitHub account, all we need to do is make sure that the commit log has the proper e-mail for his commits. To do this, we will need to rewrite the git commit history. Luckly, `git` is a baller so it provides a way to do this.

## git filter-branch
Git has a `filter-branch` command that allows you to rewrite the history of a branch. This provides a lot of power, but for our purposes we are generally only interested in mapping the `SOMEDOMAIN` user accounts to their individual respective GitHub accounts. Here is a sample script that will update the information for `Dude Feller` and `Levi Wilson`.

{% highlight bash %}
#!/bin/sh

git filter-branch -f --env-filter '

am="$GIT_AUTHOR_EMAIL"
cm="$GIT_COMMITTER_EMAIL"

if [ "$GIT_COMMITTER_EMAIL" = "SOMEDOMAIN\dfeller" ]
then
    cm="dude@feller.com"
fi
if [ "$GIT_AUTHOR_EMAIL" = "SOMEDOMAIN\dfeller" ]
then
    am="dude@feller.com"
fi

if [ "$GIT_COMMITTER_EMAIL" = "SOMEDOMAIN\lwilson" ]
then
    cm="levi@leviwilson.com"
fi
if [ "$GIT_AUTHOR_EMAIL" = "SOMEDOMAIN\lwilson" ]
then
    am="levi@leviwilson.com"
fi

export GIT_AUTHOR_EMAIL="$am"
export GIT_COMMITTER_EMAIL="$cm"
'
{% endhighlight %}

### Caveat Emptor
It is important to note that using the `filter-branch` will rewrite your git history. For the purposes of migrating, this is not a problem. But you would not want to do this if other people were using this git repository. More information can be [found here](https://help.github.com/articles/changing-author-info).

### Who to Migrate
To figure out which users we need to bring over with the `filter-branch` bash script, simply do a `git log` on the local repository to find all of the unique users that have committed over the life of the TFS project.

{% highlight ruby %}
git log --format='%aN <%aE>' | sort -u

Dude Feller <SOMEDOMAIN\dfeller>
Levi Wilson <SOMEDOMAIN\lwilson>
Other People <SOMEDOMAIN\opeople>
...
{% endhighlight %}

## Conclusion
After we push the repository we created to GitHub, not only do we have all of the commit history from TFS, but we have additionally ensured that we take advantage of some of the graph information that GitHub provides on our repository that came from TFS.

![Final Commit Log](/images/preserved-history.png)
