---
layout: post
title: "Gestire differenti versioni di gcc su OSX"
date: 2013-03-27 20:56
comments: true
categories: [osx, gcc, port]
---

Vediamo in questo post come gestire differenti versioni di gcc su Mac OS X con port.

La storia è molto semplice grazie all'utility `gcc_select` che installiamo nel seguente modo:

    $ sudo port install gcc_select
    
A questo punto possiamo vedere le varie versioni di gcc che sono installate sul nostro
sistema operativo con il comando:

    $ port select --list gcc
    Available versions for gcc:
    	llvm-gcc42
    	mp-gcc47 (active)
    	none
    	
In questo esempio il gcc che viene usato è il gcc 4.7 installato con port.
Inoltre faccio notare che `none` si riferisce al gcc che si installa con xcode.

Per switchare versione possiamo dare il comando:

    $ sudo port select --set gcc llvm-gcc42
    Selecting 'llvm-gcc42' for 'gcc' succeeded. 'llvm-gcc42' is now active.
    