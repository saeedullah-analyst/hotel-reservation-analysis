# Hotel Reservation Analysis

Operational data audit | MySQL | Hospitality | Cancellation Behavior

---

This is a training project using a hotel reservations dataset. The scenario: a hotel needs its reservation database audited and queried to support front-desk operations, resource planning, and cancellation analysis. The database is small — three tables — but the reservation ledger contains 36,275 rows covering two fiscal years.

---

## Technical Stack

| Layer | Detail |
|---|---|
| Database | MySQL |
| Concepts | Schema inspection, aggregations, date filtering, column arithmetic, dimensional lookups, IS NULL, DISTINCT, multi-condition WHERE |

---

## Database Schema

```
meal_plan ◄── reservation ──► room_type
```

| Table | Rows | Role |
|---|---|---|
| reservation | 36,275 | Central transaction ledger |
| meal_plan | 4 | Food service options |
| room_type | 7 | Room configurations |

The `reservation` table references both `meal_plan` and `room_type` via foreign keys (`meal_plan_id`, `room_type_id`).

---

## Part 1 — Schema Verification

```sql
SHOW TABLES;
-- meal_plan, reservation, room_type

DESC meal_plan;
DESC room_type;
DESC reservation;
```

Key findings from `DESC reservation`: 17 columns, one primary key (`id`), and two foreign index keys (`meal_plan_id`, `room_type_id`). The schema matches a standard star layout — a central fact table joined to smaller dimension tables.

```sql
SELECT COUNT(*) FROM meal_plan;    -- 4 rows
SELECT COUNT(*) FROM room_type;    -- 7 rows
SELECT COUNT(*) FROM reservation;  -- 36,275 rows
```

The small row counts in `meal_plan` and `room_type` indicate fixed service options rather than dynamic entries. All the transactional volume lives in `reservation`.

---

## Part 2 — Categorical and Temporal Profiling

```sql
SELECT DISTINCT market_segment_type FROM reservation;
SELECT DISTINCT required_car_parking_space FROM reservation;
SELECT DISTINCT booking_status FROM reservation;
```

| Field | Values |
|---|---|
| market_segment_type | Offline, Online, Corporate, Aviation, Complementary |
| required_car_parking_space | 0, 1 |
| booking_status | Not Canceled, Canceled |

```sql
SELECT
    MIN(YEAR(arrival_date)) AS first_year,
    MAX(YEAR(arrival_date)) AS last_year
FROM reservation;
-- 2021 to 2022
```

The dataset spans two full years, which is enough for year-over-year comparisons and seasonal occupancy analysis.

---

## Part 3 — Front-Desk Operational Queries

The following queries are structured around a single target date: February 12, 2022. They demonstrate how SQL can feed real operational data into front-desk workflows.

**Total bookings vs. active bookings:**

```sql
-- All filings for the date
SELECT COUNT(*) AS total_bookings
FROM reservation
WHERE arrival_date = '2022-02-12';
-- 106 reservations

-- Active only (non-canceled)
SELECT COUNT(*) AS active_bookings
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled';
-- 68 active reservations
```

38 of the 106 records are canceled. Filtering those out gives front-desk staff the real number to plan against.

**Guest headcount:**

```sql
-- Adults only
SELECT SUM(no_of_adults) AS sum_adults
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled';
-- 137 adults

-- Adults and children combined
SELECT SUM(no_of_adults + no_of_children) AS sum_persons
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled';
-- 148 persons total
```

**Parking:**

```sql
-- Total spaces needed
SELECT SUM(required_car_parking_space) AS car_parking
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled';
-- 2 spaces

-- Length of stay for parking guests
SELECT no_of_week_nights + no_of_weekend_nights AS no_of_nights
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled'
  AND required_car_parking_space = 1;
-- 3 nights | 3 nights
```

Both guests requiring parking stay for exactly 3 nights, so those two spaces need to be blocked from February 12 through February 15.

---

## Part 4 — Dimensional Lookups: Families Arriving February 12

Rather than joining tables immediately, this section walks through the lookup in two steps — first retrieving the foreign key IDs from `reservation`, then resolving their details in the dimension tables.

**Meal plans for families with children:**

```sql
SELECT meal_plan_id
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled'
  AND no_of_children > 0;
-- All 8 rows return meal_plan_id = 1
```

```sql
SELECT * FROM meal_plan WHERE id = 1;
-- breakfast: Yes | lunch: No | dinner: Yes
```

Every family with children on this date is on Meal Plan 1 — breakfast and dinner, no lunch. Catering can plan morning and evening shifts accordingly without expecting midday demand from this group.

**Room types for families with children:**

```sql
SELECT room_type_id
FROM reservation
WHERE arrival_date = '2022-02-12'
  AND booking_status = 'Not Canceled'
  AND no_of_children > 0;
-- Mix of room_type_id 1 and 2
```

```sql
SELECT * FROM room_type WHERE id IN (1, 2);
-- id 1 | queen bed | standard layout | no studio
-- id 2 | double bed | standard layout | no studio
```

Both room types share the same standard layout. The only difference is bed size — queen vs. double. Front-desk can proceed with allocation knowing no studio or non-standard configurations are involved.

---

## Part 5 — Cancellation Analysis

**Overall cancellation volume:**

```sql
SELECT COUNT(*) AS total_cancellations
FROM reservation
WHERE booking_status = 'Canceled';
-- 11,885 cancellations
```

11,885 cancellations out of 36,275 total reservations — roughly 33% of all bookings.

**Lead time comparison: canceled vs. active:**

```sql
-- Canceled bookings
SELECT
    MIN(lead_time) AS min_lead_time,
    AVG(lead_time) AS avg_lead_time,
    MAX(lead_time) AS max_lead_time
FROM reservation
WHERE booking_status = 'Canceled';
-- Min: 0 | Avg: 139.2 days | Max: 443 days

-- Active bookings
SELECT
    MIN(lead_time) AS min_lead_time,
    AVG(lead_time) AS avg_lead_time,
    MAX(lead_time) AS max_lead_time
FROM reservation
WHERE booking_status = 'Not Canceled';
-- Min: 0 | Avg: 58.9 days | Max: 386 days
```

| Segment | Avg Lead Time |
|---|---|
| Canceled | 139.2 days |
| Not Canceled | 58.9 days |

Canceled reservations were booked, on average, 80 days further in advance than reservations that completed. The longer the booking horizon, the higher the cancellation risk — which points toward stricter deposit policies for bookings made more than 90 days out.

---

## SQL Concepts Covered

| Concept | Used In |
|---|---|
| SHOW TABLES / DESC | Part 1 |
| COUNT, SUM, MIN, MAX, AVG | Parts 1, 3, 5 |
| WHERE with date and status filters | Parts 3, 4, 5 |
| Column arithmetic inside SUM | Part 3 |
| SELECT DISTINCT | Part 2 |
| YEAR() date function | Part 2 |
| IN (set logic) | Part 4 |
| Two-step dimensional lookup | Part 4 |

---
Data Licensing / Lizenzierung
This case study utilizes open-source hospitality operational records distributed transparently under the public Creative Commons Attribution 4.0 International License (CC BY 4.0)
