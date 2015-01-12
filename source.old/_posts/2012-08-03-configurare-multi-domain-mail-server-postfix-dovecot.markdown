---
layout: post
title: "Configurare un multi domain mail server con Postfix, Dovecot e autenticazione sicura"
date: 2009-12-11 10:22
comments: true
categories: [mail, mailserver, debian, linux, mysql, dovecot, postfix]
---

Allora in questa guida vedremo come installare e configurare un mail server professionale.
Il tutto è stato testato in una [Debian](http://www.debian.org) 5.0 stable.

Con questo server mail sarà possibile gestire più domini e quindi per esempio allo stesso tempo mantenere mail del tipo _@pioz.it_ e _@barison.org_.
Il tutto avverrà dando la possibilità all'utente se utilizzare comunicazioni criptate (come SSL o TLS) o in chiaro, se utilizzare autenticazione in chiaro (PLAIN) o criptata (CRAM-MD5 o DIGEST-MD5). Dando la possibilità di usare protocollo IMAP o POP3.

Il necessario:

*   **Postfix** come server SMTP
*   **Dovecot** come server POP3 e IMAP
*   **MySQL** come database per il salvataggio degli utenti, password e domini
*   **SASL** per criptare il trasferimento dei dati tra server e client

Installiamo il necessario:

    $ apt-get install postfix postfix-mysql
    $ apt-get install dovecot-imapd dovecot-pop3d
    $ apt-get install mysql-server-5.0
    $ apt-get install libsasl2 sasl2-bin libsasl2-modules

### Passo 0: domini
Per capirci meglio usiamo le seguenti convenzioni: il dominio associato al mail server sarà `ciccio.com`, mentre i domini che hosteremo saranno `pippo.it` e `pluto.com`.

Questo significa che tutti e tre i domini dovranno avere un record MX nel DNS chiamato mail.nomedominio che punterà all'IP del nostro server mail.

Inoltre dato che useremo anche SSL per il trasferimento dei dati tra server e client è necessario avere un certificato (valido) per il dominio `mail.ciccio.com`.

## PASSO 1: configurazione database
Per prima cosa creiamo un database per contenere gli utenti che potranno usufruire del nostro servizio mail.
Creiamo un file e lo chiamiamo `mail.sql` e ci incolliamo dentro il seguente codice SQL:

    create database mail;

    use mail;

    CREATE TABLE user (
      domain varchar(255) NOT NULL,
      username varchar(255) NOT NULL,
      password varchar(255) NOT NULL,
      quota int(255) NOT NULL default 0,
      active tinyint(1) NOT NULL default 1,
      admin tinyint(1) NOT NULL default 0,
      PRIMARY KEY (username, domain)
    );

    CREATE TABLE alias (
      address varchar(255) NOT NULL,
      goto varchar(255) NOT NULL,
      active tinyint(1) NOT NULL default 1,
      PRIMARY KEY (address, goto)
    );

A questo punto eseguiamo il codice SQL e creiamo un utente con privilegi ristretti che verrà usato da postfix e dovecot per connettersi al DB:

    $ mysql -u root -p
    mysql> source mail.sql;
    mysql> GRANT select,insert,update,delete ON mail.* TO 'postfix'@'localhost' IDENTIFIED BY 'password';
    mysql> quit;

Ottimo, ora abbiamo il nostro BD... 2 tabelle: una per gli utenti e una per gli alias. Gli alias sono delle mail che sono alias di mail "reali".

Vediamo, prima di proseguire, i campi della tabella `user`:

* **domain**: il dominio dell'indirizzo mail (per esempio luca@pippo.it ha dominio pippo.it).
* **username**: lo username per accedere alla casella di posta, nel nostro caso, **IMPORTANTE**, è l'intero indirizzo mail (quindi non luca, ma luca@pippo.it). Questo perchè avendo mail con domini diversi dobbiano distinguerle in modo univoco.
* **password**: la password per accedere alla casella di posta. Vedremo che sarà criptata con AES.
* **quota**: quota casella di posta.
* **active**: l'account mail è attivo o disattivo.
* **admin**: campo utile per verificare, magari da qualche app esterna, se l'utente può creare altri account.

I campi della tabella `alias`:

* **address**: indirizzo mail dell'alias.
* **goto**: è alias di.
* **active**: l'alias è attivo o disattivo.


## PASSO 2: creazione delle Maildir
Le nostre caselle di posta saranno sottoforma di maildir e le posizioniamo in `/etc/vmail/`. Creiamole impostando i relativi permessi:

    $ adduser postfix mail
    $ adduser dovecot mail
    $ mkdir /var/vmail
    $ chmod 770 /var/vmail
    $ chown mail.mail /var/vmail

Bene, in pratica la casella di posta per luca@pippo.it sarà in `/var/vmail/luca@pippo.it`. Essa verrà creata in automatico al primo login.


## PASSO 3: configurazione Dovecot
Adesso è arrivato il momento di configurare [Dovecot](http://www.dovecot.org/): il server pop e imap.

Dovecot, per i nostri usi consiste in due file di configurazione:

* `/etc/dovecot/dovecot.conf`: file di configurazione principale.
* `/etc/dovecot/dovecot-sql.conf`: file di configurazione per la connessione e recupero dati dal database MySQL.

Vediamo il primo:

    ## Dovecot configuration file
    
    protocols = pop3 pop3s imap imaps
    disable_plaintext_auth = no
    log_path = /var/log/mail.log
    log_timestamp = "%Y-%m-%d %H:%M:%S "
    
    ssl_disable = no
    ssl_cert_file = /etc/ssl/mycerts/mail.ciccio.com/cert.crt
    ssl_key_file = /etc/ssl/mycerts/mail.ciccio.com/key.key
    ssl_ca_file = /etc/ssl/mycerts/mail.ciccio.com/root.crt
    
    mail_location = maildir:/var/vmail/%u/
    mail_privileged_group = mail
    #first_valid_uid = 150
    first_valid_uid = 8
    
    protocol imap {
      mail_plugins = autocreate
      plugin {
        autocreate  = Trash
        autocreate2 = Junk
        autocreate3 = Drafts
        autocreate4 = Sent
        autocreate5 = INBOX
    
        autosubscribe  = Trash
        autosubscribe2 = Junk
        autosubscribe3 = Drafts
        autosubscribe4 = Sent
        autosubscribe5 = INBOX
      }
    }
    
    protocol pop3 {
      pop3_uidl_format = %08Xu%08Xv
    }
    
    protocol lda {
      # log_path = /etc/dovecot/sieve.log
      mail_plugins = cmusieve
      postmaster_address = enrico@megiston.it
    }
    
    plugin {
      sieve = /etc/dovecot/sieve/default.sieve
    }

    auth_verbose = no
    auth_debug = no
    auth_debug_passwords = no
    auth default {
      mechanisms = login plain digest-md5 cram-md5
      passdb sql {
        args = /etc/dovecot/dovecot-sql.conf
      }
      userdb sql {
        args = /etc/dovecot/dovecot-sql.conf
      }
      user = root
      socket listen {
        master {
          path = /var/run/dovecot/auth-master
          mode = 0660
          user = dovecot
          group = mail
        }
        client {
          path = /var/spool/postfix/private/auth
          mode = 0660
          user = postfix
          group = mail
        }
      }
    }

Adesso non entriamo nel dettaglio in quanto non ho voglia di star qua a scrivere, comunque due cosette sulle cose principali:

* La prima riga specifica i protocolli che vogliamo usare. Gli abilitiamo tutti per dare libertà totale ai nostri utenti. Mi raccomando di avere il certificato per il vostro dominio, nell'esempio per `mail.ciccio.com`.  
* `first_valid_uid` deve coincidere con uid dell'utente `mail`, solitamente 8. Per verificare lanciate il comando "`id mail`".  
* Importante è la riga "`mechanisms = login plain digest-md5 cram-md5
`" che specifica i meccanismi di autenticazione. Noi li abilitiamo tutti, sia quelli in chiaro che criptati in modo da permettere anche a client poco sofisticati (ad esempio quelli dei cellulari) di connettersi al nostro server. Mentre per client completi dare la possibilità di usare autenticazione cifrata (ad esempio Mozilla Thunderbird).  
* `userdb` e `passdb` vengono recuperati dall'altro file di configurazione.  
* Infine fondamentale la sezione `socket listen` che apre una socket per permettere ad altri programmi di usare il sistema di autenticazione di Dovecot, nel nostro caso Postfix.

L'altro file di configurazione per il recupero degli utenti e delle password risulta essere una cosa del genere:

    driver = mysql
    connect = host=127.0.0.1 dbname=mail user=postfix password=password
    default_pass_scheme = PLAIN
    user_query = SELECT concat('maildir:/var/vmail/', username, '/') as mail, 8 AS uid, 8 AS gid FROM user WHERE username = '%u' AND domain = '%d' AND active = '1'
    password_query = SELECT username AS user, AES_DECRYPT(UNHEX(password),'AESKEY') AS password FROM user WHERE username = '%u' AND domain = '%d' AND active = 1

Commentiamo:

* `driver` specifica che il database è MySQL.
* `connect` info per la connessione al database.
* `default_pass_scheme` usiamo PLAIN, infatti se diamo la possibilità di usare meccanismi di autenticazione in chiaro e non in chiaro (es plain e cram-md5) non c'è alternativa, non possiamo salvare la password in modo criptato per meccanismo PLAIN. Comunque niente paura, la password non sarà salvata in chiaro ma criptata con la funzione AES di mysql e trasformata da binario a text con la funzione HEX.
* `user_query` è la query per recuperare l'utente. Da notare che uid e gid devono essere lo stesso dell'utente `mail` (`id mail`). Questa in realtà deve recuperare il path della maildir dell'utente, per questo c'è il `concat`.
* `user_password` è la query per recuperare le password decriptandole usando una chiave (nell'esempio AESKEY).

Bene ora Dovecot è pronto. Riavviamo Dovecot con `/etc/init.d/dovecot restart`


## PASSO 4: configurazione Postfix
Postfix è il server SMTP. Viene gestito da due file di configurazione principali, più altri per la connessione con il database:

1. `/etc/postfix/master.cf`
2. `/etc/postfix/main.cf`
3. `/etc/postfix/mysql_virtual_alias_maps.cf`
4. `/etc/postfix/mysql_virtual_domains_maps.cf`
5. `/etc/postfix/mysql_virtual_mailbox_maps.cf`

Il master (1):

    #
    # Postfix master process configuration file.  For details on the format
    # of the file, see the master(5) manual page (command: "man 5 master").
    #
    # Do not forget to execute "postfix reload" after editing this file.
    #
    # ==========================================================================
    # service type  private unpriv  chroot  wakeup  maxproc command + args
    #               (yes)   (yes)   (yes)   (never) (100)
    # ==========================================================================
    smtp      inet  n       -       n       -       -       smtpd
    #submission inet n       -       -       -       -       smtpd
    #  -o smtpd_tls_security_level=encrypt
    #  -o smtpd_sasl_auth_enable=yes
    #  -o smtpd_client_restrictions=permit_sasl_authenticated,reject
    #  -o milter_macro_daemon_name=ORIGINATING
    # Decommentiamo il seguito per abilitare il wrapper SSL su TLS
    smtps     inet  n       -       -       -       -       smtpd
      -o smtpd_tls_wrappermode=yes
      -o smtpd_sasl_auth_enable=yes
      -o smtpd_client_restrictions=permit_sasl_authenticated,reject
    #  -o milter_macro_daemon_name=ORIGINATING
    #628      inet  n       -       -       -       -       qmqpd
    pickup    fifo  n       -       -       60      1       pickup
    cleanup   unix  n       -       -       -       0       cleanup
    qmgr      fifo  n       -       n       300     1       qmgr
    #qmgr     fifo  n       -       -       300     1       oqmgr
    tlsmgr    unix  -       -       -       1000?   1       tlsmgr
    rewrite   unix  -       -       -       -       -       trivial-rewrite
    bounce    unix  -       -       -       -       0       bounce
    defer     unix  -       -       -       -       0       bounce
    trace     unix  -       -       -       -       0       bounce
    verify    unix  -       -       -       -       1       verify
    flush     unix  n       -       -       1000?   0       flush
    proxymap  unix  -       -       n       -       -       proxymap
    proxywrite unix -       -       n       -       1       proxymap
    smtp      unix  -       -       -       -       -       smtp
    # When relaying mail as backup MX, disable fallback_relay to avoid MX loops
    relay     unix  -       -       -       -       -       smtp
            -o smtp_fallback_relay=
    #       -o smtp_helo_timeout=5 -o smtp_connect_timeout=5
    showq     unix  n       -       -       -       -       showq
    error     unix  -       -       -       -       -       error
    retry     unix  -       -       -       -       -       error
    discard   unix  -       -       -       -       -       discard
    local     unix  -       n       n       -       -       local
    virtual   unix  -       n       n       -       -       virtual
    lmtp      unix  -       -       -       -       -       lmtp
    anvil     unix  -       -       -       -       1       anvil
    scache    unix  -       -       -       -       1       scache
    #
    # ====================================================================
    # Interfaces to non-Postfix software. Be sure to examine the manual
    # pages of the non-Postfix software to find out what options it wants.
    #
    # Many of the following services use the Postfix pipe(8) delivery
    # agent.  See the pipe(8) man page for information about ${recipient}
    # and other message envelope options.
    # ====================================================================
    #
    # maildrop. See the Postfix MAILDROP_README file for details.
    # Also specify in main.cf: maildrop_destination_recipient_limit=1
    #
    maildrop  unix  -       n       n       -       -       pipe
      flags=DRhu user=vmail argv=/usr/bin/maildrop -d ${recipient}
    #
    # See the Postfix UUCP_README file for configuration details.
    #
    uucp      unix  -       n       n       -       -       pipe
      flags=Fqhu user=uucp argv=uux -r -n -z -a$sender - $nexthop!rmail ($recipient)
    #
    # Other external delivery methods.
    #
    ifmail    unix  -       n       n       -       -       pipe
      flags=F user=ftn argv=/usr/lib/ifmail/ifmail -r $nexthop ($recipient)
    bsmtp     unix  -       n       n       -       -       pipe
      flags=Fq. user=bsmtp argv=/usr/lib/bsmtp/bsmtp -t$nexthop -f$sender $recipient
    scalemail-backend unix  -       n       n       -       2       pipe
      flags=R user=scalemail argv=/usr/lib/scalemail/bin/scalemail-store ${nexthop} ${user} ${extension}
    mailman   unix  -       n       n       -       -       pipe
      flags=FR user=list argv=/usr/lib/mailman/bin/postfix-to-mailman.py
      ${nexthop} ${user}
    dovecot   unix  -       n       n       -       -       pipe
      flags=DRhu user=mail argv=/usr/lib/dovecot/deliver -n -f ${sender} -d ${recipient}

Non commentiamo, è molto simile a quello di default a parte un paio di modifiche, tra cui l'abilitazione del wrapper SSL su TLS.

Il main.cf (2):

    # MAIN POSTFIX CONFIGUTARION #
    
    # smtpd_banner = $myhostname ESMTP $mail_name (Debian/GNU)
    # biff = no
    # append_dot_mydomain = yes
    debug_peer_level = 3
    myhostname = mail.ciccio.com
    
    virtual_uid_maps = static:8
    virtual_gid_maps = static:8
    virtual_minimum_uid = 8
    virtual_mailbox_base = /var/vmail/
    virtual_alias_maps = mysql:/etc/postfix/mysql_virtual_alias_maps.cf
    virtual_mailbox_domains = mysql:/etc/postfix/mysql_virtual_domains_maps.cf
    virtual_mailbox_maps = mysql:/etc/postfix/mysql_virtual_mailbox_maps.cf
    virtual_transport = virtual
    
    # SASL #
    
    smtpd_sasl_auth_enable = yes
    smtpd_sasl_local_domain = $myhostname
    smtpd_sasl_security_options = noanonymous
    smtpd_sasl_auth_header = yes
    smtpd_sasl_type = dovecot
    smtpd_sasl_path = private/auth
    broken_sasl_auth_clients = yes
    smtpd_recipient_restrictions =
      permit_sasl_authenticated,
      reject_unauth_destination
    
    # TLS #
    
    smtp_use_tls = yes
    smtpd_use_tls = yes
    smtpd_tls_key_file = /etc/ssl/mycerts/mail.ciccio.com/key.key
    smtpd_tls_cert_file = /etc/ssl/mycerts/mail.ciccio.com/cert.crt
    smtpd_tls_CAfile = /etc/ssl/mycerts/mail.ciccio.com/root.crt
    smtpd_tls_loglevel = 1
    smtpd_tls_received_header = yes
    smtpd_tls_session_cache_timeout = 3600s
    tls_random_source = dev:/dev/urandom

Anche qua non commentiamo... da notare solo che SASL per autenticare usa la socket che gli viene messa a disposizione da Dovecot.

E gli altri tre in ordine per la connessione al BD:

    # /etc/postfix/mysql_virtual_alias_maps.cf
    user = postfix
    password = password
    hosts = 127.0.0.1
    dbname = mail
    table = alias
    select_field = goto
    where_field = address

    # /etc/postfix/mysql_virtual_domains_maps.cf
    user = postfix
    password = password
    hosts = 127.0.0.1
    dbname = mail
    table = user
    select_field = domain
    where_field = domain

    # /etc/postfix/mysql_virtual_mailbox_maps.cf
    user = postfix
    password = password
    hosts = 127.0.0.1
    dbname = mail
    query = SELECT concat(username,'/') FROM user WHERE username = '%s' AND active = 1

Ok, per Postfix abbiamo finito. Riavviamolo con `/etc/init.d/postfix/restart`


## PASSO 5: proviamo il tutto
Inseriamo gli utenti nel nosto DB e poi testiamo il tutto usando Thunderbird.
Per aggiungere un utente usiamo la seguente query:

    $ mysql -u postfix -p
    $ insert into user(domain,username,password) values("pippo.it", "luca@pippo.it", HEX(AES_ENCRIPT('passwordchevoglio', 'AESKEY')));
    $ insert into user(domain,username,password) values("pluto.com", "nicola@pluto.com", HEX(AES_ENCRIPT('passworddd', 'AESKEY')));

Vediamo per esempio come configurare Thunderbird:
![](http://media.tumblr.com/tumblr_kuhdz2ptTT1qa79c0.png)


### Alcune note
Ricordate di installare Spamassasin per Postfix e magari una bella webmail come [Roundcube](http://roundcube.net/).

Se avete dubbi, chiarimenti, o volete approfondire alcune sezioni non fatevi scrupoli a contattarmi al mio indirizzo mail [pioz@pioz.it](mailto:pioz@pioz.it).