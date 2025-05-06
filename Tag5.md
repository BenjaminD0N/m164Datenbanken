# Referenzielle Datenintegrität

## Aufgabe 1 – Warum kann man Datensätze nicht einfach löschen?

**Was passiert, wenn man einen verknüpften Datensatz löscht?** 
In anderen Tabellen verbleiben **verwaiste Fremdschlüssel** – die Datenbank wäre inkonsistent, Abfragen würden falsche Resultate liefern. 

**Wer verhindert das?** 
Das **DBMS** selbst. Jede `FOREIGN KEY`-Constraint erzwingt bei `INSERT`, `UPDATE`, `DELETE` die **referentielle Integrität (RI)** und bricht den Vorgang bei Verstoß mit Fehler 1451/1452 ab.

---

## Aufgabe 2 – „4000 Basel“ durch „3000 Bern“ ersetzen

### 2.1 Was passiert beim direkten Löschen?

~~~sql
DELETE FROM tbl_orte
WHERE PLZ = '4000' AND Ortsbezeichnung = 'Basel';
/*  Error 1451: Cannot delete or update a parent row:
    a foreign key constraint fails
    (tourenplaner.tbl_stationen …
     FOREIGN KEY (FS_ID_Ort) REFERENCES tbl_orte (ID_Ort))           */
~~~

`tbl_stationen` (und evtl. weitere Tabellen) verweisen mit `FS_ID_Ort = 5` auf diesen Datensatz.  
MySQL lässt das Löschen wegen **ON DELETE RESTRICT** nicht zu.

---

### 2.2 Variante 1 (empfohlen) – Datensatz *aktualisieren*

~~~sql
UPDATE tbl_orte
SET    PLZ             = '3000',
       Ortsbezeichnung = 'Bern'
WHERE  ID_Ort = 5;        -- 5 = bisher „4000 Basel“
~~~

Alle Fremdschlüssel behalten dieselbe `ID_Ort`; damit ist sofort überall „3000 Bern“ hinterlegt.

---

### 2.3 Variante 2 – Neuen Ort anlegen, Verweise umhängen, alten Ort löschen  

*(alles in **einer Transaktion** ausführen)*

~~~sql
START TRANSACTION;

-- 1) neuen Ort anlegen
INSERT INTO tbl_orte (PLZ, Ortsbezeichnung)
VALUES ('3000', 'Bern');
SET @idBern = LAST_INSERT_ID();

-- 2) Fremdschlüssel auf neue ID umstellen
UPDATE tbl_stationen
SET    FS_ID_Ort = @idBern
WHERE  FS_ID_Ort = 5;

-- ggf. weitere Tabellen mit FK auf tbl_orte ebenfalls updaten …

-- 3) alten Ort löschen
DELETE FROM tbl_orte
WHERE  ID_Ort = 5;

COMMIT;
~~~

---

### 2.4 Warum **nicht** einfach `ON DELETE CASCADE`?

Bei `CASCADE` würde das Löschen von „Basel“ **alle zugehörigen Stationen (und damit Teile der Fahrten-Historie)** entfernen – in der Regel unerwünscht.  
Darum nutzt man hier bewusst `RESTRICT | NO ACTION`.

---

# Aggregatsfunktionen 

## 1 Niedrigstes / höchstes Lehrer-Gehalt
```sql
SELECT 
    MIN(gehalt) AS niedrigstes_gehalt,
    MAX(gehalt) AS hoechstes_gehalt
FROM lehrer;
```

## 2 Niedrigstes Gehalt eines Mathe-Lehrers
```sql
SELECT 
    MIN(l.gehalt) AS niedrigstes_mathe_gehalt
FROM lehrer             AS l
JOIN lehrer_hat_faecher AS lf ON l.id = lf.idLehrer
JOIN faecher            AS f  ON lf.idFaecher = f.id
WHERE f.fachbezeichnung = 'Mathe';
```

## 3 Bester Ø aus Deutsch & Mathe  
*(wenn **1** beste Note ⇒ MIN, wenn **6** beste ⇒ MAX nutzen)*
```sql
-- Variante „1 ist beste Note“
SELECT MIN((noteDeutsch + noteMathe) / 2) AS bester_durchschnitt
FROM schueler;

-- Variante „6 ist beste Note“
SELECT MAX((noteDeutsch + noteMathe) / 2) AS bester_durchschnitt
FROM schueler;
```

## 4 Höchste / niedrigste Einwohnerzahl eines Ortes
```sql
SELECT
    MAX(einwohnerzahl) AS hoechste_einwohnerzahl,
    MIN(einwohnerzahl) AS niedrigste_einwohnerzahl
FROM orte;
```

## 5 Differenz zwischen größtem und kleinstem Ort
```sql
SELECT
    MAX(einwohnerzahl) - MIN(einwohnerzahl) AS differenz
FROM orte;
```

