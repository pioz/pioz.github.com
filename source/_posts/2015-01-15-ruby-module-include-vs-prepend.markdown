---
layout: post
title: "Ruby Module include vs prepend"
date: 2015-01-15 15:15:18 +0100
comments: true
categories: [ruby, modules]
---

Oggi cerchiamo di capire un po' meglio la differenza tra i metodi `Module#include` e `Module#prepend`.
Vediamo la differenza con un esempio mooolto semplice:

```
class Person
  def name
    # return name here
    puts 'Call method name in class Person'
  end
end

module Superhero
  def fly
    # fly here
    puts 'Call method fly in module Superhero'
  end

  def name
    # return name here
    puts 'Call method name in module Superhero'
  end
end

Person.include(Superhero)

Person.new.name
```

Cosa abbiamo fatto? Allora abbiamo la classe `Person` con un metodo `name`. Poi
abbiamo il modulo `Superhero` con due metodi: `fly` e `name`. Notare a questo
punto che sia la classe `Person` che il modulo `Superhero` hanno entrambi un
metodo che si chiama `name`. Ora andiamo a mixare la classe `Person` con il
modulo `Superhero` usando il metodo `include`. Se invochiamo il metodo `name` su
un oggetto di tipo `Person` vedremo che viene chiamato il metodo definito dentro
la classe `Person` e non quello definito nel modulo `Superhero`.   Questo ci
potrebbe anche andare bene, ma a volte vorremmo che quando facciamo questo tipo
di operazione i metodi del modulo che mixiamo sovrascrivessero (override) i
metodi della classe, come succede quando estendiamo una classe. Per ottenere
questo usiamo il metodo `prepend` al posto di `include` disponibile dalla
versione 2 di Ruby.

```
Person.prepend(Superhero)

Person.new.name
```

Adesso vediamo che viene invocato il metodo `name` definito nel modulo `Superhero`.
