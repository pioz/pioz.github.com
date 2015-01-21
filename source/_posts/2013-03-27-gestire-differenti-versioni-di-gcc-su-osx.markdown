---
layout: post
title: "Manage different versions of gcc on OSX"
date: 2013-03-27 20:56
comments: true
categories: [osx, gcc, port]
---

Let's see in this post how to manage different versions of gcc under Mac OS X with port.  
We can simply use the utility `gcc_select` that we can install as follows:

    $ sudo port install gcc_select

Now we can check the versions of gcc that are installed in our system with the command:

    $ port select --list gcc
    Available versions for gcc:
    	llvm-gcc42
    	mp-gcc47 (active)
    	none

In this example we are using gcc version 4.7 installed with port.  
To swith to another version type the command:

    $ sudo port select --set gcc llvm-gcc42
    Selecting 'llvm-gcc42' for 'gcc' succeeded. 'llvm-gcc42' is now active.
