# Hotel-reservation-analysis
###  1: Schema Verification & Documentation Audit
**Business Question:** Schau dir das ER-Diagramm aufmerksam an. Welche Tabellen hat die Datenbank? Wie sind sie miteinander verknüpft? Möglicherweise ist das ER-Diagramm nicht korrekt. (Examine the provided Entity-Relationship Diagram and execute an active database inventory check to verify whether the physical tables match the inherited documentation.)

```sql
SHOW TABLES;
```

* **Operational Context:** Before building pipelines or writing logic for front-desk software integrations, an analyst must confirm the database footprint. Running an active inventory check maps out the precise table names currently deployed in production, allowing us to find any missing tables or documentation gaps between the visual ER-Diagram and reality.

---

## 3. Structural Data Exploration & Schema Auditing

Before building internal integration workflows, I performed a structural schema audit to identify the precise table footprint and cross-check the system's inherited ER-Diagram for documentation gaps.

###  2: Physical Table Inventory Audit
**Business Question:** Überprüfe deine Antworten auf die letzte Frage mit SQL. Welche Tabellen gibt es in der Datenbank? (Verify the physical table footprint using SQL to confirm the active structures inside the hotel reservations database schema.)

```sql
SHOW TABLES;
```

*   **Execution Output:**
    *   `meal_plan`
    *   `reservation`
    *   `room_type`
