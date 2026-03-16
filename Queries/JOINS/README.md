
# SQL JOINS (PostgreSQL) – Comprehensive Notes

## 1. Introduction to JOINS

A **JOIN** in SQL is used to **combine rows from two or more tables based on a related column between them**.

In relational databases, data is usually **stored in multiple tables**. JOINS allow us to retrieve related information from those tables.

Example:
- Table 1: Users
- Table 2: Orders

A JOIN allows you to answer questions like:
- Which user made which order?
- Which users never placed an order?

---

# Example Tables

## Table 1: Registrations

| id | name |
|----|------|
|1|Bobby|
|2|Xavier|
|3|Albert|
|4|Zack|

## Table 2: Logins

| id | name |
|----|------|
|1|Yolanda|
|2|Bobby|
|3|William|
|4|Albert|

Important observation:

Only **Bobby and Albert exist in both tables**.

---

# Types of SQL JOINS

The most common joins are:

1. INNER JOIN
2. LEFT JOIN (LEFT OUTER JOIN)
3. RIGHT JOIN
4. FULL OUTER JOIN
5. SELF JOIN

---

# 1. INNER JOIN

## Definition

An **INNER JOIN returns only the rows that have matching values in both tables**.

If there is **no match**, the row is **not included**.

---
![SQL Joins Diagram](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*bqXfGzo3bDw7THRRuuxhHg.png)
## Syntax