## 6 Gesamtzahl Schüler
```sql
SELECT COUNT(*) AS anzahl_schueler
FROM schueler;
```

## 7 Schüler **mit** Smartphone
```sql
SELECT COUNT(*) AS schueler_mit_smartphone
FROM schueler
WHERE idSmartphones IS NOT NULL;
```

## 8 Schüler mit Smartphone **Samsung** oder **HTC**
```sql
SELECT COUNT(*) AS samsung_oder_htc
FROM schueler  AS s
JOIN smartphones AS sp ON s.idSmartphones = sp.id
WHERE sp.marke IN ('Samsung','HTC');
```

## 9 Schüler, die in **Waldkirch** wohnen
```sql
SELECT COUNT(*) AS schueler_in_waldkirch
FROM schueler AS s
JOIN orte     AS o ON s.idOrte = o.id
WHERE o.name = 'Waldkirch';
```

## 10 Schüler bei **Herrn Bohnert** aus **Emmendingen**
```sql
SELECT COUNT(*) AS schueler_bohnert_emmendingen
FROM schueler            AS s
JOIN lehrer_hat_schueler AS ls ON s.id = ls.idSchueler
JOIN lehrer              AS l  ON ls.idLehrer = l.id
JOIN orte                AS o  ON s.idOrte    = o.id
WHERE l.name = 'Bohnert'
  AND o.name = 'Emmendingen';
```

## 11 Schüler-Anzahl von **Frau Zelawat**
```sql
SELECT COUNT(DISTINCT ls.idSchueler) AS schueler_zelawat
FROM lehrer_hat_schueler AS ls
JOIN lehrer              AS l ON ls.idLehrer = l.id
WHERE l.name = 'Zelawat';
```

## 12 Russische Schüler bei Frau Zelawat
```sql
SELECT COUNT(DISTINCT s.id) AS russisch_zelawat
FROM schueler            AS s
JOIN lehrer_hat_schueler AS ls ON s.id = ls.idSchueler
JOIN lehrer              AS l  ON ls.idLehrer = l.id
WHERE l.name = 'Zelawat'
  AND s.nationalitaet = 'RU';
```

## 13 Lehrer mit höchstem Gehalt  
*(liefert alle, falls mehrere identischen Max-Wert haben)*
```sql
SELECT name, gehalt
FROM lehrer
WHERE gehalt = (SELECT MAX(gehalt) FROM lehrer);
```

# GROUP  

## 1  Anzahl aller Schüler nach Nationalität  
*(Spalten: Anzahl, Nationalität)*  
```sql
SELECT
    COUNT(*)      AS Anzahl,
    nationalitaet AS Nationalität
FROM schueler
GROUP BY nationalitaet;
```

## 2  Schüler pro Ort  
*(Spalten: Ort, Anzahl der Schüler – absteigend sortiert)*  
```sql
SELECT
    o.name                         AS Ort,
    COUNT(*)                       AS `Anzahl der Schüler`
FROM schueler AS s
JOIN orte     AS o ON s.idOrte = o.id
GROUP BY o.name
ORDER BY `Anzahl der Schüler` DESC;
```

## 3  Anzahl Lehrer pro Fach  
```sql
SELECT
    f.fachbezeichnung AS Fachbezeichnung,
    COUNT(*)          AS Anzahl
FROM lehrer_hat_faecher AS lf
JOIN faecher           AS f  ON lf.idFaecher = f.id
WHERE lf.idFaecher <> 0   -- Datensatz (0,0) ausschließen
GROUP BY f.id, f.fachbezeichnung
ORDER BY Anzahl DESC;
```

## 4  Lehrerliste pro Fach  
*(eine Zeile pro Fach, sortiert nach Anzahl Lehrer absteigend)*  
```sql
SELECT
    f.fachbezeichnung AS Fachbezeichnung,
    GROUP_CONCAT(DISTINCT l.name ORDER BY l.name SEPARATOR ', ') AS Lehrerliste
FROM lehrer_hat_faecher AS lf
JOIN faecher           AS f  ON lf.idFaecher = f.id
JOIN lehrer            AS l  ON lf.idLehrer  = l.id
WHERE lf.idFaecher <> 0
GROUP BY f.id, f.fachbezeichnung
ORDER BY COUNT(DISTINCT l.id) DESC;
```

## 5  Schüler – zugehörige Lehrer und Fächer  
*(Spalten: Schülername, Lehrer, Fächer)*  
```sql
SELECT
    s.name AS Schülername,
    GROUP_CONCAT(DISTINCT l.name ORDER BY l.name SEPARATOR ', ')      AS Lehrer,
    GROUP_CONCAT(DISTINCT f.fachbezeichnung ORDER BY f.fachbezeichnung SEPARATOR ', ') AS Fächer
FROM schueler             AS s
JOIN lehrer_hat_schueler  AS ls ON s.id = ls.idSchueler
JOIN lehrer               AS l  ON ls.idLehrer = l.id
JOIN lehrer_hat_faecher   AS lf ON l.id        = lf.idLehrer
JOIN faecher              AS f  ON lf.idFaecher = f.id
WHERE lf.idFaecher <> 0
GROUP BY s.id, s.name
ORDER BY s.name;
```