*   **Business Takeaway:** The active system runs exactly 3 tables. This lean relational architecture isolates core hospitality layers: customer scheduling workflows (`reservation`), food service log metrics (`meal_plan`), and facility capacity configurations (`room_type`). Verifying these exact names ensures all automated background systems can establish accurate connection endpoints.
###  3: Attribute and Structural Association Audit (`DESC`)
**Business Question:** Welche Spalten haben die Tabellen und über welche Spalten sind die Tabellen miteinander verknüpft? (Examine the internal column names, database data types, and structural keys across all discovered tables to map the system's active relational links.)

```sql
DESC meal_plan;
DESC room_type;
DESC reservation;
```

*   **Key Schema Metadata Discovered:**
    *   `meal_plan`: Contains 4 columns. Unique entity identifier is `id` (Primary Key).
    *   `room_type`: Contains 4 columns. Unique entity identifier is `id` (Primary Key).
    *   `reservation`: Contains 17 extensive columns. Unique entity identifier is `id` (Primary Key). Contains two explicit foreign index keys (`MUL`): `meal_plan_id` and `room_type_id`.
*   **Business Takeaway:** This multi-table layout forms a star-like operational database schema. The central transaction ledger (`reservation`) relies on foreign indexing blocks to reference separate dimensions like meal options and physical room styles. Identifying these explicit key-join pathways is essential for configuring reliable query endpoints for front-desk and reservation tracking software systems.
###  4: Entity Volume and Row Count Audit
**Business Question:** Wie viele Zeilen haben die Tabellen jeweils? (Determine the total row volume across all three relational tables to establish a baseline data scale.)

```sql
SELECT COUNT(*) FROM meal_plan;
SELECT COUNT(*) FROM room_type;
SELECT COUNT(*) FROM reservation;
```

*   **Execution Volume Results:**
    *   `meal_plan`: 4 rows
    *   `room_type`: 7 rows
    *   `reservation`: 36,275 rows
*   **Business Takeaway:** This row distribution confirms standard relational warehouse characteristics. The small row counts in `meal_plan` (4) and `room_type` (7) indicate strict, fixed internal service options, while the large 36,275 historical rows in the `reservation` ledger provide a strong statistical dataset for running robust reservation trend analyses and analyzing customer cancellation behaviors.
###  5: Dimensional Data Profiling & Record Sample Extraction
**Business Question:** Wie sehen die Daten in den Tabellen aus? Schau dir die gesamten Tabellen meal_plan und room_type an. Lass dir für die Tabelle reservation 10 Zeilen ausgeben. (Perform visual record sampling across all tables to identify categorical variables and inspect structural row outputs.)

```sql
SELECT * FROM meal_plan;
SELECT * FROM room_type;
SELECT * FROM reservation LIMIT 10;
```

*   **Dimensional Property Findings:**
    *   `meal_plan`: Relies on strict binary logic arrays (`yes` / `no`) across breakfast, lunch, and dinner to define user food options.
    *   `room_type`: Groups features by bed specifications (`single`, `double`, `queen`), pricing layouts (`standard`, `deluxe`), and optional features (`studio`).
*   **Transactional Row Sample Preview (Top 2 Checked Operational Records):**
    *   Record 1: 2 Adults | 0 Children | 1 Weekend | 2 Weekdays | Meal ID: 1 | Lead: 224 days | Arrival: 2021-10-02 | Channel: Offline | Prior Cancellations: 0 | Rate: $65.00 | Special Requests: 0 | Status: Not Canceled
    *   Record 2: 1 Adult  | 2 Children | 2 Weekends | 3 Weekdays | Meal ID: 4 | Lead: 5 days   | Arrival: 2022-11-06 | Channel: Online  | Prior Cancellations: 0 | Rate: $106.68 | Special Requests: 1 | Status: Not Canceled

*   **Business Takeaway:** Visual inspection highlights the specific transaction metrics available. The `reservation` table tracks rich transactional records, including seasonal indicators (`no_of_weekend_nights`), lead windows (`lead_time`), pricing matrices (`avg_price_per_room`), and categorical flags (`booking_status`). This dataset provides a solid foundation for building the operational planning queries required by front-desk software systems.
###  6: Column Cardinality & Categorical Level Auditing
**Business Question:** Untersuche, welche unterschiedlichen Werte es in den Spalten `market_segment_type`, `required_car_parking_space` und `booking_status` gibt. (Identify all unique operational levels and classification categories present across key transactional fields to map operational variables.)

```sql
SELECT DISTINCT market_segment_type FROM reservation;
SELECT DISTINCT required_car_parking_space FROM reservation;
SELECT DISTINCT booking_status FROM reservation;
```

*   **Categorical Levels Discovered:**
    *   `market_segment_type`: Offline, Online, Corporate, Aviation, Complementary.
    *   `required_car_parking_space`: 0, 1 (Acts as a binary storage indicator for space allocation requests).
    *   `booking_status`: Not Canceled, Canceled.
*   **Business Takeaway:** Finding these distinct entries maps out the exact business channels driving revenue. The 5 unique levels in `market_segment_type` tell us that the hotel interacts with distinct B2B corporate segments (Corporate, Aviation) alongside standard public pipelines (Online, Offline). Isolating these categories provides marketing teams with clear parameters for running targeted, channel-specific sales promotions during system integration.
###  7: Temporal Range & Historical Window Auditing
**Business Question:** Von welchen Jahren stammen die Daten? Lass dir das erste Jahr der Reservierungen als first_year und das letzte Jahr der Reservierungen als last_year ausgeben. (Calculate the chronological boundaries of the reservation logs to establish the exact historical time window stored in the database.)

```sql
SELECT 
    MIN(YEAR(arrival_date)) AS first_year,
    MAX(YEAR(arrival_date)) AS last_year 
FROM reservation;
```

*   **Execution Output:**
    *   `first_year`: 2021
    *   `last_year`: 2022
*   **Business Takeaway:** Finding these year fields reveals that the dataset spans exactly a two-year performance cycle (2021 to 2022). This structural timeline provides a clean baseline for running comparative Year-over-Year (YoY) business growth assessments, calculating seasonal occupancy patterns, and identifying macro cancellation trends across successive corporate fiscal years.
---

## 4. Front-Desk Operational Logistics & Pipeline Engineering

This section focuses on engineering highly specific, production-ready query blocks designed to be integrated directly into the hotel's front-desk scheduling and room management software frameworks.

###  8: Total Transactional Inflow Audit (Target Date Tracking)
**Business Question:** Das Hotelpersonal möchte wissen, wie viele Reservierungen es insgesamt für den 12. Februar 2022 gibt. (Calculate the absolute volume of all booking transactions recorded for arrival on the specific date of February 12, 2022.)

```sql
SELECT 
    COUNT(*) AS total_bookings
FROM reservation
WHERE arrival_date = '2022-02-12';
```

*   **Execution Output:** 106 reservations.
*   **Business Takeaway:** Finding 106 total files gives us the absolute transactional footprint for that single day, helping logistics managers track baseline pipeline traffic.

###  9: Operational Capacity Filtering (Excluding Noise Constraints)
**Business Question:** Wie viele nicht stornierte Reservierungen gibt es am 12. Februar 2022? (Isolate active, non-canceled reservation volume for arrival on February 12, 2022, to provide real operational metrics for the hotel floor staff.)

```sql
SELECT 
    COUNT(*) AS active_bookings
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled';
```

*   **Execution Output:** 68 active reservations.
*   **Business Takeaway & Operational Insight:** Out of 106 gross filings, exactly 38 files reflect canceled transactions. Filtering out this noise down to 68 active check-ins gives front-desk managers an accurate baseline to organize active check-in flows without wasting resources on ghost records.

###  10: Granular Guest Volume Aggregations
**Business Question:** Wie viele erwachsene Personen reisen insgesamt am 12. Februar 2022 an? Nenne das Ergebnis `sum_adults`. (Calculate the absolute volume of adult guests scheduled to arrive on February 12, 2022, across all active reservations.)

```sql
SELECT 
    SUM(no_of_adults) AS sum_adults
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled';
```

*   **Execution Output:** 137.0 adults.
*   **Business Takeaway:** Provides house managers with precise baseline metrics to optimize staffing configurations, luggage logistics, and front-desk check-in reception windows.

###  11: Derived Headcount Metrics (Column-Arithmetic Aggregations)
**Business Question:** Wie viele Personen - also Erwachsene und Kinder - reisen insgesamt am 12. Februar 2022 an? Nenne das Ergebnis `sum_persons`. (Calculate the true overall customer headcount arriving on February 12, 2022, by running dynamic column arithmetic inside a consolidated mathematical aggregate block.)

```sql
SELECT 
    SUM(no_of_adults + no_of_children) AS sum_persons 
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled';
```

*   **Execution Output:** 148.0 persons.
*   **Business Takeaway:** By adding adult and child integers directly inside the `SUM()` clause, this script calculates the absolute human headcount (148 guests) moving through the building. This metric serves as a reliable core baseline value for facilities planning, catering provisioning, and risk-management compliance tracking.
*   ###  12: Facility Resource Allocation (Parking Infrastructure)
**Business Question:** Wie viele Parkplätze benötigen die Gäste, die am 12. Februar 2022 anreisen? (Quantify the total number of parking spaces requested by active guests arriving on February 12, 2022.)

```sql
SELECT 
    SUM(required_car_parking_space) AS car_parking
FROM reservation
WHERE arrival_date = '2022-02-12' 
  AND booking_status = 'Not Canceled';
```

* **Execution Output:** 2.0 spaces requested.
* **Business Takeaway:** Isolating parking demands for specific dates allows facility operations to manage lot capacities in advance. Since only two parking spaces are required for arrivals on February 12, 2022, logistics managers can confidently allocate the remaining parking inventory to walk-ins or special events without risking a resource bottleneck.
###  13: Time-Delta Asset Reservation Logistics
**Business Question:** Wie lange benötigen sie die Parkplätze jeweils? Schreibe eine Query, die die Gesamtanzahl an Nächten (no_of_week_nights und no_of_weekend_nights) für diese beiden Reservierungen ausgibt, und nenne das Ergebnis no_of_nights. (Calculate the exact length of stay in total nights for the specific customers requesting parking on February 12, 2022.)

```sql
SELECT 
    no_of_week_nights + no_of_weekend_nights AS no_of_nights
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled'
  AND required_car_parking_space = 1;
```

* **Execution Output:** 
  * 3
  * 3
* **Business Takeaway:** Both active reservations requiring vehicle storage stay for exactly 3 nights. This metrics-driven calculation allows front-desk supervisors to block parking space availability for those precise slots, avoiding scheduling conflicts and optimizing yard management.
###  14: Cross-Entity Attribute Identification (Meal Plan Lookups)
**Business Question:** Welche Mahlzeiten sind für die Reservierungen am 12. Februar 2022 gebucht, bei denen Kinder anreisen? Schreibe zunächst eine Query, die in der Tabelle reservation die meal_plan_id ausgibt. (Identify the specific meal plan identifiers tied to active family reservations arriving on February 12, 2022.)

```sql
SELECT 
    meal_plan_id
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled'
  AND no_of_children > 0;
```

* **Execution Output:** 8 rows returned, all showing `meal_plan_id = 1`.
* **Business Takeaway:** The query reveals a uniform consumer pattern: every family with children arriving on this target date has booked the exact same food service package (`meal_plan_id = 1`). Isolating this shared key allows the catering department to look up the exact menu details and prepare inventory efficiently for this customer group.
###  15: Dimensional Menu Feature Extraction
**Business Question:** Welche Mahlzeiten haben die Personen gebucht? Lass dir alle Spalten der Tabelle meal_plan mit der id = 1 ausgeben. (Resolve the actual service definitions for meal plan ID 1 by extracting all attributes from the master dimensional meal table.)

```sql
SELECT *
FROM meal_plan 
WHERE id = 1;
```

* **Execution Output:** 
  * `id`: 1 | `breakfast`: Yes | `lunch`: no | `dinner`: Yes
* **Business Takeaway:** Resolving the metadata confirms that Meal Plan 1 corresponds to a standard half-board service framework (Halbpension), covering breakfast and dinner while explicitly skipping lunch. By combining this dimension lookup with the transactional check from  14, kitchen and restaurant staff can immediately allocate inventory resources for morning and evening shifts on February 12, 2022, without expecting midday traffic from these families.
###  16: Cross-Entity Capacity Identification (Room Type Lookups)
**Business Question:** Welche Zimmertypen sind für die Reservierungen am 12. Februar 2022 gebucht, bei denen Kinder anreisen? Schreibe zunächst wieder eine Query, die in der Tabelle reservation die room_type_id abfragt. (Identify the specific room type identifiers tied to active family reservations arriving on February 12, 2022.)

```sql
SELECT 
    room_type_id
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled'
  AND no_of_children > 0;
```

* **Execution Output:** 8 rows returned, showing a mix of `room_type_id = 1` and `room_type_id = 2`.
* **Business Takeaway:** The query flags a diversified inventory demand among incoming family accounts on this specific date. Families are clustered exclusively around Room Types 1 and 2, which provides front-desk coordinators with the exact structural indices required to check master room configurations next.
###  17: Dimensional Room Feature Extraction (Array Set Logic)
**Business Question:** Schreibe eine Query für die Tabelle `room_type`. Frage alle Spalten ab und setze für die Zeilen die Bedingung, dass die id eine 1 oder eine 2 sein soll. (Resolve the physical bedding and layout properties for room type identifiers 1 and 2 using set logic to extract dimensional attributes.)

```sql
SELECT *
FROM room_type 
WHERE id IN (1, 2);
```

* **Execution Output:** 
  * `id`: 1 | `bed_type`: queen  | `layout`: standard | `studio`: no
  * `id`: 2 | `bed_type`: double | `layout`: standard | `studio`: no
* **Business Takeaway:** Resolving this layout mapping reveals that both target room types share a uniform standard footprint without studio expansions. The explicit commercial difference lies in bedding dimensions, where Type 1 implements queen-size configurations and Type 2 deploys traditional double beds. Automating this structural translation directly empowers front-desk check-in teams to streamline specific room allocations for incoming families.
###  18: Baseline Macro Attrition Volume Audit
**Business Question:** Wie viele stornierte Reservierungen beinhaltet die Datenbank? (Quantify the total absolute volume of lost cancellations stored within historical hotel ledger records to establish a performance benchmark.)

```sql
SELECT 
    COUNT(*) AS total_cancellations 
FROM reservation 
WHERE booking_status = 'Canceled';
```

* **Execution Output:** 11,885 total cancellations.
* **Business Takeaway:** Finding 11,885 canceled records out of the master ledger establishes a clear baseline metric for macro attrition tracking. This high volume of lost transactions provides a massive statistical foundation for running deep business diagnostics to analyze whether scheduling horizons (short-term vs. long-term booking lead times) significantly drive this churn behavior.
###  19: Operational Timeline Volatility Diagnostics (Lead-Time Exposure Analysis)
**Business Question:** Wie hoch ist die minimale, durchschnittliche und maximale Vorlaufzeit für Reservierungen? Schreibe diese Query in zwei Varianten: einmal mit der Bedingung für stornierte und einmal mit der Bedingung für nicht stornierte Reservierungen. (Measure the minimum, average, and maximum lead-time windows separating booking creation from arrival dates across canceled versus active reservation subsets.)

**Query 19.1 (Canceled Bookings Audit):**
```sql
SELECT 
    MIN(lead_time) AS min_lead_time, 
    AVG(lead_time) AS avg_lead_time,
    MAX(lead_time) AS max_lead_time
FROM reservation
WHERE booking_status = 'Canceled';
```

**Query 19.2 (Active Bookings Audit):**
```sql
SELECT 
    MIN(lead_time) AS min_lead_time, 
    AVG(lead_time) AS avg_lead_time,
    MAX(lead_time) AS max_lead_time
FROM reservation
WHERE booking_status = 'Not Canceled';
```

* **Execution Results Summary:**
  * Canceled Bookings: Min: 0 days | **Average: 139.2 days** | Max: 443 days
  * Active Bookings: Min: 0 days | **Average: 58.9 days** | Max: 386 days
* **Business Takeaway:** While both subsets contain short-term and long-term files, the average lead-time for storniert reservations is significantly longer (139 days vs. 59 days). Booking horizons that span long periods represent high revenue volatility and pipeline risk. This statistical data indicator justifies the implementation of adaptive corporate deposit frameworks or strict upfront cancellation fees for bookings created months in advance.

---

## 13. Data Licensing / Lizenzierung
This case study utilizes open-source hospitality operational records distributed transparently under the public **Creative Commons Attribution 4.0 International License (CC BY 4.0)**. 

Diese Fallstudie verwendet open-source Hotel-Performance-Daten, welche unter den rechtlichen Bedingungen der **Creative Commons Namensnennung 4.0 International Lizenz (CC BY 4.0)** bereitgestellt werden.
