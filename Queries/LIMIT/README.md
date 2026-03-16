
# LIMIT Clause in PostgreSQL

## 1. Introduction
The **LIMIT clause** in PostgreSQL is used to **restrict the number of rows returned** by a `SELECT` query.

When a table contains a large number of records, `LIMIT` helps retrieve **only a specific number of rows** from the result set.

It is commonly used for:
- Viewing sample data
- Pagination
- Reducing query result size

---

# 2. Syntax

```sql
SELECT column1, column2, ...
FROM table_name
LIMIT number_of_rows;
````

### Explanation

* **SELECT** → Specifies the columns to retrieve
* **FROM** → Specifies the table name
* **LIMIT** → Restricts the number of rows returned

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

# 4. Basic Example of LIMIT

```sql
SELECT *
FROM students
LIMIT 3;
```

### Output

| id | name | age | marks | department |
| -- | ---- | --- | ----- | ---------- |
| 1  | Ali  | 20  | 85    | AIML       |
| 2  | Sara | 22  | 78    | CSE        |
| 3  | John | 19  | 90    | AIML       |

Explanation:
Only the **first 3 rows** from the table are returned.

---

# 5. LIMIT with Specific Columns

```sql
SELECT name, marks
FROM students
LIMIT 2;
```

### Output

| name | marks |
| ---- | ----- |
| Ali  | 85    |
| Sara | 78    |

Explanation:
The query returns **only two rows** with the selected columns.

---

# 6. LIMIT with ORDER BY

`LIMIT` is often used with `ORDER BY` to retrieve **top or bottom records**.

```sql
SELECT name, marks
FROM students
ORDER BY marks DESC
LIMIT 2;
```

### Output

| name  | marks |
| ----- | ----- |
| John  | 90    |
| Aisha | 88    |

Explanation:
The query sorts students by **marks in descending order** and returns the **top 2 students**.

---

# 7. LIMIT with OFFSET

`OFFSET` is used with `LIMIT` to skip a number of rows before returning results.

```sql
SELECT *
FROM students
LIMIT 2 OFFSET 2;
```

### Output

| id | name  | age | marks | department |
| -- | ----- | --- | ----- | ---------- |
| 3  | John  | 19  | 90    | AIML       |
| 4  | Ahmed | 23  | 65    | ECE        |

Explanation:

* `OFFSET 2` skips the first **2 rows**
* `LIMIT 2` returns the **next 2 rows**

---

# 8. Importance of LIMIT for AIML

`LIMIT` is useful in AI and ML workflows for:

* **Previewing datasets**
* **Testing queries on small samples**
* **Fetching top records**
* **Reducing processing time when exploring large datasets**

Example in data analysis:

```sql
SELECT *
FROM sales_data
LIMIT 100;
```

This retrieves **only the first 100 rows** for quick inspection.

---

# 9. Summary

* `LIMIT` restricts the number of rows returned by a query.
* It helps retrieve **only a specific number of records**.
* Often used with **ORDER BY** to get top results.
* Can be combined with **OFFSET** for pagination.


