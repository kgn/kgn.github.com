---
layout: post
title: "My Git Workflow"
date: 2012-02-29 21:01

comments: true
sharing: true
footer: true

categories: 
- git
- Gitbox
- GitX
- GitHub
- workflow

---

I love [git](http://git-scm.com) and [Github](http://github.com), I originally started out with svn then moved to [mercurial](http://mercurial.selenic.com/) and even contributed and was an admin on a mercurial app called [Murky](https://bitbucket.org/snej/murky/wiki/Home). But then it became clear that git, with the help of Github, had won. It was added to Xcode 4 and even before that most iOS and web projects were hosted on Github. So I made the switch and never looked back!

<!-- more -->

I've been using git for almost 2 years now and I'd like to share some of the tricks I've gathered up and how I work with it. I unfortunately have a quite fragmented workflow split between the command line and two apps, but this allows me to use the best tool for the job. Hopefully this post will give you some new arrows for your git quiver.

### Command Line

I use the command line for most things branching, merging, checking out new repos and other commands that I don't want a higher level tool handling for me, I wanna know exactly what's going on. I use a lot of aliases to help this workflow go faster and to remind me of some of git's more *cryptic* commands.

This is what my global config looks like:

{% gist 760788 .gitconfig %}

I also use a custom `PS1` line that shows me which directory I'm in, the current branch if I'm in a git repo, and if there are any uncommitted changes.

    username@machine[dir](*master)$
    
{% gist 760788 .profile %}

### Gitbox

[Gitbox](http://gitboxapp.com) is the newest addition to my git workflow. I had looked at it a while ago and it didn't really appeal to me, however the latest version added support for [submodules](http://book.git-scm.com/5_submodules.html) which I use in almost every project! This feature is so awesome to be able to visualize what submodules the project is using and what state they are in.

{% img center http://kgn.github.com/content/git/gitbox.png %}

It's also really nice for managing multiple git repositories because you can see them all in the list view. There are lots of other great features so be sure to check out the [Gitbox site](http://gitboxapp.com).

### GitX

GitX is the original Mac git clients. I know a lot of people have switched over to using Gitbox, [Github's Mac app](http://mac.github.com/), or [SourceTree](http://www.sourcetreeapp.com/) but for me GitX is still the best! Even with Gitbox I still use GitX to stage my commits because it has a great diff view that makes it easy to stage chunks or even individual lines.

{% img center http://kgn.github.com/content/git/gitx.png %}

Another subtile but great feature of GitX's commit view is the grey line in the commit box. I wasn't sure what this was for until I read [this post](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html) about formatting git commits. The first line of a git commit is called the *subject* and it is used in several of the command line tools like log. It is recommended that the first line of a git commit be less than 50 characters long for readability on the command line and this grey line indicates where that cutoff is!

{% img center http://kgn.github.com/content/git/log.png %}

I use [my own fork](https://github.com/kgn/gitx) of [Brotherbards's GitX](http://brotherbard.com/blog/2010/03/experimental-gitx-fork/) where I've added Lion fullscreen support and changed the commit hotkey to `⌘+enter`.


### Looking forward

I've talked to Oleg, the developer of Gitbox, and he's told me that a diff view is coming to Gitbox that will hopefully have similar functionality to GitX's! I can't wait for this so I can hopefully whittle my workflow down to just command line and Gitbox.
