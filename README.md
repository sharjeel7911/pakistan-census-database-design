<div align="center">

# 🇵🇰 Pakistan Census Database

A relational database for storing and analyzing national census data, from provinces all the way down to individual citizens, built in **MySQL**.

![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-Queries-orange?style=for-the-badge)
![Normalization](https://img.shields.io/badge/Normalized-3NF-green?style=for-the-badge)
![ERD](https://img.shields.io/badge/Design-ERD%20First-purple?style=for-the-badge)

</div>

---

## 📋 About

**Pakistan Census Database** is a centralized, normalized database that stores national demographic, household, and geographic data. It follows a strict administrative chain (Province → Division → District → Tehsil → Union Council → Census Block → Structure → Household → Person) so every citizen can be traced back to a physical address and a census block.

The schema covers housing quality, utilities (water, lighting, cooking fuel), citizen profiles (CNIC, literacy, income, employment), migration history, and the census field staff who collect all of it.

> 📌 This is our **Semester Final Project** for **Introduction to Database** (Spring 2026), University of Central Punjab.

---

## 💡 Motivation

Census data is only useful if it's clean, connected, and fast to query. A flat spreadsheet falls apart the moment you have millions of rows, duplicated district names, and people who moved houses. So we went the other way: break everything into small lookup tables, link them with proper foreign keys, and make the database enforce the rules instead of trusting whoever is typing.

The goal was not just "make the tables work" but to build something a government planner could actually slice to answer real questions like *which districts have the most households?* or *what's the literacy-income picture for a given region?*

---

## 🎯 Core Features

### 🗺️ **Geographic Hierarchy**

- Full chain from **Province** down to **Census Block**
- Every structure must belong to an active census block
- Easy roll-ups: province-level, district-level, or block-level totals

### 🏠 **Housing & Utilities**

- **Structure** table split into **Residential**, **Commercial**, and **Institution** subtypes (1:1 extension tables)
- Tracks water source, lighting, cooking fuel, washroom and kitchen status through lookup tables
- GPS coordinates stored for every structure

### 👤 **Citizen Records**

- **Person** table with CNIC, sex, DOB, nationality, religion, mother tongue, marital status, education, employment, occupation, industry, and income
- **Household head mapping** through a self-referencing `head_person_id`
- Disability tracking via a junction table (`Person_Disability`)
- Internal migration history via `Migration_History`

### 🧑‍💼 **Field Operations**

- **Census_Staff** with a recursive `supervisor_id` for management hierarchy
- Staff are assigned to census blocks and record the households/persons they enter

### 🔍 **SQL Analysis**

- **Joins**: INNER, LEFT, RIGHT, NATURAL, SELF
- **Nested & Correlated** queries with `IN`, `ANY`, `ALL`, `EXISTS`, `NOT EXISTS`
- **Aggregates**: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `GROUP BY`, `HAVING`
- **Set operations**: `UNION`, `UNION ALL`
- **Views** for profiles, household living conditions, and regional summaries

---

## 🗂️ Entity Overview

| Group | Tables |
| ----- | ------ |
| **Geography** | `Province`, `Division`, `District`, `Tehsil`, `Union_Council`, `Union_Council_Classification`, `Census_Block` |
| **Housing** | `Structure`, `Structure_Residential`, `Structure_Commercial`, `Structure_Institution`, `Structure_Type`, `Commercial_Sector_Type` |
| **Utilities** | `Water_Source`, `Lighting_Source`, `Cooking_Fuel_Source`, `Washroom_Status`, `Kitchen_Status` |
| **Households** | `Household`, `Household_Category`, `Residential_Status_Type`, `Census_Cycle` |
| **Citizens** | `Person`, `Person_Disability`, `Migration_History` |
| **Lookups** | `Biological_Sex`, `Religion`, `Mother_Tongue`, `Marital_Status`, `Education_Level`, `Employment_Status`, `Occupation`, `Industry`, `Relationship_Type`, `Nationality`, `Disability_Type` |
| **Field Staff** | `Census_Staff` |

---

## 🔗 Key Relationships

- **Geographic chain**: each level is 1:M and total on the child side (a district can't exist without a division)
- **Building extensions**: `Structure` → `Structure_Residential` / `Structure_Commercial` / `Structure_Institution` are 1:1
- **Core operations**: `Census_Block` → `Structure` → `Household` → `Person`
- **Self-references**: `Census_Staff.supervisor_id` and `Person.head_person_id`
- **Junction tables**: `Person_Disability` (M:N people ↔ disability types), `Migration_History` (people ↔ districts over time)

---

## 🛡️ Integrity & Cascading Rules

| Rule | Where | Why |
| ---- | ----- | --- |
| `ON DELETE CASCADE` | `Migration_History` → `Person` | If a citizen is removed from the census, their migration logs go with them. No orphaned records. |
| `ON DELETE SET NULL` | `Census_Staff.supervisor_id` | If a supervisor leaves, their enumerators stay in the system and become temporarily unassigned. |

---

## ⚡ Design Decisions

### **Normalization (3NF)**

Repeated text like `"Tap Water"` or `"Employed Full-Time"` is pulled out into small lookup tables. Every non-key attribute depends directly on its table's primary key.

### **Compact Data Types**

Lookup IDs use `TINYINT` (1 byte, up to 255 categories) instead of `INT` or `VARCHAR`. At census scale, those saved bytes per column add up fast and keep hot data in cache.

### **Views**

Multi-table joins are wrapped in views (`Citizen_Record`, `Highly_Educated_Persons`, `Most_Populated_Cities`, `Regional_Administrative_Summary`) so the logic lives in one place and the engine can optimize it.

---

## 📁 Project Structure
pakistan-census-database/
├── schema.sql # Table definitions, keys, constraints
├── data.sql # Sample data for testing
├── queries.sql # Joins, nested, aggregate, and view queries
├── erd.png # Entity Relationship Diagram
├── documentation.pdf # Full project report
└── README.md


---

## 🚀 How to Run

### **Prerequisites**

- MySQL (any recent 8.x release should work)
- A terminal or a client like MySQL Workbench

### **Setup**

```bash
# Log in to MySQL
mysql -u root -p

# Create and select the database
CREATE DATABASE pakistan_census;
USE pakistan_census;

# Load the schema, then the sample data
SOURCE schema.sql;
SOURCE data.sql;

# Run the queries
SOURCE queries.sql;
```

Or from the shell in one go:

```bash
mysql -u root -p pakistan_census < schema.sql
mysql -u root -p pakistan_census < data.sql
mysql -u root -p pakistan_census < queries.sql
```

---

## 🎓 Learning Outcomes

- **Database Design**: ERD-first modeling, 3NF normalization, deciding when a 1:1 extension table is worth it
- **Integrity**: declarative foreign keys, `CASCADE` vs `SET NULL` decisions
- **SQL**: every join type, nested and correlated subqueries, aggregates with `HAVING`, set operations
- **Performance Thinking**: compact data types, views, and why flat tables break at scale

---

## 👥 Team

| Member | Roll No | Role |
| ------ | ------- | ---- |
| Azma | L1F24BSCS0674 | Sub-queries, correlated queries, aggregate functions |
| Sarmad | L1F24BSCS0716 | Multi-table joins, views, documentation |
| Sharjeel | L1F24BSCS0506 | ERD, schema design, documentation |
| Zakia | L1F24BSCS0587 | ERD, data insertion, queries |

---

## 📄 License

This project was built for academic purposes at the University of Central Punjab. If you want to reuse it, feel free to open an issue or PR.

---

<div align="center">

Made with ☕ and way too many foreign keys

</div>
