# hotel-reservation-analysis
### Aufgabe 1: Schema Verification & Documentation Audit
**Business Question:** Schau dir das ER-Diagramm aufmerksam an. Welche Tabellen hat die Datenbank? Wie sind sie miteinander verknüpft? Möglicherweise ist das ER-Diagramm nicht korrekt. (Examine the provided Entity-Relationship Diagram and execute an active database inventory check to verify whether the physical tables match the inherited documentation.)

```sql
SHOW TABLES;
```

* **Operational Context:** Before building pipelines or writing logic for front-desk software integrations, an analyst must confirm the database footprint. Running an active inventory check maps out the precise table names currently deployed in production, allowing us to find any missing tables or documentation gaps between the visual ER-Diagram and reality.
