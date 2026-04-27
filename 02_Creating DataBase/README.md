# 🐘 PostgreSQL — Complete Notes
### From Absolute Beginner → Industry-Ready AI/ML Engineer

> **How to use these notes:** Every section starts with *"What & Why"* (the concept in plain English), then *"Syntax"* (the exact code), then *"Real Examples"*, and finally *"AI/ML Use-Case"* (why this matters for your career). Read top-to-bottom the first time, then use as a reference.

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
19. [PostgreSQL for ML / DL / GenAI / Agentic AI](#19-postgresql-for-ml--dl--genai--agentic-ai)
20. [Job-Ready Cheat Sheet](#20-job-ready-cheat-sheet)

---

## 1. Data Types

### What & Why

A **data type** tells PostgreSQL exactly what kind of value a column will store. Choosing the right type is critical because:

- It **saves disk space** (storing `TRUE/FALSE` as BOOLEAN uses 1 byte; as TEXT uses 5+ bytes)
- It **prevents bad data** (a DATE column won't accept `"hello"`)
- It **enables correct operations** (you can do math on INTEGER, not on TEXT)
- In ML pipelines, **wrong types cause silent bugs** (e.g., `"3.5"` vs `3.5` for a model score)

---

### Numeric Types

| Type | Storage | Range | Use When |
|------|---------|-------|----------|
| `SMALLINT` | 2 bytes | -32,768 to 32,767 | Age, small counts |
| `INTEGER` / `INT` | 4 bytes | ~-2.1B to 2.1B | Row IDs, counts |
| `BIGINT` | 8 bytes | ~-9.2 quintillion to 9.2 quintillion | User IDs at scale, timestamps |
| `DECIMAL(p,s)` / `NUMERIC(p,s)` | Variable | Exact precision | Money, scientific measurements |
| `REAL` | 4 bytes | 6 decimal digits precision | ML model scores (approx) |
| `DOUBLE PRECISION` | 8 bytes | 15 decimal digits precision | High-precision embeddings |
| `SERIAL` | 4 bytes | 1 to 2,147,483,647 | Auto-increment IDs |
| `BIGSERIAL` | 8 bytes | 1 to 9,223,372,036,854,775,807 | Large-scale auto-increment IDs |

```sql
-- DECIMAL(precision, scale)
-- precision = total digits, scale = digits after decimal point
-- DECIMAL(10, 2) can store: 12345678.99

CREATE TABLE model_metrics (
    model_id        SERIAL,
    accuracy        DECIMAL(5, 4),    -- e.g., 0.9823
    loss            DOUBLE PRECISION, -- e.g., 0.001234567891234
    epoch           SMALLINT,         -- e.g., 50
    total_params    BIGINT            -- e.g., 7000000000 (7 billion)
);
```

---

### Character / String Types

| Type | Description | Use When |
|------|-------------|----------|
| `CHAR(n)` | Fixed-length, padded with spaces | Country codes (`'US'`, `'IN'`) |
| `VARCHAR(n)` | Variable-length, max n chars | Names, email addresses |
| `TEXT` | Unlimited length | Descriptions, prompts, documents |

```sql
-- VARCHAR vs TEXT
-- VARCHAR(255): max 255 chars — good for constrained fields
-- TEXT: no limit — good for LLM prompts or article bodies

CREATE TABLE llm_prompts (
    prompt_id   SERIAL,
    user_name   VARCHAR(100),     -- bounded: names aren't infinite
    prompt_text TEXT,             -- unbounded: prompts can be very long
    model_code  CHAR(10)          -- fixed: 'gpt-4o    ' (padded)
);
```

**⚠️ Common Beginner Mistake:** Using `VARCHAR(255)` for everything. Use `TEXT` for long content — it's equally fast in PostgreSQL.

---

### Boolean Type

```sql
-- BOOLEAN stores TRUE, FALSE, or NULL
-- Accepted inputs: TRUE/FALSE, 'yes'/'no', 'on'/'off', '1'/'0'

CREATE TABLE model_registry (
    model_id        SERIAL,
    model_name      VARCHAR(100),
    is_deployed     BOOLEAN DEFAULT FALSE,
    is_fine_tuned   BOOLEAN DEFAULT NULL   -- NULL = "unknown"
);

-- Query
SELECT * FROM model_registry WHERE is_deployed = TRUE;
SELECT * FROM model_registry WHERE is_fine_tuned IS NULL;
```

---

### Date & Time Types

| Type | Storage | Description | Example |
|------|---------|-------------|---------|
| `DATE` | 4 bytes | Calendar date only | `2024-01-15` |
| `TIME` | 8 bytes | Time of day only | `14:30:00` |
| `TIMESTAMP` | 8 bytes | Date + time, no timezone | `2024-01-15 14:30:00` |
| `TIMESTAMPTZ` | 8 bytes | Date + time WITH timezone | `2024-01-15 14:30:00+05:30` |
| `INTERVAL` | 16 bytes | Duration | `3 days`, `2 hours` |

```sql
-- ALWAYS use TIMESTAMPTZ in production — timezone bugs are nasty
CREATE TABLE training_runs (
    run_id          SERIAL,
    started_at      TIMESTAMPTZ DEFAULT NOW(),
    finished_at     TIMESTAMPTZ,
    duration        INTERVAL GENERATED ALWAYS AS (finished_at - started_at) STORED
);

-- Date arithmetic
SELECT NOW() + INTERVAL '7 days';      -- a week from now
SELECT AGE('2024-01-01', '2023-01-01'); -- returns '1 year'
SELECT EXTRACT(YEAR FROM NOW());        -- returns current year
```

---

### JSON Types

| Type | Description | Use When |
|------|-------------|----------|
| `JSON` | Stores raw JSON text | You need exact original formatting |
| `JSONB` | Stores binary JSON (parsed) | **Always prefer this** — indexable, faster queries |

```sql
-- JSONB is the go-to for AI/ML metadata storage
CREATE TABLE experiment_logs (
    exp_id      SERIAL,
    run_name    VARCHAR(200),
    hyperparams JSONB,      -- {"lr": 0.001, "batch_size": 32, "epochs": 100}
    metrics     JSONB       -- {"accuracy": 0.95, "f1": 0.94}
);

-- Query inside JSONB
SELECT run_name, hyperparams->>'lr' AS learning_rate
FROM experiment_logs
WHERE (metrics->>'accuracy')::FLOAT > 0.90;

-- JSONB containment operator @>
SELECT * FROM experiment_logs
WHERE hyperparams @> '{"batch_size": 32}';
```

---

### Array Types

```sql
-- PostgreSQL supports arrays natively
CREATE TABLE documents (
    doc_id      SERIAL,
    title       TEXT,
    tags        TEXT[],          -- array of strings
    embedding   DOUBLE PRECISION[] -- array of floats (for vector storage)
);

INSERT INTO documents (title, tags, embedding)
VALUES ('AI Intro', ARRAY['ml', 'beginner', 'python'], ARRAY[0.1, 0.8, 0.3, 0.5]);

-- Query array
SELECT title FROM documents WHERE 'ml' = ANY(tags);
SELECT title, embedding[1] AS first_dim FROM documents;  -- 1-indexed!
```

---

### UUID Type

```sql
-- UUID = Universally Unique Identifier
-- Format: 550e8400-e29b-41d4-a716-446655440000
-- Use for: distributed systems, API keys, public-facing IDs

CREATE EXTENSION IF NOT EXISTS "pgcrypto";  -- enables gen_random_uuid()

CREATE TABLE api_keys (
    key_id      UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    user_id     INTEGER,
    created_at  TIMESTAMPTZ DEFAULT NOW()
);
```

---

### Special Types for AI/ML

```sql
-- pgvector extension — stores and searches ML embeddings
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE document_embeddings (
    id          SERIAL PRIMARY KEY,
    content     TEXT,
    embedding   vector(1536)    -- OpenAI ada-002 produces 1536-dim vectors
);

-- Cosine similarity search (Retrieval-Augmented Generation / RAG)
SELECT content, 1 - (embedding <=> '[0.1, 0.2, ...]'::vector) AS similarity
FROM document_embeddings
ORDER BY embedding <=> '[0.1, 0.2, ...]'::vector
LIMIT 5;
```

---

## 2. Primary Keys & Foreign Keys

### What & Why

**Primary Key (PK):** A column (or set of columns) that **uniquely identifies every row** in a table. Think of it as a student roll number — no two students share the same one.

**Foreign Key (FK):** A column in one table that **references the Primary Key of another table**. This creates a relationship and enforces referential integrity — you can't reference a row that doesn't exist.

```
users table              orders table
-----------              ------------
user_id (PK)  ←———     user_id (FK)  references users.user_id
username                 order_id (PK)
email                    amount
```

---

### Primary Key

```sql
-- Method 1: Inline definition
CREATE TABLE users (
    user_id   SERIAL PRIMARY KEY,    -- auto-increment + unique + not null
    email     VARCHAR(255)
);

-- Method 2: Table-level definition (useful for composite PKs)
CREATE TABLE user_roles (
    user_id   INTEGER,
    role_id   INTEGER,
    PRIMARY KEY (user_id, role_id)   -- composite PK: combination must be unique
);

-- SERIAL vs UUID as PK
-- SERIAL: simpler, smaller, faster joins — use for internal tables
-- UUID:   globally unique, safe for APIs — use for public-facing IDs
CREATE TABLE sessions (
    session_id  UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    user_id     INTEGER,
    created_at  TIMESTAMPTZ DEFAULT NOW()
);
```

**Rules a Primary Key automatically enforces:**
1. **NOT NULL** — every row must have a value
2. **UNIQUE** — no two rows can have the same value
3. **One PK per table** — you can only have one primary key

---

### Foreign Key

```sql
-- Basic Foreign Key
CREATE TABLE orders (
    order_id    SERIAL PRIMARY KEY,
    user_id     INTEGER REFERENCES users(user_id),  -- FK shorthand
    amount      DECIMAL(10, 2),
    ordered_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Full syntax with ON DELETE / ON UPDATE behavior
CREATE TABLE order_items (
    item_id     SERIAL PRIMARY KEY,
    order_id    INTEGER,
    product_id  INTEGER,
    quantity    INTEGER,
    
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
        ON DELETE CASCADE      -- delete items when order is deleted
        ON UPDATE CASCADE,     -- update item.order_id if order.order_id changes
    
    FOREIGN KEY (product_id) REFERENCES products(product_id)
        ON DELETE RESTRICT     -- block deletion of product if items reference it
);
```

### ON DELETE Behaviors — Explained

| Action | What happens to child rows | Use case |
|--------|---------------------------|----------|
| `RESTRICT` | Error — blocks parent deletion | Protect important data |
| `CASCADE` | Child rows are also deleted | Orders → items relationship |
| `SET NULL` | FK column set to NULL | Optional relationships |
| `SET DEFAULT` | FK column set to its DEFAULT | Reassign to a default owner |
| `NO ACTION` | Error (default, checked at end of transaction) | Most common default |

```sql
-- Real-world example: ML experiment tracking
CREATE TABLE experiments (
    exp_id      SERIAL PRIMARY KEY,
    name        VARCHAR(200),
    created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE runs (
    run_id      SERIAL PRIMARY KEY,
    exp_id      INTEGER NOT NULL,
    status      VARCHAR(50),
    metrics     JSONB,
    
    FOREIGN KEY (exp_id) REFERENCES experiments(exp_id)
        ON DELETE CASCADE   -- delete all runs when experiment is deleted
);

CREATE TABLE artifacts (
    artifact_id  SERIAL PRIMARY KEY,
    run_id       INTEGER,
    file_path    TEXT,
    
    FOREIGN KEY (run_id) REFERENCES runs(run_id)
        ON DELETE SET NULL  -- keep artifact record even if run is deleted
);
```

---

## 3. Constraints

### What & Why

Constraints are **rules enforced at the database level** to guarantee data quality. Instead of relying on application code to validate data, you embed the rules in the database itself — so no matter what inserts data (Python script, API, admin tool), the rules always apply.

---

### All Constraints at a Glance

```sql
CREATE TABLE complete_example (
    -- NOT NULL: column must have a value
    user_id     SERIAL,
    
    -- UNIQUE: no two rows can have the same value
    email       VARCHAR(255) UNIQUE,
    
    -- NOT NULL: value required
    username    VARCHAR(100) NOT NULL,
    
    -- DEFAULT: value used when none is provided
    created_at  TIMESTAMPTZ DEFAULT NOW(),
    is_active   BOOLEAN DEFAULT TRUE,
    
    -- CHECK: custom validation rule
    age         INTEGER CHECK (age >= 0 AND age <= 150),
    score       DECIMAL(5,2) CHECK (score BETWEEN 0.0 AND 100.0),
    
    -- PRIMARY KEY: unique + not null
    PRIMARY KEY (user_id),
    
    -- Named constraints (recommended for large schemas)
    CONSTRAINT chk_username_length CHECK (LENGTH(username) >= 3),
    CONSTRAINT uq_email UNIQUE (email)
);
```

---

### NOT NULL

```sql
-- Without NOT NULL, any column can be NULL (missing/unknown)
-- NULL is not zero, not empty string — it means "no value"

CREATE TABLE model_versions (
    version_id   SERIAL PRIMARY KEY,
    model_name   VARCHAR(100) NOT NULL,    -- must always have a name
    version_tag  VARCHAR(50)  NOT NULL,    -- must always have a version
    description  TEXT,                     -- optional — NULLs allowed
    deployed_at  TIMESTAMPTZ               -- optional — might not be deployed yet
);

-- NULL behavior
SELECT NULL = NULL;     -- returns NULL (not TRUE!)
SELECT NULL IS NULL;    -- returns TRUE — use IS NULL to check for nulls
SELECT NULL IS NOT NULL;-- returns FALSE
```

---

### UNIQUE

```sql
-- Single column unique
CREATE TABLE api_users (
    id      SERIAL PRIMARY KEY,
    email   VARCHAR(255) UNIQUE,        -- one account per email
    phone   VARCHAR(20)  UNIQUE         -- one account per phone
);

-- Composite unique (combination must be unique)
CREATE TABLE team_members (
    team_id     INTEGER,
    user_id     INTEGER,
    UNIQUE (team_id, user_id)           -- a user can only be in a team once
);

-- Adding unique constraint to existing table
ALTER TABLE api_users ADD CONSTRAINT uq_username UNIQUE (username);
```

---

### DEFAULT

```sql
CREATE TABLE chat_messages (
    msg_id      SERIAL PRIMARY KEY,
    content     TEXT NOT NULL,
    role        VARCHAR(20) DEFAULT 'user',    -- 'user', 'assistant', 'system'
    created_at  TIMESTAMPTZ DEFAULT NOW(),
    token_count INTEGER DEFAULT 0,
    is_deleted  BOOLEAN DEFAULT FALSE
);

-- Insert without specifying default columns
INSERT INTO chat_messages (content) VALUES ('Hello!');
-- role='user', created_at=NOW(), token_count=0, is_deleted=FALSE are auto-set
```

---

## 4. CREATE TABLE

### What & Why

`CREATE TABLE` defines the **structure (schema)** of your data. It's the blueprint before any data is stored. Getting this right upfront saves you from painful migrations later.

---

### Full Syntax

```sql
CREATE TABLE [IF NOT EXISTS] table_name (
    column_name  data_type  [constraints],
    ...
    [table_constraints]
);
```

---

### Beginner Example

```sql
-- Simple user table
CREATE TABLE users (
    user_id    SERIAL PRIMARY KEY,
    first_name VARCHAR(50)  NOT NULL,
    last_name  VARCHAR(50)  NOT NULL,
    email      VARCHAR(255) NOT NULL UNIQUE,
    age        INTEGER      CHECK (age >= 18),
    created_at TIMESTAMPTZ  DEFAULT NOW()
);
```

### Intermediate Example

```sql
-- E-commerce style
CREATE TABLE products (
    product_id   SERIAL PRIMARY KEY,
    name         VARCHAR(200) NOT NULL,
    description  TEXT,
    price        DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
    stock_qty    INTEGER DEFAULT 0 CHECK (stock_qty >= 0),
    category     VARCHAR(100),
    created_at   TIMESTAMPTZ DEFAULT NOW(),
    updated_at   TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE orders (
    order_id    SERIAL PRIMARY KEY,
    user_id     INTEGER NOT NULL REFERENCES users(user_id) ON DELETE RESTRICT,
    status      VARCHAR(50) DEFAULT 'pending' 
                    CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
    total       DECIMAL(10, 2),
    created_at  TIMESTAMPTZ DEFAULT NOW()
);
```

### Advanced: ML Experiment Tracking Schema

```sql
-- Real-world MLflow-style schema

CREATE TABLE projects (
    project_id  SERIAL PRIMARY KEY,
    name        VARCHAR(200) NOT NULL UNIQUE,
    description TEXT,
    created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE experiments (
    exp_id      SERIAL PRIMARY KEY,
    project_id  INTEGER NOT NULL REFERENCES projects(project_id) ON DELETE CASCADE,
    name        VARCHAR(200) NOT NULL,
    tags        TEXT[],
    created_at  TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE (project_id, name)
);

CREATE TABLE runs (
    run_id      UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    exp_id      INTEGER NOT NULL REFERENCES experiments(exp_id) ON DELETE CASCADE,
    status      VARCHAR(20) DEFAULT 'running'
                    CHECK (status IN ('running', 'completed', 'failed', 'killed')),
    start_time  TIMESTAMPTZ DEFAULT NOW(),
    end_time    TIMESTAMPTZ,
    hyperparams JSONB,
    metrics     JSONB,
    artifact_uri TEXT
);

-- CREATE TABLE ... LIKE (inherit structure from another table)
CREATE TABLE runs_archive (LIKE runs INCLUDING ALL);
```

---

## 5. INSERT

### What & Why

`INSERT` adds new rows of data into a table. It's how you populate your database — from loading datasets to recording new user actions.

---

### Basic Syntax

```sql
-- Insert a single row (specify columns explicitly — always safer)
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);

-- Insert a single row without column list (ORDER MUST MATCH TABLE DEFINITION)
INSERT INTO table_name VALUES (value1, value2, ...);  -- risky, avoid this

-- Insert multiple rows at once (much faster than multiple single inserts)
INSERT INTO table_name (column1, column2)
VALUES 
    (val1a, val2a),
    (val1b, val2b),
    (val1c, val2c);
```

---

### Examples

```sql
-- Single insert
INSERT INTO users (first_name, last_name, email, age)
VALUES ('Alice', 'Smith', 'alice@example.com', 28);

-- Multiple inserts (batch insert — preferred for performance)
INSERT INTO products (name, price, stock_qty, category)
VALUES
    ('Python Textbook',  49.99,  100, 'books'),
    ('GPU Server',    2999.00,    5, 'hardware'),
    ('Cloud Credits',  200.00, 1000, 'services');

-- Insert with RETURNING (get back the auto-generated ID)
INSERT INTO experiments (project_id, name, tags)
VALUES (1, 'Baseline Run', ARRAY['v1', 'baseline'])
RETURNING exp_id, created_at;
-- Returns: exp_id=42, created_at=2024-01-15 09:00:00+00

-- Insert from a SELECT (copy data from another table)
INSERT INTO runs_archive (run_id, exp_id, status, start_time)
SELECT run_id, exp_id, status, start_time
FROM runs
WHERE status = 'completed' AND start_time < NOW() - INTERVAL '30 days';
```

---

### INSERT ON CONFLICT (Upsert)

This is one of PostgreSQL's most powerful features — insert a row, but if a conflict occurs (duplicate PK/unique), do something else instead of throwing an error.

```sql
-- ON CONFLICT DO NOTHING — skip if duplicate
INSERT INTO users (email, username)
VALUES ('alice@example.com', 'alice')
ON CONFLICT (email) DO NOTHING;

-- ON CONFLICT DO UPDATE — update the existing row (UPSERT pattern)
INSERT INTO model_registry (model_name, version, accuracy, deployed_at)
VALUES ('bert-base', 'v2.1', 0.934, NOW())
ON CONFLICT (model_name, version) 
DO UPDATE SET 
    accuracy = EXCLUDED.accuracy,      -- EXCLUDED refers to the new row
    deployed_at = EXCLUDED.deployed_at;

-- Practical AI use case: upserting document embeddings
INSERT INTO document_embeddings (doc_id, content, embedding)
VALUES (101, 'Introduction to AI', '[0.1, 0.2, ...]'::vector)
ON CONFLICT (doc_id) 
DO UPDATE SET 
    embedding = EXCLUDED.embedding,
    updated_at = NOW();
```

---

## 6. UPDATE

### What & Why

`UPDATE` modifies existing rows. **Always use a WHERE clause** — without it, you update every single row in the table (a very common and devastating mistake).

---

### Syntax

```sql
UPDATE table_name
SET column1 = value1,
    column2 = value2
WHERE condition;
```

---

### Examples

```sql
-- Update a single row
UPDATE users
SET email = 'newalice@example.com'
WHERE user_id = 1;

-- Update multiple columns
UPDATE products
SET price = price * 1.10,    -- 10% price increase
    updated_at = NOW()
WHERE category = 'books';

-- Update with a subquery
UPDATE runs
SET status = 'failed'
WHERE exp_id IN (
    SELECT exp_id FROM experiments WHERE name LIKE '%deprecated%'
);

-- UPDATE with RETURNING (see what changed)
UPDATE model_registry
SET is_deployed = TRUE,
    deployed_at = NOW()
WHERE model_name = 'gpt-fine-tuned-v3'
RETURNING model_name, deployed_at;

-- UPDATE using values from another table (UPDATE ... FROM)
UPDATE orders o
SET status = 'shipped'
FROM shipments s
WHERE o.order_id = s.order_id
  AND s.shipped_at IS NOT NULL;
```

**⚠️ Safety Tip:** Before running an UPDATE, run the equivalent SELECT first:
```sql
-- First verify what you're about to update
SELECT * FROM users WHERE user_id = 1;
-- Then update
UPDATE users SET email = 'new@email.com' WHERE user_id = 1;
```

---

## 7. DELETE

### What & Why

`DELETE` removes rows from a table. Like UPDATE, **always use a WHERE clause** unless you truly want to delete everything.

---

### Syntax & Examples

```sql
-- Delete specific rows
DELETE FROM users WHERE user_id = 42;

-- Delete with a condition
DELETE FROM sessions WHERE created_at < NOW() - INTERVAL '30 days';

-- Delete with subquery
DELETE FROM runs
WHERE exp_id IN (
    SELECT exp_id FROM experiments WHERE project_id = 5
);

-- DELETE with RETURNING
DELETE FROM cart_items 
WHERE user_id = 101 AND added_at < NOW() - INTERVAL '7 days'
RETURNING item_id, product_id;

-- Delete ALL rows (but keep the table structure)
DELETE FROM temp_processing_table;
-- vs TRUNCATE (much faster for clearing large tables)
TRUNCATE TABLE temp_processing_table;
TRUNCATE TABLE temp_processing_table RESTART IDENTITY; -- also resets SERIAL counter
```

### DELETE vs TRUNCATE vs DROP

| Command | What it does | Can WHERE? | Rollback? | Speed |
|---------|-------------|------------|-----------|-------|
| `DELETE` | Removes rows | ✅ Yes | ✅ Yes | Slow |
| `TRUNCATE` | Removes all rows | ❌ No | ✅ Yes (within txn) | Very Fast |
| `DROP TABLE` | Removes table entirely | ❌ No | ✅ Yes (within txn) | Instant |

---

## 8. ALTER TABLE

### What & Why

`ALTER TABLE` modifies an existing table's **structure** — add columns, change types, rename things, add/drop constraints. This is how you evolve your schema over time without recreating tables.

---

### All Common ALTER Operations

```sql
-- ── ADD COLUMN ──────────────────────────────────────────────
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users ADD COLUMN last_login TIMESTAMPTZ DEFAULT NOW();

-- ── DROP COLUMN ─────────────────────────────────────────────
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users DROP COLUMN IF EXISTS phone;  -- safe, no error if missing

-- ── RENAME COLUMN ───────────────────────────────────────────
ALTER TABLE users RENAME COLUMN username TO display_name;

-- ── CHANGE DATA TYPE ────────────────────────────────────────
ALTER TABLE products ALTER COLUMN price TYPE NUMERIC(12, 2);
-- Note: type change fails if existing data can't be cast automatically

-- ── SET / DROP DEFAULT ──────────────────────────────────────
ALTER TABLE orders ALTER COLUMN status SET DEFAULT 'pending';
ALTER TABLE orders ALTER COLUMN status DROP DEFAULT;

-- ── SET / DROP NOT NULL ─────────────────────────────────────
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
ALTER TABLE users ALTER COLUMN phone DROP NOT NULL;

-- ── ADD CONSTRAINT ──────────────────────────────────────────
ALTER TABLE products ADD CONSTRAINT chk_price CHECK (price >= 0);
ALTER TABLE users ADD CONSTRAINT uq_email UNIQUE (email);
ALTER TABLE orders ADD CONSTRAINT fk_user
    FOREIGN KEY (user_id) REFERENCES users(user_id);

-- ── DROP CONSTRAINT ─────────────────────────────────────────
ALTER TABLE products DROP CONSTRAINT chk_price;
ALTER TABLE users DROP CONSTRAINT uq_email;

-- ── RENAME TABLE ────────────────────────────────────────────
ALTER TABLE users RENAME TO app_users;
```

### Real-World Migration Example

```sql
-- Adding vector support to an existing table (common in RAG upgrades)
CREATE EXTENSION IF NOT EXISTS vector;

ALTER TABLE documents 
    ADD COLUMN embedding vector(1536),
    ADD COLUMN embedding_model VARCHAR(100) DEFAULT 'text-embedding-ada-002',
    ADD COLUMN embedded_at TIMESTAMPTZ;

-- After populating embeddings, add an index
CREATE INDEX idx_docs_embedding ON documents 
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);
```

---

## 9. DROP TABLE

### What & Why

`DROP TABLE` permanently deletes a table — structure AND all data. It's irreversible (outside a transaction). Use with extreme caution.

---

### Syntax

```sql
-- Basic drop
DROP TABLE table_name;

-- Safe drop (no error if table doesn't exist)
DROP TABLE IF EXISTS table_name;

-- Drop multiple tables at once
DROP TABLE IF EXISTS table1, table2, table3;

-- RESTRICT vs CASCADE
DROP TABLE experiments;           -- fails if other tables reference this (default)
DROP TABLE experiments CASCADE;   -- also drops dependent objects (FKs, views, etc.)

-- Drop, but preserve dependent views
DROP TABLE old_reports RESTRICT;  -- explicit — fails if anything depends on it
```

### Production Safety Pattern

```sql
-- ALWAYS wrap destructive DDL in a transaction
BEGIN;
    -- First verify you're dropping the right thing
    SELECT COUNT(*) FROM old_logs;         -- check: 0 rows expected before dropping
    DROP TABLE old_logs;
ROLLBACK;  -- change to COMMIT when you're sure
```

---

## 10. CHECK Constraint

### What & Why

`CHECK` lets you define **custom validation rules** directly in the table schema. The database enforces the rule on every INSERT and UPDATE, so invalid data can never enter.

---

### Syntax & Examples

```sql
-- Inline CHECK
CREATE TABLE employees (
    emp_id      SERIAL PRIMARY KEY,
    name        VARCHAR(100) NOT NULL,
    salary      DECIMAL(10,2) CHECK (salary > 0),
    age         INTEGER CHECK (age BETWEEN 18 AND 100),
    department  VARCHAR(50)  CHECK (department IN ('eng', 'data', 'product', 'sales'))
);

-- Named CHECK (recommended — descriptive error messages)
CREATE TABLE model_evaluations (
    eval_id     SERIAL PRIMARY KEY,
    model_id    INTEGER NOT NULL,
    accuracy    DECIMAL(5,4),
    precision_  DECIMAL(5,4),
    recall      DECIMAL(5,4),
    f1_score    DECIMAL(5,4),
    
    CONSTRAINT chk_accuracy  CHECK (accuracy  BETWEEN 0 AND 1),
    CONSTRAINT chk_precision CHECK (precision_ BETWEEN 0 AND 1),
    CONSTRAINT chk_recall    CHECK (recall     BETWEEN 0 AND 1),
    CONSTRAINT chk_f1        CHECK (f1_score   BETWEEN 0 AND 1)
);

-- Multi-column CHECK
CREATE TABLE date_ranges (
    id          SERIAL PRIMARY KEY,
    start_date  DATE NOT NULL,
    end_date    DATE NOT NULL,
    
    CONSTRAINT chk_date_order CHECK (end_date >= start_date)
);

-- Add CHECK to existing table
ALTER TABLE employees 
ADD CONSTRAINT chk_email_format CHECK (email LIKE '%@%.%');
```

**⚠️ Important:** CHECK constraints with NULL — if any column in the check is NULL, the constraint is considered **satisfied** (not violated). NULL means "unknown", so the check can't determine it's false.

```sql
-- This INSERT succeeds even with CHECK (age >= 18)
INSERT INTO employees (name, salary) VALUES ('Bob', 50000);
-- age is NULL → check is skipped → insert succeeds
```

---

## 11. Conditional Expressions & Procedures

### What & Why

Conditional expressions let you add **if-then-else logic inside SQL queries** — without needing application code. Procedures let you store and execute **blocks of SQL logic** on the server.

---

### PL/pgSQL Basics (Stored Procedures & Functions)

```sql
-- CREATE FUNCTION: returns a value
CREATE OR REPLACE FUNCTION get_model_grade(accuracy DECIMAL)
RETURNS VARCHAR AS $$
BEGIN
    IF accuracy >= 0.95 THEN RETURN 'A';
    ELSIF accuracy >= 0.90 THEN RETURN 'B';
    ELSIF accuracy >= 0.80 THEN RETURN 'C';
    ELSE RETURN 'F';
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Usage
SELECT model_name, accuracy, get_model_grade(accuracy) AS grade
FROM model_evaluations;

-- CREATE PROCEDURE: executes logic, no return value
CREATE OR REPLACE PROCEDURE archive_old_runs(days_old INTEGER)
LANGUAGE plpgsql AS $$
BEGIN
    INSERT INTO runs_archive SELECT * FROM runs
    WHERE start_time < NOW() - (days_old || ' days')::INTERVAL;
    
    DELETE FROM runs
    WHERE start_time < NOW() - (days_old || ' days')::INTERVAL;
    
    RAISE NOTICE 'Archived runs older than % days', days_old;
END;
$$;

-- Call the procedure
CALL archive_old_runs(90);
```

---

## 12. CASE

### What & Why

`CASE` is SQL's **if-else expression**. It evaluates conditions and returns a value. You can use it anywhere an expression is valid — SELECT, WHERE, ORDER BY, and even inside aggregations.

---

### Two Forms of CASE

```sql
-- ── FORM 1: Searched CASE (most flexible) ───────────────────
SELECT 
    model_name,
    accuracy,
    CASE
        WHEN accuracy >= 0.95 THEN 'Excellent'
        WHEN accuracy >= 0.90 THEN 'Good'
        WHEN accuracy >= 0.80 THEN 'Fair'
        ELSE 'Poor'
    END AS performance_grade
FROM model_evaluations;

-- ── FORM 2: Simple CASE (equality check) ────────────────────
SELECT 
    order_id,
    CASE status
        WHEN 'pending'    THEN '⏳ Waiting'
        WHEN 'shipped'    THEN '🚚 On the way'
        WHEN 'delivered'  THEN '✅ Done'
        WHEN 'cancelled'  THEN '❌ Cancelled'
        ELSE '❓ Unknown'
    END AS status_display
FROM orders;
```

### Advanced CASE Patterns

```sql
-- CASE inside COUNT (conditional aggregation)
SELECT
    experiment_name,
    COUNT(*) AS total_runs,
    COUNT(CASE WHEN status = 'completed' THEN 1 END) AS completed,
    COUNT(CASE WHEN status = 'failed'    THEN 1 END) AS failed,
    ROUND(
        COUNT(CASE WHEN status = 'completed' THEN 1 END)::DECIMAL
        / COUNT(*) * 100, 2
    ) AS success_rate_pct
FROM runs r
JOIN experiments e ON r.exp_id = e.exp_id
GROUP BY experiment_name;

-- CASE in ORDER BY
SELECT model_name, status
FROM deployments
ORDER BY
    CASE status
        WHEN 'critical' THEN 1
        WHEN 'warning'  THEN 2
        WHEN 'healthy'  THEN 3
    END;

-- CASE for data transformation (ML feature engineering in SQL)
SELECT 
    user_id,
    CASE WHEN age < 18             THEN 'teen'
         WHEN age BETWEEN 18 AND 35 THEN 'young_adult'
         WHEN age BETWEEN 36 AND 60 THEN 'adult'
         ELSE 'senior'
    END AS age_group,
    CASE WHEN total_purchases > 1000 THEN 1 ELSE 0 END AS high_value_flag
FROM user_profiles;
```

---

## 13. COALESCE

### What & Why

`COALESCE(val1, val2, ..., valN)` returns the **first non-NULL value** from its arguments. It's the standard way to handle NULLs — replace them with a fallback value.

---

### Syntax & Examples

```sql
-- Basic: replace NULL with a default
SELECT 
    user_id,
    COALESCE(display_name, username, 'Anonymous') AS shown_name
FROM users;
-- If display_name is NULL, use username. If that's also NULL, use 'Anonymous'.

-- Replace NULL numbers with 0
SELECT 
    product_id,
    COALESCE(discount_price, regular_price) AS final_price,
    COALESCE(review_count, 0) AS reviews
FROM products;

-- Real ML use case: fill missing metric values
SELECT
    run_id,
    COALESCE(metrics->>'accuracy', '0') AS accuracy,
    COALESCE(metrics->>'f1_score', 'N/A') AS f1
FROM runs;

-- COALESCE vs CASE (equivalent, COALESCE is cleaner for NULL replacement)
-- These are identical:
SELECT COALESCE(phone, 'No phone') FROM users;
SELECT CASE WHEN phone IS NULL THEN 'No phone' ELSE phone END FROM users;

-- Use in aggregations: treat NULLs as 0
SELECT AVG(COALESCE(score, 0)) AS avg_score FROM assessments;

-- Multiple fallbacks — great for default configurations
SELECT
    exp_id,
    COALESCE(
        custom_config->>'max_epochs',   -- user setting
        default_config->>'max_epochs',  -- project default
        '100'                           -- hardcoded fallback
    )::INTEGER AS max_epochs
FROM experiments;
```

---

## 14. CAST

### What & Why

`CAST` converts a value from one data type to another. SQL requires **type compatibility** — you can't compare a TEXT `'42'` to an INTEGER `42` without converting one of them.

---

### Two Syntax Forms

```sql
-- Standard SQL syntax
CAST(value AS target_type)

-- PostgreSQL shorthand (preferred in practice)
value::target_type
```

---

### Common Conversions

```sql
-- Text → Number
SELECT CAST('42' AS INTEGER);           -- standard
SELECT '42'::INTEGER;                   -- shorthand
SELECT '3.14'::DECIMAL(10, 2);

-- Number → Text
SELECT 42::TEXT;
SELECT TO_CHAR(3.14159, 'FM9999.99');  -- formatted conversion

-- Text → Date/Timestamp
SELECT '2024-01-15'::DATE;
SELECT '2024-01-15 09:30:00'::TIMESTAMP;
SELECT TO_TIMESTAMP('15/01/2024', 'DD/MM/YYYY');  -- with format

-- Timestamp → Date (drop the time part)
SELECT NOW()::DATE;

-- Integer → Boolean
SELECT 1::BOOLEAN;   -- TRUE
SELECT 0::BOOLEAN;   -- FALSE

-- JSON extraction needs casting
SELECT (metrics->>'accuracy')::DECIMAL(5,4) FROM runs;
-- metrics->>'accuracy' returns TEXT '0.9342'
-- ::DECIMAL converts it to 0.9342 for math operations

-- Array to string and back
SELECT ARRAY[1,2,3]::TEXT;           -- '{1,2,3}'
SELECT '{1,2,3}'::INTEGER[];         -- ARRAY[1,2,3]

-- Real-world: compare a JSONB field numerically
SELECT run_id
FROM runs
WHERE (metrics->>'accuracy')::FLOAT > 0.90  -- CAST needed here
ORDER BY (metrics->>'accuracy')::FLOAT DESC;
```

**⚠️ CAST Failures:** If the value can't be converted, PostgreSQL throws an error. Use `TRY` patterns or validate first:

```sql
-- Safe cast pattern using CASE
SELECT 
    value_text,
    CASE 
        WHEN value_text ~ '^\d+(\.\d+)?$'  -- regex check: is it numeric?
        THEN value_text::DECIMAL
        ELSE NULL
    END AS safe_numeric
FROM raw_import_data;
```

---

## 15. NULLIF

### What & Why

`NULLIF(val1, val2)` returns **NULL if val1 equals val2**, otherwise returns val1. Its primary use is **preventing division-by-zero errors** and turning "sentinel" values (like `0` or `''`) back into proper NULLs.

---

### Syntax & Examples

```sql
-- Prevent division by zero
SELECT 
    product_id,
    total_revenue / NULLIF(total_units_sold, 0) AS avg_revenue_per_unit
    -- If total_units_sold = 0, NULLIF returns NULL → division returns NULL (not error)
FROM sales;

-- Turn empty strings into NULL (common after CSV imports)
SELECT NULLIF(TRIM(phone_number), '') AS clean_phone FROM contacts;
-- '' → NULL, '  ' → NULL, '555-1234' → '555-1234'

-- Combined with COALESCE
SELECT 
    COALESCE(NULLIF(TRIM(email), ''), 'no-email@placeholder.com') AS email
FROM raw_users;
-- Step 1: NULLIF turns '' → NULL
-- Step 2: COALESCE turns NULL → 'no-email@placeholder.com'

-- Calculate accuracy rate safely
SELECT
    model_id,
    ROUND(
        correct_predictions::DECIMAL 
        / NULLIF(total_predictions, 0) * 100, 2
    ) AS accuracy_pct
FROM model_results;
```

---

## 16. Views

### What & Why

A **View** is a **named, saved SQL query** that behaves like a virtual table. When you query a view, PostgreSQL runs the underlying query and returns the results. Views don't store data themselves (unless materialized).

**Why use views?**
- **Simplify complex queries** — wrap a 20-join query into a single clean name
- **Security** — grant access to a view but not the underlying tables
- **Abstraction** — change the underlying query without changing how apps use it
- **Reusability** — write once, use everywhere

---

### Regular Views

```sql
-- Create a view
CREATE VIEW active_model_deployments AS
SELECT 
    m.model_id,
    m.model_name,
    m.version,
    d.deployed_at,
    d.environment,
    d.endpoint_url
FROM models m
JOIN deployments d ON m.model_id = d.model_id
WHERE d.is_active = TRUE
  AND d.health_status = 'healthy';

-- Use it like a table
SELECT * FROM active_model_deployments WHERE environment = 'production';
SELECT COUNT(*) FROM active_model_deployments;

-- Replace/update a view
CREATE OR REPLACE VIEW active_model_deployments AS
SELECT ... ;  -- new query here

-- Drop a view
DROP VIEW IF EXISTS active_model_deployments;
DROP VIEW IF EXISTS active_model_deployments CASCADE;  -- also drops dependent views
```

### Materialized Views (Very Important for ML/Analytics)

A **Materialized View** actually **stores the query results on disk**. Much faster to query, but needs manual refresh when data changes.

```sql
-- Create a materialized view
CREATE MATERIALIZED VIEW experiment_summary AS
SELECT
    e.exp_id,
    e.name AS experiment_name,
    COUNT(r.run_id) AS total_runs,
    COUNT(CASE WHEN r.status = 'completed' THEN 1 END) AS completed_runs,
    MAX((r.metrics->>'accuracy')::FLOAT) AS best_accuracy,
    AVG((r.metrics->>'accuracy')::FLOAT) AS avg_accuracy
FROM experiments e
LEFT JOIN runs r ON e.exp_id = r.exp_id
GROUP BY e.exp_id, e.name
WITH DATA;  -- populate immediately

-- Query it (blazing fast — reads stored results)
SELECT * FROM experiment_summary ORDER BY best_accuracy DESC;

-- Refresh when underlying data changes
REFRESH MATERIALIZED VIEW experiment_summary;

-- Refresh without locking (allows reads during refresh — production-safe)
REFRESH MATERIALIZED VIEW CONCURRENTLY experiment_summary;
-- Note: requires a UNIQUE index on the materialized view

-- Add index to materialized view
CREATE UNIQUE INDEX idx_exp_summary_id ON experiment_summary(exp_id);
```

### View vs Materialized View

| Feature | View | Materialized View |
|---------|------|------------------|
| Stores data | No | Yes |
| Query speed | Runs query each time | Reads stored results |
| Always fresh | Yes | Only after REFRESH |
| Use case | Simple abstraction | Heavy analytics, dashboards |

---

## 17. Import & Export

### What & Why

Real-world data lives in CSV files, JSON dumps, and external databases. PostgreSQL provides tools to import and export data efficiently — essential for loading ML datasets, sharing data, and ETL pipelines.

---

### COPY Command (Server-Side — Fastest)

```sql
-- Export to CSV (runs on the database server)
COPY users TO '/tmp/users_export.csv' 
    WITH (FORMAT CSV, HEADER TRUE, DELIMITER ',');

-- Import from CSV
COPY users (first_name, last_name, email, age) 
FROM '/tmp/users_import.csv' 
WITH (FORMAT CSV, HEADER TRUE, DELIMITER ',');

-- Export only specific rows
COPY (
    SELECT user_id, email, created_at 
    FROM users 
    WHERE created_at > '2024-01-01'
) TO '/tmp/new_users.csv' WITH (FORMAT CSV, HEADER);

-- Handle NULLs in CSV
COPY products FROM '/tmp/products.csv'
WITH (FORMAT CSV, HEADER, NULL 'N/A', DELIMITER ',');
```

### \copy Command (Client-Side — psql)

```sql
-- \copy runs on YOUR machine (the client), not the server
-- Use this when you don't have server filesystem access
\copy users TO '~/users.csv' WITH CSV HEADER
\copy users FROM '~/new_users.csv' WITH CSV HEADER
```

### Python + pandas Export/Import

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine('postgresql://user:pass@localhost:5432/mydb')

# Export table to DataFrame
df = pd.read_sql('SELECT * FROM model_evaluations', engine)
df.to_csv('evaluations.csv', index=False)

# Import DataFrame to table
df = pd.read_csv('new_evaluations.csv')
df.to_sql('model_evaluations', engine, if_exists='append', index=False)

# Fast bulk insert with COPY
from io import StringIO
def copy_from_df(df, table, conn):
    buffer = StringIO()
    df.to_csv(buffer, index=False, header=False)
    buffer.seek(0)
    cursor = conn.cursor()
    cursor.copy_from(buffer, table, sep=',', null='')
    conn.commit()
```

---

## 18. Python + PostgreSQL (psycopg2)

### What & Why

**psycopg2** is the most popular Python library for connecting to PostgreSQL. It lets you execute SQL queries, fetch results, and manage transactions from Python code — essential for ML pipelines, data ingestion, and AI applications.

---

### Installation & Basic Connection

```python
pip install psycopg2-binary  # easier install, includes C extensions
# or: pip install psycopg2   # compile from source

import psycopg2
import psycopg2.extras  # for RealDictCursor, execute_values, etc.

# Connect to database
conn = psycopg2.connect(
    host="localhost",
    port=5432,
    database="mydb",
    user="postgres",
    password="your_password"
)

# Or via connection string (URL format)
conn = psycopg2.connect("postgresql://postgres:password@localhost:5432/mydb")

cursor = conn.cursor()
```

---

### Basic CRUD Operations

```python
# ── CREATE ────────────────────────────────────────────────────
cursor.execute(
    "INSERT INTO users (name, email) VALUES (%s, %s) RETURNING user_id",
    ("Alice", "alice@example.com")    # ALWAYS use parameterized queries!
)
new_id = cursor.fetchone()[0]
conn.commit()

# ── READ ──────────────────────────────────────────────────────
cursor.execute("SELECT * FROM users WHERE age > %s", (25,))
rows = cursor.fetchall()     # list of tuples
row  = cursor.fetchone()     # single tuple
many = cursor.fetchmany(10)  # first 10 results

# Use RealDictCursor for dict-like access
dict_cursor = conn.cursor(cursor_factory=psycopg2.extras.RealDictCursor)
dict_cursor.execute("SELECT * FROM users WHERE user_id = %s", (1,))
user = dict_cursor.fetchone()
print(user['email'])  # dict access instead of user[1]

# ── UPDATE ────────────────────────────────────────────────────
cursor.execute(
    "UPDATE users SET email = %s WHERE user_id = %s",
    ("new@email.com", 1)
)
conn.commit()

# ── DELETE ────────────────────────────────────────────────────
cursor.execute("DELETE FROM sessions WHERE expires_at < NOW()")
print(f"Deleted {cursor.rowcount} expired sessions")
conn.commit()
```

---

### Batch Operations (For ML Data Pipelines)

```python
from psycopg2.extras import execute_values, execute_batch

# execute_values — fastest for bulk inserts
records = [
    (1, 'doc1', [0.1, 0.2, 0.3]),
    (2, 'doc2', [0.4, 0.5, 0.6]),
    # ... thousands more
]
execute_values(
    cursor,
    "INSERT INTO document_embeddings (doc_id, content, embedding) VALUES %s",
    records
)
conn.commit()

# execute_batch — runs many statements efficiently
updates = [(0.95, 'run_001'), (0.87, 'run_002')]
execute_batch(
    cursor,
    "UPDATE runs SET accuracy = %s WHERE run_id = %s",
    updates,
    page_size=1000   # process 1000 at a time
)
conn.commit()
```

---

### Context Manager (Best Practice)

```python
# Use context managers for safe transaction handling
with psycopg2.connect("postgresql://...") as conn:
    with conn.cursor() as cur:
        cur.execute("INSERT INTO logs (event) VALUES (%s)", ("model_loaded",))
    # conn.commit() is called automatically when the `with` block exits
    # conn.rollback() is called automatically if an exception occurs
```

---

### Connection Pooling (Production Must-Have)

```python
from psycopg2 import pool

# Create a connection pool (don't open/close connections for every query)
connection_pool = pool.ThreadedConnectionPool(
    minconn=2,
    maxconn=20,
    host="localhost",
    database="mydb",
    user="postgres",
    password="password"
)

def get_user(user_id):
    conn = connection_pool.getconn()
    try:
        with conn.cursor(cursor_factory=psycopg2.extras.RealDictCursor) as cur:
            cur.execute("SELECT * FROM users WHERE user_id = %s", (user_id,))
            return cur.fetchone()
    finally:
        connection_pool.putconn(conn)  # always return connection to pool
```

---

### ⚠️ SQL Injection — Always Use Parameterized Queries

```python
# ❌ NEVER do this — vulnerable to SQL injection
user_input = "1; DROP TABLE users; --"
cursor.execute(f"SELECT * FROM users WHERE id = {user_input}")  # DANGEROUS

# ✅ ALWAYS use parameterized queries with %s
cursor.execute("SELECT * FROM users WHERE id = %s", (user_input,))
# psycopg2 safely escapes the input — no injection possible
```

---

## 19. PostgreSQL for ML / DL / GenAI / Agentic AI

### What & Why

PostgreSQL is not just a database — it's a **central nervous system** for modern AI systems. Here's how it's used in real production AI architectures.

---

### Pattern 1: Feature Store (ML Training Data)

```sql
-- Store preprocessed features for ML training
CREATE TABLE feature_store (
    entity_id       BIGINT NOT NULL,
    entity_type     VARCHAR(50) NOT NULL,    -- 'user', 'product', 'session'
    feature_name    VARCHAR(100) NOT NULL,
    feature_value   DOUBLE PRECISION,
    computed_at     TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (entity_id, entity_type, feature_name)
);

-- Pivot features for model input
SELECT
    entity_id,
    MAX(CASE WHEN feature_name = 'avg_session_duration' THEN feature_value END) AS avg_session_duration,
    MAX(CASE WHEN feature_name = 'purchase_count_30d'   THEN feature_value END) AS purchase_count_30d,
    MAX(CASE WHEN feature_name = 'churn_risk_score'     THEN feature_value END) AS churn_risk_score
FROM feature_store
WHERE entity_type = 'user'
GROUP BY entity_id;
```

### Pattern 2: Experiment Tracking (MLflow-style)

```sql
-- Already created above, but here's how you query it
-- Find top 5 runs across all experiments
SELECT 
    e.name AS experiment,
    r.run_id,
    r.hyperparams->>'learning_rate' AS lr,
    r.hyperparams->>'batch_size' AS batch_size,
    (r.metrics->>'accuracy')::FLOAT AS accuracy,
    (r.metrics->>'val_loss')::FLOAT AS val_loss,
    r.end_time - r.start_time AS duration
FROM runs r
JOIN experiments e ON r.exp_id = e.exp_id
WHERE r.status = 'completed'
ORDER BY (r.metrics->>'accuracy')::FLOAT DESC
LIMIT 5;
```

### Pattern 3: RAG (Retrieval-Augmented Generation) with pgvector

```sql
-- Full RAG system schema
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE knowledge_base (
    chunk_id    SERIAL PRIMARY KEY,
    source_url  TEXT,
    chunk_text  TEXT NOT NULL,
    metadata    JSONB,
    embedding   vector(1536),    -- OpenAI ada-002 dimensions
    created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Create HNSW index (faster, better recall than IVFFlat)
CREATE INDEX idx_kb_embedding_hnsw ON knowledge_base
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);

-- Semantic search query (used in RAG retrieval step)
CREATE OR REPLACE FUNCTION search_knowledge_base(
    query_embedding vector(1536),
    top_k INTEGER DEFAULT 5,
    similarity_threshold FLOAT DEFAULT 0.7
)
RETURNS TABLE(chunk_text TEXT, similarity FLOAT, metadata JSONB) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        kb.chunk_text,
        1 - (kb.embedding <=> query_embedding) AS similarity,
        kb.metadata
    FROM knowledge_base kb
    WHERE 1 - (kb.embedding <=> query_embedding) > similarity_threshold
    ORDER BY kb.embedding <=> query_embedding
    LIMIT top_k;
END;
$$ LANGUAGE plpgsql;
```

### Pattern 4: Agentic AI — Tool Call Logging & Memory

```sql
-- Store agent conversation history
CREATE TABLE agent_sessions (
    session_id  UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    user_id     INTEGER REFERENCES users(user_id),
    agent_type  VARCHAR(100),
    started_at  TIMESTAMPTZ DEFAULT NOW(),
    metadata    JSONB
);

CREATE TABLE agent_messages (
    msg_id      BIGSERIAL PRIMARY KEY,
    session_id  UUID REFERENCES agent_sessions(session_id) ON DELETE CASCADE,
    role        VARCHAR(20) CHECK (role IN ('user', 'assistant', 'system', 'tool')),
    content     TEXT,
    tool_name   VARCHAR(100),   -- which tool was called
    tool_input  JSONB,          -- tool arguments
    tool_output JSONB,          -- tool result
    tokens_used INTEGER,
    created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Agent memory table (persistent facts between sessions)
CREATE TABLE agent_memory (
    memory_id   SERIAL PRIMARY KEY,
    user_id     INTEGER REFERENCES users(user_id),
    memory_key  VARCHAR(200) NOT NULL,
    memory_val  TEXT NOT NULL,
    embedding   vector(1536),   -- embed the memory for semantic retrieval
    created_at  TIMESTAMPTZ DEFAULT NOW(),
    expires_at  TIMESTAMPTZ,
    UNIQUE (user_id, memory_key)
);

-- Retrieve relevant memories for an agent
SELECT memory_key, memory_val,
       1 - (embedding <=> $1::vector) AS relevance
FROM agent_memory
WHERE user_id = $2
  AND (expires_at IS NULL OR expires_at > NOW())
ORDER BY embedding <=> $1::vector
LIMIT 10;
```

### Pattern 5: LLM Output Logging & Evaluation

```sql
CREATE TABLE llm_calls (
    call_id         UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    session_id      UUID,
    model           VARCHAR(100) NOT NULL,  -- 'gpt-4o', 'claude-3-5-sonnet'
    prompt_tokens   INTEGER,
    completion_tokens INTEGER,
    total_tokens    INTEGER,
    latency_ms      INTEGER,
    prompt_hash     TEXT,   -- SHA256 of prompt for dedup/caching
    input_text      TEXT,
    output_text     TEXT,
    cost_usd        DECIMAL(10, 6),
    called_at       TIMESTAMPTZ DEFAULT NOW(),
    metadata        JSONB   -- extra: temperature, top_p, etc.
);

-- LLM cost dashboard query
SELECT
    DATE_TRUNC('day', called_at) AS day,
    model,
    COUNT(*) AS total_calls,
    SUM(total_tokens) AS total_tokens,
    SUM(cost_usd) AS total_cost_usd,
    AVG(latency_ms) AS avg_latency_ms
FROM llm_calls
WHERE called_at > NOW() - INTERVAL '30 days'
GROUP BY DATE_TRUNC('day', called_at), model
ORDER BY day DESC, model;
```

---

## 20. Job-Ready Cheat Sheet

### Essential Commands Reference

```sql
-- ════════════════════════════════════════
--  DATABASE & TABLE MANAGEMENT
-- ════════════════════════════════════════
CREATE DATABASE mydb;
DROP DATABASE mydb;
\c mydb                          -- connect to db (psql)
\dt                              -- list tables
\d table_name                    -- describe table structure
\di                              -- list indexes

-- ════════════════════════════════════════
--  DATA RETRIEVAL
-- ════════════════════════════════════════
SELECT * FROM users;
SELECT id, name FROM users WHERE age > 18;
SELECT * FROM users ORDER BY name ASC LIMIT 10 OFFSET 20;
SELECT DISTINCT city FROM users;
SELECT COUNT(*), AVG(age), MIN(age), MAX(age) FROM users;
SELECT city, COUNT(*) FROM users GROUP BY city HAVING COUNT(*) > 5;

-- ════════════════════════════════════════
--  JOINS
-- ════════════════════════════════════════
-- INNER JOIN — only matching rows from both tables
SELECT u.name, o.total FROM users u
INNER JOIN orders o ON u.user_id = o.user_id;

-- LEFT JOIN — all rows from left, matching from right (NULL if no match)
SELECT u.name, COUNT(o.order_id) FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.name;

-- RIGHT JOIN — all rows from right, matching from left
-- FULL OUTER JOIN — all rows from both tables

-- ════════════════════════════════════════
--  WINDOW FUNCTIONS
-- ════════════════════════════════════════
-- ROW_NUMBER, RANK, DENSE_RANK, LAG, LEAD, SUM OVER, AVG OVER

SELECT 
    user_id,
    order_id,
    total,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY total DESC) AS rank_by_user,
    SUM(total) OVER (PARTITION BY user_id) AS user_lifetime_value,
    LAG(total) OVER (PARTITION BY user_id ORDER BY ordered_at) AS prev_order_total
FROM orders;

-- ════════════════════════════════════════
--  INDEXES (Critical for Performance)
-- ════════════════════════════════════════
CREATE INDEX idx_users_email ON users(email);
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);
CREATE INDEX idx_orders_user ON orders(user_id);   -- FK columns should always be indexed
CREATE INDEX idx_runs_metrics ON runs USING GIN(metrics);  -- JSONB index
CREATE INDEX idx_docs_embedding ON documents USING hnsw(embedding vector_cosine_ops);

DROP INDEX idx_users_email;
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@example.com';

-- ════════════════════════════════════════
--  TRANSACTIONS
-- ════════════════════════════════════════
BEGIN;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;   -- make permanent
-- or:
ROLLBACK; -- undo everything

SAVEPOINT my_savepoint;
ROLLBACK TO SAVEPOINT my_savepoint;

-- ════════════════════════════════════════
--  COMMON FUNCTIONS
-- ════════════════════════════════════════
-- String
LOWER(text), UPPER(text), TRIM(text), LENGTH(text)
CONCAT(a, b), a || b              -- string concatenation
SUBSTRING(text, start, length)
REPLACE(text, from, to)
REGEXP_REPLACE(text, pattern, replacement)
SPLIT_PART(text, delimiter, n)

-- Numeric
ROUND(value, decimals), CEIL(value), FLOOR(value)
ABS(value), MOD(a, b)
RANDOM()                          -- random float 0-1

-- Date/Time
NOW(), CURRENT_DATE, CURRENT_TIMESTAMP
DATE_TRUNC('month', timestamp)    -- truncate to month
DATE_PART('year', timestamp)      -- extract part
AGE(timestamp1, timestamp2)       -- interval between two dates
TO_CHAR(timestamp, 'YYYY-MM-DD')  -- format date as string

-- JSONB
data->'key'                       -- returns JSONB
data->>'key'                      -- returns TEXT
data#>'{key1,key2}'               -- nested access
data @> '{"key": "value"}'        -- containment check
jsonb_array_elements(data->'arr') -- expand JSONB array to rows
```

---

### Data Types Quick Reference

| Category | Type | Example |
|----------|------|---------|
| Integer | `SERIAL`, `INTEGER`, `BIGINT` | `42`, `7000000000` |
| Decimal | `DECIMAL(10,2)`, `DOUBLE PRECISION` | `3.14`, `0.001234` |
| Text | `VARCHAR(n)`, `TEXT` | `'hello'`, large text |
| Boolean | `BOOLEAN` | `TRUE`, `FALSE` |
| Date/Time | `DATE`, `TIMESTAMPTZ` | `2024-01-15`, `2024-01-15 09:00+00` |
| JSON | `JSONB` | `'{"key": "val"}'::JSONB` |
| Array | `TEXT[]`, `INTEGER[]` | `ARRAY['a','b','c']` |
| UUID | `UUID` | `gen_random_uuid()` |
| Vector | `vector(n)` | `'[0.1, 0.2, 0.3]'::vector` |

---

### Constraints Quick Reference

| Constraint | Purpose | Example |
|------------|---------|---------|
| `PRIMARY KEY` | Unique + Not Null identifier | `id SERIAL PRIMARY KEY` |
| `NOT NULL` | Value required | `name TEXT NOT NULL` |
| `UNIQUE` | No duplicates | `email VARCHAR UNIQUE` |
| `FOREIGN KEY` | Referential integrity | `REFERENCES users(id)` |
| `CHECK` | Custom rule | `CHECK (age >= 0)` |
| `DEFAULT` | Auto-fill value | `DEFAULT NOW()` |

---

### The "AI Engineer" Toolkit in PostgreSQL

| Need | PostgreSQL Solution |
|------|-------------------|
| Store ML metadata | `JSONB` columns |
| Vector similarity search | `pgvector` extension |
| Semantic caching | Vector + HNSW index |
| Experiment tracking | Structured tables + JSONB |
| RAG retrieval | `pgvector` + cosine distance |
| Agent memory | Table + vector embeddings |
| LLM cost tracking | `llm_calls` table |
| Feature store | Time-partitioned tables |
| Batch ML inference | `COPY` + `execute_values` |
| Analytical dashboards | Materialized views |

---

### Top 10 Performance Best Practices

1. **Always index foreign key columns** — PostgreSQL does NOT auto-index FKs
2. **Use `TIMESTAMPTZ` not `TIMESTAMP`** — avoid timezone disasters
3. **Use `JSONB` not `JSON`** — it's indexed, binary, and faster
4. **Use connection pooling** — never open a new connection per query
5. **Use `EXPLAIN ANALYZE`** — always check query plans for slow queries
6. **Parameterize all queries** — prevents SQL injection AND enables plan caching
7. **Use partial indexes** for filtered queries: `CREATE INDEX ON orders(status) WHERE status = 'pending'`
8. **Batch inserts with `execute_values`** — 10-100x faster than row-by-row
9. **Use `MATERIALIZED VIEW`** for heavy reports — refresh on schedule
10. **Vacuum regularly** — `VACUUM ANALYZE table_name` keeps statistics fresh

---

*Built for ML / DL / GenAI / Agentic AI engineers. From beginner to job-ready.* 🐘
