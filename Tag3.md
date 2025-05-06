# Aufgabe 1 Datentypen

| Datentyp                                    | MariaDB (MySQL)                                                      | Beispiel                                             | Bemerkung / Einstellungen                                    |
|---------------------------------------------|----------------------------------------------------------------------|------------------------------------------------------|--------------------------------------------------------------|
| **Ganze Zahlen**                            | `TINYINT`, `SMALLINT`, `MEDIUMINT`, `INT`, `BIGINT`                  | `INT`                                                | Vorzeichenbehaftet (–2³¹…2³¹–1), mit `UNSIGNED` optional      |
| **Natürliche Zahlen**                       | dieselben Integer-Typen mit `UNSIGNED`                               | `INT UNSIGNED`                                       | Nur nicht-negative Werte (0…2³²–1)                           |
| **Festkommazahlen (Dezimalzahlen)**         | `DECIMAL(M,D)`                                                       | `DECIMAL(10,2)` → 12345678.90                        | Genaue (keine Rundungsfehler), M=Gesamtstellen, D=Nachkommastellen |
| **Aufzählungstypen**                        | `ENUM('Wert1','Wert2',…)`                                            | `ENUM('rot','grün','blau')`                          | Bis zu 65 535 Werte, intern als Integer abgespeichert         |
| **Boolean (logische Werte)**                | `BOOLEAN` (Alias für `TINYINT(1)`)                                   | `BOOLEAN`                                            | 0=false, ≠0=true; `TRUE`/`FALSE` Keywords                    |
| **Zeichen (einzelnes Zeichen)**             | `CHAR(1)`                                                            | `CHAR(1)`                                            | Fixe Länge 1 Byte, wird rechts mit Leerzeichen aufgefüllt    |
| **Gleitkommazahlen**                        | `FLOAT`, `DOUBLE` (bzw. `FLOAT(p)`)                                   | `DOUBLE`                                             | Approximate; `FLOAT(p)` erlaubt Präzisionsangabe              |
| **Zeichenkette fester Länge**               | `CHAR(n)`                                                            | `CHAR(50)`                                           | Fixe Byte-Länge n, rechte Padding mit Leerzeichen            |
| **Zeichenkette variabler Länge**            | `VARCHAR(n)`, `TINYTEXT`, `TEXT`, `MEDIUMTEXT`, `LONGTEXT`           | `VARCHAR(255)`                                       | Variable Länge ≤ n; TEXT-Typen bis 4 GB                      |
| **Datum und/oder Zeit**                     | `DATE`, `TIME`, `DATETIME`, `YEAR`                                   | `DATETIME`                                           | `DATE` (YYYY-MM-DD), `TIME` (HH:MM:SS), `DATETIME` kombiniert |
| **Zeitstempel**                             | `TIMESTAMP`                                                          | `TIMESTAMP DEFAULT CURRENT_TIMESTAMP`                | Speichert UTC, kann automatisch gesetzt/aktualisiert werden  |
| **Binäre Datenobjekte variabler Länge**     | `BLOB`, `TINYBLOB`, `MEDIUMBLOB`, `LONGBLOB`, `VARBINARY(n)`         | `BLOB`                                               | Für Binärdaten (Bilder, Dateien); TEXT-Äquivalente für Binär |
| **Verbund (Record/Struct)**                 | **nicht nativ unterstützt**                                          | —                                                    | Mehrere Spalten oder `JSON` statt komplexer Verbundtypen      |
| **JSON**                                    | `JSON`                                                               | `JSON`                                               | Speichert validiertes JSON-Dokument; unterstützt Indizes auf Pfade |

# Aufgabe 2 SELECT

```sql
-- a) Alle Datensätze der DVD-Sammlung ausgeben
SELECT *
FROM dvd_sammlung;

-- b) Filmtitel und zugehörige Nummer ausgeben
SELECT film, nummer
FROM dvd_sammlung;

-- c) Filmtitel und zugehörigen Regisseur ausgeben
SELECT film, regisseur
FROM dvd_sammlung;

-- d) Liste aller Filme von Quentin Tarantino
SELECT *
FROM dvd_sammlung
WHERE regisseur = 'Quentin Tarantino';

-- e) Liste aller Filme von Steven Spielberg
SELECT *
FROM dvd_sammlung
WHERE regisseur = 'Steven Spielberg';

-- f) Liste aller Filme, in denen der Regisseur den Vornamen "Steven" hat
SELECT *
FROM dvd_sammlung
WHERE regisseur LIKE 'Steven %';

-- g) Liste aller Filme, die länger als 2 Stunden sind (120 Minuten)
SELECT *
FROM dvd_sammlung
WHERE laenge_minuten > 120;

-- h) Liste aller Filme, die von Tarantino oder Spielberg gedreht wurden
SELECT *
FROM dvd_sammlung
WHERE regisseur IN ('Quentin Tarantino', 'Steven Spielberg');

-- i) Filme von Tarantino, die kürzer als 90 Minuten sind
SELECT *
FROM dvd_sammlung
WHERE regisseur = 'Quentin Tarantino'
  AND laenge_minuten < 90;

-- j) Film suchen, in dessen Titel "Sibirien" vorkommt
SELECT *
FROM dvd_sammlung
WHERE film LIKE '%Sibirien%';

-- k) Alle Teile von "Das grosse Rennen" ausgeben
SELECT *
FROM dvd_sammlung
WHERE film LIKE 'Das grosse Rennen%';

-- l) Alle Filme sortiert nach Regisseur
SELECT *
FROM dvd_sammlung
ORDER BY regisseur;

-- m) Alle Filme sortiert nach Regisseur, dann nach Filmtitel
SELECT *
FROM dvd_sammlung
ORDER BY regisseur, film;

-- n) Alle Filme von Tarantino, die längsten zuerst
SELECT *
FROM dvd_sammlung
WHERE regisseur = 'Quentin Tarantino'
ORDER BY laenge_minuten DESC;
```

