---
layout: post
title: "Configurare più monitor con scheda video NVidia e Xorg"
date: 2014-01-25 13:04
comments: true
categories: [ITA, linux, xorg, nvidia]
---

L'altro giorno mi sono trovato a configurare un dual monitor più connessione alla TV sul mio
pc fisso che monta una Debian con una scheda video NVidia GeForce GTX 650.

La configurazione che ho fatto è questa:  

- monitor Samsung 24" come principale
- monitor QBel 19" a destra del monitor principale
- TV che clona il monitor principale

Dato che il il monitor principale ha una risoluzione massima di 1920x1080,
mentre la TV soltanto 720x480 (si è una di quelle ancora col tubo catodico) non si riesce a fare un clone pieno, ovvero proiettare i 1920x1080 pixel del monitor principale sui 720x480 pixel della TV. Quindi mi sono accontentato di proiettare i 720x480 pixel della parte centrale del monitor principale sulla TV. In questo modo se apro un film con VLC basta che ridimensioni la finestra a 720x480 al centro del monitor per vedere il film in full screen sulla TV (ho uno scriptino che me lo fa, ma questo magari lo vedremo un altro giorno).

Quindi vediamo come fare sta roba.
Apriamo il nostro file `/etc/X11/xorg.conf`, il file di configurazione principale per Xorg (open source implementation of the X Window System). Il mio si presenta più o meno così:

    Section "ServerLayout"
        Identifier     "Layout0"
        Screen      0  "Screen0" 0 0
        InputDevice    "Keyboard0" "CoreKeyboard"
        InputDevice    "Mouse0" "CorePointer"
        Option         "Xinerama" "0"
    EndSection

    Section "Files"
    EndSection

    Section "InputDevice"
        # generated from default
        Identifier     "Mouse0"
        Driver         "mouse"
        Option         "Protocol" "auto"
        Option         "Device" "/dev/psaux"
        Option         "Emulate3Buttons" "no"
        Option         "ZAxisMapping" "4 5"
    EndSection

    Section "InputDevice"
        # generated from default
        Identifier     "Keyboard0"
        Driver         "kbd"
    EndSection

    Section "Device"
        Identifier     "Device0"
        Driver         "nvidia"
        VendorName     "NVIDIA Corporation"
        BoardName      "GeForce GTX 650"
    EndSection

    Section "Screen"
        Identifier     "Screen0"
        Device         "Device0"
        DefaultDepth    24
        Option         "Stereo" "0"
        SubSection     "Display"
            Depth       24
        EndSubSection
    EndSection

Quello che dobbiamo fare è andare a modificare la sezione `Screen` che permette di configurare la disposizione dei monitor. Ma prima di far ciò ci serve conoscrere gli ID dei nostri monitor connessi alla nostra scheda video. Usiamo il comando `xrand --query`. Dovremmo ottenere un output di questo tipo:

    Screen 0: minimum 8 x 8, current 3200 x 1080, maximum 16384 x 16384
    VGA-0 connected 1280x1024+1920+0 (normal left inverted right x axis y axis) 376mm x 301mm
       1280x1024      60.0*+   75.0  
       1280x960       60.0  
       1152x864       75.0  
       1024x768       75.0     70.1     60.0  
       800x600        75.0     72.2     60.3     56.2  
       640x480        75.0     72.8     59.9  
    HDMI-0 connected 720x480+600+300 (normal left inverted right x axis y axis) 386mm x 290mm
       1280x720       60.0 +   59.9  
       1920x1080      60.1     60.0  
       1440x480       60.1  
       720x480        59.9*    60.1  
       640x480        60.0     59.9  
    DVI-D-0 connected primary 1920x1080+0+0 (normal left inverted right x axis y axis) 531mm x 299mm
       1920x1080      60.0*+   59.9     50.0  
       1680x1050      60.0  
       1600x900       60.0  
       1440x900       59.9  
       1280x1024      75.0     60.0  
       1280x800       59.8  
       1280x720       60.0     59.9     50.0  
       1152x864       75.0  
       1024x768       75.0     70.1     60.0  
       800x600        75.0     72.2     60.3     56.2  
       720x576        50.0  
       720x480        59.9  
       640x480        75.0     72.8     59.9  
    DVI-D-1 disconnected (normal left inverted right x axis y axis)

In questo caso `DVI-D-0` è il monitor principale (24" Samsung), `VGA-0` è il monitor secondario (19" QBel) e `HDMI-0` è la TV.

Con questi dati possiamo iniziare a modificare il file `xorg.conf`. Nella sezione `Screen` aggiungiamo:

    Option         "TwinView" "1"
    Option         "TVStandard" "PAL-B"
    Option         "nvidiaXineramaInfoOrder" "DVI-D-0"
    Option         "metamodes" "DVI-D-0: nvidia-auto-select +0+0, VGA-0: 1280x1024 +1920+0, HDMI-0: 720x480 +600+300"

- Ovvero abilitiamo il TwinView della nostra scheda NVidia.
- Diciamo che lo standard della TV è il PAL-B.
- Diciamo che il monitor principale è il `DVI-D-0`, ovvero il Samsung da 24".
- Disponiamo i vari monitor:
  - Il monitor `DVI-D-0` avrà risoluzione auto calcolata (1920x1080) e l'angolo top-left avrà coordinate 0,0
  - Il monitor `VGA-0` avrà risoluzione 1280x1024 e l'angolo top-left avrà coordinate 1920,0 ovvero a destra del monitor principale infatti la larghezza del monitor principale (risoluzione orizzontale) è di 1920 pixel, quando termina inizia il monitor secondario.
  - La TV invece `HDMI-0` avrà risoluzione 720x480 e l'angono top-left avrà coordinate 600,300 ovvero più o meno al centro del monitor principale.

A questo punto salvate il file `xorg.conf` e riavviate il server X!
