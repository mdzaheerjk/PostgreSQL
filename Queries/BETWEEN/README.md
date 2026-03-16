
# BETWEEN Operator in PostgreSQL

## 1. Introduction
The **BETWEEN operator** in PostgreSQL is used in the `WHERE` clause to **select values within a specific range**.

It checks whether a column value lies **between two given values** (inclusive of both limits).

`BETWEEN` can be used with:
- Numbers
- Dates
- Text values

---

# 2. Syntax

```sql
SELECT column1, column2, ...
FROM table_name
WHERE column_name BETWEEN value1 AND value2;
````

### Explanation

* **SELECT** → columns to retrieve
* **FROM** → table name
* **WHERE** → filters rows
* **BETWEEN** → selects values within a range
* **value1 AND value2** → starting and ending values of the range

Important:
`BETWEEN` includes **both boundary values**.

---

# 3. Example Table (students)

| id | name  | age | marks | department |
| -- | ----- | --- | ----- | ---------- |
| 1  | Ali   | 20  | 85    | AIML       |
| 2  | Sara  | 22  | 78    | CSE        |
| 3  | John  | 19  | 90    | AIML       |
| 4  | Ahmed | 23  | 65    | ECE        |
| 5  | Aisha | 21  | 88    | AIML       |

---

# 4. BETWEEN with Numbers

```sql
SELECT *
FROM students
WHERE marks BETWEEN 80 AND 90;
```

### Output

| id | name  | age | marks | department |
| -- | ----- | --- | ----- | ---------- |
| 1  | Ali   | 20  | 85    | AIML       |
| 3  | John  | 19  | 90    | AIML       |
| 5  | Aisha | 21  | 88    | AIML       |

### Explanation

The query returns students whose **marks are between 80 and 90**, including both 80 and 90.

---

# 5. BETWEEN with Age

```sql
SELECT name, age
FROM students
WHERE age BETWEEN 20 AND 22;
```

### Output

| name  | age |
| ----- | --- |
| Ali   | 20  |
| Sara  | 22  |
| Aisha | 21  |

Explanation:
Returns students whose **age lies between 20 and 22**.

---

# 6. NOT BETWEEN

`NOT BETWEEN` is used to select values **outside the given range**.

## Syntax

```sql
SELECT column1
FROM table_name
WHERE column_name NOT BETWEEN value1 AND value2;
```

---

## Example

```sql
SELECT name, marks
FROM students
WHERE marks NOT BETWEEN 80 AND 90;
```

### Output

| name  | marks |
| ----- | ----- |
| Sara  | 78    |
| Ahmed | 65    |

Explanation:
Returns students whose **marks are outside the range 80–90**.

---

# 7. BETWEEN with Dates (Example)

```sql
SELECT *
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

Explanation:
Returns orders placed **between January 1 and December 31, 2024**.

---

# 8. Importance for AIML

`BETWEEN` is useful in AI/ML data preparation for:

* Selecting data within a **specific range**
* Filtering training datasets
* Data preprocessing
* Exploratory data analysis

Example:

```sql
SELECT *
FROM sales_data
WHERE sales BETWEEN 1000 AND 5000;
```

---

# 9. Summary

| Operator    | Purpose                       |
| ----------- | ----------------------------- |
| BETWEEN     | Select values within a range  |
| NOT BETWEEN | Select values outside a range |

`BETWEEN` is commonly used to **filter numerical, textual, or date ranges in SQL queries**.


