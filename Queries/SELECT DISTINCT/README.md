
# SELECT DISTINCT in PostgreSQL

## Introduction
The **SELECT DISTINCT** statement in PostgreSQL is used to retrieve **unique values** from a column or a combination of columns.  
It removes duplicate rows from the result set and displays only **distinct (non-repeated) values**.

This is useful when working with datasets where duplicate values exist and only **unique records** are needed.

---

# Syntax

```sql
SELECT DISTINCT column1, column2, ...
FROM table_name;
````

**Explanation**

* `SELECT DISTINCT` → removes duplicate values
* `column1, column2` → columns for which unique values are required
* `FROM` → specifies the table

---

# Example Table

| id | name  | department |
| -- | ----- | ---------- |
| 1  | Ali   | AIML       |
| 2  | Sara  | CSE        |
| 3  | John  | AIML       |
| 4  | Ahmed | CSE        |
| 5  | Aisha | AIML       |

---

# Example 1: DISTINCT on One Column

```sql
SELECT DISTINCT department
FROM students;
```

### Output

| department |
| ---------- |
| AIML       |
| CSE        |

Even though **AIML and CSE appear multiple times**, `DISTINCT` shows them only once.

---

# Example 2: DISTINCT on Multiple Columns

```sql
SELECT DISTINCT name, department
FROM students;
```

This returns **unique combinations of name and department**.

---

# Example 3: DISTINCT with SELECT *

```sql
SELECT DISTINCT *
FROM students;
```

This removes **duplicate rows** if the table contains identical records.

---

# Importance for AIML

`SELECT DISTINCT` is useful for:

* Finding **unique categories in datasets**
* Removing duplicate data before **machine learning preprocessing**
* Identifying **unique labels or classes**
* Data cleaning during **exploratory data analysis (EDA)**

---

# Summary

* `SELECT DISTINCT` is used to retrieve **unique values**.
* It removes **duplicate records** from the result.
* Can be used on **single or multiple columns**.
* Helpful in **data cleaning and preprocessing for AI/ML datasets**.


