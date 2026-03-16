
# ILIKE Operator in PostgreSQL

## 1. Introduction
The **ILIKE operator** in PostgreSQL is used for **case-insensitive pattern matching**.

It works the same as `LIKE`, but it **ignores letter case**.

- `LIKE` → case-sensitive  
- `ILIKE` → case-insensitive

---

# 2. Syntax

```sql
SELECT column1, column2
FROM table_name
WHERE column_name ILIKE pattern;
````

Explanation:

* `SELECT` → columns to retrieve
* `FROM` → table name
* `WHERE` → filter condition
* `ILIKE` → case-insensitive pattern matching

---

# 3. Example Table (students)

| id | name  | department |
| -- | ----- | ---------- |
| 1  | Ali   | AIML       |
| 2  | sara  | CSE        |
| 3  | JOHN  | AIML       |
| 4  | Ahmed | ECE        |
| 5  | aisha | AIML       |

---

# 4. Example: Case-Insensitive Search

```sql
SELECT *
FROM students
WHERE name ILIKE 'a%';
```

### Output

| id | name  | department |
| -- | ----- | ---------- |
| 1  | Ali   | AIML       |
| 4  | Ahmed | ECE        |
| 5  | aisha | AIML       |

Explanation:
`a%` matches **Ali, Ahmed, and aisha** because `ILIKE` ignores case.

---

# 5. Difference Between LIKE and ILIKE

### Using LIKE

```sql
SELECT *
FROM students
WHERE name LIKE 'a%';
```

Output:

```
aisha
```

Explanation:
`LIKE` is **case-sensitive**, so it only matches lowercase `a`.

---

### Using ILIKE

```sql
SELECT *
FROM students
WHERE name ILIKE 'a%';
```

Output:

```
Ali
Ahmed
aisha
```

Explanation:
`ILIKE` ignores case and matches **A or a**.

---

# 6. ILIKE with Wildcards

| Pattern | Meaning            |
| ------- | ------------------ |
| 'a%'    | starts with a or A |
| '%a'    | ends with a or A   |
| '%h%'   | contains h or H    |

Example:

```sql
SELECT *
FROM students
WHERE name ILIKE '%h%';
```

Returns names containing **h or H**.

---

# 7. Importance for AIML

`ILIKE` is useful when working with **text datasets**, such as:

* Searching words in **NLP datasets**
* Filtering **text features**
* Handling **case-insensitive text data**

Example:

```sql
SELECT *
FROM reviews
WHERE review_text ILIKE '%excellent%';
```

---

# 8. Summary

| Operator | Purpose                           |
| -------- | --------------------------------- |
| LIKE     | Case-sensitive pattern matching   |
| ILIKE    | Case-insensitive pattern matching |

`ILIKE` is mainly used when searching text without worrying about **uppercase or lowercase letters**.


