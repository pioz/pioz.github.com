---
layout: post
title: "ALSA: Switch audio output to USB headset"
date: 2014-09-17 13:31
comments: true
categories: [debian, alsa, linux, os]
---

Ho acquistato delle cuffie wireless USB, per essere preciso esattamente queste:
http://www.amazon.it/gp/product/B005FY61MM . Sono molto comode e funzionano su
Debian out-of-the-box ovvero senza installare alcun driver.

Adesso nella mia Debian ho questo problema: vorrei che quando inserisco il
ricevitore USB in automatico l'output di ALSA cambi dalla scheda audio integrata
alle cuffie wireless.

Questo può essere fatto usando le regole di udev (udev è il gestore dei
dispositivi per il kernel Linux e amministra dinamicamente i dispositivi a
blocchi per ogni periferica rilevata nel sistema).
Iniziamo: apriamo il file `/etc/udev/rules.d/00-local.rules` e se non esiste lo
creiamo; aggiungiamo le seguenti regole:
```
# Set USB headset as default sound card when plugged in
KERNEL=="pcmC[D0-9cp]*", ACTION=="add", PROGRAM="/bin/sh -c 'K=%k; K=$${K#pcmC}; K=$${K%%D*}; echo defaults.ctl.card $$K > /etc/asound.conf; echo defaults.pcm.card $$K >>/etc/asound.conf'"

# Restore default sound card when USB headset unplugged
KERNEL=="pcmC[D0-9cp]*", ACTION=="remove", PROGRAM="/bin/sh -c 'echo defaults.ctl.card 0 > /etc/asound.conf; echo defaults.pcm.card 0 >>/etc/asound.conf'"
```
In pratica quando viene inserito il ricevitore USB udev lo detecta e lancia
uno script per cambiare il file `/etc/asound.conf` cambiando il device di output
con quello delle cuffie appena inserite.

L'unica pecca è che quando viene cambiato l'output di ALSA tutti i programmi che
stanno usando ALSA devono essere riavviti. Se trovate un modo per risolvere
questo problema fatemi sarepe!
