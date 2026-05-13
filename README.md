# 🐘 PostgreSQL — Complete Learning Resource

> Structured, production-ready PostgreSQL notes covering queries, database creation, and core concepts. Perfect for beginners through advanced users, with real-world examples and AI/ML applications.

![License](https://img.shields.io/badge/license-MIT-blue)
![Stars](https://img.shields.io/github/stars/mdzaheerjk/PostgreSQL?style=social)
![Last Updated](https://img.shields.io/badge/last_updated-May%202026-brightgreen)

---

## 📚 What's Inside

This repository contains **comprehensive, well-organized PostgreSQL documentation** structured into two main learning paths:

### 📖 **01_SEARCH** — Querying & Data Retrieval
Complete reference for reading and analyzing data without any DDL operations. Pure **SELECT power**.

**Topics covered:**
- ✅ Basic SELECT statements & column operations
- ✅ WHERE clauses with all filtering operators (LIKE, BETWEEN, IN, regex)
- ✅ ORDER BY, LIMIT, and pagination strategies
- ✅ Aggregate functions (COUNT, SUM, AVG, GROUP_CONCAT, etc.)
- ✅ GROUP BY & HAVING with advanced features (GROUPING SETS, ROLLUP, CUBE)
- ✅ JOINs (INNER, LEFT, RIGHT, FULL OUTER, CROSS, SELF, LATERAL)
- ✅ Subqueries (scalar, IN, EXISTS, derived tables, correlated)
- ✅ CTEs (WITH clauses) including recursive queries
- ✅ Window functions (ranking, aggregation, value offsets)
- ✅ String, numeric, and date/time functions
- ✅ Conditional expressions (CASE, COALESCE, NULLIF)
- ✅ Array and JSON/JSONB operations
- ✅ Full-text search capabilities
- ✅ Set operations (UNION, INTERSECT, EXCEPT)
- ✅ Utility queries (stats, deduplication, running totals)

**[→ Full 01_SEARCH guide](./01_SEARCH/)**

---

### 🛠️ **02_CREATE** — Database Design & Manipulation
Complete guide to building, modifying, and populating databases from the ground up.

**Topics covered:**
- ✅ All PostgreSQL data types (numeric, string, boolean, date/time, JSON, arrays, UUID, vectors)
- ✅ Primary keys & foreign keys with relationship patterns
- ✅ Constraints (NOT NULL, UNIQUE, CHECK, DEFAULT)
- ✅ CREATE TABLE with beginner to advanced examples
- ✅ INSERT (single/batch) and INSERT ON CONFLICT (upsert)
- ✅ UPDATE statements and bulk operations
- ✅ DELETE and TRUNCATE
- ✅ ALTER TABLE (add/drop columns, constraints, renames)
- ✅ DROP TABLE safely
- ✅ Stored procedures & functions (PL/pgSQL)
- ✅ Views and data abstraction
- ✅ Import/Export (CSV, JSON)
- ✅ Python integration (psycopg2)
- ✅ ML/AI use cases (embeddings, experiment tracking, RAG)

**[→ Full 02_CREATE guide](./02_CREATE/)**

---

## 🚀 Quick Start

### For Beginners
1. Start with **[01_SEARCH/README.md](./01_SEARCH/)** — learn how to query data first
2. Then move to **[02_CREATE/README.md](./02_CREATE/)** — learn how to build tables

### For Intermediate Users
- Jump to specific sections you need
- Use as a reference while building applications
- Study the real-world ML examples

### For Advanced Users
- Reference the window functions, recursive CTEs, and full-text search sections
- See advanced patterns for experiment tracking and embeddings
- Explore PL/pgSQL stored procedures

---

## 🎯 Use Cases

### 💼 **General Database Design**
- Building scalable schema from scratch
- Optimizing query performance
- Managing relationships between tables
- Enforcing data integrity with constraints

### 🤖 **AI/ML Applications**
- Storing embeddings with pgvector
- Building experiment tracking systems (MLflow-style)
- Managing training runs and hyperparameters
- Retrieval-Augmented Generation (RAG) with vector search
- Storing LLM prompts and responses

### 📊 **Data Analysis**
- Complex aggregations and grouping
- Time-series analysis with window functions
- Full-text search on document collections
- JSON/JSONB querying for semi-structured data

### 🔌 **Integration**
- Python data pipelines (psycopg2 examples)
- Bulk data imports/exports
- Application backend design

---

## 📝 Key Features of This Resource

✨ **Beginner-Friendly**
- Plain English explanations ("What & Why")
- Every concept includes syntax and real examples
- Progressive complexity from basics to advanced

✨ **Production-Ready**
- Best practices throughout
- Safety patterns (transactional DDL, data validation)
- Performance considerations (indexing hints, query optimization)

✨ **Comprehensive**
- Nearly 1000+ lines of reference material
- All major PostgreSQL features covered
- ML/AI and modern use cases included

✨ **Well-Organized**
- Logical structure for easy navigation
- Clear section headers and table of contents
- Consistent formatting across all guides

---

## 📂 Repository Structure
