# Skalare Subquery

```sql
/* 1) Teuerstes Buch (höchster Verkaufspreis) */
SELECT *
FROM   buecher b
WHERE  b.verkaufspreis = (SELECT MAX(verkaufspreis) FROM buecher);

/* 2) Billigstes Buch (niedrigster Verkaufspreis) */
SELECT *
FROM   buecher b
WHERE  b.verkaufspreis = (SELECT MIN(verkaufspreis) FROM buecher);

/* 3) Bücher, deren Einkaufspreis über dem Durchschnitt aller Bücher liegt */
SELECT *
FROM   buecher b
WHERE  b.einkaufspreis > (SELECT AVG(einkaufspreis) FROM buecher);

/* 4) Bücher, deren Einkaufspreis über dem Durchschnitts-Einkaufspreis der Thriller liegt */
SELECT DISTINCT b.*
FROM   buecher b
WHERE  b.einkaufspreis >
       ( SELECT AVG(b_in.einkaufspreis)
         FROM   buecher                b_in
         JOIN   buecher_has_sparten    bs_in ON b_in.buecher_id = bs_in.buecher_buecher_id
         JOIN   sparten                s_in  ON bs_in.sparten_sparten_id = s_in.sparten_id
         WHERE  s_in.bezeichnung = 'Thriller'
       );

/* 5) Thriller, deren Einkaufspreis über dem Durchschnitts-Einkaufspreis der Thriller liegt */
SELECT b.*
FROM   buecher              b
JOIN   buecher_has_sparten  bs ON b.buecher_id = bs.buecher_buecher_id
JOIN   sparten              s  ON bs.sparten_sparten_id = s.sparten_id
WHERE  s.bezeichnung = 'Thriller'
  AND  b.einkaufspreis >
       ( SELECT AVG(b_in.einkaufspreis)
         FROM   buecher                b_in
         JOIN   buecher_has_sparten    bs_in ON b_in.buecher_id = bs_in.buecher_buecher_id
         JOIN   sparten                s_in  ON bs_in.sparten_sparten_id = s_in.sparten_id
         WHERE  s_in.bezeichnung = 'Thriller'
       );

/* 6) Bücher mit überdurchschnittlichem Gewinn – Buch-ID 22 NICHT in der Durchschnitts­berechnung */
SELECT b.*,
       (b.verkaufspreis - b.einkaufspreis) AS gewinn
FROM   buecher b
WHERE  (b.verkaufspreis - b.einkaufspreis) >
       ( SELECT AVG(verkaufspreis - einkaufspreis)
         FROM   buecher
         WHERE  buecher_id <> 22
       );
```

# Subquery nach FROM

```sql
/* --- Unterabfrage: gewünschte Sparten + deren Ø-Einkaufspreis --- */
SELECT SUM(avg_einkaufspreis)   AS summe_avg_einkaufspreise
FROM (
        SELECT   s.sparten_id,
                 s.bezeichnung,
                 AVG(b.einkaufspreis) AS avg_einkaufspreis
        FROM     sparten              s
        JOIN     buecher_has_sparten  bs ON s.sparten_id = bs.sparten_sparten_id
        JOIN     buecher              b  ON bs.buecher_buecher_id = b.buecher_id
        GROUP BY s.sparten_id, s.bezeichnung
        HAVING   s.bezeichnung <> 'Humor'         -- Humor ausschließen
             AND AVG(b.einkaufspreis) > 10         -- nur > 10 €
     ) AS sparten_preise;


/* Liste zur Kontrolle – Name + #Publikationen -------------------- */
SELECT   vorname,
         nachname,
         anzahl_buecher
FROM (
        SELECT  a.autoren_id,
                a.vorname,
                a.nachname,
                COUNT(ab.buecher_buecher_id) AS anzahl_buecher
        FROM    autoren              a
        JOIN    autoren_has_buecher  ab ON a.autoren_id = ab.autoren_autoren_id
        GROUP BY a.autoren_id, a.vorname, a.nachname
        HAVING   COUNT(*) > 4
     ) AS bekannte_autoren;

/* Reine Zählung der bekannten Autor-innen ------------------------ */
SELECT COUNT(*) AS anzahl_bekannter_autoren
FROM (
        SELECT  a.autoren_id
        FROM    autoren              a
        JOIN    autoren_has_buecher  ab ON a.autoren_id = ab.autoren_autoren_id
        GROUP BY a.autoren_id
        HAVING  COUNT(*) > 4
     ) AS bekannte_autoren;

/* Unterabfrage: jeder Verlag + Ø-Gewinn pro Buch (< 10 €) -------- */
SELECT AVG(avg_gewinn)  AS gesamt_ø_gewinn_der_verlage
FROM (
        SELECT  v.verlage_id,
                v.name,
                AVG(b.verkaufspreis - b.einkaufspreis) AS avg_gewinn
        FROM    verlage v
        JOIN    buecher  b ON v.verlage_id = b.verlage_verlage_id
        GROUP BY v.verlage_id, v.name
        HAVING  avg_gewinn < 10                  -- „weniger als 10 € Gewinn“
     ) AS schwachverdiener;
```