```sql
SELECT *
FROM Registrations
INNER JOIN Logins
ON Registrations.name = Logins.name;
````

---

## Result

| Registrations.id | Registrations.name | Logins.id | Logins.name |
| ---------------- | ------------------ | --------- | ----------- |
| 1                | Albert             | 2         | Albert      |
| 3                | Bobby              | 4         | Bobby       |

---

## Explanation

Only **Albert and Bobby appear in both tables**, so only those rows are returned.

Think of INNER JOIN as:

> **Intersection of both tables**

---

# 2. LEFT JOIN (LEFT OUTER JOIN)

## Definition

A **LEFT JOIN returns all rows from the left table** and the matching rows from the right table.

If no match exists, **NULL values appear for the right table columns**.

---


## Syntax

```sql
SELECT *
FROM Registrations
LEFT JOIN Logins
ON Registrations.name = Logins.name;
```

---

## Result

| id | name   | id   | name   |
| -- | ------ | ---- | ------ |
| 1  | Albert | 2    | Albert |
| 2  | Xavier | NULL | NULL   |
| 3  | Bobby  | 4    | Bobby  |
| 4  | Zack   | NULL | NULL   |

---

## Explanation

All records from **Registrations** appear.

If the name doesn't exist in Logins:

* Xavier
* Zack

their login fields become **NULL**.

---

# 3. RIGHT JOIN

## Definition

A **RIGHT JOIN returns all rows from the right table and matching rows from the left table**.

If no match exists, **NULL values appear for the left table**.

---

## Syntax

```sql
SELECT *
FROM Registrations
RIGHT JOIN Logins
ON Registrations.name = Logins.name;
```

---

## Result

| id   | name   | id | name    |
| ---- | ------ | -- | ------- |
| 1    | Albert | 2  | Albert  |
| 3    | Bobby  | 4  | Bobby   |
| NULL | NULL   | 1  | Yolanda |
| NULL | NULL   | 3  | William |

---

## Explanation

All rows from **Logins** appear.

Yolanda and William are not in Registrations, so left columns become **NULL**.

---

# 4. FULL OUTER JOIN

## Definition

A **FULL OUTER JOIN returns all rows from both tables**.

If a match exists → combined row
If no match → NULL for the missing side

---
![FULL OUTER JOIN](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*vHBTEYV08I4h_rMOVZpYfg.png)

## Syntax

```sql
SELECT *
FROM Registrations
FULL OUTER JOIN Logins
ON Registrations.name = Logins.name;
```

---

## Result

| id   | name   | id   | name    |
| ---- | ------ | ---- | ------- |
| 1    | Albert | 2    | Albert  |
| 2    | Xavier | NULL | NULL    |
| 3    | Bobby  | 4    | Bobby   |
| 4    | Zack   | NULL | NULL    |
| NULL | NULL   | 1    | Yolanda |
| NULL | NULL   | 3    | William |

---

## Explanation

All records from both tables appear.

Unmatched rows show **NULL values**.

---

# 5. SELF JOIN

## Definition

A **SELF JOIN joins a table with itself**.

It is used when a table contains **related data within itself**.

Example:
Employees table where each employee has a supervisor.

---

## Example Table: Employees

| empid | name | supervisor_id |
| ----- | ---- | ------------- |
| 1     | Bob  | 3             |
| 2     | Amy  | 3             |
| 3     | John | NULL          |

---

## Query

```sql
SELECT e.name AS employee, s.name AS supervisor
FROM Employees e
JOIN Employees s
ON e.supervisor_id = s.empid;
```

---

## Result

| employee | supervisor |
| -------- | ---------- |
| Bob      | John       |
| Amy      | John       |

---

# Using JOIN with WHERE for Filtering

Sometimes we use JOIN with **NULL checks** to find unmatched rows.

---

# Example: Records only in Registrations

```sql
SELECT *
FROM Registrations
LEFT JOIN Logins
ON Registrations.name = Logins.name
WHERE Logins.id IS NULL;
```

---

## Result

| id | name   |
| -- | ------ |
| 2  | Xavier |
| 4  | Zack   |

Explanation:

These users **registered but never logged in**.

---

# Example: Unique records in both tables

```sql
SELECT *
FROM Registrations
FULL OUTER JOIN Logins
ON Registrations.name = Logins.name
WHERE Registrations.id IS NULL
OR Logins.id IS NULL;
```

---

Result includes:

* Xavier
* Zack
* Yolanda
* William

These exist **only in one table**.

---

# JOIN vs UNION

## JOIN

* Combines **columns from multiple tables**
* Requires a **relationship condition**

Example:

```sql
SELECT *
FROM Orders
JOIN Customers
ON Orders.customer_id = Customers.id;
```

---

## UNION

* Combines **rows from multiple queries**

Example:

```sql
SELECT name FROM TableA
UNION
SELECT name FROM TableB;
```

---

# UNION vs UNION ALL

| Feature            | UNION  | UNION ALL |
| ------------------ | ------ | --------- |
| Duplicates removed | Yes    | No        |
| Speed              | Slower | Faster    |

---

# WHERE vs HAVING

| Clause | Purpose                      |
| ------ | ---------------------------- |
| WHERE  | Filters rows before grouping |
| HAVING | Filters after GROUP BY       |

Example:

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

---

# Interview-Level Insight

Most SQL interview questions test:

* INNER JOIN logic
* LEFT JOIN with NULL filtering
* Multi-table joins
* Aggregation with JOIN

Example:

Find users who **registered but never logged in**.

```sql
SELECT r.name
FROM Registrations r
LEFT JOIN Logins l
ON r.name = l.name
WHERE l.name IS NULL;
```

---

# Real Data Science Usage

In AI / ML pipelines, JOINS are used to:

* Merge datasets
* Combine features
* Join user activity tables
* Join transactional datasets

Example:

```sql
SELECT users.id, users.name, orders.amount
FROM users
JOIN orders
ON users.id = orders.user_id;
```

This merges **user data with order data**.

---

# Quick Summary

| Join Type       | Result                    |
| --------------- | ------------------------- |
| INNER JOIN      | Only matching rows        |
| LEFT JOIN       | All left rows + matches   |
| RIGHT JOIN      | All right rows + matches  |
| FULL OUTER JOIN | All rows from both tables |
| SELF JOIN       | Table joined with itself  |

---

# Final Tip

Don't memorize joins blindly.

Always think:

1️⃣ What rows should appear?
2️⃣ Which table is primary?
3️⃣ What happens when there is no match?

That's how good SQL engineers actually solve joins.

```


