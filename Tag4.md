# Aufgabe 1 Mengenlehre

## Aufgabe 1

Gegeben  
A = {c, e, z, r, d, g, u, x}
B = {c, e, g}
C = {r, d, g, t}
D = {e, z, u}
E = {z, r, u}

Beurteilung der Teilmengen-Beziehungen:  
- **a** B ⊂ A  Richtig (c, e, g ∈ A)  
- **b** C ⊂ A  Falsch (t ∉ A)  
- **c** E ⊂ A  Richtig (z, r, u ∈ A)  
- **d** B ⊂ C  Falsch (c, e ∉ C)  
- **e** E ⊂ C  Falsch (z, u ∉ C)  

---

## Aufgabe 2

Gegeben  
A = {1, 2, 3, 4, 5}
B = {2, 5}
C = {3, 5, 7, 9}

a **A ∩ B**  
{2, 5}


b **A ∪ C**  
{1, 2, 3, 4, 5, 7, 9}


c **Bc** (Komplement von B in A, also A \ B)  
{1, 3, 4}

d **B \ C**  
{2}

e **C \ B**  
{3, 7, 9}


---

## Aufgabe 3

Eigene Beispiele mit bezeichnenden Mengen:

1. **⊂ (Teilmenge)**  
   - Mengen:  
     ```
     Nahrungsmittel = { Obst, Gemüse, Fleisch, Getreide }
     Pflanzliches    = { Obst, Gemüse, Getreide }
     ```  
   - Beziehung:  
     ```
     Pflanzliches ⊂ Nahrungsmittel
     ```

2. **∩ (Schnittmenge)**  
   - Mengen:  
     ```
     Obst           = { Apfel, Banane, Orange }
     GelbeFrüchte   = { Banane, Zitrone, Ananas }
     ```  
   - Schnittmenge:  
     ```
     Obst ∩ GelbeFrüchte = { Banane }
     ```

# Aufgabe 2 JOINs

```sql
-- 1.  Datenbank löschen und neu anlegen
DROP DATABASE IF EXISTS `kunden`;
CREATE DATABASE IF NOT EXISTS `kunden`;
USE `kunden`;

-- 2.  Tabellenstrukturen

-- Tabelle `orte`
DROP TABLE IF EXISTS `orte`;
CREATE TABLE `orte` (
  `id_postleitzahl` VARCHAR(5)  NOT NULL,
  `name`            VARCHAR(255) NOT NULL,
  `einwohnerzahl`   INT(11)      DEFAULT NULL,
  PRIMARY KEY (`id_postleitzahl`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

-- Tabelle `kunden`
DROP TABLE IF EXISTS `kunden`;
CREATE TABLE `kunden` (
  `kunde_id`            INT(11)      NOT NULL AUTO_INCREMENT,
  `name`                VARCHAR(200) NOT NULL,
  `fk_ort_postleitzahl` VARCHAR(5)   NOT NULL,
  PRIMARY KEY (`kunde_id`),
  KEY (`fk_ort_postleitzahl`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=latin1;

-- 3.  Beispieldaten einfügen

-- Orte
INSERT INTO `orte` (`id_postleitzahl`, `name`, `einwohnerzahl`) VALUES
  ('80995','München',      1000000),
  ('79312','Emmendingen',     40000),
  ('79111','Freiburg',       280000),
  ('20095','Hamburg',       2000000);

-- Kunden
INSERT INTO `kunden` (`kunde_id`,`name`,`fk_ort_postleitzahl`) VALUES
  (1,'John',      '79111'),
  (2,'Herbert',   '79312'),
  (3,'Sabina',    '79312'),
  (4,'Mary',      '79111'),
  (5,'Heinrich',  '79111'),
  (6,'Usal',      '80995'),
  (7,'Johannes',  '80995'),
  (8,'Carla',     '79312'),
  (9,'Ludowika',  '79111'),
  (10,'Niemand',  '99999');   -- 99999 existiert NICHT in `orte`

-- 4.  Warum das folgende Statement ein „kartesisches Produkt“ erzeugt
-- SELECT * FROM kunden INNER JOIN orte;
--  ↑ Ohne ON-Klausel verbindet MySQL jede Kunden-Zeile mit jeder Orts-Zeile.
--    Das Ergebnis ist riesig und inhaltlich bedeutungslos.
-- 5.  Beispielabfragen (a – g)

-- a) Name, Postleitzahl und Wohnort aller Kunden
SELECT
  k.name                   AS kundenname,
  k.fk_ort_postleitzahl    AS postleitzahl,
  o.name                   AS ort
FROM kunden k
JOIN orte  o ON k.fk_ort_postleitzahl = o.id_postleitzahl;

-- b) Name und Wohnort aller Kunden mit der Postleitzahl 79312
SELECT
  k.name AS kundenname,
  o.name AS ort
FROM kunden k
JOIN orte  o ON k.fk_ort_postleitzahl = o.id_postleitzahl
WHERE k.fk_ort_postleitzahl = '79312';

-- c) Name und Wohnort aller Kunden, die in Emmendingen wohnen
SELECT
  k.name AS kundenname,
  o.name AS ort
FROM kunden k
JOIN orte  o ON k.fk_ort_postleitzahl = o.id_postleitzahl
WHERE o.name = 'Emmendingen';

-- d) Name, Wohnort und Einwohnerzahl für Kunden in Orten > 70 000 Einwohner
SELECT
  k.name          AS kundenname,
  o.name          AS ort,
  o.einwohnerzahl AS einwohner
FROM kunden k
JOIN orte  o ON k.fk_ort_postleitzahl = o.id_postleitzahl
WHERE o.einwohnerzahl > 70000;

-- e) Alle Orte mit weniger als 1 000 000 Einwohnern
SELECT
  id_postleitzahl,
  name          AS ort,
  einwohnerzahl
FROM orte
WHERE einwohnerzahl < 1000000;

-- f) Kundenname und Ortname für Orte mit 100 000–1 500 000 Einwohnern
SELECT
  k.name AS kundenname,
  o.name AS ort
FROM kunden k
JOIN orte  o ON k.fk_ort_postleitzahl = o.id_postleitzahl
WHERE o.einwohnerzahl BETWEEN 100000 AND 1500000;

-- g) Kundenname, Postleitzahl und Ort
--    Bedingung:
--      • Kundenname enthält „e“
--      • Ortsname enthält „u“ ODER „r“
SELECT
  k.name                AS kundenname,
  k.fk_ort_postleitzahl AS postleitzahl,
  o.name                AS ort
FROM kunden k
JOIN orte  o ON k.fk_ort_postleitzahl = o.id_postleitzahl
WHERE k.name LIKE '%e%'
  AND (o.name LIKE '%u%' OR o.name LIKE '%r%');
```
