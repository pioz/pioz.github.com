---
layout: post
title: "Chiamate ricorsive Ruby vs Python"
date: 2012-08-03 21:27
comments: true
categories: [ruby, python, c]
---

Facciamo un piccolo test: calcoliamo Fibonacci ricorsivamente in Ruby (v2.0.0) e in Python (v2.7.2) e vediamo chi vince in termini di tempo

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

    $ time ruby f.rb && time python f.py
    real  0m4.795s
    user  0m4.787s
    sys   0m0.007s

    real  0m12.935s
    user  0m12.849s
    sys   0m0.049s

Ecco Ruby impiega **4.795** secondi mentre Python **12.935** secondi!

**Ruby vince nettamente su Python!!**

Oh beh provate però questo codice in C (`f.c`):

    int
    fib (int n)
    {
      if (n == 0 || n == 1)
        return n;
      else
        return fib (n-1) + fib (n-2);
    }

    int
    main ()
    {
      int i;
      for (i = 0; i < 36; i++)
        fib (i);
    }

Compiliamo:

     gcc f.c

Ed eseguiamo:

    $ time ./a.out
    real  0m0.232s
    user  0m0.229s
    sys   0m0.003s
    
0.232 secondi!