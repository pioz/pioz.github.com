---
layout: post
title: "Shell colors"
date: 2012-07-27 21:43
comments: true
categories: [shell, linux]
---

To color a string in our shell we can use this template:

    \033[CODE1;CODE2;CODE3mSTRING\033[0m

CODE1 can be a series of values that change the style of the text, below a
table:

* `0` Normal Characters
* `1` Bold Characters
* `4` Underlined Characters
* `5` Blinking Characters
* `7` Reverse video Characters

CODE2 rappresents the foreground color, below a table:

* `30` Black
* `31` Red
* `32` Green
* `33` Yellow
* `34` Blue
* `35` Magenta
* `36` Cyan
* `37` White

CODE3 rappresents the background color, below a table:

* `40` Black
* `41` Red
* `42` Green
* `43` Yellow
* `44` Blue
* `45` Magenta
* `46` Cyan
* `47` White

To reset the text to the default style and color we can use the character
`\033[0m`.

Some examples:

    echo -e "\033[31mTHIS STRING WILL BE RED\033[0m"
    echo -e "\033[1;36mTHIS STRING WILL BE CYAN IN BOLD\033[0m"
    echo -e "\033[1;35;47mTHIS STRING WILL BE MAGENTA IN BOLD WITH WHITE BACKGROUND\033[0m"
