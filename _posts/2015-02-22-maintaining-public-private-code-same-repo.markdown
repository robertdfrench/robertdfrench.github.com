---
layout: post
title:  "Maintaining public and private code in the same repo"
date:   2015-02-22 10:53:24
categories: git branch open-source submodule
---
Let's face it, git submodules suck. They are confusing, fragile, and orthogonal to the ecosystem of commits and refs that make up the rest of git's functionality. Fortunately, for some use cases, there are other routes.

### Private Apps with Public Libs
Sometimes while working on a private application, you might whip up a compact, unrelated set of modules to solve a small problem, or provide some extra functionality to the rest of your development environment. Often, this stuff might be applicable in a wider context. Occasionally, your employer will even give you the thumbs up to share it with the outside world. Pretty cool right? For a brief moment, you can get paid to work on your own open source project.

Now you're faced with the task of restructuring this code as a separate package with and including it into your main codebase as a proper dependency. This means you'll have to maintain it as an entirely separate package, and any new functionality you add needs to be rebuilt and then distributed to your private app just as any third party dependency would: while this is certainly The Right Way to Do It, it can also be a pain in the ass.

### Merge-only branches
In some narrow cases, it can make sense to have special "merge-only" branches that don't contain any proprietary code from your private app. Instead, they only contain the code for the open source software you want to share publicly.

{% highlight bash %}
git checkout <SHA of first commit>
git checkout -b public
# Hack away on public code
{% endhighlight %}

If you're starting this public project in the middle of an existing proprietary project, you'll want to make sure you branch from a commit that has no proprietary code. The first commit to your project might be an appropriate place to start, but it might also have sensitive data (i.e. the first import of your code from a previous version control system). If that is the case, take a look at GitHub's guide to [Removing sensitive data](https://help.github.com/articles/remove-sensitive-data/).

Once you've got a clean commit to branch from, you can set up special remotes for your public branch. For example:

{% highlight bash %}
git remote add public-origin https://github.com/CoolwareInc/cool-public-lib.git
git push -u public-origin master
{% endhighlight %}

Now you can work on your publicly available libs, tag them, and the merge them into your proprietary codebase
{% highlight bash %}
vim lib/awesome.rb
git commit -m "Refactor Awesome#helper to be 1,000 times faster"
git tag -a v1.2.3 -m "Performance improvements to libAwesome"
git push --follow-tags #Push any tags that reference pushed commits
# Now import back into private app
git checkout master
git merge public
{% endhighlight %}

### Downsides
This approach isn't perfect. Namely, it can be pretty easy to do public work in a private branch, making it a mess to back those changes out later. Or worse, you could accidentally push your private commits to the public remote. If you are interested in going down this route, take a look at [Git Hooks](http://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks), specifically the `pre-push` hook, which should allow you to double check whether you are trying to push private refs to the wrong remote. 

#### References
[Push git commits & tags simultaneously](http://stackoverflow.com/a/3745250)
