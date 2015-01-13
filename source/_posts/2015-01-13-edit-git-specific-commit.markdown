---
layout: post
title: "Edit git specific commit"
date: 2015-01-13 14:44:28 +0100
comments: true
categories: git
---

Oggi mi è capitato questa situazione mentre stavo fixando una issue di
[Chess](https://github.com/pioz/chess), una mia gemma di Ruby su github: in
pratica ho fixato il problema e ho fatto un commit `git commit -am "fix issue
bla bla"`. Mi sono poi accorto che non ho aumentato il numero di versione della
gemma e che quindi non potevo pusharla su Rubygems. A questo punto potevo
modificare il numero di versione e fare un secondo commit, ma questo non mi
andava, mi sarebbe piaciuto modificare l'ultimo commit e includere la modifica
del file `version.rb`.  
Come fare? Facciamo la nostra modifica al file `version.rb` aumentando la
versione della gemma e committiamo il tutto con il comando:

    git commit -a --amend --no-edit

Bene! Abbiamo modificato l'ultimo commit. Ultima cosa da aggiungere: se avevate
già fatto il push dell'ultimo commit lo SHA-1 di questo sarà cambiato dopo
questa operazione e quindi bisognerà pushare le modifiche usando l'opzione
`--force`

    git push -f  
