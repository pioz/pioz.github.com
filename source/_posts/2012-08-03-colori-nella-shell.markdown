---
layout: post
title: "Colori nella shell"
date: 2012-07-27 21:43
comments: true
categories: [shell, linux]
---

Per colorare una string nella shell possiamo usare questo template:

    \033[CODE1;CODE2;CODE3mTESTO\033[0m

CODE1 possono essere una serie di valori che alterano lo stile del testo, di seguito una tabella:

* `0` Normal Characters
* `1` Bold Characters
* `4` Underlined Characters
* `5` Blinking Characters
* `7` Reverse video Characters

CODE2 invece rappresenta il colore, di seguito una tabella:

* `30` Black
* `31` Red
* `32` Green
* `33` Yellow
* `34` Blue
* `35` Magenta
* `36` Cyan
* `37` White

CODE3 rappresenta il colore di sfondo del testo, di seguito una tabella:

* `40` Black
* `41` Red
* `42` Green
* `43` Yellow
* `44` Blue
* `45` Magenta
* `46` Cyan
* `47` White

Per resettare il testo allo stile di default usare il carattere `\033[0m`.

Alcuni esempi:

    echo -e "\033[31mQUESTO TESTO SARA STAMPATO IN ROSSO\033[0m"
    echo -e "\033[1;36mQUESTO TESTO SARA STAMPATO IN GROSSETTO E COLOR CIANO\033[0m"
    echo -e "\033[1;35;47mQUESTO TESTO SARA STAMPATO IN GROSSETTO E COLOR MAGENTA SU SFONDO BIANCO\033[0m"