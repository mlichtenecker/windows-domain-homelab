# DHCP-Dienst zeigt einen roten Pfeil, obwohl er autorisiert ist

**01.09.2026**

## Symptom

Bei der Kontrolle mit `dcdiag /c` auf DC01 ist der Test `SystemLog`
durchgefallen. In der Ausgabe standen unter anderem diese beiden Einträge:

```
The DHCP service failed to see a directory server for authorization.
The DHCP/BINL service ... has determined that it is not authorized to start.
It has stopped servicing clients.
```

Passend dazu zeigte die DHCP-Konsole neben dem Server einen roten Pfeil nach
unten statt des grünen nach oben.

## Suche

Meine erste Vermutung war, dass die Autorisierung in Active Directory fehlt oder
verloren gegangen ist. Geprüft habe ich das mit:

```
Get-DhcpServerInDC
```

Das Ergebnis war allerdings korrekt:

```
IPAddress            DnsName
---------            -------
10.10.10.10          dc01.ad.mlab.internal
```

Die Autorisierung war also die ganze Zeit vorhanden, der Fehler musste woanders
liegen.

## Ursache

Dem Dienst reicht es nicht, autorisiert zu sein. Er prüft das nur einmal beim
eigenen Start und danach nicht mehr laufend. Auf DC01 laufen AD DS, DNS und DHCP
auf derselben Maschine und starten beim Booten nahezu gleichzeitig. Startet der
DHCP-Dienst, bevor Active Directory vollständig ansprechbar ist, findet er beim
Autorisierungs-Check keinen erreichbaren Verzeichnisdienst. Er hält sich
daraufhin selbst für nicht autorisiert und stellt die Adressvergabe ein, obwohl
der Eintrag in AD die ganze Zeit korrekt vorhanden war.

Es handelt sich also um ein Problem der Startreihenfolge und nicht um einen
tatsächlichen Autorisierungsfehler.

## Behoben

Den Dienst einmal manuell neu gestartet, nachdem AD sicher vollständig oben war:

```
Restart-Service DHCPServer
```

Danach hat der Dienst die Autorisierung erneut geprüft, diesmal erfolgreich, und
der Pfeil wurde grün.

## Merken

Auf einer einzelnen Maschine, auf der AD, DNS und DHCP zusammen laufen, kann es
beim Kaltstart zu diesem Wettlauf kommen. Zeigt der DHCP-Dienst nach einem
Neustart der VM einen roten Pfeil, obwohl `Get-DhcpServerInDC` eine korrekte
Autorisierung ausweist, reicht meistens ein `Restart-Service DHCPServer`, statt
an der Autorisierung selbst herumzudoktern.

Dauerhaft ließe sich das über eine verzögerte Startart des DHCP-Dienstes oder
eine Dienstabhängigkeit lösen. In einer echten Unternehmensumgebung würde man
den DHCP-Dienst meistens auf einen eigenen Server auslagern, um den
Domänencontroller zu entlasten. Für mein Homelab war die Kombination aber die
beste Wahl, um Arbeitsspeicher auf meinem Host-PC zu sparen.
