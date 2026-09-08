# Technische Details

Nachschlagteil zur [Übersicht](README.md) mit den konkreten Werten. Die
Begründungen für die einzelnen Entscheidungen stehen dort, hier stehen die
Einstellungen.

## Netz

| | |
|---|---|
| Netz | 10.10.10.0/24 |
| Gateway | 10.10.10.1 |
| DNS für Clients | 10.10.10.10 |
| DHCP-Bereich | 10.10.10.100 bis .199, Leasedauer 8 Tage |
| Domäne | ad.mlab.internal (NetBIOS MLAB) |

Feste Adressen: .1 FW01, .5 Host-PC, .10 DC01, .20 FS01.

## Virtuelle Switches

| Switch | Typ | Wofür |
|---|---|---|
| Default-Switch | NAT | WAN-Seite von FW01 |
| LAB-Intern | intern | das Lab-Netz |

## Virtuelle Maschinen

Alle vier VMs laufen als Generation 2 mit zwei virtuellen Prozessoren.

| VM | Zugewiesen | Dynamischer RAM | Minimum | Maximum |
|---|---|---|---|---|
| DC01 | 2048 MB | ja | 1024 MB | 4096 MB |
| FS01 | 1536 MB | ja | 1024 MB | 3072 MB |
| CL01 | 2048 MB | ja | 1024 MB | 4096 MB |
| FW01 | 1536 MB | nein | – | – |

Der Arbeitsspeicherpuffer liegt überall bei 20 Prozent. Bei FW01 ist der
dynamische Arbeitsspeicher abgeschaltet, weil FreeBSD damit nicht umgehen kann.
Dort ist außerdem Secure Boot deaktiviert, während die Windows-Maschinen mit
Secure Boot und virtuellem TPM laufen.

FS01 hat eine zweite virtuelle Festplatte für die Freigaben, FW01 zwei
Netzwerkkarten für LAN und WAN.

## DNS

Auf DC01 läuft die Active-Directory-integrierte Zone `ad.mlab.internal`. Als
Weiterleitungen sind 1.1.1.1 und 9.9.9.9 eingetragen.

Dazu kommt die Reverse-Lookup-Zone `10.10.10.in-addr.arpa`. Damit lässt sich zu
einer IP-Adresse der zugehörige Rechnername ermitteln, was beim Auswerten von
Protokollen hilft.

## DHCP

Der Dienst läuft auf DC01 und ist in Active Directory autorisiert.

| Option | Wert |
|---|---|
| 003 Router | 10.10.10.1 |
| 006 DNS-Server | 10.10.10.10 |
| 015 DNS-Domänenname | ad.mlab.internal |

## Firewall-Aliase

| Name | Typ | Inhalt |
|---|---|---|
| ADMIN_PC | Host | 10.10.10.5 |
| SRV_DC | Host | 10.10.10.10 |
| SERVERS | Host | 10.10.10.10, 10.10.10.20 |
| CLIENTS | Host | 10.10.10.100-10.10.10.199 |
| PORTS_WEB | Port | 80, 443 |

## Firewall-Regeln, Interface LAN eingehend

| # | Aktion | Protokoll | Quelle | Ziel | Port | Beschreibung |
|---|---|---|---|---|---|---|
| 1 | Pass | any | ADMIN_PC | any | any | Admin-PC: alles |
| 2 | Pass | TCP/UDP | SRV_DC | any | 53 | DC01: DNS extern |
| 3 | Pass | UDP | LAN net | LAN address | 123 | LAN: NTP an FW01 |
| 4 | Pass | ICMP | LAN net | any | echo req | LAN: Ping-Diagnose |
| 5 | Pass | TCP | CLIENTS | any | PORTS_WEB | Clients: Webzugriff |
| 6 | Pass | TCP | SERVERS | any | PORTS_WEB | Server: Updates |
| 7 | Block, Log | any | LAN net | any | any | Default Deny + Log |

Über den eigenen Regeln stehen die beiden von OPNsense mitgelieferten Regeln
*Default allow LAN to any* für IPv4 und IPv6. Ich habe sie deaktiviert, weil
sonst bereits die erste Regel jeden Verkehr aus dem LAN erlaubt und das gesamte
Regelwerk darunter wirkungslos bleibt. Wie mir das aufgefallen ist, steht in
[Störungsbericht 2](stoerungsberichte/02-firewall-regelwerk-greift-nicht.md).

