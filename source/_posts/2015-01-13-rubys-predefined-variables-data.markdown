---
layout: post
title: "Ruby's predefined variables: DATA"
date: 2015-01-13 00:09:44 +0100
comments: true
categories: ruby
---

I recently found out that one of the Ruby's predefined variables there is one
named `DATA`. It is very interesting. In a Ruby source file anything following
`__END__` becomes a kind of virtual file that can be read with the variable
`DATA`.  
Check this example:
```
puts DATA.read.split("\n")
__END__
tomatoes
potatoes
milk
beer
salt
beer again
```
