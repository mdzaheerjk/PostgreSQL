
# WHERE Clause in PostgreSQL

## 1. Introduction
The **WHERE clause** in PostgreSQL is used to **filter records** from a table based on a specified condition.  
It allows the database to return only those rows that satisfy the given condition.

Without `WHERE`, a `SELECT` statement retrieves **all rows** from the table.

---

# 2. Basic Syntax

```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition;
````

### Explanation

* **SELECT** → Specifies the columns to retrieve
* **FROM** → Specifies the table name
* **WHERE** → Filters rows based on the given condition

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

# 4. Example of WHERE Clause

```sql
SELECT *
FROM students
WHERE department = 'AIML';
```

### Output

| id | name  | age | marks | department |
| -- | ----- | --- | ----- | ---------- |
| 1  | Ali   | 20  | 85    | AIML       |
| 3  | John  | 19  | 90    | AIML       |
| 5  | Aisha | 21  | 88    | AIML       |

Explanation:
The query retrieves only the rows where **department is AIML**.

---

# 5. WHERE with Comparison Operators

## Equal (=)

Used to select rows where a column value **exactly matches** a specified value.

```sql
SELECT *
FROM students
WHERE department = 'CSE';
```

Returns students from the **CSE department**.

---

## Not Equal (!= or <>)

Used to select rows where the value **does not match** the specified value.

```sql
SELECT *
FROM students
WHERE department != 'AIML';
```

Returns students **not belonging to AIML**.

---

## Greater Than (>)

```sql
SELECT name, age
FROM students
WHERE age > 21;
```

Returns students whose **age is greater than 21**.

---

## Less Than (<)

```sql
SELECT name, marks
FROM students
WHERE marks < 80;
```

Returns students with **marks less than 80**.

---

## Greater Than or Equal To (>=)

```sql
SELECT name, marks
FROM students
WHERE marks >= 85;
```

Returns students with **marks greater than or equal to 85**.

---

## Less Than or Equal To (<=)

```sql
SELECT name, age
FROM students
WHERE age <= 20;
```

Returns students whose **age is less than or equal to 20**.

---

# 6. Logical Operators in WHERE

Logical operators allow combining multiple conditions.

## AND

Both conditions must be true.

```sql
SELECT *
FROM students
WHERE department = 'AIML' AND marks > 85;
```

Returns AIML students with marks greater than 85.

---

## OR

At least one condition must be true.

```sql
SELECT *
FROM students
WHERE department = 'AIML' OR department = 'CSE';
```

Returns students from **either AIML or CSE**.

---

## NOT

Negates a condition.

```sql
SELECT *
FROM students
WHERE NOT department = 'ECE';
```

Returns students **not in the ECE department**.

---

# 7. WHERE with Range (BETWEEN)

Used to select values within a range.

```sql
SELECT *
FROM students
WHERE marks BETWEEN 80 AND 90;
```

Returns students whose **marks are between 80 and 90**.

---

# 8. WHERE with Multiple Values (IN)

```sql
SELECT *
FROM students
WHERE department IN ('AIML', 'CSE');
```

Returns students belonging to **AIML or CSE departments**.

---

# 9. WHERE with Pattern Matching (LIKE)

Used for searching text patterns.

```sql
SELECT *
FROM students
WHERE name LIKE 'A%';
```

Returns names **starting with letter A**.

---

# 10. Importance of WHERE for AIML

The WHERE clause is very useful for:

* **Filtering large datasets**
* **Selecting training data**
* **Data cleaning**
* **Data preprocessing**
* **Exploratory Data Analysis (EDA)**

Example in ML workflow:

```sql
SELECT *
FROM sales_data
WHERE year = 2024;
```

This retrieves only **2024 data for analysis or model training**.

---

# 11. Summary

* `WHERE` is used to **filter rows in a query**.
* It works with **comparison operators and logical operators**.
* It allows selecting only the **required data from a table**.
* It is essential for **data extraction and preprocessing in AI/ML workflows**.


