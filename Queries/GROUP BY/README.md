
# GROUP BY Clause in PostgreSQL

## 1. Introduction
The **GROUP BY clause** in PostgreSQL is used to **group rows that have the same values in specified columns**.

It is commonly used with **aggregate functions** such as:

- COUNT()
- SUM()
- AVG()
- MAX()
- MIN()

The purpose of `GROUP BY` is to **summarize data by grouping similar rows together**.

---

# 2. Syntax

```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
GROUP BY column_name;
````

### Explanation

* **SELECT** → columns to display
* **FROM** → table name
* **GROUP BY** → groups rows with the same values
* **aggregate_function()** → performs calculations on each group

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

# 4. Example: GROUP BY with COUNT()

```sql
SELECT department, COUNT(*)
FROM students
GROUP BY department;
```

### Output

| department | count |
| ---------- | ----- |
| AIML       | 3     |
| CSE        | 2     |
| ECE        | 1     |

### Explanation

1. The table is grouped by **department**.
2. `COUNT(*)` counts the number of students in each department.
3. Each department becomes a **separate group**.

---

# 5. Example: GROUP BY with AVG()

```sql
SELECT department, AVG(marks)
FROM students
GROUP BY department;
```

### Output

| department | avg   |
| ---------- | ----- |
| AIML       | 87.67 |
| CSE        | 74    |
| ECE        | 65    |

### Explanation

1. Students are grouped by **department**.
2. The **average marks** of students are calculated for each department.

---

# 6. Example: GROUP BY with SUM()

```sql
SELECT department, SUM(marks)
FROM students
GROUP BY department;
```

### Output

| department | sum |
| ---------- | --- |
| AIML       | 263 |
| CSE        | 148 |
| ECE        | 65  |

Explanation:
This calculates the **total marks of students in each department**.

---

# 7. Example: GROUP BY with MAX()

```sql
SELECT department, MAX(marks)
FROM students
GROUP BY department;
```

### Output

| department | max |
| ---------- | --- |
| AIML       | 90  |
| CSE        | 78  |
| ECE        | 65  |

Explanation:
Finds the **highest marks in each department**.

---

# 8. Example: GROUP BY with MIN()

```sql
SELECT department, MIN(marks)
FROM students
GROUP BY department;
```

### Output

| department | min |
| ---------- | --- |
| AIML       | 85  |
| CSE        | 70  |
| ECE        | 65  |

Explanation:
Finds the **lowest marks in each department**.

---

# 9. Importance for AIML

`GROUP BY` is very useful for:

* Data summarization
* Exploratory Data Analysis (EDA)
* Feature engineering
* Dataset statistics

Example:

```sql
SELECT category, COUNT(*)
FROM sales_data
GROUP BY category;
```

This counts how many records exist for each **category**.

---

# 10. Summary

* `GROUP BY` groups rows with similar values.
* It is usually used with **aggregate functions**.
* It helps summarize large datasets.
* Useful for **data analysis and machine learning preprocessing**.


