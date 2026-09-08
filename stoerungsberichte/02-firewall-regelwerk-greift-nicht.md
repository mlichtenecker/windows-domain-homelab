# Das Regelwerk greift nicht, weil eine Default-Regel darüber steht

**06.09.2026**

## Symptom

Beim Nachprüfen des Firewall-Regelwerks blieb mein eigener Test wirkungslos. Am
Client (10.10.10.101) habe ich testweise 1.1.1.1 als DNS-Server eingetragen
statt des Domänencontrollers. Erlaubt ist das laut Regelwerk nur für DC01, das Surfen
hätte also sofort aufhören müssen.

Es funktionierte weiter. Im Live-Log der Firewall stand dazu passend kein
einziger Block-Eintrag vom LAN-Interface.

## Suche

Mein erster Verdacht waren die bestehenden States, weil eine Regel nur über das
erste Paket einer Verbindung entscheidet. Also die State Table zurückgesetzt und
den Browser am Client komplett geschlossen. Der Test blieb trotzdem ohne
Wirkung.

Damit war klar, dass nicht der Zeitpunkt das Problem ist, sondern das Regelwerk
selbst. Bis dahin hatte ich immer nur meine sieben eigenen Regeln angesehen.
Beim Durchgehen der kompletten Liste von oben nach unten standen über meinen
Regeln zwei Einträge, die ich nie angelegt hatte:

```
Default allow LAN to any rule        IPv4
Default allow LAN IPv6 to any rule   IPv6
```

## Ursache

OPNsense legt diese beiden Regeln bei der Installation selbst an, damit man nach
der Ersteinrichtung überhaupt aus dem LAN heraus arbeiten kann. Sie stehen ganz
oben und sind aktiv.

Das Regelwerk wird von oben nach unten ausgewertet, die erste passende Regel
gewinnt. Damit war jeder Verkehr aus dem LAN bereits durch die erste Regel
erlaubt, und meine sieben eigenen Regeln darunter sind nie zur Anwendung
gekommen.

## Behoben

Beide Default-Regeln deaktiviert und nicht gelöscht, damit ich sie bei Bedarf
wieder einschalten kann. Danach *Apply* und die State Table zurückgesetzt.

Aussperren konnte ich mich dabei nicht: Die erste meiner eigenen Regeln erlaubt
dem Admin-PC alles, und der Verkehr zwischen CL01, DC01 und FS01 läuft im selben
Subnetz an der Firewall vorbei.

Beim wiederholten Test war das Surfen sofort weg, und im Live-Log stand der
erwartete Eintrag:

```
LAN  In  UDP  10.10.10.101:52439 → 1.1.1.1:53  block  Default Deny + Log
```

![Der Block-Eintrag im Live-Log nach der Korrektur](../screenshots/11-opnsense-log.png)

## Merken

Das war schlicht Unachtsamkeit: Ich hätte das Regelwerk von Anfang an
vollständig durchsehen sollen und nicht nur meinen eigenen Teil. Bei einer
Firewall, die von oben nach unten auswertet, entscheidet die erste passende
Regel, und alles darunter spielt keine Rolle mehr.
