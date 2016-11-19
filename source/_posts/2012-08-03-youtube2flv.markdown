---
layout: post
title: "Youtube flv 2 mp3"
date: 2010-06-08 21:24
comments: true
categories: [youtube, flv, convert]
---

Vediamo come convertire un video flv di Youtube in mp3 usando ffmpeg.

Per prima cosa dobbiamo ottenere l'URL esatto del video per poter scaricare l'flv.
Per far ciò ci serve un token che cambia nel tempo ma che possiamo ottenere dal file `video_info`; scarichiamolo:

    http://www.youtube.com/get_video_info?video_id=VugK063j0Zo

Questo file contiene una serie di valori in stile parametri http. A noi serve quello con chiave **token**: nell'esempio `token=vjVQa1PpcFNk6LFs5OQXj-teHu8POPhx7isbKGtGbOc%3D`.

A questo punto abbiamo tutto il necessario per generare l'url per il download diretto del video flv:

    http://www.youtube.com/get_video?video_id=VIDEOID&t=TOKEN

nell'esempio:

    http://www.youtube.com/get_video?video_id=VugK063j0Zo&t=vjVQa1PpcFNk6LFs5OQXj-teHu8POPhx7isbKGtGbOc%3D

Bene, ora possiamo con ffmpeg ottenere direttamente l'audio del video in formato mp3 con il seguente comando:

    ffmpeg -i 'URL DIRETTO DEL VIDEO' -acodec libmp3lame -ac 2 -ab 128kb -vn -y file_output.mp3

Nell'esempio

    ffmpeg -i 'http://www.youtube.com/get_video?video_id=VugK063j0Zo&t=vjVQa1PpcFMpsZHHpcVB1lB6w_EvoJYGbZTNi4KQ2g4%3D' -acodec libmp3lame -ac 2 -ab 128kb -vn -y magically_magical.mp3

It's magical...
