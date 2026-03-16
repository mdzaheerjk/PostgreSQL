
# Subquery in PostgreSQL

## 1. Introduction
A **Subquery** is a query written **inside another SQL query**.  
It is also called an **Inner Query** or **Nested Query**.

The **outer query** uses the result of the **subquery** to perform further operations.

Subqueries are commonly used in:
- SELECT
- WHERE
- FROM clauses

---

# 2. Basic Syntax

```sql
SELECT column_name
FROM table_name
WHERE column_name operator (
    SELECT column_name
    FROM table_name
    WHERE condition
);
````

### Explanation

* **Outer Query** → Main query that returns the final result
* **Subquery** → Inner query executed first
* The result of the subquery is used by the outer query

---

# 3. Example Table (students)

| id | name  | marks | department |
| -- | ----- | ----- | ---------- |
| 1  | Ali   | 85    | AIML       |
| 2  | Sara  | 78    | CSE        |
| 3  | John  | 90    | AIML       |
| 4  | Ahmed | 65    | ECE        |
| 5  | Aisha | 88    | AIML       |

---

# 4. Example: Subquery in WHERE Clause

```sql
SELECT name, marks
FROM students
WHERE marks > (
    SELECT AVG(marks)
    FROM students
);
```

### Explanation

1. The **subquery** calculates the average marks.
2. The **outer query** selects students whose marks are **greater than the average**.

### Result

| name  | marks |
| ----- | ----- |
| John  | 90    |
| Aisha | 88    |

---

# 5. Example: Subquery with IN

```sql
SELECT name
FROM students
WHERE department IN (
    SELECT department
    FROM students
    WHERE marks > 85
);
```

### Explanation

1. Subquery finds departments where marks are **greater than 85**.
2. Outer query selects students from those departments.

---

# 6. Subquery in FROM Clause

A subquery can also act as a **temporary table**.

```sql
SELECT department, avg_marks
FROM (
    SELECT department, AVG(marks) AS avg_marks
    FROM students
    GROUP BY department
) AS dept_avg;
```

### Explanation

1. Subquery calculates **average marks for each department**.
2. Outer query retrieves the results from this temporary table.

---

# 7. Types of Subqueries

### 1. Single Row Subquery

Returns only **one row**.

Example:

```sql
SELECT *
FROM students
WHERE marks > (
    SELECT AVG(marks)
    FROM students
);
```

---

### 2. Multiple Row Subquery

Returns **multiple rows**.

Example:

```sql
SELECT *
FROM students
WHERE department IN (
    SELECT department
    FROM students
);
```

---

# 8. Importance for AIML

Subqueries are useful for:

* Filtering datasets using calculated values
* Data preprocessing
* Extracting complex data conditions
* Advanced data analysis

Example:

```sql
SELECT *
FROM sales
WHERE sales_amount > (
    SELECT AVG(sales_amount)
    FROM sales
);
```

This retrieves records **above average sales**.

---

# 9. Summary

* A **Subquery is a query inside another query**.
* The **inner query runs first**.
* The outer query uses the result of the subquery.
* Used for **complex filtering and data analysis**.


