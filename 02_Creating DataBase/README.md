# 🐘 PostgreSQL — Complete Notes for ML / DL / GenAI / Agentic AI
> From absolute beginner → job-ready data engineer / AI engineer

---

## Table of Contents

1. [Data Types](#1-data-types)
2. [Primary Keys & Foreign Keys](#2-primary-keys--foreign-keys)
3. [Constraints](#3-constraints)
4. [CREATE TABLE](#4-create-table)
5. [INSERT](#5-insert)
6. [UPDATE](#6-update)
7. [DELETE](#7-delete)
8. [ALTER TABLE](#8-alter-table)
9. [DROP TABLE](#9-drop-table)
10. [CHECK Constraint](#10-check-constraint)
11. [Conditional Expressions & Procedures](#11-conditional-expressions--procedures)
12. [CASE](#12-case)
13. [COALESCE](#13-coalesce)
14. [CAST](#14-cast)
15. [NULLIF](#15-nullif)
16. [Views](#16-views)
17. [Import & Export](#17-import--export)
18. [Python + PostgreSQL (psycopg2)](#18-python--postgresql-psycopg2)
19. [PostgreSQL for ML / DL / GenAI / Agentic AI — Real-World Patterns](#19-postgresql-for-ml--dl--genai--agentic-ai--real-world-patterns)
20. [Job-Ready Cheat Sheet](#20-job-ready-cheat-sheet)

---

## 1. Data Types

Data types define **what kind of data** a column can hold. Choosing the right type saves storage, speeds up queries, and prevents bad data.

### Numeric Types

| Type | Storage | Range / Use |
|------|---------|------------|
| `SMALLINT` | 2 bytes | –32,768 to 32,767 |
| `INTEGER` / `INT` | 4 bytes | –2.1B to 2.1B (most common) |
| `BIGINT` | 8 bytes | Very large integers |
| `SERIAL` | 4 bytes | Auto-incrementing integer (shorthand) |
| `BIGSERIAL` | 8 bytes | Auto-incrementing big integer |
| `NUMERIC(p, s)` | Variable | Exact decimal; `p` = precision, `s` = scale |
| `REAL` | 4 bytes | Floating-point (6 decimal digits) |
| `DOUBLE PRECISION` | 8 bytes | Floating-point (15 decimal digits) |

```sql
-- Examples
salary    NUMERIC(10, 2)   -- up to 99,999,999.99
score     REAL             -- 0.93847
model_id  BIGSERIAL        -- auto-incremented ID
```

> **ML note:** Use `NUMERIC` for financial features (no rounding error). Use `REAL` or `DOUBLE PRECISION` for model scores, embeddings dimensions, probabilities.

---

### Character / Text Types

| Type | Description |
|------|-------------|
| `CHAR(n)` | Fixed-length string, padded with spaces |
| `VARCHAR(n)` | Variable-length string, max `n` characters |
| `TEXT` | Unlimited length string (most flexible) |

```sql
username   VARCHAR(50)
bio        TEXT
country_code CHAR(2)    -- always 'IN', 'US', etc.
```

> **GenAI note:** Use `TEXT` for storing prompts, completions, chat history, and document chunks — no length limit.

---

### Boolean

```sql
is_active   BOOLEAN   -- TRUE, FALSE, NULL
```

```sql
SELECT * FROM models WHERE is_deployed = TRUE;
```

---

### Date / Time Types

| Type | Description |
|------|-------------|
| `DATE` | Date only (YYYY-MM-DD) |
| `TIME` | Time only |
| `TIMESTAMP` | Date + time (no timezone) |
| `TIMESTAMPTZ` | Date + time **with** timezone (recommended) |
| `INTERVAL` | Duration (e.g., `'3 days'`) |

```sql
created_at   TIMESTAMPTZ DEFAULT NOW()
trained_on   DATE
duration     INTERVAL
```

> **ML note:** Always use `TIMESTAMPTZ` for experiment timestamps to avoid timezone bugs across servers.

---

### JSON Types

| Type | Description |
|------|-------------|
| `JSON` | Stores raw JSON text |
| `JSONB` | Binary JSON — **indexed, faster queries** |

```sql
hyperparams  JSONB   -- {"lr": 0.001, "epochs": 10, "batch": 32}
metadata     JSONB
```

```sql
-- Query inside JSONB
SELECT * FROM experiments WHERE hyperparams->>'lr' = '0.001';
SELECT * FROM experiments WHERE hyperparams @> '{"epochs": 10}';
```

> **Agentic AI note:** `JSONB` is the go-to for storing agent state, tool call results, memory, and structured LLM outputs.

---

### Array Types

```sql
tags       TEXT[]       -- {'machine-learning', 'nlp'}
scores     REAL[]       -- {0.91, 0.87, 0.94}
embedding  VECTOR(1536) -- (with pgvector extension)
```

```sql
SELECT * FROM articles WHERE 'nlp' = ANY(tags);
```

---

### UUID

```sql
id   UUID DEFAULT gen_random_uuid()
```

> **Best practice:** Use `UUID` as primary key for distributed systems, APIs, and microservices — avoids ID collision across services.

---

### Special Types

| Type | Use Case |
|------|----------|
| `BYTEA` | Raw binary data (images, model weights) |
| `INET` | IP addresses |
| `TSVECTOR` | Full-text search |
| `VECTOR(n)` | pgvector — AI embeddings |

---

## 2. Primary Keys & Foreign Keys

### Primary Key

A **Primary Key (PK)** uniquely identifies each row. Rules:
- Must be **unique**
- Cannot be **NULL**
- Each table should have exactly **one** PK

```sql
CREATE TABLE users (
    user_id   SERIAL PRIMARY KEY,
    username  VARCHAR(50) NOT NULL,
    email     TEXT UNIQUE NOT NULL
);
```

Using UUID as PK (modern approach):
```sql
CREATE TABLE sessions (
    session_id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    user_id    INTEGER,
    started_at TIMESTAMPTZ DEFAULT NOW()
);
```

Composite Primary Key (using multiple columns):
```sql
CREATE TABLE model_metrics (
    model_id   INTEGER,
    epoch      INTEGER,
    accuracy   REAL,
    PRIMARY KEY (model_id, epoch)   -- combination must be unique
);
```

---

### Foreign Key

A **Foreign Key (FK)** links a column in one table to the PK of another table. Enforces **referential integrity** — you can't have orphan records.

```sql
CREATE TABLE experiments (
    exp_id      SERIAL PRIMARY KEY,
    user_id     INTEGER REFERENCES users(user_id),
    model_name  TEXT,
    created_at  TIMESTAMPTZ DEFAULT NOW()
);
```

Full FK syntax with actions:
```sql
CREATE TABLE predictions (
    pred_id    SERIAL PRIMARY KEY,
    exp_id     INTEGER,
    input_text TEXT,
    output     TEXT,
    FOREIGN KEY (exp_id)
        REFERENCES experiments(exp_id)
        ON DELETE CASCADE    -- delete predictions if experiment deleted
        ON UPDATE CASCADE    -- update if experiment id changes
);
```

### ON DELETE / ON UPDATE Options

| Option | Behavior |
|--------|----------|
| `CASCADE` | Automatically delete/update child rows |
| `SET NULL` | Set FK column to NULL |
| `SET DEFAULT` | Set FK column to default value |
| `RESTRICT` | Prevent deletion if child rows exist |
| `NO ACTION` | Same as RESTRICT (default) |

---

### Visual: Relationship Diagram

```
users (user_id PK)
    │
    ├──→ experiments (exp_id PK, user_id FK → users)
    │         │
    │         └──→ predictions (pred_id PK, exp_id FK → experiments)
    │
    └──→ sessions (session_id PK, user_id FK → users)
```

---

## 3. Constraints

Constraints are **rules** that enforce data integrity at the database level — the last line of defense before bad data enters.

### Types of Constraints

| Constraint | Description |
|------------|-------------|
| `NOT NULL` | Column cannot be empty |
| `UNIQUE` | All values in column must differ |
| `PRIMARY KEY` | Unique + Not Null (identifier) |
| `FOREIGN KEY` | References another table's PK |
| `CHECK` | Custom condition must be true |
| `DEFAULT` | Default value if none provided |

### Column-Level Constraints

```sql
CREATE TABLE ml_models (
    model_id    SERIAL PRIMARY KEY,
    model_name  VARCHAR(100) NOT NULL,
    version     VARCHAR(20)  NOT NULL DEFAULT '1.0.0',
    accuracy    REAL         CHECK (accuracy BETWEEN 0 AND 1),
    framework   TEXT         CHECK (framework IN ('pytorch', 'tensorflow', 'jax')),
    created_at  TIMESTAMPTZ  DEFAULT NOW()
);
```

### Table-Level Constraints

```sql
CREATE TABLE ab_tests (
    test_id    SERIAL,
    model_a    INTEGER NOT NULL,
    model_b    INTEGER NOT NULL,
    start_date DATE,
    end_date   DATE,
    PRIMARY KEY (test_id),
    CHECK (model_a <> model_b),              -- can't A/B test same model
    CHECK (end_date > start_date)            -- end must be after start
);
```

### Adding Constraints to Existing Tables

```sql
ALTER TABLE ml_models ADD CONSTRAINT chk_version
    CHECK (version ~ '^\d+\.\d+\.\d+$');    -- semantic version regex

ALTER TABLE users ADD CONSTRAINT uq_email UNIQUE (email);
```

### Removing Constraints

```sql
ALTER TABLE ml_models DROP CONSTRAINT chk_version;
```

---

## 4. CREATE TABLE

`CREATE TABLE` defines a new table's structure — columns, types, and constraints.

### Basic Syntax

```sql
CREATE TABLE table_name (
    column1  datatype  constraints,
    column2  datatype  constraints,
    ...
    table_constraints
);
```

### Simple Example

```sql
CREATE TABLE datasets (
    dataset_id   SERIAL PRIMARY KEY,
    name         VARCHAR(100) NOT NULL,
    description  TEXT,
    rows_count   BIGINT,
    size_mb      NUMERIC(10, 2),
    is_public    BOOLEAN DEFAULT TRUE,
    created_at   TIMESTAMPTZ DEFAULT NOW()
);
```

### Real ML System Example

```sql
-- Users who train models
CREATE TABLE data_scientists (
    ds_id       UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    full_name   VARCHAR(100) NOT NULL,
    email       TEXT UNIQUE NOT NULL,
    team        VARCHAR(50),
    joined_at   DATE DEFAULT CURRENT_DATE
);

-- ML experiment tracking
CREATE TABLE experiments (
    exp_id        SERIAL PRIMARY KEY,
    ds_id         UUID REFERENCES data_scientists(ds_id) ON DELETE SET NULL,
    dataset_id    INTEGER REFERENCES datasets(dataset_id),
    model_type    VARCHAR(50) NOT NULL,
    hyperparams   JSONB,
    train_loss    REAL,
    val_loss      REAL,
    accuracy      REAL CHECK (accuracy BETWEEN 0.0 AND 1.0),
    status        VARCHAR(20) DEFAULT 'running'
                  CHECK (status IN ('running','completed','failed')),
    started_at    TIMESTAMPTZ DEFAULT NOW(),
    finished_at   TIMESTAMPTZ
);
```

### CREATE TABLE IF NOT EXISTS

```sql
-- Safe: doesn't throw error if table already exists
CREATE TABLE IF NOT EXISTS chat_history (
    id         SERIAL PRIMARY KEY,
    session_id UUID,
    role       TEXT CHECK (role IN ('user','assistant','system')),
    content    TEXT NOT NULL,
    tokens     INTEGER,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### CREATE TABLE AS (from a query)

```sql
-- Create a new table from query results
CREATE TABLE top_models AS
SELECT model_id, model_name, accuracy
FROM experiments
WHERE accuracy > 0.95
ORDER BY accuracy DESC;
```

### Temporary Tables

```sql
-- Exists only for current session
CREATE TEMP TABLE batch_predictions (
    input_id  INTEGER,
    score     REAL
);
```

---

## 5. INSERT

`INSERT` adds new rows to a table.

### Basic Syntax

```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```

### Single Row Insert

```sql
INSERT INTO datasets (name, description, rows_count, size_mb, is_public)
VALUES ('IMDB Reviews', 'Sentiment analysis dataset', 50000, 125.4, TRUE);
```

### Multiple Row Insert (Bulk)

```sql
INSERT INTO ml_models (model_name, framework, accuracy)
VALUES
    ('BERT-base',   'pytorch',     0.923),
    ('RoBERTa',     'pytorch',     0.941),
    ('DistilBERT',  'pytorch',     0.911),
    ('T5-small',    'tensorflow',  0.887);
```

### INSERT with RETURNING

```sql
-- Get the generated ID back immediately
INSERT INTO experiments (ds_id, model_type, hyperparams)
VALUES (
    'a1b2c3d4-....',
    'transformer',
    '{"lr": 0.0001, "epochs": 20, "batch_size": 32}'::JSONB
)
RETURNING exp_id, started_at;
```

### INSERT … ON CONFLICT (Upsert)

```sql
-- Insert or update if conflict on unique key
INSERT INTO model_registry (model_name, version, accuracy)
VALUES ('GPT-4-mini', '1.2.0', 0.952)
ON CONFLICT (model_name, version)
DO UPDATE SET
    accuracy   = EXCLUDED.accuracy,
    updated_at = NOW();

-- Insert or ignore if already exists
INSERT INTO users (email, username)
VALUES ('alice@ai.com', 'alice')
ON CONFLICT (email) DO NOTHING;
```

> **Agentic AI note:** Upsert is critical for agent memory updates — you don't want duplicate memories, just updates.

### INSERT from SELECT

```sql
-- Copy rows from one table into another
INSERT INTO model_archive (model_id, model_name, accuracy)
SELECT model_id, model_name, accuracy
FROM experiments
WHERE status = 'completed' AND accuracy > 0.90;
```

---

## 6. UPDATE

`UPDATE` modifies existing rows.

### Basic Syntax

```sql
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE condition;
```

> ⚠️ **ALWAYS use WHERE.** Without it, every row gets updated!

### Simple Update

```sql
UPDATE experiments
SET status = 'completed', finished_at = NOW()
WHERE exp_id = 42;
```

### Update Multiple Columns

```sql
UPDATE ml_models
SET
    accuracy   = 0.967,
    version    = '2.1.0',
    updated_at = NOW()
WHERE model_name = 'BERT-base';
```

### Update with Calculation

```sql
UPDATE products
SET price = price * 1.10     -- 10% price increase
WHERE category = 'premium';
```

### Update Using Another Table (JOIN)

```sql
UPDATE experiments e
SET accuracy = m.best_accuracy
FROM model_benchmarks m
WHERE e.model_type = m.model_type;
```

### UPDATE with RETURNING

```sql
UPDATE experiments
SET status = 'failed'
WHERE started_at < NOW() - INTERVAL '24 hours'
  AND status = 'running'
RETURNING exp_id, model_type, started_at;
```

### Conditional Update (CASE in SET)

```sql
UPDATE predictions
SET confidence_label =
    CASE
        WHEN score >= 0.9  THEN 'high'
        WHEN score >= 0.7  THEN 'medium'
        ELSE 'low'
    END;
```

---

## 7. DELETE

`DELETE` removes rows from a table.

### Basic Syntax

```sql
DELETE FROM table_name WHERE condition;
```

> ⚠️ **ALWAYS use WHERE.** `DELETE FROM table;` deletes ALL rows!

### Simple Delete

```sql
DELETE FROM experiments WHERE status = 'failed';
```

### Delete with Subquery

```sql
-- Delete predictions for experiments older than 1 year
DELETE FROM predictions
WHERE exp_id IN (
    SELECT exp_id FROM experiments
    WHERE created_at < NOW() - INTERVAL '1 year'
);
```

### DELETE with RETURNING

```sql
DELETE FROM chat_history
WHERE session_id = 'abc-123' AND created_at < NOW() - INTERVAL '30 days'
RETURNING id, content;
```

### TRUNCATE (fast delete all rows)

```sql
TRUNCATE TABLE temp_predictions;               -- delete all, reset identity
TRUNCATE TABLE temp_predictions RESTART IDENTITY CASCADE;
```

| Command | Speed | Rollback | Triggers | WHERE clause |
|---------|-------|----------|----------|--------------|
| `DELETE` | Slow | Yes | Yes | Yes |
| `TRUNCATE` | Fast | Yes (in transaction) | No | No |

---

## 8. ALTER TABLE

`ALTER TABLE` modifies an existing table's structure — add columns, change types, rename, etc.

### Add a Column

```sql
ALTER TABLE experiments
ADD COLUMN test_accuracy REAL;

ALTER TABLE ml_models
ADD COLUMN embedding_dim INTEGER DEFAULT 768;

ALTER TABLE chat_history
ADD COLUMN token_count INTEGER DEFAULT 0;
```

### Drop a Column

```sql
ALTER TABLE experiments DROP COLUMN old_metric;
ALTER TABLE experiments DROP COLUMN IF EXISTS deprecated_field;
```

### Rename a Column

```sql
ALTER TABLE experiments RENAME COLUMN val_loss TO validation_loss;
```

### Change Column Data Type

```sql
ALTER TABLE experiments
ALTER COLUMN accuracy TYPE DOUBLE PRECISION;

-- With casting
ALTER TABLE logs
ALTER COLUMN event_count TYPE BIGINT USING event_count::BIGINT;
```

### Set / Drop Default

```sql
ALTER TABLE ml_models ALTER COLUMN framework SET DEFAULT 'pytorch';
ALTER TABLE ml_models ALTER COLUMN framework DROP DEFAULT;
```

### Set / Drop NOT NULL

```sql
ALTER TABLE experiments ALTER COLUMN model_type SET NOT NULL;
ALTER TABLE experiments ALTER COLUMN description DROP NOT NULL;
```

### Rename Table

```sql
ALTER TABLE experiments RENAME TO ml_experiments;
```

### Add / Drop Constraints

```sql
-- Add constraint
ALTER TABLE ml_models
ADD CONSTRAINT chk_accuracy CHECK (accuracy BETWEEN 0 AND 1);

-- Drop constraint
ALTER TABLE ml_models DROP CONSTRAINT chk_accuracy;

-- Add FK
ALTER TABLE predictions
ADD CONSTRAINT fk_exp FOREIGN KEY (exp_id) REFERENCES experiments(exp_id);
```

---

## 9. DROP TABLE

`DROP TABLE` permanently deletes a table and all its data.

### Basic Syntax

```sql
DROP TABLE table_name;
DROP TABLE IF EXISTS table_name;            -- safe — no error if not found
DROP TABLE table_name CASCADE;             -- also drops dependent objects
DROP TABLE table_name RESTRICT;            -- fail if dependencies exist (default)
```

### Examples

```sql
DROP TABLE IF EXISTS temp_results;

DROP TABLE predictions CASCADE;     -- also drops views/FKs pointing to it
```

> ⚠️ `DROP TABLE` is **irreversible** unless inside a transaction block or you have backups.

### Drop Multiple Tables

```sql
DROP TABLE IF EXISTS table_a, table_b, table_c;
```

### Safe Pattern (use transactions)

```sql
BEGIN;
DROP TABLE old_experiments;
-- verify other things look fine...
COMMIT;   -- or ROLLBACK; if something looks wrong
```

---

## 10. CHECK Constraint

`CHECK` enforces a custom boolean expression on a column or table. If the expression is `FALSE`, the insert/update is rejected.

### Column-Level CHECK

```sql
CREATE TABLE model_metrics (
    model_id  INTEGER,
    accuracy  REAL CHECK (accuracy BETWEEN 0.0 AND 1.0),
    f1_score  REAL CHECK (f1_score >= 0),
    loss      REAL CHECK (loss > 0),
    split     TEXT CHECK (split IN ('train', 'val', 'test'))
);
```

### Named CHECK Constraint

```sql
CREATE TABLE training_runs (
    run_id     SERIAL PRIMARY KEY,
    epochs     INTEGER,
    batch_size INTEGER,
    CONSTRAINT chk_epochs     CHECK (epochs > 0 AND epochs <= 10000),
    CONSTRAINT chk_batch      CHECK (batch_size IN (8, 16, 32, 64, 128, 256)),
    CONSTRAINT chk_dates      CHECK (end_time > start_time)
);
```

### Adding CHECK to Existing Table

```sql
ALTER TABLE experiments
ADD CONSTRAINT chk_status
CHECK (status IN ('queued', 'running', 'completed', 'failed'));
```

### CHECK with NOT VALID (skip existing rows)

```sql
-- Add constraint without checking existing rows (safe for large tables)
ALTER TABLE predictions
ADD CONSTRAINT chk_score CHECK (score BETWEEN 0 AND 1)
NOT VALID;

-- Then validate separately
ALTER TABLE predictions VALIDATE CONSTRAINT chk_score;
```

---

## 11. Conditional Expressions & Procedures

PostgreSQL has several **conditional expressions** that let you add logic directly inside SQL queries — no Python required.

| Expression | Purpose |
|------------|---------|
| `CASE` | If-elif-else logic in SQL |
| `COALESCE` | Return first non-NULL value |
| `NULLIF` | Return NULL if two values are equal |
| `CAST` | Convert data type |
| `GREATEST` / `LEAST` | Max / min across values |

---

## 12. CASE

`CASE` is SQL's if-else. It returns different values based on conditions.

### Simple CASE (equality checks)

```sql
SELECT
    model_name,
    CASE framework
        WHEN 'pytorch'     THEN 'PyTorch 🔥'
        WHEN 'tensorflow'  THEN 'TensorFlow 🌊'
        WHEN 'jax'         THEN 'JAX ⚡'
        ELSE 'Unknown'
    END AS framework_label
FROM ml_models;
```

### Searched CASE (range/condition checks)

```sql
SELECT
    exp_id,
    accuracy,
    CASE
        WHEN accuracy >= 0.95 THEN 'Excellent'
        WHEN accuracy >= 0.90 THEN 'Good'
        WHEN accuracy >= 0.80 THEN 'Acceptable'
        WHEN accuracy >= 0.70 THEN 'Poor'
        ELSE 'Failing'
    END AS performance_grade
FROM experiments
WHERE status = 'completed';
```

### CASE in ORDER BY

```sql
-- Prioritize 'completed' experiments first
SELECT * FROM experiments
ORDER BY
    CASE status
        WHEN 'completed' THEN 1
        WHEN 'running'   THEN 2
        WHEN 'failed'    THEN 3
        ELSE 4
    END;
```

### CASE in UPDATE

```sql
UPDATE experiments
SET priority =
    CASE
        WHEN accuracy > 0.95 THEN 'high'
        WHEN accuracy > 0.85 THEN 'medium'
        ELSE 'low'
    END
WHERE status = 'completed';
```

### CASE in Aggregation

```sql
-- Count experiments by performance category
SELECT
    COUNT(*) AS total,
    COUNT(CASE WHEN accuracy >= 0.90 THEN 1 END) AS high_accuracy,
    COUNT(CASE WHEN accuracy < 0.70  THEN 1 END) AS poor_accuracy,
    SUM(CASE WHEN framework = 'pytorch' THEN 1 ELSE 0 END) AS pytorch_count
FROM experiments;
```

> **ML note:** Use CASE in SELECT for feature engineering directly in SQL — e.g., bucketing continuous values into categorical bins.

---

## 13. COALESCE

`COALESCE(val1, val2, ..., valN)` returns the **first non-NULL value** in the list.

### Basic Usage

```sql
SELECT
    model_name,
    COALESCE(accuracy, val_accuracy, 0.0) AS best_accuracy
FROM experiments;
```

### Replacing NULLs with Defaults

```sql
SELECT
    exp_id,
    COALESCE(description, 'No description provided') AS description,
    COALESCE(tags, ARRAY['untagged'])                  AS tags,
    COALESCE(finished_at, NOW())                       AS effective_end
FROM experiments;
```

### COALESCE in WHERE

```sql
-- Treat NULL scores as 0 for comparison
SELECT * FROM predictions
WHERE COALESCE(score, 0) > 0.5;
```

### Multiple Fallback Sources

```sql
-- First try manual_label, then predicted_label, then 'unknown'
SELECT
    row_id,
    COALESCE(manual_label, predicted_label, 'unknown') AS final_label
FROM dataset_rows;
```

> **AI note:** Essential when building data pipelines — raw data always has NULLs. Always COALESCE before feeding features to a model.

---

## 14. CAST

`CAST` converts a value from one data type to another.

### Syntax Options

```sql
CAST(value AS target_type)
value::target_type          -- PostgreSQL shorthand (preferred)
```

### Common Conversions

```sql
-- String to number
SELECT '42'::INTEGER;
SELECT '3.14'::REAL;
SELECT '99.99'::NUMERIC(10,2);

-- Number to string
SELECT 42::TEXT;
SELECT 3.14::VARCHAR(10);

-- String to date/time
SELECT '2024-01-15'::DATE;
SELECT '2024-01-15 09:30:00'::TIMESTAMP;
SELECT '2024-01-15 09:30:00+05:30'::TIMESTAMPTZ;

-- To boolean
SELECT 'true'::BOOLEAN;
SELECT 1::BOOLEAN;      -- TRUE
SELECT 0::BOOLEAN;      -- FALSE

-- To JSON
SELECT '{"key": "value"}'::JSONB;

-- Array to text
SELECT ARRAY[1,2,3]::TEXT;    -- '{1,2,3}'
```

### CAST in Queries

```sql
-- Calculate percentage (integer division pitfall!)
SELECT
    correct_predictions,
    total_predictions,
    -- Without CAST: integer / integer = integer!
    correct_predictions / total_predictions                     AS wrong_pct,
    -- With CAST: float division
    CAST(correct_predictions AS REAL) / total_predictions       AS accuracy,
    correct_predictions::REAL / total_predictions               AS accuracy_v2
FROM model_results;
```

### CAST for Type Matching

```sql
-- Comparing text ID to integer FK
SELECT * FROM experiments
WHERE exp_id = '42'::INTEGER;

-- Aggregating mixed types
SELECT AVG(score::NUMERIC) FROM raw_predictions;
```

### Safe CAST (no error on failure)

```sql
-- Returns NULL instead of error if cast fails
SELECT CASE
    WHEN value ~ '^\d+$' THEN value::INTEGER
    ELSE NULL
END AS safe_int
FROM raw_import;
```

---

## 15. NULLIF

`NULLIF(val1, val2)` returns `NULL` if `val1 = val2`, otherwise returns `val1`.

Think of it as the **inverse of COALESCE** — it *creates* NULL from a sentinel value.

### Basic Usage

```sql
SELECT NULLIF(10, 10);    -- returns NULL
SELECT NULLIF(10, 20);    -- returns 10
SELECT NULLIF('', '');    -- returns NULL (empty string → NULL)
```

### Preventing Division by Zero

```sql
-- Without NULLIF: ERROR: division by zero
SELECT total_correct / total_predictions FROM results;

-- With NULLIF: returns NULL instead of crashing
SELECT
    total_correct * 1.0 / NULLIF(total_predictions, 0) AS accuracy
FROM results;
```

### Treating Empty Strings as NULL

```sql
-- Empty strings from CSV imports treated as real NULL
SELECT
    NULLIF(TRIM(model_notes), '')  AS model_notes,
    NULLIF(error_message, 'N/A')   AS error_message
FROM experiment_logs;
```

### NULLIF + COALESCE Combination

```sql
-- Replace empty/zero values with defaults
SELECT
    COALESCE(NULLIF(accuracy, 0), -1)         AS accuracy_or_minus1,
    COALESCE(NULLIF(TRIM(notes), ''), 'N/A')  AS clean_notes
FROM experiments;
```

> **Data pipeline note:** `NULLIF` is essential when importing messy CSVs where missing values come in as `''`, `'N/A'`, `'NULL'`, `0`, etc.

---

## 16. Views

A **View** is a saved SQL query that behaves like a virtual table. It doesn't store data — it runs the underlying query each time you SELECT from it.

### Why Use Views?

- **Simplify** complex queries into reusable names
- **Security** — expose only specific columns/rows to users
- **Abstraction** — hide table structure changes from apps
- **Reporting** — pre-define commonly needed reports

### CREATE VIEW

```sql
CREATE VIEW view_name AS
SELECT ...;
```

### Simple View

```sql
CREATE VIEW completed_experiments AS
SELECT
    e.exp_id,
    d.full_name AS scientist,
    e.model_type,
    e.accuracy,
    e.finished_at
FROM experiments e
JOIN data_scientists d ON e.ds_id = d.ds_id
WHERE e.status = 'completed'
ORDER BY e.accuracy DESC;

-- Use it like a table
SELECT * FROM completed_experiments WHERE accuracy > 0.95;
```

### View for AI Chat History

```sql
CREATE VIEW recent_conversations AS
SELECT
    session_id,
    role,
    content,
    token_count,
    created_at,
    ROW_NUMBER() OVER (PARTITION BY session_id ORDER BY created_at) AS turn_number
FROM chat_history
WHERE created_at > NOW() - INTERVAL '7 days';
```

### CREATE OR REPLACE VIEW

```sql
CREATE OR REPLACE VIEW model_leaderboard AS
SELECT
    model_name,
    framework,
    MAX(accuracy)   AS best_accuracy,
    AVG(accuracy)   AS avg_accuracy,
    COUNT(*)        AS num_runs
FROM experiments
WHERE status = 'completed'
GROUP BY model_name, framework
ORDER BY best_accuracy DESC;
```

### Materialized View (stores results — faster but needs refresh)

```sql
-- Creates a physical snapshot of the query result
CREATE MATERIALIZED VIEW mv_model_stats AS
SELECT
    model_type,
    COUNT(*)         AS total_runs,
    AVG(accuracy)    AS avg_accuracy,
    MAX(accuracy)    AS best_accuracy
FROM experiments
GROUP BY model_type;

-- Refresh when underlying data changes
REFRESH MATERIALIZED VIEW mv_model_stats;

-- Refresh without locking reads
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_model_stats;
```

> **ML note:** Use Materialized Views for pre-computing expensive aggregations (e.g., daily model performance summaries) that you query often.

### Dropping Views

```sql
DROP VIEW IF EXISTS completed_experiments;
DROP MATERIALIZED VIEW IF EXISTS mv_model_stats;
```

---

## 17. Import & Export

### Export (COPY TO)

```sql
-- Export full table to CSV
COPY experiments TO '/tmp/experiments.csv' WITH (FORMAT CSV, HEADER TRUE);

-- Export query result to CSV
COPY (
    SELECT exp_id, model_type, accuracy
    FROM experiments
    WHERE status = 'completed'
) TO '/tmp/good_experiments.csv' WITH (FORMAT CSV, HEADER TRUE);

-- Export as TSV (tab-separated)
COPY experiments TO '/tmp/experiments.tsv' WITH (FORMAT CSV, HEADER TRUE, DELIMITER E'\t');
```

### Import (COPY FROM)

```sql
-- Import CSV into table (table must exist with matching columns)
COPY experiments (model_type, accuracy, status)
FROM '/tmp/new_experiments.csv'
WITH (FORMAT CSV, HEADER TRUE);

-- Handle NULL values in CSV
COPY raw_data FROM '/tmp/data.csv'
WITH (FORMAT CSV, HEADER TRUE, NULL 'NA');
```

### Using `\copy` in psql (client-side, no superuser needed)

```sql
\copy experiments TO '~/experiments.csv' CSV HEADER;
\copy experiments FROM '~/experiments.csv' CSV HEADER;
```

### Import via Python (pandas + psycopg2)

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine('postgresql://user:pass@localhost:5432/mydb')

df = pd.read_csv('experiments.csv')
df.to_sql('experiments', engine, if_exists='append', index=False)

# Read from DB into DataFrame
df = pd.read_sql('SELECT * FROM experiments WHERE accuracy > 0.9', engine)
```

---

## 18. Python + PostgreSQL (psycopg2)

`psycopg2` is the most popular Python library for connecting to PostgreSQL. It's the backbone of ML data pipelines.

### Installation

```bash
pip install psycopg2-binary
# or for production (compiled):
pip install psycopg2
```

### Basic Connection

```python
import psycopg2

conn = psycopg2.connect(
    host     = "localhost",
    port     = 5432,
    database = "ml_platform",
    user     = "postgres",
    password = "yourpassword"
)

cursor = conn.cursor()
```

### Using Environment Variables (best practice)

```python
import os
import psycopg2

conn = psycopg2.connect(os.environ['DATABASE_URL'])
# DATABASE_URL = "postgresql://user:pass@host:5432/dbname"
```

### SELECT Query

```python
cursor.execute("SELECT exp_id, model_type, accuracy FROM experiments WHERE accuracy > %s", (0.9,))
rows = cursor.fetchall()

for row in rows:
    print(f"ID: {row[0]}, Model: {row[1]}, Accuracy: {row[2]:.4f}")

# Fetch one
row = cursor.fetchone()

# Fetch as dict (using RealDictCursor)
from psycopg2.extras import RealDictCursor
cursor = conn.cursor(cursor_factory=RealDictCursor)
cursor.execute("SELECT * FROM experiments LIMIT 5")
rows = cursor.fetchall()   # list of dict-like rows
print(rows[0]['model_type'])
```

### INSERT with Parameters

```python
# ✅ Always use parameterized queries — NEVER f-strings (SQL injection risk!)
cursor.execute("""
    INSERT INTO experiments (model_type, hyperparams, accuracy, status)
    VALUES (%s, %s, %s, %s)
    RETURNING exp_id
""", ('transformer', '{"lr": 0.0001}', 0.945, 'completed'))

exp_id = cursor.fetchone()[0]
conn.commit()
print(f"Inserted experiment: {exp_id}")
```

### UPDATE and DELETE

```python
cursor.execute("""
    UPDATE experiments
    SET status = %s, finished_at = NOW()
    WHERE exp_id = %s
""", ('completed', 42))

cursor.execute("DELETE FROM predictions WHERE score < %s", (0.1,))
conn.commit()
```

### Bulk Insert (executemany)

```python
records = [
    ('bert', 0.91, 'completed'),
    ('gpt2', 0.88, 'completed'),
    ('t5',   0.93, 'completed'),
]
cursor.executemany("""
    INSERT INTO experiments (model_type, accuracy, status)
    VALUES (%s, %s, %s)
""", records)
conn.commit()
```

### Bulk Insert (execute_values — much faster)

```python
from psycopg2.extras import execute_values

records = [(f'model_{i}', 0.85 + i*0.01, 'completed') for i in range(1000)]

execute_values(cursor, """
    INSERT INTO experiments (model_type, accuracy, status)
    VALUES %s
""", records)
conn.commit()
```

### Transaction Management

```python
try:
    cursor.execute("INSERT INTO experiments ...")
    cursor.execute("UPDATE model_registry ...")
    conn.commit()           # all or nothing
except Exception as e:
    conn.rollback()         # undo everything on error
    print(f"Transaction failed: {e}")
finally:
    cursor.close()
    conn.close()
```

### Context Manager Pattern (recommended)

```python
with psycopg2.connect(os.environ['DATABASE_URL']) as conn:
    with conn.cursor() as cursor:
        cursor.execute("SELECT * FROM experiments LIMIT 10")
        rows = cursor.fetchall()
# Connection and cursor auto-closed + committed
```

### Connection Pooling (production)

```python
from psycopg2 import pool

# Create pool once at app startup
connection_pool = pool.SimpleConnectionPool(
    minconn=1,
    maxconn=20,
    dsn=os.environ['DATABASE_URL']
)

def get_conn():
    return connection_pool.getconn()

def release_conn(conn):
    connection_pool.putconn(conn)
```

### JSON / JSONB with psycopg2

```python
import json
from psycopg2.extras import Json

hyperparams = {"lr": 0.001, "epochs": 50, "batch_size": 32, "optimizer": "adam"}

cursor.execute("""
    INSERT INTO experiments (model_type, hyperparams)
    VALUES (%s, %s)
""", ('transformer', Json(hyperparams)))
conn.commit()

# Read back
cursor.execute("SELECT hyperparams FROM experiments WHERE exp_id = %s", (1,))
row = cursor.fetchone()
params = row[0]   # automatically parsed as Python dict
print(params['lr'])   # 0.001
```

### Using SQLAlchemy (ORM layer over psycopg2)

```python
from sqlalchemy import create_engine, text
import pandas as pd

engine = create_engine('postgresql+psycopg2://user:pass@localhost/mldb')

# Execute raw SQL
with engine.connect() as conn:
    result = conn.execute(text("SELECT * FROM experiments WHERE accuracy > :acc"), {"acc": 0.9})
    rows = result.fetchall()

# Pandas integration
df = pd.read_sql_query("SELECT * FROM experiments", engine)
df.to_sql('new_table', engine, if_exists='replace', index=False)
```

---

## 19. PostgreSQL for ML / DL / GenAI / Agentic AI — Real-World Patterns

### 19.1 Experiment Tracking Database

```sql
CREATE TABLE experiments (
    exp_id        UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    project       TEXT NOT NULL,
    run_name      TEXT,
    model_arch    TEXT,
    hyperparams   JSONB NOT NULL DEFAULT '{}',
    metrics       JSONB DEFAULT '{}',
    tags          TEXT[] DEFAULT '{}',
    git_commit    CHAR(40),
    status        TEXT DEFAULT 'running' CHECK (status IN ('running','done','failed')),
    started_at    TIMESTAMPTZ DEFAULT NOW(),
    finished_at   TIMESTAMPTZ,
    duration_secs REAL GENERATED ALWAYS AS (
                      EXTRACT(EPOCH FROM (finished_at - started_at))
                  ) STORED
);

-- Log metrics per step (loss curves)
CREATE TABLE metric_logs (
    log_id   BIGSERIAL PRIMARY KEY,
    exp_id   UUID REFERENCES experiments(exp_id) ON DELETE CASCADE,
    step     INTEGER NOT NULL,
    metric   TEXT NOT NULL,
    value    DOUBLE PRECISION NOT NULL,
    logged_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX ON metric_logs (exp_id, metric, step);
```

```python
# Log metrics from training loop
def log_metric(exp_id, step, metric_name, value):
    cursor.execute("""
        INSERT INTO metric_logs (exp_id, step, metric, value)
        VALUES (%s, %s, %s, %s)
    """, (exp_id, step, metric_name, value))
    conn.commit()

# Call in training loop
for epoch, loss, acc in training_results:
    log_metric(exp_id, epoch, 'train_loss', loss)
    log_metric(exp_id, epoch, 'val_accuracy', acc)
```

---

### 19.2 GenAI Chat History Storage

```sql
CREATE TABLE sessions (
    session_id   UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    user_id      UUID,
    model        TEXT DEFAULT 'claude-3-5-sonnet',
    system_prompt TEXT,
    metadata     JSONB DEFAULT '{}',
    created_at   TIMESTAMPTZ DEFAULT NOW(),
    last_active  TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE messages (
    msg_id       BIGSERIAL PRIMARY KEY,
    session_id   UUID REFERENCES sessions(session_id) ON DELETE CASCADE,
    role         TEXT NOT NULL CHECK (role IN ('user', 'assistant', 'system', 'tool')),
    content      TEXT NOT NULL,
    tool_calls   JSONB,           -- for function/tool call results
    input_tokens  INTEGER,
    output_tokens INTEGER,
    latency_ms   INTEGER,
    created_at   TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_messages_session ON messages(session_id, created_at);
```

```python
def save_message(session_id, role, content, input_tokens=None, output_tokens=None):
    cursor.execute("""
        INSERT INTO messages (session_id, role, content, input_tokens, output_tokens)
        VALUES (%s, %s, %s, %s, %s)
        RETURNING msg_id
    """, (session_id, role, content, input_tokens, output_tokens))
    conn.commit()

def get_conversation(session_id, last_n=20):
    cursor.execute("""
        SELECT role, content FROM messages
        WHERE session_id = %s
        ORDER BY created_at DESC
        LIMIT %s
    """, (session_id, last_n))
    return list(reversed(cursor.fetchall()))
```

---

### 19.3 Vector Embeddings with pgvector

```bash
# Install pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;
```

```sql
CREATE TABLE documents (
    doc_id      UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    title       TEXT,
    content     TEXT NOT NULL,
    source_url  TEXT,
    embedding   VECTOR(1536),       -- OpenAI text-embedding-3-small
    metadata    JSONB DEFAULT '{}',
    created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Create HNSW index for fast approximate nearest neighbor search
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
```

```python
import openai

def embed_text(text: str) -> list[float]:
    response = openai.embeddings.create(
        model="text-embedding-3-small",
        input=text
    )
    return response.data[0].embedding

def store_document(title, content, url):
    embedding = embed_text(content)
    cursor.execute("""
        INSERT INTO documents (title, content, source_url, embedding)
        VALUES (%s, %s, %s, %s)
    """, (title, content, url, embedding))
    conn.commit()

def semantic_search(query: str, top_k: int = 5):
    query_embedding = embed_text(query)
    cursor.execute("""
        SELECT doc_id, title, content,
               1 - (embedding <=> %s::vector) AS similarity
        FROM documents
        ORDER BY embedding <=> %s::vector
        LIMIT %s
    """, (query_embedding, query_embedding, top_k))
    return cursor.fetchall()
```

> **RAG (Retrieval Augmented Generation) pattern:** Store docs as embeddings in PostgreSQL → retrieve top-k similar chunks → inject into LLM prompt.

---

### 19.4 Agentic AI — Tool Call & Memory Storage

```sql
-- Agent runs
CREATE TABLE agent_runs (
    run_id       UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    agent_name   TEXT NOT NULL,
    user_goal    TEXT NOT NULL,
    status       TEXT DEFAULT 'running' CHECK (status IN ('running','success','failed','cancelled')),
    context      JSONB DEFAULT '{}',
    started_at   TIMESTAMPTZ DEFAULT NOW(),
    finished_at  TIMESTAMPTZ,
    total_steps  INTEGER DEFAULT 0,
    total_tokens INTEGER DEFAULT 0
);

-- Individual agent steps / tool calls
CREATE TABLE agent_steps (
    step_id     BIGSERIAL PRIMARY KEY,
    run_id      UUID REFERENCES agent_runs(run_id) ON DELETE CASCADE,
    step_num    INTEGER NOT NULL,
    step_type   TEXT CHECK (step_type IN ('think','tool_call','tool_result','respond')),
    tool_name   TEXT,
    tool_input  JSONB,
    tool_output JSONB,
    reasoning   TEXT,
    tokens_used INTEGER,
    duration_ms INTEGER,
    created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Long-term agent memory
CREATE TABLE agent_memory (
    mem_id      UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    agent_name  TEXT NOT NULL,
    user_id     UUID,
    memory_key  TEXT NOT NULL,
    memory_val  TEXT NOT NULL,
    embedding   VECTOR(1536),
    importance  REAL DEFAULT 0.5 CHECK (importance BETWEEN 0 AND 1),
    access_count INTEGER DEFAULT 0,
    last_accessed TIMESTAMPTZ DEFAULT NOW(),
    expires_at  TIMESTAMPTZ,
    created_at  TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE (agent_name, user_id, memory_key)
);
```

```python
def log_tool_call(run_id, step_num, tool_name, tool_input, tool_output, duration_ms):
    cursor.execute("""
        INSERT INTO agent_steps
            (run_id, step_num, step_type, tool_name, tool_input, tool_output, duration_ms)
        VALUES (%s, %s, 'tool_call', %s, %s, %s, %s)
    """, (run_id, step_num, tool_name,
          Json(tool_input), Json(tool_output), duration_ms))
    conn.commit()

def upsert_memory(agent_name, user_id, key, value):
    embedding = embed_text(value)
    cursor.execute("""
        INSERT INTO agent_memory (agent_name, user_id, memory_key, memory_val, embedding)
        VALUES (%s, %s, %s, %s, %s)
        ON CONFLICT (agent_name, user_id, memory_key)
        DO UPDATE SET
            memory_val   = EXCLUDED.memory_val,
            embedding    = EXCLUDED.embedding,
            last_accessed = NOW(),
            access_count  = agent_memory.access_count + 1
    """, (agent_name, user_id, key, value, embedding))
    conn.commit()
```

---

### 19.5 Feature Store (ML Features)

```sql
CREATE TABLE feature_store (
    entity_id   TEXT NOT NULL,
    entity_type TEXT NOT NULL,
    feature     TEXT NOT NULL,
    value       DOUBLE PRECISION,
    value_str   TEXT,
    value_json  JSONB,
    computed_at TIMESTAMPTZ DEFAULT NOW(),
    valid_from  TIMESTAMPTZ DEFAULT NOW(),
    valid_to    TIMESTAMPTZ DEFAULT 'infinity',
    PRIMARY KEY (entity_id, entity_type, feature, valid_from)
);

-- Latest value view
CREATE VIEW current_features AS
SELECT DISTINCT ON (entity_id, entity_type, feature)
    entity_id, entity_type, feature, value, value_str, value_json, computed_at
FROM feature_store
ORDER BY entity_id, entity_type, feature, valid_from DESC;

-- Point-in-time lookup (prevents data leakage in training)
CREATE OR REPLACE FUNCTION get_features_at(
    p_entity_id TEXT,
    p_as_of TIMESTAMPTZ
) RETURNS TABLE (feature TEXT, value DOUBLE PRECISION) AS $$
    SELECT DISTINCT ON (feature) feature, value
    FROM feature_store
    WHERE entity_id = p_entity_id
      AND valid_from <= p_as_of
    ORDER BY feature, valid_from DESC;
$$ LANGUAGE SQL;
```

---

### 19.6 Model Registry

```sql
CREATE TABLE model_registry (
    model_id      UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    model_name    TEXT NOT NULL,
    version       TEXT NOT NULL,
    framework     TEXT,
    task          TEXT,
    metrics       JSONB DEFAULT '{}',
    artifact_path TEXT,       -- s3://bucket/path or /mnt/models/...
    is_champion   BOOLEAN DEFAULT FALSE,
    promoted_at   TIMESTAMPTZ,
    created_at    TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE (model_name, version)
);

-- Champion/Challenger pattern
CREATE VIEW champion_models AS
SELECT * FROM model_registry
WHERE is_champion = TRUE;
```

```python
def promote_champion(model_name, version):
    with conn:  # transaction
        # Demote current champion
        cursor.execute("""
            UPDATE model_registry
            SET is_champion = FALSE
            WHERE model_name = %s AND is_champion = TRUE
        """, (model_name,))
        # Promote new champion
        cursor.execute("""
            UPDATE model_registry
            SET is_champion = TRUE, promoted_at = NOW()
            WHERE model_name = %s AND version = %s
        """, (model_name, version))
```

---

### 19.7 Useful SQL Patterns for ML Engineers

```sql
-- Training/validation split by hash (reproducible, no leakage)
SELECT *,
    CASE
        WHEN MOD(ABS(hashtext(id::TEXT)), 10) < 8 THEN 'train'
        WHEN MOD(ABS(hashtext(id::TEXT)), 10) < 9 THEN 'val'
        ELSE 'test'
    END AS split
FROM dataset_rows;

-- Rolling average of loss (window function)
SELECT
    step,
    loss,
    AVG(loss) OVER (ORDER BY step ROWS BETWEEN 9 PRECEDING AND CURRENT ROW)
        AS loss_rolling_avg_10
FROM metric_logs
WHERE exp_id = 'abc-123' AND metric = 'train_loss'
ORDER BY step;

-- Compare two model runs side by side
SELECT
    a.exp_id   AS exp_a,
    b.exp_id   AS exp_b,
    a.accuracy AS acc_a,
    b.accuracy AS acc_b,
    b.accuracy - a.accuracy AS delta
FROM experiments a
JOIN experiments b ON a.model_type = b.model_type
WHERE a.exp_id = 'exp-001' AND b.exp_id = 'exp-002';

-- Find best hyperparameter combination
SELECT
    hyperparams->>'lr'         AS learning_rate,
    hyperparams->>'batch_size' AS batch_size,
    AVG(accuracy)              AS avg_accuracy,
    STDDEV(accuracy)           AS std_accuracy,
    COUNT(*)                   AS num_runs
FROM experiments
WHERE status = 'completed'
GROUP BY hyperparams->>'lr', hyperparams->>'batch_size'
ORDER BY avg_accuracy DESC;
```

---

## 20. Job-Ready Cheat Sheet

### DDL Quick Reference

```sql
-- Create
CREATE TABLE t (id SERIAL PRIMARY KEY, name TEXT NOT NULL, val REAL DEFAULT 0.0);
CREATE TABLE IF NOT EXISTS t (...);
CREATE TABLE t2 AS SELECT * FROM t1 WHERE ...;
CREATE VIEW v AS SELECT ...;
CREATE MATERIALIZED VIEW mv AS SELECT ...; REFRESH MATERIALIZED VIEW mv;

-- Modify
ALTER TABLE t ADD COLUMN col TEXT;
ALTER TABLE t DROP COLUMN col;
ALTER TABLE t RENAME COLUMN old TO new;
ALTER TABLE t ALTER COLUMN col TYPE BIGINT USING col::BIGINT;
ALTER TABLE t ALTER COLUMN col SET DEFAULT 0;
ALTER TABLE t ALTER COLUMN col SET NOT NULL;
ALTER TABLE t ADD CONSTRAINT name CHECK (condition);
ALTER TABLE t DROP CONSTRAINT name;
ALTER TABLE t RENAME TO new_name;

-- Delete
DROP TABLE IF EXISTS t CASCADE;
DROP VIEW IF EXISTS v;
TRUNCATE TABLE t RESTART IDENTITY;
```

### DML Quick Reference

```sql
-- Insert
INSERT INTO t (col1, col2) VALUES (v1, v2);
INSERT INTO t (col1, col2) VALUES (v1, v2) RETURNING id;
INSERT INTO t (col1) VALUES (v1) ON CONFLICT (col1) DO UPDATE SET col1=EXCLUDED.col1;
INSERT INTO t (col1) VALUES (v1) ON CONFLICT DO NOTHING;
INSERT INTO t SELECT * FROM t2;

-- Update
UPDATE t SET col1=v1, col2=v2 WHERE condition;
UPDATE t SET col1=v1 WHERE id IN (SELECT id FROM t2 WHERE ...);
UPDATE t SET col1=v1 RETURNING *;

-- Delete
DELETE FROM t WHERE condition;
DELETE FROM t WHERE id IN (...) RETURNING *;
```

### Conditional Expressions

```sql
CASE WHEN cond1 THEN r1 WHEN cond2 THEN r2 ELSE rn END
COALESCE(a, b, c)                -- first non-null
NULLIF(a, b)                     -- null if a=b, else a
CAST(x AS type) / x::type
GREATEST(a, b, c) / LEAST(a, b, c)
```

### Constraints

```sql
NOT NULL
UNIQUE
PRIMARY KEY                      -- NOT NULL + UNIQUE
FOREIGN KEY (col) REFERENCES t(col) ON DELETE CASCADE
CHECK (col > 0)
DEFAULT value
```

### Python psycopg2 Template

```python
import psycopg2, os
from psycopg2.extras import RealDictCursor, Json, execute_values

conn = psycopg2.connect(os.environ['DATABASE_URL'])

with conn:
    with conn.cursor(cursor_factory=RealDictCursor) as cur:
        # SELECT
        cur.execute("SELECT * FROM t WHERE val > %s", (threshold,))
        rows = cur.fetchall()

        # INSERT
        cur.execute("INSERT INTO t (col) VALUES (%s) RETURNING id", (value,))
        new_id = cur.fetchone()['id']

        # BULK INSERT
        execute_values(cur, "INSERT INTO t (a, b) VALUES %s", records)

        # JSON
        cur.execute("INSERT INTO t (data) VALUES (%s)", (Json({'key': 'val'}),))

conn.close()
```

### pgvector Cheat Sheet

```sql
-- Setup
CREATE EXTENSION vector;

-- Column
embedding VECTOR(1536)

-- Distance operators
<=>   -- cosine distance (most common for text)
<->   -- L2 (Euclidean) distance
<#>   -- negative inner product

-- Index
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);
CREATE INDEX ON docs USING ivfflat (embedding vector_l2_ops) WITH (lists=100);

-- Query
SELECT id, 1-(embedding <=> '[...]'::vector) AS sim
FROM docs ORDER BY embedding <=> '[...]'::vector LIMIT 10;
```

### Key Best Practices

| Practice | Why |
|----------|-----|
| Always use parameterized queries (`%s`) | Prevent SQL injection |
| Use `TIMESTAMPTZ` not `TIMESTAMP` | Timezone-aware; no bugs across servers |
| Use `UUID` for distributed PKs | Globally unique, no collision |
| Use `JSONB` not `JSON` | Indexable, faster, binary |
| Use `COALESCE` before aggregating | Avoid NULL-ruined averages |
| Use `ON CONFLICT DO UPDATE` for upserts | Idempotent data pipelines |
| Use connection pooling in prod | Don't open/close per request |
| Use `EXPLAIN ANALYZE` to debug slow queries | Find bottlenecks |
| Use `MATERIALIZED VIEW` for heavy aggregations | Pre-compute, then refresh |
| Use `TRANSACTION` for multi-step operations | All-or-nothing consistency |

---

*Generated for ML / DL / GenAI / Agentic AI engineers — covers PostgreSQL topics 60–82 with production AI system patterns.*
