# CarbonTrack: A Database-Driven Personal Carbon Footprint Calculator

**Project Check-in 1: Scope, Schema, and Strategy**

| Detail | Information |
|--------|-------------|
| Course | Introduction to Database Systems, Fall 2026 |
| Student | Nitinsai Karuturi |
| Check-in | Check-in 1: Scope, Schema, and Strategy |
| Platform | Android mobile application |

---

## 1. Problem Definition and Mobile Scope

### Problem

Most people have no clear idea how much CO₂ their everyday choices produce. Commuting, meals, and home energy use all add up, but the numbers are hidden. Without data, it is hard to know which habits matter most or whether things are improving over time.

CarbonTrack solves this by letting a user log everyday activities, automatically converting each one into kilograms of CO₂ using a stored emission factor, and showing the results as simple totals and summaries.

### Target Platform

Android mobile application.

### Scope for the Semester

The scope emphasizes personal tracking and clear calculation over social or gamified features, which keeps it manageable for one semester.

| In Scope | Out of Scope |
|----------|--------------|
| Log daily activities in three categories: Transport, Food, Energy | Social features, sharing, or leaderboards |
| Store an emission factor (kg CO₂ per unit) for each activity type | Automatic tracking through GPS or smart meters |
| Calculate CO₂ per log entry as quantity × emission factor | Carbon offset purchases |
| View weekly CO₂ totals and see which category produces the most CO₂ | Multi-user accounts and login (single user on one device) |

---

## 2. Initial Database Design and Mechanics

### Tables, Primary Keys, and Foreign Key

| Table | Columns | Primary Key | Foreign Key |
|-------|---------|-------------|-------------|
| ActivityType (parent) | type_id, name, category, unit, kg_co2_per_unit | type_id | None |
| ActivityLog (child) | log_id, type_id, log_date, quantity | log_id | type_id → ActivityType(type_id) |

### Logical Relationship

One ActivityType can appear in many ActivityLog entries (for example, "Car commute" is logged on many days). Each ActivityLog entry refers to exactly one ActivityType through `type_id`, forming a one-to-many relationship. Storing emission factors in ActivityType means each factor is stored once, and the CO₂ for any log entry is derived from it instead of being typed in repeatedly.

### SQL: Schema

```sql
CREATE TABLE ActivityType (
    type_id          INTEGER PRIMARY KEY,
    name             VARCHAR(50)  NOT NULL,
    category         VARCHAR(20)  NOT NULL,  -- Transport, Food, Energy
    unit             VARCHAR(20)  NOT NULL,  -- km, meal, kWh
    kg_co2_per_unit  DECIMAL(6,3) NOT NULL
);

CREATE TABLE ActivityLog (
    log_id    INTEGER PRIMARY KEY,
    type_id   INTEGER      NOT NULL,
    log_date  DATE         NOT NULL,
    quantity  DECIMAL(8,2) NOT NULL,
    FOREIGN KEY (type_id) REFERENCES ActivityType(type_id)
);
```

### SQL: Sample Data

*Emission factors below are illustrative placeholders and will be replaced with sourced values (for example, from the EPA) in later check-ins.*

```sql
INSERT INTO ActivityType VALUES
(1, 'Car commute',      'Transport', 'km',   0.250),
(2, 'Bus ride',         'Transport', 'km',   0.100),
(3, 'Beef meal',        'Food',      'meal', 6.000),
(4, 'Veggie meal',      'Food',      'meal', 0.800),
(5, 'Home electricity', 'Energy',    'kWh',  0.400);

INSERT INTO ActivityLog VALUES
(1, 1, '2026-09-28', 20),
(2, 3, '2026-09-28', 1),
(3, 5, '2026-09-29', 12),
(4, 2, '2026-09-30', 15),
(5, 4, '2026-10-01', 2);
```

### Five Example Queries in Relational Algebra

In every join below, the join condition is `ActivityType.type_id = ActivityLog.type_id`. Results are shown for the sample data above.

