---
layout: post
title: "Mini guida su Xen e Debian"
date: 2013-03-27 21:12
comments: true
categories: [xen, debian, vm, macchine virtuali, os]
---

![image](http://www.debian.org/Pics/openlogo-50.png)
Su questo post vedremo una mini guida su come installare una macchina virtuale con Xen
in ambiente GNU/Linux Debian.

Xen è un monitor di macchine virtuali open source rilasciato sotto licenza GPL per piattaforma
x86 e compatibili (al momento sono in corso dei port per x86-64 e per IA-64) sviluppato
presso il Computer Laboratory dell'Università di Cambridge.

Per chiarezza ricordo che la macchina fisica di solito viene identificata con il
nome `dom0` (domain 0), mentre le macchine virtuali con il nome `domU`.

Detto ciò iniziamo.

## dom0

Per prima cosa configuriamo alcune opzioni di rete per permettere alle varie domU di
usare la rete di dom0. Nel file `/etc/sysctl.conf` la linea:

    net.ipv4.ip_forward=1

e applichiamo la configurazione con il comando:

    sysctl -p

Nel famoso file `/etc/network/interfaces` aggiungiamo alla fine:

    # Routing for VMs
    up route add -host IP_STATICO_MACCHINA_VIRTUALE gw IP_STATICO_MACCHINA_VIRTUALE

Fatto questo installiamo il necessario:

    $ apt-get install xen-hypervisor xen-linux-system xen-tools xen-utils xenstore-utils xenwatch
    
A questo punto dovremmo aver installato anche un nuovo kernel per la nostra
Debian in grado di gestire la virtualizzazione. Dovremmo fare una modifica alla conf di grub
per dare priorità al kernel xen rispetto a quello standard: in pratica bisogna rinominare il
file dentro la directory `/etc/grub.d` con nome `xx_linux` DOPO il file `yy_linux_xen`.
Nel mio caso i file in questione sono `10_linux` e `20_linux_xen`. Spostiamo il primo dopo
il secondo semplicemente rinominando il numero con cui inizia il file (10) in un numero maggiore:

    $ cd /etc/grub.d/
    $ mv 10_linux 50_linux
    
Fatto ciò rigeneriamo il file di configurazione di grub (il nostro bootloader preferito) con
il comando:

    $ update-grub2
    
e facciamo un reboot:

    $ reboot
    
Controlliamo che sia partito il kernel giusto con il comando:

    $ uname -a
    Linux 2.6.32-5-xen-amd64 #1 SMP Sun May 6 08:57:29 UTC 2012 x86_64 GNU/Linux
    
(Eh si... Debian stable)

Ora abbiamo un paio di file di configurazione su cui mettere le mani

* `/etc/xen/xend-config.sxp`
* `/etc/xen-tools/xen-tools.conf`

Il primo file `/etc/xen/xend-config.sxp` gestisce la configurazione del demone xend.
I valori di default vanno bene, anche se dobbiamo fare alcune modifiche per abilitare
un network bridge tra la la dom0 e le nostre domU decommentando le seguenti righe:

    ##
    # To bridge network traffic, like this:
    #
    # dom0: ----------------- bridge -> real eth0 -> the network
    #                            |
    # domU: fake eth0 -> vifN.0 -+
    #
    # use
    #
    (network-script network-bridge)
    #
    ...
    #
    # If you are using only one bridge, the vif-bridge script will discover that,
    # so there is no need to specify it explicitly.
    #
    (vif-script vif-bridge)
    
In questo modo la rete di dom0 tramite un ponte diventa disponibile anche per le varie
macchine virtuali domU.  

Le altre opzioni vanno bene così come sono, ma se volte approfondire c'è sempre la doc.

Il secondo file `/etc/xen-tools/xen-tools.conf` serve per gestire la configurazione con
cui vengono create le macchine virtuali. Vediamo alcune opzioni:

    ##
    #  Output directory for storing loopback images.
    #
    #  If you choose to use loopback images, which are simple to manage but
    # slower than LVM partitions, then specify a directory here and uncomment
    # the line.
    #
    #  New instances will be stored in subdirectories named after their
    # hostnames.
    #
    ##
    dir = /var/xen

Specifica la directory dove verranno salvate le immagini delle macchine virtuali.
In pratica i file che sono a tutti gli effetti gli HD delle vostre macchine virtuali.
Infatti se volete fare un backup di un'intera macchina virtuale salvatevi il suo file
immagine.

    #
    ##
    #
    # If you don't wish to use loopback images then you may specify an
    # LVM volume group here instead
    #
    ##
    # lvm = vg0

Se non volete usare le immagini, ma HD fisici allora dovrete usate un LVM (Logical Volume Manager).

    #
    ##
    #
    #  Installation method.
    #
    #  There are four distinct methods which you may to install a new copy
    # of Linux to use in your Xen guest domain:
    #
    #   - Installation via the debootstrap command.
    #   - Installation via the rpmstrap command.
    #   - Installation via the rinse command.
    #   - Installation by copying a directory containing a previous installation.
    #   - Installation by untarring a previously archived image.
    #
    #  NOTE That if you use the "untar", or "copy" options you should ensure
    # that the image you're left with matches the 'dist' setting later in
    # this file.
    #
    #
    ##
    #
    #
    # install-method = [ debootstrap | rinse | rpmstrap | copy | tar ]
    #
    #
    install-method = debootstrap
    
In questa sezione di decice in quale modo il sistema operativo delle vostre macchine
virtuali verrà installato. Ci sono diversi modi, ma il più comune e semplice è quello di
usare l'utility `debootstrap` che viene usato per creare un sistema Debian da zero,
senza la necessità di dpkg o apt. Per fare questo, vengono scaricati i file .deb da
un sito mirror e vengono spacchettati in una directory su cui è eventualmente
possibile fare il chroot.

    #
    ##
    #  Disk and Sizing options.
    ##
    #
    size   = 4Gb      # Disk image size.
    memory = 128Mb    # Memory size
    swap   = 128Mb    # Swap size
    # noswap = 1      # Don't use swap at all for the new system.
    fs     = ext4     # use the EXT3 filesystem for the disk image.
    dist   = `xt-guess-suite-and-mirror --suite` # Default distribution to install.
    image  = sparse   # Specify sparse vs. full disk images.

Molto intuitivo, un commento solo alle ultime due opzioni: `dist` specifica se
installare la versione stable, testing o unstable. Mentre `image` il tipo di immagine
da usare come file system: possiamo scegliere tra un'immagine _sparse_ oppure tra
un'immagine <em>full disk</em>. Nel primo caso il file aumenta la sua dimensione dinamicamente,
mentre nel secondo caso il file avrà la stessa grandezza del valore scelto qualche riga sopra (`size`).

    #
    # Uncomment the following line if you wish to interactively setup a
    # new root password for images.
    #
    passwd = 1
    #

Immettiamo noi quando creiamo le nostre macchine virtuali la password per l'utente root.

Per le altre opzioni vi rimando alla doc.

A questo punto creiamo la directory che conterrà i file immagini delle nostre macchine virtuali
che deve essere quella specificata nella variabile `dir` del file `/etc/xen-tools/xen-tools.conf`:

    $ mkdir -p /var/xen
    $ chmod 755 /var/xen
    
Riavviamo xend:

    $ /etc/init.d/xend restart
    
## domU

Ora il nostro ambiente Debian/Xen dovrebbe essere pronto per la virtualizzazione.
Andiamo a creare la nostra macchina virtuale. Per far ciò usiamo i tools che xen ci mette a
disposizione, in questo caso `xen-create-image`. Questo comando necessita di un solo parametro
fondamentale ovvero il nome della vostra macchina virtuale. Poi utilizzerà la configurazione
salvata nel file `/etc/xen-tools/xen-tools.conf` (vi ricordate?) per il resto. Possiamo comunque
ovverridare alcune opzioni con dei parametri da linea di comando:

    $ xen-create-image --hostname=vm1 --size=200Gb --memory=32Gb --ip=www.xxx.yyy.zzz --netmask=255.255.255.248 --gateway=ip.di.dom.0
    
Questo comando non farà altro che create il file di configurazione specifico per la macchina
virtuale "vm1" in `/etc/xen/vm1.cfg`. Comunque abbiamo creato una configurazione per una
macchina virtuale di nome _vm1_ con HD di capacità 200Gb, memoria ram da 32Gb, ip statico www.xxx.yyy.zzz,
netmask 255.255.255.248 e gateway ip.di.dom.0 (ovviamente dom0 deve avere un HD > di 200Gb e memoria RAM > di 32Gb).

Ok, adesso finalmente creiamo sta maledetta macchina:

    $ xm create /etc/xen/vm1.cfg

La macchina virtuale si avvia, per vedere lo stato del sistema usate il comodo comando:

    $ xm list
    Name                                        ID   Mem VCPUs      State   Time(s)
    Domain-0                                     0  2353     8     r-----  24266.4
    vm1                                          3 30720     8     -b---- 150851.7

I valori di `State` possono essere:

* r - *running*: la macchina sta girando e usando una CPU.
* b - *blocked*: la macchina è bloccata e non sta usando la CPU. Questo perchè sta aspettando per operazioni di IO oppure è in sleep perchè non ha niente da fare.
* p - *paused*: la macchina è in pausa (`xm pause`).
* s - *shutdown*: la macchina si sta spegnendo o riavviando o sospendendo.
* c - *crashed*: la macchina è crashata.
* d - *dying*.

Per accedere alla vostra macchina virtuale potete o connettervi via ssh oppure usare il comando:

    $ xm console vm1
  
che vi apre una console alla vostra macchina.

Per spegnere la macchina:

    $ xm shutdown vm1
    
Per mettere in pausa la macchina:

    $ xm pause vm1
    
Per riattivarla:

    $ xm restore vm1
    
Per eliminare la macchina:

    $ xm delete vm1
