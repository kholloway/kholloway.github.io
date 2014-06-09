---
layout: post
title: "Git - Move subdirectory to new repo"
date: 2013-04-04 15:08:22 -0500
comments: true
categories: [Git, Gitorious]
---
I had a need to move/detach a subdirectory that was inside a larger Git repository into it's own smaller/standalone repository.
After a few Google searches it turns out this is a fairly common and relatively easy thing to do.

Not required but I like to start with a fresh clone of the repo I'm working with into a temp directory.

    mkdir tmp
    cd tmp
    git clone my_original_repo_url

Now clone the repo again (this time it's a local only clone of the repo above):

    git clone --no-hardlinks my_original_repo new_repo_name
    cd new_repo_name

Extract just the subdirectory you want:

    git filter-branch --subdirectory-filter mysubdir

Now lets remove the old remotes, any unneeded history and repack the repo:

    git remote rm origin
    git update-ref -d refs/original/refs/heads/master
    git reflog expire --expire=now --all
    git repack -ad

Now you can add your new remote(s) in and push your changes up to the server:

    git remote add origin my_new_repo_url
    git push origin master

Note: If you are using Gitorious it may at this point complain about a 'invalid ref' when you push it to the server.
As far as I can tell this does not cause any problems and only occurs on the first push.

So that covers making your new repo from a subdirectory now lets go remove the now old subdirectory from the original repo so we don't commit to it by accident.
I'm using a simplified removal process, you could remove all references and commit info for the subdirectory if you like but for my case that was overkill.

    cd ../my_original_repo
    rm -rf mysubdir
    git rm -r mysubdir
    git commit -m "Removing subdir, it has been moved to its own repo now"
    git push origin master

All done!
Your subdirectory has now been moved from the original repository into a new repository with all your history and commits intact.
