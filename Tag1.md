# Aufgabe 1 

## Auftrag 1 
![image](https://github.com/user-attachments/assets/450978a4-85bc-4a99-a0ac-b94669bc0a36)

## Auftrag 2 
![Aufgabe_1_TourenplanerWorkbench](https://github.com/user-attachments/assets/b0de7785-68b3-4afa-934e-192d94639e87)

# Aufgabe 2

## 1. Zusammenfassung Datenmodellierung

- *Zweck:* Strukturierung der zu verarbeitenden und zu speichernden Daten in Informationssystemen  
- *Phasen der Datenmodellierung:*  
  - *Konzeptionelles Datenmodell (ERM):* 
    - Fachliche Abbildung der Anforderungen  
    - Implementierungs‐unabhängig  
  - *Logisches Datenmodell (ERD):* 
    - Abbildung auf das Ziel‐Datenbanksystem (z. B. relational)  
    - Definition von Primär­schlüsseln (PK), Fremd­schlüsseln (FK) und Datentypen  
  - *Physisches Datenmodell:* 
    - DDL-Skripte (CREATE TABLE, …)  
    - Indizes, Partitionierung, produktspezifische Optimierungen  

- *Modellierungstypen im Data-Warehouse-Umfeld:*  
  - *3NF-Modellierung:* Operative Systeme, Core/Enterprise DWH, anwendungsneutrale Speicherung  
  - *Star Schema:* Reporting Layer/Data Mart, optimiert für Auswertungen  
  - *Data Vault:* Core DWH/Integration Layer, Fokus auf Erweiterbarkeit und automatisierte Ladung  

---

## 2. Normalisierungsschritte

1. *1. Normalform (1NF)*
   - Entfernen mehrfach auftretender Gruppen  
   - Alle Attribute haben atomare (unteilbare) Werte  

2. *2. Normalform (2NF)*  
   - Voraussetzung: 1NF erfüllt  
   - Keine partiellen Abhängigkeiten bei zusammengesetztem Primärschlüssel  

3. *3. Normalform (3NF)*  
   - Voraussetzung: 2NF erfüllt  
   - Keine transitiven Abhängigkeiten (kein Nicht-Schlüsselattribut von einem anderen Nicht-Schlüsselattribut abhängig)  

4. *Weiterführende Formen (optional):* BCNF, 4NF, 5NF
