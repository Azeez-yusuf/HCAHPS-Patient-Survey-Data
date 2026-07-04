# HCAHPS-Patient-Survey-Data
A deep-dive exploratory data analysis (EDA) of HCAHPS patient experience surveys to identify key drivers of hospital ratings and care quality.

## HCAHPS Patient Survey Data Transformation Pipeline

A comprehensive end-to-end ETL (Extract, Transform, Load) and data-cleansing project optimizing multi-table, historical healthcare survey datasets for strict schema migration into MySQL Workbench. 

This project addresses complex relational data anomalies—including mixed data types, structural formatting mismatches, broken key integrity, and unescaped embedded strings—converting raw, "dirty" CSV matrices into an enterprise-ready, normalized relational database.

---

#### 📊 Database Schema Architecture & Interdependencies

The dataset tracks **Hospital Consumer Assessment of Healthcare Providers and Systems (HCAHPS)** surveys over multiple operational timeline release periods. The structured environment maps across 8 distinct tables, strictly utilizing SQL naming standards (`lower_case_snake_case`).


          ┌───────────────────┐       ┌─────────────────┐
          │      reports      │       │     states      │
          └─────────┬─────────┘       └────────┬─────────
                    │                          │         
     ┌──────────────┴──────────────┐           │         
     │                             │           │         
┌────────▼─────────┐         ┌─────────▼───────────▼────────┐
│ national_results │         │        state_results         │
└──────────────────┘         └──────────────────────────────┘
▲

│

┌──────────────────┐         ┌─────────────┴────────────────┐
│     measures     │─────────►│          responses          │
└────────┬─────────┘         └──────────────────────────────┘
│

┌────────▼─────────┐

│    questions     │

└──────────────────┘



### Relational Entity Roles
* **Core Parents (Dimension Lookups):** * `reports`: Timeline anchors keyed by `release_period`.
    * `states`: Geographic regional bounds keyed by 2-letter abbreviation `state`.
    * `measures`: HCAHPS clinical operational classifications keyed by `measure_id`.
* **Sub-Dimension Mapping:**
    * `questions`: Granular survey text vectors tied via Foreign Key (`measure_id`).
    * `data_dictionary`: Metadata master repository cataloging structural fields.
* **Downstream Fact Tables:**
    * `national_results`: Country-wide normalized summary scores.
    * `state_results`: Cross-sectional metrics filtered by temporal and geographical planes.
    * `responses`: Highly granular hospital facility-level transactional survey aggregates.

---

### 🛠️ Data Vulnerabilities Addressed & Cleansing Log

The raw ingestion files exhibited severe real-world data corruption that would instantly break a strict database compiler. The execution pipeline resolved these anomalies via **Excel Power Query (M Language Engine)** and strict text qualification:

#### 1. Structural Footnote Removal (`data_dictionary.csv`)
* **Anomaly:** Meta-legend indicators (`(PK) = Primary Key`) were appended directly beneath the operational grid rows, resulting in row shape disparities and unparseable null clusters.
* **Resolution:** Implemented row-filtering layers isolating target data blocks and cleanly truncated metadata anomalies.

#### 2. Destructive Character Mitigation & File Isolation
* **Anomaly:** The `questions.csv` column matrices embed internal phrase syntax commas (`"..., how often did nurses treat you with courtesy and respect? "`). Standard flat CSV parsers split fields inside quotes on raw commas, triggering row dimension truncation errors.
* **Resolution:** Stripped trailing whitespace blocks, cleansed non-printable string characters via standard Unicode sweeps, and exported tables using double-quote (`"`) text qualifiers to preserve field isolation.

