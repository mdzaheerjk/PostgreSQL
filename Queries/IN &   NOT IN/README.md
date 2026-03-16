
# IN Operator and NOT IN Operator in PostgreSQL

## 1. Introduction
The **IN operator** in PostgreSQL is used in the `WHERE` clause to check whether a value **matches any value in a given list**.

The **NOT IN operator** is used to select rows where the value **does not match any value in the given list**.

These operators make queries **simpler and more readable** when checking multiple values.

---

# 2. Example Table (students)

| id | name  | age | marks | department |
|----|-------|-----|------|-----------|
| 1 | Ali | 20 | 85 | AIML |
| 2 | Sara | 22 | 78 | CSE |
| 3 | John | 19 | 90 | AIML |
| 4 | Ahmed | 23 | 65 | ECE |
| 5 | Aisha | 21 | 88 | AIML |

---

# 3. IN Operator

The **IN operator** selects rows where the column value **matches any value in the list**.

## Syntax

```sql
SELECT column1, column2
FROM table_name
WHERE column_name IN (value1, value2, value3);
````

---

## Example

```sql
SELECT *
FROM students
WHERE department IN ('AIML', 'CSE');
```

### Output

| id | name  | age | marks | department |
| -- | ----- | --- | ----- | ---------- |
| 1  | Ali   | 20  | 85    | AIML       |
| 2  | Sara  | 22  | 78    | CSE        |
| 3  | John  | 19  | 90    | AIML       |
| 5  | Aisha | 21  | 88    | AIML       |

### Explanation

The query retrieves students whose **department is either AIML or CSE**.

Instead of writing:

```sql
WHERE department = 'AIML' OR department = 'CSE'
```

we can use the **IN operator**.

---

# 4. NOT IN Operator

The **NOT IN operator** selects rows where the column value **does not match any value in the list**.

## Syntax

```sql
SELECT column1, column2
FROM table_name
WHERE column_name NOT IN (value1, value2);
```

---

## Example

```sql
SELECT *
FROM students
WHERE department NOT IN ('AIML', 'CSE');
```

### Output

| id | name  | age | marks | department |
| -- | ----- | --- | ----- | ---------- |
| 4  | Ahmed | 23  | 65    | ECE        |

### Explanation

The query returns students **not belonging to AIML or CSE departments**.

---

# 5. Another Example

```sql
SELECT name, age
FROM students
WHERE age IN (19, 21);
```

### Output

| name  | age |
| ----- | --- |
| John  | 19  |
| Aisha | 21  |

Explanation:
Returns students whose **age is either 19 or 21**.

---

# 6. Advantages of IN Operator

* Simplifies multiple **OR conditions**
* Makes queries **clean and readable**
* Efficient when checking **multiple values**

Example comparison:

Without IN:

```sql
WHERE department = 'AIML'
OR department = 'CSE'
OR department = 'ECE'
```

With IN:

```sql
WHERE department IN ('AIML','CSE','ECE')
```

---

# 7. Importance for AIML

The `IN` operator is useful in AI/ML workflows for:

* Selecting data from **specific categories**
* Filtering datasets by **multiple labels**
* Preparing training data from **selected classes**

Example:

```sql
SELECT *
FROM dataset
WHERE class_label IN ('cat','dog','horse');
```

---

# 8. Summary

| Operator | Purpose                                              |
| -------- | ---------------------------------------------------- |
| IN       | Select rows that match **any value in a list**       |
| NOT IN   | Select rows that **do not match values in the list** |

Both operators are commonly used with the **WHERE clause to filter multiple values easily**.


