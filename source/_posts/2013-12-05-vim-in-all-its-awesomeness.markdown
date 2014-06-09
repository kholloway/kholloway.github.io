---
layout: post
title: "Vim in all it's awesomeness"
date: 2013-12-05 08:36:29 -0500
comments: true
categories: [Janus, Vim]
---
I have tried many text editors (and various IDE's) over the years and I always come back to VIM for just about everything, especially anything programming related.
Vim on it's own is great but with some plugins and some simple changes it's awesome.
Everything from syntax highlighting (using Syntastic) to autocompletion of functions and variable names.
Since I write a lot of code in Python and Puppet (Syntastic has checkers for both and much more) which has saved me an incredible amount of time since I started using it.

Do yourself a favor and take 5 minutes to go get the incredible [Janus: Vim Distribution](https://github.com/carlhuda/janus).
It's a quick and painless install.
From the Janus README:

    $ brew install macvim    (optional - requires [Homebrew](http://brew.sh/))
    $ curl -Lo- https://bit.ly/janus-bootstrap | bash

The above two commands are all you need, that second command will also backup your existing Vim files in your home directory so you don't lose anything you may already have setup.

Customization is fairly simple using ~/.vimrc.before and ~/.vimrc.after files in your home dir.
The only changes I make to the base Janus setup is shown below, put these changes in ~/.vimrc.after if you like.

    " Clear searches easily with ,/ after
    nmap <silent> ,/ :nohlsearch<CR>

    " Give a shortcut key to NERD Tree
    map <F2> :NERDTreeToggle<CR>

    " Disable F1 help crap, map to ESC instead
    map <F1> <Esc>
    imap <F1> <Esc>

One of my favorite commands once you have Janus installed is to reformat an entire file with a simple keystroke:

* \<leader\>fef formats the entire file

The default \<leader\> character is \ so \fef reformats your current file.
Stop living in the dark ages, go install Janus right now!

