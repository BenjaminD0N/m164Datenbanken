# Generalisierung/Spezialisierung

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


[]
