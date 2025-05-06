## Aufgabe 1 Generalisierung/Spezialisierung

## 1. Theoretische Zusammenfassung: Generalisierung & Spezialisierung

- *Problem:* Mehrere Entitätstypen (z. B. Mitarbeiter, Kunde, Fahrer) teilen sich gemeinsame Attribute (z. B. Name, Adresse), aber haben auch je eigene Attribute.  
- *Redundanzgefahr:* Würden alle Attribute in jede Tabelle kopiert, entstünde Daten­redundanz und Inkonsistenz (z. B. dieselbe Person einmal als Mitarbeiter, einmal als Kunde).  
- *Lösung – Generalisierung / Spezialisierung:*  
  1. *Generalisiert* gemeinsame Attribute in einer Ober­entität („*Supertype*“), z. B. *Person* (ID, Nachname, Vorname).  
  2. *Spezialisiert* nur die unterschiedlichen Attribute in Unter­entitäten („*Subtypes*“), z. B. *Mitarbeiter* (Personalnummer, Eintrittsdatum), *Kunde* (Rabattstufe), *Fahrer* (Fahrereintritt).  
  3. *Verknüpfung* über *is_a*–Beziehungen (Spezialisten haben Fremd­schlüssel auf die Supertype-Tabelle); so bleibt jede Information genau einmal gespeichert.  
- *Parallele in OOP:* Vererbung von Klassen (Superklasse `Person`, Subklassen `Mitarbeiter`, `Kunde`, `Fahrer`).

---

## 2. Eigenes Beispiel einer Generalisierung

Stellen wir uns ein System für *Fahrzeuge* vor:

- *Supertype* `Fahrzeug`  
  - `FahrzeugID` (PK), `Hersteller`, `Modell`, `Baujahr`  
- *Subtype* `PKW`  
  - `AnzahlSitze`, `Kofferraumvolumen`  
- *Subtype* `LKW`  
  - `ZulässigesGesamtgewicht`, `AnzahlAchsen`  
- *Subtype* `Motorrad`  
  - `Hubraum`, `Motortyp`  

Jedes spezialisierte Fahrzeug verweist per FK auf einen Datensatz in `Fahrzeug`.

---

## 3. Sammlung weiterer Beispiele von KlassenkameradInnen

- *Produkt → Buch / DVD / Software*
- *Benutzer → Admin / Moderator / Gast* 
- *Tier → Hund / Katze / Vogel*
- *Mitarbeiter → Lehrer / Ingenieur / Sekretärin*

---

## 4. Praktische Umsetzungsansätze (Recherche mit ChatGPT / phind)

1. *Datenbankseitig:*  
   - *PostgreSQL TABLE INHERITS*:  
     ```sql
     CREATE TABLE person (
       id SERIAL PRIMARY KEY,
       nachname VARCHAR(45),
       vorname  VARCHAR(45)
     );
     CREATE TABLE mitarbeiter (
       personalnummer INT,
       eintrittsdatum DATE
     ) INHERITS (person);
     ```
   - *MySQL / MariaDB:* Kein native Vererbung – stattdessen SUPERTYPE-Tabelle + Subtype-Tabellen mit PK=FK; ggf. Views zur einfachen Abfrage.  

