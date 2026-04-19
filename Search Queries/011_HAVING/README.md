
# HAVING Clause in PostgreSQL

## 1. Introduction
The **HAVING clause** in PostgreSQL is used to **filter grouped data after using the GROUP BY clause**.

While the **WHERE clause filters rows before grouping**, the **HAVING clause filters groups after aggregation functions are applied**.

It is commonly used with aggregate functions such as:

- COUNT()
- SUM()
- AVG()
- MAX()
- MIN()

---

# 2. Syntax

```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1
HAVING condition;
````

### Explanation

* **SELECT** → specifies the columns to retrieve
* **FROM** → specifies the table name
* **GROUP BY** → groups rows based on a column
* **HAVING** → filters the grouped results

---

# 3. Example Table (students)

| id | name  | marks | department |
| -- | ----- | ----- | ---------- |
| 1  | Ali   | 85    | AIML       |
| 2  | Sara  | 78    | CSE        |
| 3  | John  | 90    | AIML       |
| 4  | Ahmed | 65    | ECE        |
| 5  | Aisha | 88    | AIML       |
| 6  | David | 70    | CSE        |

---

# 4. Example Using HAVING with COUNT()

```sql
SELECT department, COUNT(*)
FROM students
GROUP BY department
HAVING COUNT(*) > 2;
```

### Output

| department | count |
| ---------- | ----- |
| AIML       | 3     |

### Explanation

1. The table is grouped by **department**.
2. COUNT(*) calculates the number of students in each department.
3. HAVING filters departments where **student count is greater than 2**.

So only **AIML** appears because it has **3 students**.

---

# 5. Example Using HAVING with AVG()

```sql
SELECT department, AVG(marks)
FROM students
GROUP BY department
HAVING AVG(marks) > 80;
```

### Output

| department | avg   |
| ---------- | ----- |
| AIML       | 87.66 |

### Explanation

1. Students are grouped by **department**.
2. The **average marks** are calculated for each department.
3. HAVING filters departments where **average marks are greater than 80**.

---

# 6. Difference Between WHERE and HAVING

| Clause | Purpose                                   |
| ------ | ----------------------------------------- |
| WHERE  | Filters rows before grouping              |
| HAVING | Filters grouped results after aggregation |

Example:

```sql
SELECT department, AVG(marks)
FROM students
WHERE marks > 70
GROUP BY department
HAVING AVG(marks) > 80;
```

Explanation:

* **WHERE marks > 70** filters rows first
* **GROUP BY department** groups the remaining rows
* **HAVING AVG(marks) > 80** filters the grouped results

---

# 7. Importance for AIML

The HAVING clause is useful for:

* Analyzing grouped datasets
* Filtering aggregated results
* Finding patterns in grouped data
* Preparing summarized data for machine learning analysis

Example:

```sql
SELECT category, COUNT(*)
FROM sales_data
GROUP BY category
HAVING COUNT(*) > 100;
```

This retrieves categories with **more than 100 records**.

---

# 8. Summary

* `HAVING` is used to **filter grouped data**.
* It is used **after GROUP BY**.
* Works mainly with **aggregate functions**.
* Helps analyze summarized datasets in SQL queries.


