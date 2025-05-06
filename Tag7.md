# Datensicherung 

## Warum Backups nötig sind
- Datenbanken versorgen Websites / Unternehmens-Apps mit allen Informationen.  
- Häufige Verlustursachen: Hardware-Defekte oder Bedienfehler, nicht Angriffe.  
- Ausfall = Funktionsstörung, Daten- oder Vertrauensverlust.

## Grundprinzip
Ein *Backup* ist eine Sicherheitskopie, mit der sich der Datenbankzustand zu einem früheren Zeitpunkt wiederherstellen lässt.

## Online- vs. Offline-Backups
| Modus  | Merkmale | Vor- und Nachteile |
|--------|----------|--------------------|
| *Online* | DB bleibt in Betrieb; Änderungen werden währenddessen zwischengespeichert | keine Downtime, aber etwas komplexer |
| *Offline* | DB wird während der Sicherung heruntergefahren | einfacher Prozess, aber Service-Unterbrechung – daher nachts/bei geringer Last durchführen |

## Backup-Methoden
| Methode | Was wird gesichert? | Speicherbedarf | Wiederherstellung |
|---------|--------------------|---------------|-------------------|
| *Voll-Backup* | komplette Datenbank | hoch | nur ein Backup nötig |
| *Differentielles Backup* | alle Änderungen seit letztem *Voll*-Backup | mittel | letztes Voll + gewünschtes diff. Backup |
| *Inkrementelles Backup* | nur Änderungen seit *letztem* Backup (gleich welcher Art) | gering | Voll-Backup + alle inkrementellen Backups bis Zielstand |

## Best Practices
- *Regelmäßig* sichern – Speicher sparen ist kein Argument gegen Backups.  
- Backups *extern & geschützt* (getrenntes Brand-/Diebstahlrisiko) lagern.  
- Sicherungen *verschlüsseln*.  
- Für logische Backups einen *eigenen Datenbank-User* anlegen.

## Werkzeuge (Auswahl)
| Tool | Eigenschaften |
|------|---------------|
| `mysqldump` | Shell-Komando; schnell, aber evtl. vom Hoster gesperrt |
| *phpMyAdmin* | Web-GUI; einfach, Upload-Limit ≈ 2 MB |
| *BigDump* | spielt beliebig große Dumps ein (kein eigenes Backup) |
| *HeidiSQL* | Windows-Client ohne PHP-Limits, keine Automatisierung |
| *Mariabackup* | Open Source; physische *Online*-Backups (InnoDB, Aria, MyISAM) |

## Fazit
Datenbanken müssen doppelt geschützt sein:  
1. *Präventiv* gegen Angriffe.  
2. *Reaktiv* durch verlässliche, regelmäßige Backups, um im Schadfall alles vollständig wiederherstellen zu können.
