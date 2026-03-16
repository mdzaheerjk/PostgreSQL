
# ORDER BY Clause in PostgreSQL

## 1. Introduction
The **ORDER BY clause** in PostgreSQL is used to **sort the result of a query** based on one or more columns.

By default, PostgreSQL sorts data in **ascending order (ASC)**.  
You can also sort data in **descending order (DESC)**.

---

# 2. Syntax

```sql
SELECT column1, column2, ...
FROM table_name
ORDER BY column_name [ASC | DESC];
````

### Explanation

* **SELECT** → specifies the columns to retrieve
* **FROM** → specifies the table name
* **ORDER BY** → sorts the result based on a column
* **ASC** → ascending order (default)
* **DESC** → descending order

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

# 4. ORDER BY in Ascending Order (Default)

```sql
SELECT name, marks
FROM students
ORDER BY marks;
```

### Output

| name  | marks |
| ----- | ----- |
| Ahmed | 65    |
| Sara  | 78    |
| Ali   | 85    |
| Aisha | 88    |
| John  | 90    |

### Explanation

The query sorts students **from lowest marks to highest marks**.

---

# 5. ORDER BY in Descending Order

```sql
SELECT name, marks
FROM students
ORDER BY marks DESC;
```

### Output

| name  | marks |
| ----- | ----- |
| John  | 90    |
| Aisha | 88    |
| Ali   | 85    |
| Sara  | 78    |
| Ahmed | 65    |

### Explanation

The query sorts students **from highest marks to lowest marks**.

---

# 6. ORDER BY Multiple Columns

PostgreSQL allows sorting by **more than one column**.

```sql
SELECT name, department, marks
FROM students
ORDER BY department ASC, marks DESC;
```

### Explanation

1. Data is first sorted by **department (ascending)**
2. Within each department, students are sorted by **marks (descending)**

---

# 7. ORDER BY with LIMIT

`ORDER BY` is often used with `LIMIT` to get **top records**.

```sql
SELECT name, marks
FROM students
ORDER BY marks DESC
LIMIT 3;
```

### Output

| name  | marks |
| ----- | ----- |
| John  | 90    |
| Aisha | 88    |
| Ali   | 85    |

Explanation:
Returns the **top 3 students with highest marks**.

---

# 8. Importance for AIML

The `ORDER BY` clause is useful in AI and ML workflows for:

* Sorting datasets
* Finding **top or lowest values**
* Ranking results
* Organizing data before analysis

Example:

```sql
SELECT *
FROM sales_data
ORDER BY sales DESC;
```

This sorts the data by **highest sales first**.

---

# 9. Summary

| Keyword  | Purpose                   |
| -------- | ------------------------- |
| ORDER BY | Sorts query results       |
| ASC      | Sorts in ascending order  |
| DESC     | Sorts in descending order |

The **ORDER BY clause helps organize query results in a meaningful order**, making it easier to analyze data.


