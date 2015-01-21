---
layout: post
title: "Recursion: Ruby vs Python"
date: 2012-08-03 21:27
comments: true
categories: [ruby, python, c]
---

Let's do a litte test: we compute Fibonacci sequence with recursion in Ruby
(v2.0.0) and Python (v2.7.2) and let's see who wins in terms of time:

Ruby Fibonacci code (`f.rb`):
```
def fib(n)
  if n == 0 || n == 1
    n
  else
    fib(n-1) + fib(n-2)
  end
end

36.times { |i| fib(i) }
```

Python Fibonacci code (`f.py`)
```
def fib(n):
  if n == 0 or n == 1:
    return n
  else:
    return fib(n-1) + fib(n-2)

for i in range(36):
  fib(i)
```

Now let's run the programs as follows:

    $ time ruby f.rb && time python f.py
    real  0m4.795s
    user  0m4.787s
    sys   0m0.007s

    real  0m12.935s
    user  0m12.849s
    sys   0m0.049s


Look: Ruby took **4.795** seconds, while Python **12.935** seconds!

**Ruby wins clearly on Python!!**

But now check this C code (`f.c`):
```
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
```

Compile:

     gcc f.c

Run:

    $ time ./a.out
    real  0m0.232s
    user  0m0.229s
    sys   0m0.003s

0.232 seconds!
