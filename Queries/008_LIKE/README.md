
# LIKE Operator in PostgreSQL

## 1. Introduction
The **LIKE operator** in PostgreSQL is used in the `WHERE` clause to **search for a specific pattern in a column**.

It is mainly used with **text or string values** to find rows that match a particular pattern.

`LIKE` uses **wildcard characters** to perform pattern matching.

---

# 2. Syntax

```sql
SELECT column1, column2
FROM table_name
WHERE column_name LIKE pattern;
````

### Explanation

* **SELECT** → specifies the columns to retrieve
* **FROM** → specifies the table name
* **WHERE** → filters rows
* **LIKE** → searches for a pattern in a column

---

# 3. Wildcards Used in LIKE

| Wildcard | Meaning                                |
| -------- | -------------------------------------- |
| %        | Represents **zero or more characters** |
| _        | Represents **a single character**      |

---

# 4. Example Table (students)

| id | name  | age | department |
| -- | ----- | --- | ---------- |
| 1  | Ali   | 20  | AIML       |
| 2  | Sara  | 22  | CSE        |
| 3  | John  | 19  | AIML       |
| 4  | Ahmed | 23  | ECE        |
| 5  | Aisha | 21  | AIML       |

---

# 5. LIKE with % (Starts With)

```sql
SELECT *
FROM students
WHERE name LIKE 'A%';
```

### Output

| id | name  | age | department |
| -- | ----- | --- | ---------- |
| 1  | Ali   | 20  | AIML       |
| 4  | Ahmed | 23  | ECE        |
| 5  | Aisha | 21  | AIML       |

### Explanation

`A%` means the name **starts with the letter A**.

---

# 6. LIKE with % (Ends With)

```sql
SELECT *
FROM students
WHERE name LIKE '%a';
```

### Output

| id | name | age | department |
| -- | ---- | --- | ---------- |
| 2  | Sara | 22  | CSE        |

### Explanation

`%a` means the name **ends with the letter a**.

---

# 7. LIKE with % (Contains)

```sql
SELECT *
FROM students
WHERE name LIKE '%h%';
```

### Output

| id | name  | age | department |
| -- | ----- | --- | ---------- |
| 3  | John  | 19  | AIML       |
| 4  | Ahmed | 23  | ECE        |
| 5  | Aisha | 21  | AIML       |

Explanation:
`%h%` finds names that **contain the letter h anywhere**.

---

# 8. LIKE with _ (Single Character)

```sql
SELECT *
FROM students
WHERE name LIKE '_l%';
```

### Output

| id | name | age | department |
| -- | ---- | --- | ---------- |
| 1  | Ali  | 20  | AIML       |

### Explanation

`_l%` means:

* first character → anything
* second character → **l**
* remaining characters → anything

So **Ali** matches this pattern.

---

# 9. NOT LIKE

`NOT LIKE` is used to **exclude a pattern**.

```sql
SELECT *
FROM students
WHERE name NOT LIKE 'A%';
```

### Output

| id | name | age | department |
| -- | ---- | --- | ---------- |
| 2  | Sara | 22  | CSE        |
| 3  | John | 19  | AIML       |

Explanation:
Returns students whose names **do not start with A**.

---

# 10. Importance for AIML

The `LIKE` operator is useful in AI/ML workflows for:

* Searching **text patterns in datasets**
* Filtering **specific text-based records**
* Data preprocessing for **NLP datasets**
* Finding matching **keywords or patterns**

Example:

```sql
SELECT *
FROM reviews
WHERE review_text LIKE '%good%';
```

This retrieves reviews that contain the word **good**.

---

# 11. Summary

| Operator | Purpose                         |
| -------- | ------------------------------- |
| LIKE     | Searches for a specific pattern |
| %        | Represents multiple characters  |
| _        | Represents a single character   |
| NOT LIKE | Excludes a pattern              |

The **LIKE operator is widely used for pattern matching in text data** within PostgreSQL queries.