# ORDER 

## 1  Schafe im Jahr 2018 (absteigend nach Gebiet)
```sql
SELECT *
FROM   nutztiere
WHERE  tierart LIKE 'Schafe%'   -- nur Schafe
  AND  jahr = 2018              -- Jahr 2018
ORDER BY gebiet_name DESC;
```

## 2  Gesamtzahl Schafe im Jahr 2018
```sql
SELECT SUM(anzahl) AS total_schafe_2018
FROM   nutztiere
WHERE  tierart LIKE 'Schafe%'
  AND  jahr = 2018;
```

## 3  Durchschnittliche Kühe in **Region Zürich** (alle Jahre)
```sql
SELECT AVG(anzahl) AS avg_kuehe_zuerich
FROM   nutztiere
WHERE  tierart LIKE 'Kühe%'
  AND  gebiet_name = 'Region Zürich';
```

## 4  Höchste Kühe-Anzahl in **Region Zürich**
```sql
SELECT MAX(anzahl) AS max_kuehe_zuerich
FROM   nutztiere
WHERE  tierart LIKE 'Kühe%'
  AND  gebiet_name = 'Region Zürich';
```

## 5  Niedrigste Kühe-Anzahl in **Region Zürich**
```sql
SELECT MIN(anzahl) AS min_kuehe_zuerich
FROM   nutztiere
WHERE  tierart LIKE 'Kühe%'
  AND  gebiet_name = 'Region Zürich';
```

## 6  Totale Nutztiere pro Region im Jahr 2016
```sql
SELECT
    gebiet_name,
    SUM(anzahl) AS total_2016
FROM   nutztiere
WHERE  jahr = 2016
GROUP BY gebiet_name
ORDER BY total_2016 DESC;
```

## 7  Totale Nutztiere **pro Region & Jahr**
```sql
SELECT
    gebiet_name,
    jahr,
    SUM(anzahl) AS total_nutztiere
FROM   nutztiere
GROUP BY gebiet_name, jahr
ORDER BY gebiet_name, jahr;
```

## 8  Totale Nutztiere pro Region & Jahr (nach Jahr sortiert)
```sql
SELECT
    gebiet_name,
    jahr,
    SUM(anzahl) AS total_nutztiere
FROM   nutztiere
GROUP BY gebiet_name, jahr
ORDER BY jahr, gebiet_name;
```

## 9  Totale Nutztiere pro Region & Jahr **ab 2015** (nach Jahr sortiert)
```sql
SELECT
    gebiet_name,
    jahr,
    SUM(anzahl) AS total_nutztiere
FROM   nutztiere
WHERE  jahr >= 2015
GROUP BY gebiet_name, jahr
ORDER BY jahr, gebiet_name;
```

# HAVING 

## 1  Durchschnittsnoten < 4  
```sql
SELECT
    name                     AS Schülername,
    (noteDeutsch + noteMathe) / 2.0 AS Durchschnittsnote
FROM schueler
WHERE (noteDeutsch + noteMathe) / 2.0 < 4;
```

## 2  Durchschnitt (gerundet & sortiert)  
```sql
SELECT
    name                                                AS Schülername,
    ROUND((noteDeutsch + noteMathe) / 2.0, 1)           AS Durchschnittsnote
FROM schueler
HAVING Durchschnittsnote < 4
ORDER BY Durchschnittsnote ASC;
```

## 3  Lehrer mit Nettogehalt > 3000 €  
```sql
SELECT
    name,
    ROUND(gehalt * 0.70, 2) AS Nettogehalt
FROM lehrer
WHERE gehalt * 0.70 > 3000;
```

## 4  Klassenzimmer mit < 10 Schülern  
```sql
SELECT
    klassenzimmer            AS Klassenzimmer,
    COUNT(*)                 AS Anzahl
FROM schueler
GROUP BY klassenzimmer
HAVING COUNT(*) < 10
ORDER BY klassenzimmer;
```

## 5a  Russische Schüler nach Ort  
```sql
SELECT
    o.name          AS `Ort-Name`,
    COUNT(*)        AS Anzahl
FROM schueler AS s
JOIN orte     AS o ON s.idOrte = o.id
WHERE s.nationalitaet = 'RU'
GROUP BY o.id, o.name
ORDER BY o.name;
```

## 5b  …nur Orte mit ≥ 10 russischen Schülern  
```sql
SELECT
    o.name   AS `Ort-Name`,
    COUNT(*) AS Anzahl
FROM schueler AS s
JOIN orte     AS o ON s.idOrte = o.id
WHERE s.nationalitaet = 'RU'
GROUP BY o.id, o.name
HAVING COUNT(*) >= 10
ORDER BY o.name;
```

