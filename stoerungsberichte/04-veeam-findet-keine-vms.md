# Veeam bietet keine virtuellen Maschinen zum Sichern an

**09.09.2026**

## Symptom

Wenn ich bisher mit Veeam gearbeitet habe, lief es immer auf einem Server. Dafür
hätte ich hier eine weitere virtuelle Maschine gebraucht, und der Arbeitsspeicher
ist auf meinem Host-PC ohnehin knapp. Also habe ich Veeam diesmal direkt auf dem
Host installiert, also unter Windows 11 Pro.

Anschließend habe ich den Hyper-V-Host in Veeam hinzugefügt, ein Sicherungsziel
angelegt und einen Sicherungsauftrag gestartet. Bei dem Schritt, in dem man die
virtuellen Maschinen auswählt, blieb die Liste leer. Es kam weder ein Fehler noch
eine Meldung, es stand schlicht nichts darin. Unter *Managed Servers* war der
Host korrekt eingetragen und ein Rescan lief sauber durch, angeboten wurde mir
trotzdem nichts.

## Suche

Zuerst bin ich davon ausgegangen, dass ich falsch navigiere, und habe die Ansicht
mehrmals umgestellt.

Beim Hinzufügen des Hosts hatte ich vorher bereits zwei andere Probleme gelöst.
Veeam kam zunächst nicht auf die administrative Freigabe meines Rechners, und
danach war mein Windows-Konto wegen zu vieler Fehlversuche gesperrt. Beides habe
ich behoben, und beides hatte mit der leeren Liste nichts zu tun.

## Ursache

Die Antwort stand in der Warnmeldung, die beim Hinzufügen des Hosts erscheint:

> Hyper-V on Windows Client OS is supported only as an Instant Recovery target
> and for the Data Labs functionality. Host-based backup of VMs running on such
> Hyper-V host is not supported, but you can use agent-based backup instead.
> Proceed anyway?

Das "not supported" habe ich so verstanden, dass es funktioniert, der Hersteller
aber keinen Support dafür übernimmt. Gemeint war, dass es gar nicht geht.
Hyper-V unter Windows 11 bringt die Funktionen nicht mit, die man braucht, um
eine laufende virtuelle Maschine von außen zu sichern. Sie stehen nur unter
Windows Server zur Verfügung. Der Host lässt sich zwar eintragen, sichern kann
man von dort aber nichts.

## Behoben

Ich habe auf agentenbasierte Sicherung umgestellt. Statt eines Auftrags für
virtuelle Maschinen legt man dabei einen für Windows-Rechner an, und in den
Servern läuft anschließend ein kleines Programm von Veeam, das sich selbst
sichert. Wie das im Einzelnen eingerichtet ist, steht in [backup.md](../backup.md).

## Merken

Weicht man vom üblichen Aufbau ab, weil die Hardware nicht mehr hergibt, fällt
meistens auch Funktionalität weg. Ich hatte erwartet, dass Veeam auf einem Client
dasselbe leistet wie auf einem Server, nur langsamer. Beim nächsten Mal sehe ich
vorher nach, welche Einschränkungen der Hersteller für genau die Umgebung nennt,
die ich tatsächlich zur Verfügung habe.
