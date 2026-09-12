# Screenshots

| Datei | Inhalt |
|---|---|
| 01-hyperv-uebersicht.png | Hyper-V-Manager, alle VMs laufen |
| 02-netzplan.png | Netzplan |
| 03-dc01-serverrollen.png | Server-Manager DC01 mit den Rollen |
| 04-ad-ou-struktur.png | ADUC, OU-Struktur aufgeklappt |
| 05-ad-gruppe-mitglieder.png | Gruppenkette Buchhaltung: Benutzer in G-Buchhaltung, G-Buchhaltung in DL-Vertrieb-RO |
| 06-gpo-uebersicht.png | Gruppenrichtlinienverwaltung |
| 07-cl01-gpresult.png | gpresult /r am Client |
| 08-cl01-laufwerk-v.png | Explorer mit Laufwerk V: |
| 09-cl01-zugriff-verweigert.png | Buchhaltung kann im Vertriebsordner nicht speichern |
| 10-opnsense-regeln.png | Firewall > Rules > LAN |
| 11-opnsense-log.png | Live-Log mit greifender Block-Regel |
| 12-fs01-berechtigungen.png | NTFS-Berechtigungen auf D:\Freigaben\Vertrieb |

## Datensicherung

| Datei | Inhalt |
|---|---|
| 21-wsb-uebersicht.png | Windows Server Backup, Sicherung und Wiederherstellung des Systemzustands erfolgreich |
| 22-wsb-eventlog.png | Ereignisprotokoll mit dem Abschluss der Wiederherstellung |
| 23-ad-papierkorb.png | Der gelöschte Benutzer im Container Deleted Objects |
| 24-ad-benutzer-zurueck.png | Das Konto liegt nach dem Restore wieder in der OU Buchhaltung |
| 25-veeam-repository.png | Ordner auf der externen SSD mit Vollsicherung, Inkrement und Metadaten |
| 26-fs01-geloescht.png | Hyper-V-Manager nach dem Löschen von FS01 |
| 27-veeam-instant-recovery.png | Restore-Session mit Start- und Endzeit |
| 28-fs01-laeuft-wieder.png | FS01 läuft nach der Instant Recovery wieder |
| 29-fs01-freigabe-nach-restore.png | Die Freigabe Vertrieb ist nach der Wiederherstellung wieder da |
| 30-opnsense-backup.png | OPNsense, System > Configuration > Backups |
| 31-dc01-zeitquelle.png | w32tm /query /status auf DC01, Quelle ist die Firewall |
| 32-msconfig-safeboot.png | msconfig, Safe boot mit Active Directory repair war aktiv geblieben |
| 33-veeam-schutzgruppe.png | Schutzgruppe Lab-Server mit DC01 und FS01, Agent installiert |
| 34-veeam-job.png | Auftrag Lab-Täglich mit Ergebnis und Statistik |
| 35-svc-backup.png | Dienstkonto svc-backup in der OU Dienstkonten |
| 36-opnsense-vor-restore.png | Frisch installierte OPNsense mit den beiden Standardregeln |
| 37-opnsense-konsole-interfaces.png | Konsole mit vertauschten Schnittstellen nach dem Einspielen |
| 38-opnsense-nach-restore.png | Alle sieben Regeln nach dem Einspielen der Konfiguration |
