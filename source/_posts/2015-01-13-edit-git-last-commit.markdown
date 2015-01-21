---
layout: post
title: "Edit git last commit"
date: 2015-01-13 14:44:28 +0100
comments: true
categories: git
---

Today, while I was fixing a github issue on
[Chess](https://github.com/pioz/chess), I noticed this situation: I fixed that
issue and I did a commit `git commit -am "fix issue bla bla"`. Then I realized
that I didn't increment the version number of the gem and so I couldn't push it
on Rubygems. Now I can edit the version number and perform a second commit, or
find a way to edit the last commit and include the change of the file
`version.rb`.
How to do this? First edit the file `version.rb`, then increment the version of
the gem, and finally commit with the follow command:

    git commit -a --amend --no-edit

Good! We now have edited the last commit. If you already pushed, the SHA-1 of the
last commit will be different after this operation and so we need to push with
the option `--force`:

    git push -f  
