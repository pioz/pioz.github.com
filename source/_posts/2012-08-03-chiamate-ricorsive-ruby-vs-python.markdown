---
layout: post
title: "Chiamate ricorsive Ruby vs Python"
date: 2012-08-03 21:27
comments: true
categories: [ruby, python, c]
---

Facciamo un piccolo test: calcoliamo Fibonacci ricorsivamente in Ruby (v1.9.3) e in Python (v2.7.2) e vediamo chi vince in termini di tempo

Ruby Fibonacci code (`f.rb`)


    def fib(n)
      if n == 0 || n == 1
        n
      else
        fib(n-1) + fib(n-2)
      end
    end
    
    36.times { |i| fib(i) }

Python Fibonacci code (`f.py`)

    def fib(n):
      if n == 0 or n == 1:
        return n
      else:
        return fib(n-1) + fib(n-2)
    
    for i in range(36):
      fib(i)

Ora lanciamo i due programmi nel seguente modo:

    time ruby f.rb && time python f.py

E vediamo l'output:


    real  0m6.417s
    user  0m6.411s
    sys   0m0.005s

    real  0m12.951s
    user  0m12.925s
    sys   0m0.023s

Ecco Ruby impiega **6.471** secondi mentre Python **12.951** secondi!

**Ruby vince nettamente su Python!!**

Oh beh provate però questo codice in C (`f.c`):

    int
    fib(int n)
    {
      if(n == 0 || n == 1)
        return n;
      else
        return fib(n-1) + fib(n-2);
    }

    int
    main()
    {
      int i;
      for(i = 0; i < 36; i++)
        fib(i);
    }

Compiliamo:

     gcc f.c

Ed eseguiamo:

    time ./a.out

0.002 secondi!