2. *ORM-Layer / Applikation:*  
   - *JPA / Hibernate*:  
     - `@Inheritance(strategy = InheritanceType.JOINED)` für one-to-one-Subtyping  
     - `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)` für Single-Table-Mapping  
   - *Entity Framework (C#/.NET):* TPH (Table-Per-Hierarchy) oder TPT (Table-Per-Type).

3. *Modellierungstools:*  
   - *UML-Editors* (StarUML, Visual Paradigm) zur Visualisierung von Generalisierungspfeilen.  
   - *Workbench/ER-Tools*: Unterstützung für Supertypen und Subtypen mit automatischer Generierung der DDL-Skripte.

4. *Bewertung/Konsequenzen:*  
   - Trade-off zwischen Abfrage­performance (Joins bei JOINED-Strategie) vs. Wartbarkeit.  
   - Daten­integrität durch FK-Constraints sichern.

# Aufgabe 2 Beziehungsarten

## 1. Zusammenfassung Datenmodellierung

- **Zweck:** Strukturierung der zu verarbeitenden und zu speichernden Daten in Informationssystemen  
- **Phasen der Datenmodellierung:**  
  - **Konzeptionelles Datenmodell (ERM):**  
    - Fachliche Abbildung der Anforderungen  
    - Implementierungs‐unabhängig  
  - **Logisches Datenmodell (ERD):**  
    - Abbildung auf das Ziel‐Datenbanksystem (z. B. relational)  
    - Definition von Primärschlüsseln (PK), Fremdschlüsseln (FK) und Datentypen  
  - **Physisches Datenmodell:**  
    - DDL-Skripte (CREATE TABLE, …)  
    - Indizes, Partitionierung, produktspezifische Optimierungen  

- **Modellierungstypen im Data-Warehouse-Umfeld:**  
  - **3NF-Modellierung:** Operative Systeme, Core/Enterprise DWH, anwendungsneutrale Speicherung  
  - **Star Schema:** Reporting-Layer/Data-Mart, optimiert für Auswertungen  
  - **Data Vault:** Core-DWH/Integration-Layer, Fokus auf Erweiterbarkeit und automatisierte Ladung  

---

## 2. Normalisierungsschritte

1. **1. Normalform (1NF)**  
   - Entfernen mehrfach auftretender Gruppen  
   - Alle Attribute haben atomare (unteilbare) Werte  

2. **2. Normalform (2NF)**  
   - Voraussetzung: 1NF erfüllt  
   - Keine partiellen Abhängigkeiten bei zusammengesetztem Primärschlüssel  

3. **3. Normalform (3NF)**  
   - Voraussetzung: 2NF erfüllt  
   - Keine transitiven Abhängigkeiten (kein Nicht-Schlüsselattribut von einem anderen Nicht-Schlüsselattribut abhängig)  

4. **Weiterführende Formen (optional):** BCNF, 4NF, 5NF  

---

## 3. Aufgabe: Tourenplaner

### 3.1 Konzeptionelles Datenmodell (ERM)

- **Entitäten & Attribute**  
  - **Tour** (TourID, Name, StartDatum, EndDatum)  
  - **Teilnehmer** (TeilnehmerID, Vorname, Nachname, Email)  
  - **Ort** (OrtID, Bezeichnung, Adresse)  

- **Beziehungen**  
  - **Teilnahme** (n:m zwischen Teilnehmer und Tour)  
    - Attribut: Buchungsdatum  
  - **TourEtappe** (n:m zwischen Tour und Ort)  
    - Attribut: Reihenfolge  

### 3.2 Logisches Datenmodell (ERD)

| Tabelle        | Attribute                                      | Schlüssel                                                      |
| -------------- | ---------------------------------------------- | -------------------------------------------------------------- |
| **Tour**       | TourID, Name, StartDatum, EndDatum             | PK: TourID                                                     |
| **Teilnehmer** | TeilnehmerID, Vorname, Nachname, Email         | PK: TeilnehmerID                                               |
| **Ort**        | OrtID, Bezeichnung, Adresse                    | PK: OrtID                                                      |
| **Buchung**    | TourID, TeilnehmerID, Buchungsdatum            | PK: (TourID, TeilnehmerID)<br>FK: TourID → Tour<br>FK: TeilnehmerID → Teilnehmer |
| **Tour_Ort**   | TourID, OrtID, Reihenfolge                     | PK: (TourID, OrtID)<br>FK: TourID → Tour<br>FK: OrtID → Ort     |

#### Normalisierungs-Check

- **1NF:** Alle Werte atomar  
- **2NF:** Keine partiellen Abhängigkeiten in den n:m-Tabellen (Buchung, Tour_Ort)  
- **3NF:** Keine transitiven Abhängigkeiten; alle Nicht-Schlüsselattribute hängen nur vom Primärschlüssel ab  

---

## 4. Identifying & Non-Identifying Relationships

### 4.1 Theorie-Zusammenfassung

**Identifying Relationship**  
- FK ist Teil des Primärschlüssels der Child-Tabelle  
- Child-Entität existiert nur im Kontext des Parent  
- Ändert sich der Parent, ändert sich damit die Identität des Child

# Aufgabe 3 DBMS

## Zusammenfassung Datenbanksysteme (DBS / DBMS)

**1. Begriff & Aufbau**  
- **Datenbanksystem (DBS):** Software + Datenbestand zur elektronischen Verwaltung großer Datenmengen.  
- **Datenbankmanagementsystem (DBMS):** Verwaltungskomponente, die Speicherung, Konsistenz und Zugriffskontrolle übernimmt.  
- **Datenbank (DB):** Die tatsächlichen, persistent gespeicherten Daten.

**2. Kernfunktionen eines DBMS**  
1. **Integrierte Datenhaltung**  
   - Einheitliche Ablage aller Daten– keine Mehrfachspeicherung von logischen Daten­elementen  
   - Definition komplexer Beziehungen, Verknüpfungen, kontrollierte Redundanz  
2. **Datenbanksprache**  
   - **DQL/DRL (SELECT)** – Abfragen  
   - **DDL (CREATE, ALTER, DROP)** – Strukturdefinition  
   - **DML (INSERT, UPDATE, DELETE)** – Datenmanipulation  
   - **DCL (GRANT, REVOKE)** – Rechteverwaltung  
   - **TCL (COMMIT, ROLLBACK)** – Transaktionskontrolle  
3. **Katalog (Data Dictionary)**  
   - Metadaten über Tabellen, Spalten, Beziehungen, Constraints  
4. **Benutzersichten (Views)**  
   - Externe Schemata für unterschiedliche Benutzer­anforderungen  
5. **Konsistenz- und Integritätskontrolle**  
   - Constraints (Prüfbedingungen)  
   - Physische Integritätsprüfung (Speicher­konsistenz)  
6. **Zugriffskontrolle (Access Control)**  
   - Benutzer- und Rollen-Rechte auf Tabellen, Spalten und Views  
7. **Transaktionen & Mehrbenutzerfähigkeit**  
   - Atomarität, Konsistenz, Isolation, Dauerhaftigkeit (ACID)  
   - Sperr- und Synchronisations­mechanismen  
8. **Datensicherung & Recovery**  
   - Backup/Restore, Journaling, Crash-Recovery

**3. Vorteile eines DBMS**  
- Standardisierung von Daten­namen und Formaten  
- Effiziente Speicherung & Retrieval großer Daten­mengen  
- Kürzere Entwicklungs­zeiten durch fertige DB-Funktionen  
- Hohe Flexibilität (Daten­unabhängigkeit)  
- Hohe Verfügbarkeit & gleichzeitiger Zugriff  
- Kosteneinsparung durch Zentralisierung

**4. Nachteile eines DBMS**  
- Hohe Anschaffungs- und Betriebskosten  
- Geringere Performance in Spezialfällen  
- Komplexität: Fachpersonal (DBA) erforderlich  
- Kosten für Security, Concurrency-Management  
- Single Point of Failure (Lösung: verteilte DB)

**5. Wann statt DBMS reguläre Dateien?**  
- Kein Mehr­benutzer­zugriff nötig  
- Harte Echtzeit­anforderungen  
- Sehr einfache, unveränderliche Daten­strukturen

## DB-Ranking

### Vergleich: Ursprüngliche Liste vs. Aktuelles DB-Engines Ranking (April 2025)

| DBMS               | Hersteller        | In Top 10? (Rang) |
|--------------------|-------------------|-------------------|
| Adabas             | Software AG       | Nein              |
| Cache              | InterSystems      | Nein              |
| IBM Db2            | IBM               | Ja (9)            |
| Firebird           | —                 | Nein              |
| IMS                | IBM               | Nein              |
| Informix           | IBM               | Nein              |
| InterBase          | Borland           | Nein              |
| MS Access          | Microsoft         | Nein              |
| MS SQL Server      | Microsoft         | Ja (3)            |
| MySQL              | MySQL AB          | Ja (2)            |
| Oracle             | Oracle            | Ja (1)            |
| PostgreSQL         | —                 | Ja (4)            |
| Sybase ASE         | Sybase            | Nein              |
| Versant            | Versant           | Nein              |
| Visual FoxPro      | Microsoft         | Nein              |
| Teradata           | NCR Teradata      | Nein              |

### Mindmap der Top 10 DB-Engines
mindmap
  root((Top 10 DB-Engines))
    Oracle
    MySQL
    Microsoft SQL Server
    PostgreSQL
    MongoDB
    Snowflake
    Redis
    Elasticsearch
    IBM Db2
    SQLite

# Aufgabe 4 DDL 
```sql
-- 1. CREATE SCHEMA und Tabellen anlegen
CREATE SCHEMA IF NOT EXISTS `AufgabenDB`
  DEFAULT CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
USE `AufgabenDB`;

-- Tabelle tbl_fahrer
CREATE TABLE `tbl_fahrer` (
  `ID` INT NOT NULL AUTO_INCREMENT,
  `Name` VARCHAR(45) NOT NULL,
  `Vorname` VARCHAR(45) NOT NULL,
  `Telefonnummer` VARCHAR(45),
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4;

-- Tabelle tbl_disponent
CREATE TABLE `tbl_disponent` (
  `ID` INT NOT NULL AUTO_INCREMENT,
  `Name` VARCHAR(45) NOT NULL,
  `Vorname` VARCHAR(45) NOT NULL,
  `Telefonnummer` VARCHAR(45),
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4;


-- 2. DROP TABLE und Wiederherstellung
DROP TABLE IF EXISTS `tbl_disponent`;

-- Wiederherstellung per erneutem CREATE TABLE
CREATE TABLE `tbl_disponent` (
  `ID` INT NOT NULL AUTO_INCREMENT,
  `Name` VARCHAR(45) NOT NULL,
  `Vorname` VARCHAR(45) NOT NULL,
  `Telefonnummer` VARCHAR(45),
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4;


-- 3. Generalisierung: Oberentität tbl_Mitarbeiter
CREATE TABLE `tbl_Mitarbeiter` (
  `MA_ID` INT NOT NULL AUTO_INCREMENT,
  `Name` VARCHAR(50) NOT NULL,
  `Vorname` VARCHAR(30) NOT NULL,
  `Geburtsdatum` DATETIME,
  `Telefonnummer` VARCHAR(12),
  `Einkommen` FLOAT(10,2),
  PRIMARY KEY (`MA_ID`)
) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4;


-- 4. ALTER TABLE MODIFY: Charset von Name & Vorname in latin1 ändern
ALTER TABLE `tbl_Mitarbeiter`
  MODIFY `Name`    VARCHAR(50) CHARACTER SET latin1 NOT NULL,
  MODIFY `Vorname` VARCHAR(30) CHARACTER SET latin1 NOT NULL;


-- 5. ALTER TABLE DROP COLUMN: gemeinsame Attribute löschen
ALTER TABLE `tbl_fahrer`
  DROP COLUMN `Name`,
  DROP COLUMN `Vorname`,
  DROP COLUMN `Telefonnummer`;

ALTER TABLE `tbl_disponent`
  DROP COLUMN `Name`,
  DROP COLUMN `Vorname`,
  DROP COLUMN `Telefonnummer`;


-- 6. Spezialisierung: Fremdschlüssel auf tbl_Mitarbeiter hinzufügen
ALTER TABLE `tbl_fahrer`
  ADD COLUMN `MA_ID` INT NOT NULL,
  ADD CONSTRAINT `FK_fahrer_mitarbeiter`
    FOREIGN KEY (`MA_ID`) REFERENCES `tbl_Mitarbeiter`(`MA_ID`);

ALTER TABLE `tbl_disponent`
  ADD COLUMN `MA_ID` INT NOT NULL,
  ADD CONSTRAINT `FK_disponent_mitarbeiter`
    FOREIGN KEY (`MA_ID`) REFERENCES `tbl_Mitarbeiter`(`MA_ID`);
```

# Aufgabe 5 Forward Engineering eines DB Modells

### 1.1 Modell erstellen
- Erstelle in deinem Modellierungs‐Tool (z. B. MySQL Workbench) ein ER-Diagramm  
  - Verwende dein Tourenplaner-Modell oder ein eigenes Beispiel mit mind. 3 Tabellen und Beziehungen  

### 1.2 Forward Engineering starten
1. **Forward Engineering**-Assistent öffnen  
2. **Ziel‐Datenbank** auswählen (z. B. MySQL auf TBZ-Cloudserver, localhost:127.0.0.1 oder AWS-Instanz)  
3. **Benutzername/Passwort** eingeben  
4. **Optionen wählen**  
   - DDL-Statements (CREATE SCHEMA, CREATE TABLE, ALTER TABLE, FOREIGN KEYS)  
   - ggf. Indizes, Views, Stored Procedures  
   - Filter, was nicht erzeugt werden soll  

### 1.3 SQL-Skript sichern
- Klicke auf **Save to File…** und sichere das generierte SQL-Script (z. B. `tourenplaner_forward.sql`)  
- Füge das SQL-Script in dein Lernportfolio ein oder verlinke es  

### 1.4 Skript auf DB-Server übertragen
- Mit **Next** die Statements an den Server senden  
- Verbindung prüfen: keine Fehlermeldungen  

### 1.5 Struktur im DB-Server prüfen
USE AufgabenDB;
SHOW TABLES;
DESCRIBE Tour;
DESCRIBE Teilnehmer;
DESCRIBE Tour_Ort;

# Aufgabe 6 Synchronize

## Synchronize Model

1. **Modell anpassen**  
   - Füge im ER-Diagramm eine neue Tabelle `tbl_MA` hinzu (z. B. für Administrationszwecke)  
   - Ziehe eine Relation von `tbl_MA` zu einer bestehenden Tabelle, z. B. `tbl_Tour`  
   - Erweitere eine bestehende Tabelle, z. B. `tbl_Tour`, um ein neues Attribut `Beschreibung VARCHAR(255)`

2. **Synchronize Model**  
   - Rechtsklick auf das Physical Schema → **Edit Schema** und ggf. Schema-Name anpassen  
   - Menü **Database → Synchronize Model…** öffnen  
   - Verbindung zum Ziel-Server (z. B. localhost:3306, AWS-Endpoint o. Ä.)  
   - Im Assistenten die Unterschiede zwischen Modell und DB-Schema prüfen  
   - Generierte DDL-Scripts anwenden oder in Datei exportieren  

3. **Ergebnis prüfen**  
   USE AufgabenDB;
   SHOW TABLES LIKE 'tbl_MA';
   DESCRIBE tbl_Tour;

# Aufgabe 7 Partitionen

### 1.1 Partition (Datenbanken)
- **Definition:** Aufteilung einer großen Datenbank-Tabelle oder -Datenbank in kleinere, unabhängige Teile (Partitionen). Jede Partition enthält einen disjunkten Teil der Daten, aber zusammen repräsentieren alle Partitionen den gesamten Datensatz.  
- **Arten:**
  - *Horizontale Partitionierung* (Shard, Subtable): Zeilen werden nach Kriterien (z. B. Datum, geografische Region) verteilt.  
  - *Vertikale Partitionierung:* Spalten oder Spalten-Gruppen werden getrennt abgelegt.  
- **Ziele:** Skalierbarkeit, Manageability, Performance-Optimierung, Verfügbarkeit und Lastverteilung  

### 1.2 Storage Engine
- **Definition:** Kernkomponente eines DBMS, die die physischen CRUD-Operationen (Create, Read, Update, Delete) auf den Datendateien übernimmt.  
- **Funktion:** Verwaltung von Datenseiten, Sperr­mechanismen, Transaktionen, Indizes, Puffer-Pooling, Crash-Recovery.  
- **Beispiele (MySQL):**  
  - **InnoDB:** Transaktional, MVCC, Foreign Keys, ACID  
  - **MyISAM:** Non-transaktional, Full-Text-Index, schneller bei reinen Lese­zugriffen  
  - **Memory:** Daten im RAM, sehr schnell, flüchtig  


### 1.3 Tablespace (InnoDB)
- **Definition:** Logische Einheit in InnoDB, die eine oder mehrere physische Datendateien (`ibdata-N` oder `.ibd`) kapselt und darauf Tabellen sowie Indexstrukturen speichert.  
- **Typen:**
  - **System Tablespace:**
