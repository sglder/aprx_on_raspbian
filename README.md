
# Vorher

Nachdem das Raspian-Image aufgespielt ist und der Pi gestatet hat.
Erstmal ein neues Passwort für den pi-Benutzer

```
pi@strudel:~ $ passwd
```

Dann Updates...

```
pi@strudel:~ $ sudo apt autoremove wolfram-engine
pi@strudel:~ $ sudo apt update
pi@strudel:~ $ sudo apt full-upgrade
```

Falls Du kein Freund von vi/vim/nano bist würde ich dir empfehlen die
Konfiguration später mit dem Editor von Midnight Commander zu machen.
`wget` nehme ich nur weiter unten um APRX runterzuladen.

```
pi@strudel:~ $ sudo apt install mc wget
pi@strudel:~ $ sudo update-alternatives --config editor # mcedit auswählen wenn gewünscht
```

Danach kannst Du `sudoedit` zum Editieren von Systemdateien verwenden.


# Download und Installation und Konfiguration von APRX

```
pi@strudel:~ $ wget 'http://thelifeofkenneth.com/aprx/debs/aprx_2.9.0_raspi.deb'
pi@strudel:~ $ sudo dpkg -i aprx_2.9.0_raspi.deb
pi@strudel:~ $ sudoedit /etc/aprx.conf
```

Einstellungen vornehmen `sudoedit /etc/aprx.conf`:

```
# Define the parameters in following order:
#   1)  <aprsis>     ** zero or one
#   2)  <logging>    ** zero or one
#   3)  <interface>  ** there can be multiple!
#   4)  <beacon>     ** zero to many
#   5)  <telemetry>  ** zero to many
#   6)  <digipeater> ** zero to many (at most one for each Tx)

mycall  XX5XX

myloc lat 5000.00N lon 00800.00E

<aprsis>
#    login     OTHERCALL-7  # login defaults to $mycall
# Passcode for your callsign:
# http://apps.magicbug.co.uk/passcode/index.php
    passcode NNNNN
    server   germany.aprs2.net
</aprsis>

<logging>
    pidfile /var/run/aprx.pid
    rflog /var/log/aprx/aprx-rf.log
    aprxlog /var/log/aprx/aprx.log
</logging>

<interface>
    # null-device $mycall
    serial-device /dev/ttyUSB0  19200 8n1    KISS
    # callsign     $mycall  # callsign defaults to $mycall
    # initstring "\x0dKISS ON\x0dRESET\x0d" # Ist ggf. notwendig, damit der TNC nach Kaltstart vom Host- in den Kiss-Mode umschaltet.
    tx-ok        true    # transmitter enable defaults to false
    # telem-to-is  true # set to 'false' to disable
</interface>

<digipeater>
    transmitter $mycall
    <source>
        source $mycall
    </source>
</digipeater>

<beacon>
    cycle-size 10m
    # Für das Symbol (Quelle http://www.aprs-dl.de/?APRS_Detailwissen:SSID%2BSymbole:)
    # "L#""   für WIDEn-N Digis WIDE1-1 ohne Bundesland oder Distrikts-Routing (L Digi)
    # "X#""   für eXperimentelle Digis (X Digi)
    # ...
    beacon via TRACE1-1 \
        symbol "X#" lat "5054.33N" lon "00801.77E" \
        comment "Testbetrieb mit Raspberry Pi (Aprx v2.9.0)"
</beacon>
```

Danach zum Test APRX manuell und im Vordergrund ausführen (das Terminal blockiert
und APRX lässt sich auch nicht mit Strg+C beenden. Zum Beenden einfach
`sudo killall aprx` von einer anderen Konsole aus ausführen):

```
pi@strudel:~ $ sudo aprx -d
```

Wenn alles Funktioniert muss der Start von APRX bei Systemstart aktiviert werden.
Dazu `sudoedit /etc/default/aprx` und STARTAPRX auf yes setzen.

```
#
# STARTAPRX: start aprx on boot. Should be set to "yes" once you have
#            configured aprx.
#
STARTAPRX="yes"

#
# Additional options that are passed to the Daemon.
#
DAEMON_OPTS=""
```

Danach lässt sich APRX (auch) über den `service` Befehl im Hintergund starten,
beenden und neustarten.

```
pi@strudel:~ $ sudo service aprx start
pi@strudel:~ $ sudo service aprx stop
pi@strudel:~ $ sudo service aprx restart
```

# Konfiguration der Wetterstation

Für die Lacrosse Wetterstation benötigt man open2300

<https://s55ma.radioamater.si/2016/05/03/la-crosse-ws2300-ws2307-series-aprs-with-xastir/>

<http://www.lavrsen.dk/foswiki/bin/view/Open2300>

Mehr hab ich noch nicht geschafft.

# Siehe auch

<http://ham.zmailer.org/oh2mqk/aprx/aprx-manual.pdf>

<http://www.radio.cc/post/aprs-igate-with-raspberr-pi-setup>

<http://www.aprs-dl.de/?APRS_Detailwissen:SSID%2BSymbole>
