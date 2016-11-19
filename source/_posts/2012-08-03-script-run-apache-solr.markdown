---
layout: post
title: "init.d script to run Apache SOLR"
date: 2011-02-25 21:40
comments: true
categories: [apache, daemon, solr, linux]
---

Copiate il seguente script in `/etc/init.d/solr`

    #!/bin/sh

    PATH=/opt/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
    NAME=solr
    DESC=apache-solr
    VERSION=1.4.1
    SOLR_PATH=/opt/solr-$VERSION
    COMMAND=/usr/bin/java
    OPTIONS="-Dsolr.solr.home=$SOLR_PATH/solr -Djetty.home=$SOLR_PATH -jar $SOLR_PATH/start.jar"    
    PIDFILE=/var/run/$NAME.pid
        
    test -x $COMMAND || exit 0
    test -f $SOLR_PATH/start.jar || exit 0
    
    set -e
    
    case "$1" in
      start)
            if [ -f $PIDFILE ]; then
              echo "$PIDFILE exists. $NAME may be running."
            else
              echo -n "Starting $DESC: "
              start-stop-daemon -b -p$PIDFILE -d$SOLR_PATH --start --quiet --exec $COMMAND -- $OPTIONS
              sleep 3
              echo `ps -ef | grep -v grep | grep "$COMMAND $OPTIONS" | awk '{print $2}'` > $PIDFILE
              echo "$NAME."
            fi
            ;;
      stop)
            if [ -f $PIDFILE ]; then
              echo -n "Stopping $DESC: "
              kill `cat $PIDFILE`
              rm -f $PIDFILE
              echo "$NAME."
            else
              echo "$PIDFILE does not exist. $NAME may be not running."
            fi
            ;;
      restart)
            /etc/init.d/$NAME stop
            sleep 1
            /etc/init.d/$NAME start
            ;;
      *)
            echo "Usage: /etc/init.d/$NAME {start|stop|restart}" >&2
            exit 1
            ;;
    esac
    
    exit 0

