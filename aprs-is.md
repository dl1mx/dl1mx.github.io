---
layout: post
title: APRS-IS Bake mit dem Raspberry Pi
date: 2012-12-30 13:45
author: admin
comments: true
categories: [Amateurfunk, Amateurfunk, APRS, Raspberry Pi]
---
Möchte man eine Stationskennung per Internet an das APRS-IS Netzwerk absetzen, so bietet sich ein stromsparender Rechner wie z.B. der Raspberry Pi an. Diesen habe ich mit dem Debian Derivat Raspbian ausgestattet (http://www.raspbian.org).

Ich gehe davon aus das Raspbian vollständig eingerichtet ist und sich über das Netzwerk per SSH fernbedienen läßt. Unter Windows logge ich mich dann mit PuTTY oder unter Linux mit SSH auf dem Minirechner ein. Als erstes werden alle Pakete auf den neuesten Stand gebracht:
<pre>sudo apt-get update
sudo apt-get upgrade</pre>
Dann hole ich mir die benötigten Pakete und lege mir ein Arbeitsverzeichnis an:
<pre>apt-get install vim netcat aprsd
mkdir ~/aprs
cd ~/aprs</pre>
Um sich auf einem der APRS-IS server einzuloggen braucht man ein Passwort, dass vom Benutzernamen abhängt. Dieses Passwort kann komfortabel mittels des Programms aprspass aus dem Paket aprsd erzeugt werden was wir uns für später merken.
<pre>aprspass DB0ABC
APRS password for DB0ABC is = xxxxx</pre>
Nun erstellen wir eine Datei mit den APRS Daten, die an den APRS-IS Server gesendet werden sollen und unsere Position sowie den Status darstellen:
<pre>vi DB0ABC.txt</pre>
In diese Datei fügen wir (beispielsweise) folgendes hinzu:
<pre>user DB0ABC pass xxxxx
DB0ABC&gt;APRS,TCPIP*:!0102.03N/00405.06Er1750 R20k 145.600MHz DB0ABC
DB0ABC&gt;APRS,TCPIP*:&gt;http://www.darc.de</pre>
In der ersten Zeile steht unser Rufzeichen und das Passwort. Die zweite Zeile enhält das APRS-Frame für die Position mit ergänzendem Text. Die dritte Zeile ist ein APRS-Frame für den Statustext. Der Aufbau der jeweiligen Frames ist z.B. unter http://www.aprs-dl.de/index.php?APRS_Detailwissen:Lokale_Informationen sehr gut beschrieben und muss für jede Station angepasst werden.

Was jetzt noch fehlt ist das regelmäßige Einloggen auf einem APRS-IS Server und das Absetzen der Frames. Dazu erstellen wir ein bash-Script was durch den cron Daemon regelmäßig aufgerufen wird.
<pre>vi aprsbake.sh</pre>
Der Inhalt dieser Datei ist wie folgt:
<pre>#!/bin/sh
nc -v 195.190.142.207 14580 &lt; /home/pi/aprs/DB0ABC.txt
</pre>
Die o.g. IP-Adresse gehört dem APRS-IS Server DB0ERF in Erfurt. Es kann aber auch ein beliebig anderer APRS-IS Server genutzt werden. Zuletzt muss nur noch der cron Daemon programmiert werden:
<pre>crontab -e</pre>
Diese sich öffnende Datei enthält am Ende eine neue Zeile mit folgendem Inhalt, was einem 15 Minuten Intervall entspricht:
<pre>*/15 * * * * /home/pi/aprs/aprsbake.sh</pre>
Viel Spaß!