**Query 1: List all food activity types.**

σ category='Food' (ActivityType)

**Supports:** choosing an activity by category. **Result:** Beef meal, Veggie meal.

**Query 2: Show log entries after September 29, 2026.**

σ log_date > '2026-09-29' (ActivityLog)

**Supports:** the recent activity view. **Result:** Log 4 (Bus ride, 09-30) and log 5 (Veggie meal, 10-01).

**Query 3: Show full log history with activity names and units.**

π name, log_date, quantity, unit (ActivityType ⋈ ActivityLog)

**Supports:** the main history screen. **Result:** All five log entries with readable names and units.

**Query 4: Show only transport logs.**

π name, log_date, quantity (σ category='Transport' (ActivityType) ⋈ ActivityLog)

**Supports:** the category filter on the history screen. **Result:** Car commute on 09-28 (20), Bus ride on 09-30 (15).

**Query 5: Find high-emission activity types (over 1 kg CO₂ per unit) that have been logged.**

π name, category (σ kg_co2_per_unit > 1 (ActivityType) ⋈ ActivityLog)

**Supports:** the "biggest impact habits" insight. **Result:** Beef meal (Food).

### How These Queries Support the App's Main Features

- Logging an activity inserts into ActivityLog, and the activity choices come from ActivityType (Query 1).
- The history screen joins the two tables so each log shows a readable name and unit (Queries 2, 3, and 4).
- The insights screen uses the emission factor in ActivityType to highlight the highest-impact habits (Query 5).

### Note on Weekly CO₂ Totals

The app's headline feature is the weekly CO₂ total, which needs quantity × kg_co2_per_unit summed per week. Standard relational algebra cannot express a sum, so this uses the extended aggregation operator:

γ SUM(quantity × kg_co2_per_unit) (ActivityType ⋈ ActivityLog)

For the sample data, CO₂ per entry is 5.0, 6.0, 4.8, 1.5, and 1.6 kg, for a total of 18.9 kg CO₂ for the week of September 28, 2026. In SQL this is a `SUM` with `GROUP BY` on the week, to be implemented in the app.

---

## 3. AI Utilization Plan

### Goal

I am using AI in this project to build my own understanding of databases and problem-solving, not to have working code handed to me. AI is mainly a tutor and reviewer: I design the schema and write the queries myself first, then use AI to challenge and check my reasoning.

### Tools and Roles

| Tool | Role | Not used for |
|------|------|--------------|
| Claude (chat) | Tutor: explains concepts, checks my reasoning, helps diagnose SQL errors through guiding questions | Generating my full schema or final write-ups |
| SQLite or an online SQL playground | Verification: I run every schema and query myself on the sample data (not an AI tool; my source of truth) | N/A |
| GitHub Copilot (later, app development) | Boilerplate and syntax help in the mobile app code | Designing tables or deciding how data is related |

### Example Prompts

1. **Concept tutoring:** "Explain the difference between selection (σ) and projection (π) in relational algebra with a small example. Then give me three practice problems without the answers."
2. **Reasoning check:** "Here is my relational algebra for 'show only transport logs.' Tell me whether the join and filter order are correct, but do not rewrite it. Ask me questions that help me find any mistakes."
3. **Guided debugging:** "My INSERT into ActivityLog fails with a foreign key error. Do not fix it for me. Ask me guiding questions so I can find the cause."
4. **Self-testing:** "Quiz me on primary keys versus foreign keys using my ActivityType and ActivityLog tables. Wait for my answer before revealing the correct one."
5. **Design critique:** "Here is my two-table design and the reason for each column. What questions should I ask about whether any column depends on something other than the key? Do not give me the answer."

### How I Will Keep This Honest

- I attempt each task on my own first, then use AI to review it.
- I run every SQL statement and compare the results with what I worked out by hand.
- I keep notes on where AI helped me understand something and where I got stuck, for my later reflections.
- If I cannot explain a design decision without the AI in front of me, I study it until I can.