# Aufgabe 3 INSERT

```sql
-- ========================================================
-- 1. Aufgabe: Neue Kunden anlegen
-- ========================================================

-- a) Kurzform: Heinrich Schmitt aus Zürich, Schweiz (land_id = 2)
INSERT INTO kunden
VALUES (NULL, 'Heinrich', 'Schmitt', 2, 'Zürich');

-- b) Kurzform: Sabine Müller aus Bern, Schweiz (land_id = 2)
INSERT INTO kunden
VALUES (NULL, 'Sabine', 'Müller', 2, 'Bern');

-- c) Kurzform: Markus Mustermann aus Wien, Österreich (land_id = 1)
INSERT INTO kunden
VALUES (NULL, 'Markus', 'Mustermann', 1, 'Wien');


-- d) Langform: Herr Maier (nur Vorname, Nachname)
INSERT INTO kunden (vorname, nachname)
VALUES ('Herr', 'Maier');

-- e) Langform: Herr Bulgur aus Sirnach (land_id = 2)
INSERT INTO kunden (vorname, nachname, land_id, wohnort)
VALUES ('Herr', 'Bulgur', 2, 'Sirnach');

-- f) Langform: Maria Manta (nur Vorname, Nachname)
INSERT INTO kunden (vorname, nachname)
VALUES ('Maria', 'Manta');


-- ========================================================
-- 2. Fehlerhafte INSERTs: Korrekturen
-- ========================================================

-- a) Fehlend: Tabellenname
--   INSERT INTO (nachname, wohnort, land_id) VALUES (...);
-- ✔ Korrekt:
INSERT INTO kunden (nachname, wohnort, land_id)
VALUES ('Fesenkampp', 'Duisburg', 3);

-- b) Falsche Anführungszeichen um Spaltenname
--   INSERT INTO kunden ('vorname') VALUES ('Herbert');
-- ✔ Korrekt:
INSERT INTO kunden (vorname)
VALUES ('Herbert');

-- c) land_id muss numerisch sein, nicht 'Deutschland'
--   INSERT INTO kunden (nachname, vorname, wohnort, land_id)
--   VALUES ('Schulter','Albert','Duisburg','Deutschland');
-- ✔ Korrekt (z. B. land_id=4 für Deutschland):
INSERT INTO kunden (nachname, vorname, wohnort, land_id)
VALUES ('Schulter','Albert','Duisburg', 4);

-- d) Ungültige Syntax, keine Spalten, falsche VALUES-Anzahl
--   INSERT INTO kunden ('', 'Brunhild', 'Sulcher', 1, 'Süderstade');
-- ✔ Korrekt (langform):
INSERT INTO kunden (vorname, nachname, land_id, wohnort)
VALUES ('Brunhild','Sulcher', 1,'Süderstade');

-- e) VALUES ohne Spaltenliste, nur 4 Werte statt 5
--   INSERT INTO kunden VALUES ('Jochen','Schmied',2,'Solingen');
-- ✔ Korrekt:
INSERT INTO kunden
VALUES (NULL,'Jochen','Schmied', 2,'Solingen');

-- f) VALUES ohne Spaltenliste, leere Strings
--   INSERT INTO kunden VALUES ('','Doppelbrecher',2,'');
-- ✔ Korrekt (NULL für Auto-ID, NULL für unbekannte Spalte):
INSERT INTO kunden
VALUES (NULL,'','Doppelbrecher', 2, NULL);

-- g) Falsche Anzahl Werte vs. Spalten
--   INSERT INTO kunden (nachname, wohnort, land_id)
--   VALUES ('Christoph','Fesenkampp','Duisburg',3);
-- ✔ Korrekt: entweder Spalte film weglassen oder Werte matchen
INSERT INTO kunden (nachname, wohnort, land_id)
VALUES ('Christoph','Fesenkampp',3);

-- h) Dieser Eintrag ist syntaktisch korrekt, fügt nur Vorname ein:
INSERT INTO kunden (vorname)
VALUES ('Herbert');

-- i) Fehlende Anführungszeichen um Literale
--   INSERT INTO kunden (nachname, vorname, wohnort, land_id)
--   VALUES (Schulter, Albert, Duisburg, 1);
-- ✔ Korrekt:
INSERT INTO kunden (nachname, vorname, wohnort, land_id)
VALUES ('Schulter','Albert','Duisburg',1);

-- j) Falsches Schlüsselwort VALUE statt VALUES
--   INSERT INTO kunden VALUE ('', "Brunhild", "Sulcher", 1, "Süderstade");
-- ✔ Korrekt:
INSERT INTO kunden
VALUES (NULL,'Brunhild','Sulcher',1,'Süderstade');

-- k) Gleiches wie j), plus unquoted String
--   INSERT INTO kunden VALUE ('','Jochen','Schmied',2,Solingen);
-- ✔ Korrekt:
INSERT INTO kunden
VALUES (NULL,'Jochen','Schmied',2,'Solingen');
```
# Aufgabe 4 UPDATE/DELETE/ALTER/DROP