Der Verkehr zwischen CL01, DC01 und FS01 läuft im selben Subnetz über den
virtuellen Switch und erreicht die Firewall gar nicht. Für Anmeldung, interne
Namensauflösung und die Dateifreigabe sind deshalb keine eigenen Regeln nötig.

Auf dem WAN-Interface gibt es keine eigenen Regeln, dort sind lediglich *Block
private networks* und *Block bogon networks* abgeschaltet.

## Benutzer

| Abteilung | Name | Anmeldename |
|---|---|---|
| IT | Max Mustermann | mmustermann |
| IT | Tim Fischer | tfischer |
| Vertrieb | Anna Schmidt | aschmidt |
| Vertrieb | Jane Smith | jsmith |
| Buchhaltung | Lukas Weber | lweber |
| Buchhaltung | Laura Meyer | lmeyer |
| Helpdesk | Björn Krüger | bkrueger |

Bei den Anmeldenamen habe ich Umlaute ersetzt, weil angeschlossene Systeme und
Schnittstellen damit oft nicht zurechtkommen. Zu Testzwecken habe ich es
trotzdem einmal mit Umlaut ausprobiert. Active Directory selbst nimmt den Namen
an.

## Gruppen

- Global: G-Vertrieb, G-Buchhaltung, G-Helpdesk
- Domänenlokal: DL-Vertrieb-RW, DL-Vertrieb-RO, DL-Helpdesk

```
aschmidt  -> G-Vertrieb     -> DL-Vertrieb-RW -> NTFS Ändern
lweber    -> G-Buchhaltung  -> DL-Vertrieb-RO -> NTFS Lesen
bkrueger  -> G-Helpdesk     -> DL-Helpdesk    -> Delegierung auf der OU Benutzer
```

Die Helpdesk-Rechte hängen an DL-Helpdesk und sind über den Assistenten
*Delegate Control* auf der OU Benutzer vergeben, beschränkt auf das Zurücksetzen
von Kennwörtern und das Lesen von Benutzerinformationen.

## Gruppenrichtlinien

| GPO | Verknüpft an | Einstellung |
|---|---|---|
| GPO-Bildschirmsperre | OU=MLAB | Machine inactivity limit 900 |
| GPO-Laufwerk-Vertrieb | OU=Vertrieb | V: auf `\\FS01\Vertrieb` |
| GPO-Firewall | OU=MLAB | alle drei Profile an, eingehend blockieren |
| GPO-USB-Sperre | OU=Computer | Removable Storage: Deny all access |

In der Default Domain Policy stehen Mindestlänge 12, Komplexität aktiviert,
Chronik 5, Höchstalter 365 Tage sowie eine Sperre nach 5 Fehlversuchen für 15
Minuten.

Neue Computerkonten werden mit `redircmp` nach `OU=Computer,OU=MLAB` umgeleitet,
damit sie nicht im Container Computers landen, an den sich keine
Gruppenrichtlinien verknüpfen lassen.

## Freigabe `D:\Freigaben\Vertrieb`

| Ebene | Wer | Recht |
|---|---|---|
| Freigabe | Authentifizierte Benutzer | Vollzugriff |
| NTFS | SYSTEM | Vollzugriff |
| NTFS | Lokale Administratoren (FS01) | Vollzugriff |
| NTFS | MLAB\DL-Vertrieb-RW | Ändern |
| NTFS | MLAB\DL-Vertrieb-RO | Lesen und Ausführen |

Beide Server sind englisch installiert. In dieser Dokumentation stehen die
Berechtigungen auf Deutsch, in den Screenshots heißen sie entsprechend *Full
control*, *Modify* und *Read & execute*.

## Zeit

FW01 läuft als NTP-Server auf dem LAN-Interface und ist auf DC01 als Zeitquelle
eingetragen. Die Clients bekommen ihre Zeit über die Domäne vom
Domänencontroller.

Bei DC01 ist in den Hyper-V-Einstellungen die Zeitsynchronisierung mit dem Host
abgeschaltet, damit nicht zwei Quellen gleichzeitig an der Uhr drehen.
