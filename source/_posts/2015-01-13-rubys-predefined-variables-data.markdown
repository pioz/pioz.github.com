---
layout: post
title: "Ruby's predefined variables: DATA"
date: 2015-01-13 00:09:44 +0100
comments: true
categories: ruby
---

Ho scoperto da poco che tra le variabili predefiniti di Ruby ne esiste una che
si chiama `DATA`. È molto interessante.  
In un file sorgente Ruby tutto quello che segue `__END__` diventa una specie
di file virtuale e può essere letto appunto tramite la variabile predefinita
`DATA`.
Ecco un esempio:

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