```sql
-- ############################################################
-- Skript: filmeDatenbank_management.sql
-- Zweck: vollständiges, idempotentes Setup und Pflege
-- ############################################################

-- 1) Schema sicherstellen
CREATE SCHEMA IF NOT EXISTS `filmeDatenbank`
  DEFAULT CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
USE `filmeDatenbank`;

-- 2) Tabelle anlegen (falls nicht vorhanden)
CREATE TABLE IF NOT EXISTS `dvd_sammlung` (
  `id`             INT(11)      NOT NULL AUTO_INCREMENT,
  `film`           VARCHAR(255) NOT NULL,
  `nummer`         INT(11)      NOT NULL,
  `laenge_minuten` INT(11)      NOT NULL,
  `regisseur`      VARCHAR(255) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_nummer` (`nummer`)
) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4;

-- 3) Ausgangsdaten einfügen (UPSERT für Idempotenz)
INSERT INTO `dvd_sammlung` (`id`, `film`, `nummer`, `laenge_minuten`, `regisseur`)
VALUES
  (1,  'Meine Grossmutter lacht nie',             1, 119, 'Quentin Tarantino'),
  (2,  'Angst',                                   2,  92, 'Steven Spielberg'),
  (3,  'Wenn ich nur könnte',                    3,  89, 'Quentin Tarantino'),
  (4,  'Men and Mice',                            4,  88, 'Cohen'),
  (6,  'Grün ist die Farbe der Liebe',            5, 201, 'Quentin Tarantino'),
  (7,  'Frühstück in Sibirien',                   6,  72, 'Steven Spielberg'),
  (8,  'Das grosse Rennen',                        8,  83, 'Cohen'),
  (9,  'Das grosse Rennen, Teil 2',                9,  85, 'Cohen'),
  (10, 'Adlatus',                                 7, 131, 'Quentin Tarantino'),
  (11, 'Angriff auf Rom',                        10, 138, 'Steven Burghofer')
ON DUPLICATE KEY UPDATE
  `film`           = VALUES(`film`),
  `nummer`         = VALUES(`nummer`),
  `laenge_minuten` = VALUES(`laenge_minuten`),
  `regisseur`      = VALUES(`regisseur`);

-- 4) Korrekturen und Erweiterungen

-- 4.1 Regisseur "Cohen" vervollständigen
UPDATE `dvd_sammlung`
SET   `regisseur` = 'Etan Cohen'
WHERE `regisseur` = 'Cohen';

-- 4.2 Filmlänge für "Angst" korrigieren
UPDATE `dvd_sammlung`
SET   `laenge_minuten` = 120
WHERE `film` = 'Angst';

-- 4.3 Tabelle umbenennen: dvd_sammlung → bluray_sammlung
DROP TABLE IF EXISTS `bluray_sammlung`;
RENAME TABLE `dvd_sammlung` TO `bluray_sammlung`;

-- 4.4 Neue Spalte "Preis" hinzufügen
ALTER TABLE `bluray_sammlung`
  ADD COLUMN IF NOT EXISTS `Preis` DECIMAL(10,2) AFTER `laenge_minuten`;

-- 4.5 Entfernen des Films "Angriff auf Rom"
DELETE FROM `bluray_sammlung`
WHERE `kinofilme` = 'Angriff auf Rom'  -- wird gleich umbenannt, siehe 4.7
   OR `film` = 'Angriff auf Rom';

-- 4.6 Spalte "film" in "kinofilme" umbenennen
ALTER TABLE `bluray_sammlung`
  CHANGE COLUMN `film` `kinofilme` VARCHAR(255) NOT NULL;

-- 4.7 Spalte "nummer" entfernen
ALTER TABLE `bluray_sammlung`
  DROP COLUMN IF EXISTS `nummer`;

-- 5) Rückbau: gesamte Tabelle löschen
DROP TABLE IF EXISTS `bluray_sammlung`;
```