#### 3. Mixed Structural Data Type Normalization (`responses.csv`)
* **Anomaly:** The `completed_surveys` dimension underwent a data reporting tracking transition mid-lifecycle. Early iterations stored categorical string labels (`"300 or more"`, `"Fewer than 100"`), while modern cycles logged explicit integer scalars (`616`, `76`). Furthermore, thousands of cells across dimensions were populated with the string text `"Not Available"`.
* **Resolution:** * Isolated `"Not Available"` artifacts and converted them to blank strings so the MySQL migration layer correctly evaluates them as valid relational database `NULL` objects.
    * Enforced clean `VARCHAR(50)` allocations on the structural column to safely absorb both categorical historical buckets and precise values without generating calculation script bugs.

#### 4. Zero Padding Preservation (`responses.csv`)
* **Anomaly:** US federal `facility_id` string sequences contain leading zeros (e.g., `010001`). Default spreadsheet engines truncate these strings into standard numeric variables, permanently altering primary code integrity.
* **Resolution:** Force-cast the extraction data layer pipeline to explicitly interpret identifiers as absolute text configurations (`VARCHAR`) ahead of computational array transforms.

---

### 🚀 Execution Pipelines & Workflow Steps

To clone and migrate this database environment into an error-free MySQL Workbench deployment, execute the files utilizing these explicit parameters:

#### Phase A: Target CSV Pre-Processing Specifications
Every underlying module must be saved with the following configuration flags inside Excel before launching the migration framework:
* **Format Selection:** `CSV (Comma delimited) (*.csv)`
* **Text Qualification:** Forced explicit double-quoting (`"`) around string fields.
* **Encoding Parameter:** `Unicode (UTF-8)` via Save Selection -> Web Options -> Encoding.

#### Phase B: RDBMS Table Construction & Migration Sequence
Because relational data models strictly enforce internal `FOREIGN KEY CONSTRAINTS`, parent records must build completely before reference tables are generated. Run your data loading scripts inside MySQL Workbench following this sequence map:

| Import Order | Target Table Name | Key Constraints Enforced | Target MySQL Data Types |
| :--- | :--- | :--- | :--- |
| **1** | `reports` | `PRIMARY KEY (release_period)` | `VARCHAR(7)`, `DATE`, `DATE` |
| **2** | `states` | `PRIMARY KEY (state)` | `CHAR(2)`, `VARCHAR(50)`, `VARCHAR(50)` |
| **3** | `measures` | `PRIMARY KEY (measure_id)` | `VARCHAR(20)`, `VARCHAR(100)`, `VARCHAR(50)` |
| **4** | `data_dictionary` | `PRIMARY KEY (table_name, field_name)` | `VARCHAR(30)`, `VARCHAR(30)`, `TEXT` |
| **5** | `questions` | `PRIMARY KEY (question_num)`, `FOREIGN KEY (measure_id)` | `INT`, `VARCHAR(20)`, `TEXT`, `VARCHAR(50)` |
| **6** | `national_results`| `PRIMARY KEY (release_period, measure_id)` | `VARCHAR(7)`, `VARCHAR(20)`, `INT`, `INT`, `INT` |
| **7** | `state_results` | `PRIMARY KEY (release_period, state, measure_id)` | `VARCHAR(7)`, `CHAR(2)`, `VARCHAR(20)`, `INT`, `INT`, `INT` |
| **8** | `responses` | `PRIMARY KEY (release_period, facility_id)` | `VARCHAR(7)`, `CHAR(2)`, `VARCHAR(10)`, `VARCHAR(50)`, `INT` |

---

### 🔧 Database Import Configuration

When configuring the **Table Data Import Wizard** inside MySQL Workbench, apply the parameters below to match the optimized output of the cleansing workflow:

* **Field Separator:** `,` (Comma)
* **Line Separator:** `\r\n` (CRLF Standard Windows/Excel line endings)
* **Enclosed By:** `"` (Double-quote text qualifier)

---
#### Technical Author Note
*This ETL data pipeline and transformation documentation was systematically engineered to reflect production-grade architectures, ensuring math-logical precision and comprehensive referential consistency across all analytical reporting layers.*
