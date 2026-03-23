# PostgreSQL — Complete Query & Search Reference

> Covers every way to **read and search data** in PostgreSQL. No DDL (CREATE/ALTER/DROP). Pure SELECT power.

---

## Basic SELECT

```sql
SELECT * FROM employees;
SELECT name, age, salary FROM employees;
SELECT name AS full_name, salary AS pay FROM employees;   -- column aliases
SELECT DISTINCT department FROM employees;                 -- unique values
SELECT DISTINCT ON (department) name, department          -- first row per group
  FROM employees ORDER BY department, salary DESC;
```

---

## WHERE — Filtering Rows

### Comparison operators

```sql
SELECT * FROM employees WHERE salary > 50000;
SELECT * FROM employees WHERE salary >= 50000;
SELECT * FROM employees WHERE salary <> 50000;   -- not equal (also !=)
SELECT * FROM employees WHERE salary = 50000;
SELECT * FROM employees WHERE name = 'Alice';    -- string: single quotes only
SELECT * FROM employees WHERE hired_at < '2023-01-01';
```

### Logical operators

```sql
WHERE age > 25 AND salary > 60000
WHERE department = 'Sales' OR department = 'Marketing'
WHERE NOT active
WHERE (salary > 80000 OR bonus > 10000) AND department = 'Engineering'
```

### NULL checks

```sql
WHERE manager_id IS NULL
WHERE manager_id IS NOT NULL
WHERE COALESCE(bonus, 0) > 5000    -- treat NULL as 0 for comparison
```

### BETWEEN (inclusive)

```sql
WHERE salary BETWEEN 40000 AND 80000
WHERE hired_at BETWEEN '2023-01-01' AND '2023-12-31'
WHERE age NOT BETWEEN 20 AND 30
```

### IN / NOT IN

```sql
WHERE department IN ('Sales', 'Marketing', 'Support')
WHERE id IN (1, 5, 10, 42)
WHERE department NOT IN ('HR', 'Finance')
WHERE id IN (SELECT employee_id FROM managers)   -- subquery
```

### LIKE / ILIKE (pattern matching)

```sql
WHERE name LIKE 'A%'          -- starts with A (case-sensitive)
WHERE name LIKE '%son'        -- ends with son
WHERE name LIKE '%ali%'       -- contains ali
WHERE name LIKE '_ohn'        -- any single char then ohn
WHERE name ILIKE 'alice%'     -- case-INsensitive (PostgreSQL extension)
WHERE name NOT LIKE 'temp%'
WHERE email LIKE '%@gmail.com'
```

### SIMILAR TO (SQL regex)

```sql
WHERE name SIMILAR TO '(Alice|Bob)%'     -- alternation
WHERE code SIMILAR TO '[A-Z]{3}[0-9]+'   -- char classes
```

### Regex with `~`

```sql
WHERE name ~ '^A'            -- case-sensitive regex match
WHERE name ~* '^alice'       -- case-insensitive regex match
WHERE name !~ 'temp'         -- does NOT match regex
WHERE name !~* 'temp'        -- case-insensitive NOT match
WHERE phone ~ '^\+?[0-9\s\-]{7,15}$'
WHERE description ~* '\b(error|fail|crash)\b'
```

---

## ORDER BY

```sql
SELECT * FROM employees ORDER BY salary;                  -- ascending (default)
SELECT * FROM employees ORDER BY salary DESC;
SELECT * FROM employees ORDER BY department ASC, salary DESC;
SELECT * FROM employees ORDER BY LENGTH(name);            -- expression
SELECT * FROM employees ORDER BY salary DESC NULLS LAST;  -- NULLs at end
SELECT * FROM employees ORDER BY salary ASC  NULLS FIRST; -- NULLs at start
SELECT * FROM employees ORDER BY 2, 3;                    -- by column position (fragile)
```

---

## LIMIT & OFFSET (Pagination)

```sql
SELECT * FROM employees LIMIT 10;                 -- first 10 rows
SELECT * FROM employees LIMIT 10 OFFSET 20;       -- rows 21–30
SELECT * FROM employees ORDER BY id LIMIT 10 OFFSET 40;  -- page 5 (size 10)

-- Keyset / cursor pagination (faster on large tables)
SELECT * FROM employees
  WHERE id > 1000                  -- last seen id from previous page
  ORDER BY id
  LIMIT 20;
```

---

## Aggregate Functions

```sql
SELECT COUNT(*) FROM employees;                          -- total rows
SELECT COUNT(manager_id) FROM employees;                 -- non-NULL count
SELECT COUNT(DISTINCT department) FROM employees;

SELECT SUM(salary) FROM employees;
SELECT AVG(salary) FROM employees;
SELECT ROUND(AVG(salary), 2) FROM employees;
SELECT MIN(salary), MAX(salary) FROM employees;
SELECT STDDEV(salary) FROM employees;                    -- population std dev
SELECT STDDEV_SAMP(salary) FROM employees;               -- sample std dev
SELECT VARIANCE(salary) FROM employees;
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) FROM employees;  -- median
SELECT PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY salary) FROM employees;
SELECT MODE() WITHIN GROUP (ORDER BY department) FROM employees;

-- String aggregation
SELECT STRING_AGG(name, ', ' ORDER BY name) FROM employees;
SELECT STRING_AGG(DISTINCT department, ', ' ORDER BY department) FROM employees;

-- Array aggregation
SELECT ARRAY_AGG(name ORDER BY salary DESC) FROM employees;
SELECT ARRAY_AGG(DISTINCT department) FROM employees;

-- JSON aggregation
SELECT JSON_AGG(row_to_json(employees)) FROM employees;
SELECT JSONB_AGG(name) FROM employees;
```

---

## GROUP BY & HAVING

```sql
-- Basic grouping
SELECT department, COUNT(*) AS headcount
FROM employees
GROUP BY department;

-- Multiple columns
SELECT department, job_title, AVG(salary) AS avg_sal
FROM employees
GROUP BY department, job_title;

-- HAVING — filter groups (like WHERE but after aggregation)
SELECT department, AVG(salary)
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;

SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) >= 5;

-- HAVING with WHERE (both can coexist)
SELECT department, AVG(salary)
FROM employees
WHERE active = true                -- filters rows BEFORE grouping
GROUP BY department
HAVING AVG(salary) > 60000;        -- filters groups AFTER aggregation

-- GROUPING SETS — multiple groupings in one query
SELECT department, job_title, SUM(salary)
FROM employees
GROUP BY GROUPING SETS (
    (department, job_title),
    (department),
    ()                             -- grand total row
);

-- ROLLUP — hierarchical subtotals
SELECT department, job_title, SUM(salary)
FROM employees
GROUP BY ROLLUP(department, job_title);

-- CUBE — all combinations of subtotals
SELECT region, department, SUM(salary)
FROM employees
GROUP BY CUBE(region, department);
```

---

## JOINs

### INNER JOIN (only matching rows)

```sql
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

### LEFT JOIN (all left rows, NULLs for unmatched right)

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;

-- Find rows with NO match (anti-join pattern)
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;
```

### RIGHT JOIN

```sql
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

### FULL OUTER JOIN

```sql
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;

-- Rows that have NO match on EITHER side
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id
WHERE e.id IS NULL OR d.id IS NULL;
```

### CROSS JOIN (cartesian product)

```sql
SELECT a.color, b.size
FROM colors a
CROSS JOIN sizes b;
```

### SELF JOIN

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### Multiple JOINs

```sql
SELECT e.name, d.department_name, l.city
FROM employees e
JOIN departments d ON e.department_id = d.id
JOIN locations  l ON d.location_id = l.id
WHERE l.country = 'US';
```

### JOIN with additional conditions

```sql
SELECT e.name, p.project_name
FROM employees e
JOIN project_assignments pa
  ON e.id = pa.employee_id
  AND pa.role = 'lead'              -- condition on the join itself
JOIN projects p ON pa.project_id = p.id;
```

### LATERAL JOIN (each row can reference outer query)

```sql
SELECT e.name, recent.title
FROM employees e
JOIN LATERAL (
    SELECT title
    FROM job_history
    WHERE employee_id = e.id
    ORDER BY start_date DESC
    LIMIT 1
) AS recent ON true;
```

---

## Subqueries

### In WHERE

```sql
-- Scalar subquery
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- IN subquery
SELECT * FROM employees
WHERE department_id IN (
    SELECT id FROM departments WHERE region = 'West'
);

-- NOT IN subquery (careful with NULLs — use NOT EXISTS instead)
SELECT * FROM employees
WHERE id NOT IN (
    SELECT employee_id FROM managers WHERE active = true
);

-- EXISTS (more efficient than IN for large subqueries)
SELECT * FROM employees e
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.employee_id = e.id AND o.total > 1000
);

-- NOT EXISTS (preferred anti-join)
SELECT * FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM managers m WHERE m.employee_id = e.id
);
```

### In FROM (derived table)

```sql
SELECT dept, avg_sal
FROM (
    SELECT department AS dept, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
) AS dept_stats
WHERE avg_sal > 70000;
```

### In SELECT (scalar subquery)

```sql
SELECT name,
       salary,
       (SELECT AVG(salary) FROM employees) AS company_avg,
       salary - (SELECT AVG(salary) FROM employees) AS diff_from_avg
FROM employees;
```

### Correlated subquery (references outer query)

```sql
SELECT name, salary, department
FROM employees e1
WHERE salary = (
    SELECT MAX(salary)
    FROM employees e2
    WHERE e2.department = e1.department   -- references outer row
);
```

---

## CTEs — Common Table Expressions (WITH)

```sql
-- Basic CTE
WITH dept_stats AS (
    SELECT department, AVG(salary) AS avg_sal, COUNT(*) AS cnt
    FROM employees
    GROUP BY department
)
SELECT * FROM dept_stats WHERE avg_sal > 70000;

-- Multiple CTEs
WITH
    high_earners AS (
        SELECT * FROM employees WHERE salary > 100000
    ),
    their_departments AS (
        SELECT DISTINCT d.*
        FROM departments d
        JOIN high_earners h ON h.department_id = d.id
    )
SELECT * FROM their_departments;

-- Recursive CTE — org chart / tree traversal
WITH RECURSIVE org_chart AS (
    -- Base case: top-level managers (no manager)
    SELECT id, name, manager_id, 1 AS depth
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive step: employees reporting to already-found rows
    SELECT e.id, e.name, e.manager_id, oc.depth + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY depth, name;

-- Recursive CTE — path string
WITH RECURSIVE path_cte AS (
    SELECT id, name, manager_id, name::text AS path
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id, p.path || ' > ' || e.name
    FROM employees e
    JOIN path_cte p ON e.manager_id = p.id
)
SELECT * FROM path_cte;
```

---

## Window Functions

Window functions compute values across a set of rows **without collapsing them** into one (unlike GROUP BY).

### Syntax

```sql
function_name() OVER (
    PARTITION BY col          -- group (optional)
    ORDER BY col              -- ordering within group (optional)
    ROWS/RANGE frame_clause   -- window frame (optional)
)
```

### Ranking functions

```sql
SELECT name, department, salary,
    ROW_NUMBER()  OVER (PARTITION BY department ORDER BY salary DESC) AS row_num,
    RANK()        OVER (PARTITION BY department ORDER BY salary DESC) AS rank,
    DENSE_RANK()  OVER (PARTITION BY department ORDER BY salary DESC) AS dense_rank,
    NTILE(4)      OVER (ORDER BY salary)                              AS quartile,
    PERCENT_RANK()OVER (ORDER BY salary)                              AS pct_rank,
    CUME_DIST()   OVER (ORDER BY salary)                              AS cum_dist
FROM employees;

-- RANK vs DENSE_RANK vs ROW_NUMBER
-- salaries: 100, 100, 80
-- RANK:        1,   1,  3   (gap after tie)
-- DENSE_RANK:  1,   1,  2   (no gap)
-- ROW_NUMBER:  1,   2,  3   (always unique)
```

### Value / offset functions

```sql
SELECT name, department, salary,
    LAG(salary)  OVER (PARTITION BY department ORDER BY hired_at) AS prev_salary,
    LEAD(salary) OVER (PARTITION BY department ORDER BY hired_at) AS next_salary,
    LAG(salary, 2) OVER (ORDER BY hired_at) AS two_rows_back,
    LAG(salary, 1, 0) OVER (ORDER BY hired_at) AS prev_or_zero,  -- default 0

    FIRST_VALUE(salary) OVER (PARTITION BY department ORDER BY salary DESC) AS top_sal,
    LAST_VALUE(salary)  OVER (
        PARTITION BY department ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS bottom_sal,
    NTH_VALUE(salary, 2) OVER (PARTITION BY department ORDER BY salary DESC) AS second_sal
FROM employees;
```

### Aggregate window functions

```sql
SELECT name, salary, department,
    SUM(salary)   OVER (PARTITION BY department) AS dept_total,
    AVG(salary)   OVER (PARTITION BY department) AS dept_avg,
    COUNT(*)      OVER (PARTITION BY department) AS dept_count,
    MAX(salary)   OVER ()                        AS company_max,

    -- Running total (cumulative sum)
    SUM(salary) OVER (ORDER BY hired_at
                      ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total,

    -- 3-row moving average
    AVG(salary) OVER (ORDER BY hired_at
                      ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg_3,

    -- Percentage of department total
    ROUND(salary * 100.0 / SUM(salary) OVER (PARTITION BY department), 2) AS pct_of_dept

FROM employees;
```

### Window frame clauses

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW   -- from start to here
ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING           -- ±3 rows
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING   -- from here to end
RANGE BETWEEN INTERVAL '7 days' PRECEDING AND CURRENT ROW  -- time range
```

### FILTER with window functions

```sql
SELECT department,
    COUNT(*) FILTER (WHERE salary > 80000) AS high_earner_count,
    AVG(salary) FILTER (WHERE active = true) AS active_avg
FROM employees
GROUP BY department;
```

### Named windows (reuse definitions)

```sql
SELECT name, salary,
    RANK()    OVER dept_win AS rank,
    AVG(salary) OVER dept_win AS dept_avg
FROM employees
WINDOW dept_win AS (PARTITION BY department ORDER BY salary DESC);
```

---

## String Functions

```sql
-- Case
UPPER(name);  LOWER(name);  INITCAP(name)

-- Length
LENGTH(name);  CHAR_LENGTH(name);  BIT_LENGTH(name);  OCTET_LENGTH(name)

-- Trimming
TRIM('  hello  ')           -- 'hello'
LTRIM(name);  RTRIM(name)
TRIM(LEADING '0' FROM code)
TRIM(BOTH 'x' FROM 'xxhelloxx')

-- Padding
LPAD(code, 5, '0')          -- '00042'
RPAD(name, 20, '.')

-- Concatenation
name || ' ' || surname
CONCAT(first, ' ', last)
CONCAT_WS(', ', city, state, country)   -- with separator (skips NULLs)

-- Substring extraction
SUBSTRING(name FROM 1 FOR 3)     -- first 3 chars
SUBSTR(name, 1, 3)               -- same
LEFT(name, 3);  RIGHT(name, 4)

-- Position / search
POSITION('ali' IN name)          -- 1-based, 0 if not found
STRPOS(name, 'ali')              -- same
CHARINDEX(name, 'ali')

-- Replace & remove
REPLACE(name, 'Alice', 'Alicia')
TRANSLATE(code, 'ABC', '123')    -- char-by-char substitution
REGEXP_REPLACE(name, '\s+', ' ', 'g')   -- global regex replace
OVERLAY(name PLACING 'XX' FROM 3 FOR 2) -- replace chars at position

-- Split
SPLIT_PART('a,b,c', ',', 2)     -- 'b' (1-based)
STRING_TO_ARRAY('a,b,c', ',')   -- '{a,b,c}'  array

-- Reverse
REVERSE(name)

-- Repeat / space
REPEAT('ha', 3)                  -- 'hahaha'
SPACE(5)                         -- '     '

-- Format
FORMAT('Hello %s, you are %s years old', name, age)
TO_CHAR(salary, 'FM$999,999.00')
TO_CHAR(hired_at, 'YYYY-MM-DD')

-- Pattern matching in SELECT
name LIKE '%son'
name ILIKE '%ali%'
name ~ '^\d+'
name ~* 'alice'

-- Levenshtein edit distance (needs fuzzystrmatch extension)
SELECT levenshtein('Alice', 'Alicia');      -- 2
SELECT similarity('Alice', 'Alicia');       -- pg_trgm extension
```

---

## Numeric Functions

```sql
ABS(value);  SIGN(value)
ROUND(value, 2);  TRUNC(value, 2);  CEIL(value);  FLOOR(value)
MOD(17, 5)                  -- 2  (remainder)
POWER(2, 10)                -- 1024
SQRT(144)                   -- 12
CBRT(27)                    -- 3 (cube root)
EXP(1)                      -- e
LN(value);  LOG(10, value);  LOG(value)  -- natural / base-10 / base-10
PI()                        -- 3.14159...
RANDOM()                    -- 0.0 to 1.0
FLOOR(RANDOM() * 100)::INT  -- random int 0–99

-- Rounding modes
ROUND(2.5)    -- 3  (round half up)
ROUND(3.5)    -- 4
ROUND(2.55, 1) -- 2.6

-- Integer division and modulo
17 / 5        -- 3  (integer division when both are int)
17.0 / 5      -- 3.4
17 % 5        -- 2

-- Infinity / NaN
'Infinity'::float
'-Infinity'::float
'NaN'::float
```

---

## Date & Time Functions

```sql
-- Current timestamps
NOW()                            -- timestamptz
CURRENT_TIMESTAMP                -- same as NOW()
CURRENT_DATE                     -- date only
CURRENT_TIME                     -- time only
LOCALTIME                        -- time without timezone
LOCALTIMESTAMP                   -- timestamp without timezone
CLOCK_TIMESTAMP()                -- changes within a statement (NOW() does not)

-- Extracting parts
EXTRACT(YEAR  FROM hired_at)
EXTRACT(MONTH FROM hired_at)
EXTRACT(DAY   FROM hired_at)
EXTRACT(HOUR  FROM hired_at)
EXTRACT(DOW   FROM hired_at)     -- day of week 0=Sun 6=Sat
EXTRACT(DOY   FROM hired_at)     -- day of year 1–366
EXTRACT(WEEK  FROM hired_at)     -- ISO week number
EXTRACT(EPOCH FROM hired_at)     -- Unix timestamp (seconds)
EXTRACT(QUARTER FROM hired_at)

DATE_PART('month', hired_at)     -- same as EXTRACT but returns float8

-- Truncation
DATE_TRUNC('month', hired_at)    -- first day of month at midnight
DATE_TRUNC('year',  hired_at)
DATE_TRUNC('week',  hired_at)    -- Monday
DATE_TRUNC('hour',  hired_at)
DATE_TRUNC('day',   hired_at)

-- Arithmetic
hired_at + INTERVAL '30 days'
hired_at - INTERVAL '1 year'
hired_at + INTERVAL '1 month 15 days 2 hours'
hired_at::date + 7              -- add 7 days to a date

-- Difference
AGE(end_date, start_date)                    -- interval: '2 years 3 mons 5 days'
AGE(hired_at)                                -- from today
end_date - start_date                        -- integer days (for date type)
EXTRACT(EPOCH FROM (end_ts - start_ts))     -- difference in seconds
EXTRACT(DAY FROM NOW() - hired_at)          -- days difference

-- Construction
MAKE_DATE(2024, 12, 25)
MAKE_TIMESTAMP(2024, 12, 25, 9, 0, 0)
MAKE_INTERVAL(years => 1, months => 6)
TO_DATE('25-12-2024', 'DD-MM-YYYY')
TO_TIMESTAMP('2024-12-25 09:00:00', 'YYYY-MM-DD HH24:MI:SS')

-- Formatting
TO_CHAR(hired_at, 'YYYY-MM-DD')
TO_CHAR(hired_at, 'Day, DD Month YYYY')
TO_CHAR(hired_at, 'YYYY-"W"IW')            -- ISO week

-- Timezone
hired_at AT TIME ZONE 'UTC'
hired_at AT TIME ZONE 'America/New_York'
TIMEZONE('UTC', hired_at)

-- Useful filters
WHERE DATE_TRUNC('month', hired_at) = DATE_TRUNC('month', NOW())  -- this month
WHERE hired_at >= NOW() - INTERVAL '7 days'                        -- last 7 days
WHERE EXTRACT(DOW FROM hired_at) IN (0, 6)                        -- weekends
WHERE hired_at::date = CURRENT_DATE                               -- today
WHERE hired_at BETWEEN '2024-01-01' AND '2024-12-31'
```

---

## Conditional Expressions

### CASE

```sql
-- Simple CASE
SELECT name,
    CASE grade
        WHEN 'A' THEN 'Excellent'
        WHEN 'B' THEN 'Good'
        WHEN 'C' THEN 'Average'
        ELSE 'Below average'
    END AS grade_label
FROM students;

-- Searched CASE
SELECT name, salary,
    CASE
        WHEN salary > 100000 THEN 'High'
        WHEN salary > 60000  THEN 'Mid'
        WHEN salary > 30000  THEN 'Low'
        ELSE 'Entry'
    END AS band
FROM employees;

-- CASE in WHERE
SELECT * FROM employees
WHERE CASE WHEN active THEN salary > 50000 ELSE salary > 30000 END;

-- CASE in ORDER BY
ORDER BY
    CASE WHEN department = 'Engineering' THEN 1
         WHEN department = 'Sales'       THEN 2
         ELSE 3
    END;

-- CASE in aggregate
SELECT department,
    SUM(CASE WHEN gender = 'F' THEN 1 ELSE 0 END) AS female_count,
    SUM(CASE WHEN gender = 'M' THEN 1 ELSE 0 END) AS male_count,
    COUNT(*) FILTER (WHERE gender = 'F') AS female_count2   -- cleaner alternative
FROM employees GROUP BY department;
```

### COALESCE, NULLIF, GREATEST, LEAST

```sql
COALESCE(bonus, 0)                   -- first non-NULL value
COALESCE(nickname, first_name, 'Unknown')
NULLIF(value, 0)                     -- returns NULL if value = 0
NULLIF(status, 'active')             -- returns NULL if status = 'active'
GREATEST(a, b, c)                    -- max of values (ignores NULLs)
LEAST(a, b, c)                       -- min of values (ignores NULLs)
IIF(condition, true_val, false_val)   -- not standard in PG; use CASE
```

---

## Array Queries

```sql
-- Array literals
SELECT ARRAY[1, 2, 3];
SELECT '{1,2,3}'::int[];

-- Access
SELECT tags[1] FROM posts;             -- 1-based indexing
SELECT tags[1:3] FROM posts;           -- slice
SELECT tags[ARRAY_LENGTH(tags, 1)] FROM posts;  -- last element

-- Operators
WHERE tags @> ARRAY['python']          -- contains
WHERE tags <@ ARRAY['python','sql']    -- is contained by
WHERE tags && ARRAY['python','java']   -- overlap (any element in common)
WHERE 'python' = ANY(tags)             -- any element equals
WHERE 'python' = ALL(tags)             -- all elements equal

-- Functions
ARRAY_LENGTH(tags, 1)                  -- length of dim 1
ARRAY_DIMS(tags)                       -- '[1:5]'
CARDINALITY(tags)                      -- total elements (all dims)
ARRAY_UPPER(tags, 1)                   -- upper bound of dim 1
ARRAY_LOWER(tags, 1)                   -- lower bound

ARRAY_APPEND(tags, 'rust')             -- add to end
ARRAY_PREPEND('go', tags)             -- add to start
ARRAY_CAT(tags1, tags2)               -- concatenate
tags1 || tags2                         -- same with operator
ARRAY_REMOVE(tags, 'java')            -- remove all occurrences
ARRAY_REPLACE(tags, 'java', 'kotlin')
ARRAY_POSITION(tags, 'python')        -- index of first match (1-based)
ARRAY_POSITIONS(tags, 'python')       -- all matching indices
ARRAY_TO_STRING(tags, ', ')           -- join to string

-- unnest — expand array to rows
SELECT id, UNNEST(tags) AS tag FROM posts;
SELECT UNNEST(ARRAY[1,2,3]) AS n;

-- Generate subscripts
SELECT id, i, tags[i]
FROM posts, GENERATE_SUBSCRIPTS(tags, 1) AS i;
```

---

## JSON / JSONB Queries

```sql
-- Operators
data->'key'                       -- get JSON value (returns JSON)
data->>'key'                      -- get as text
data->'key'->'nested'             -- chained access
data->>0                          -- array element as text
data#>'{a,b,c}'                   -- path access (returns JSON)
data#>>'{a,b,c}'                  -- path access (returns text)

-- Existence operators (JSONB only)
data ? 'key'                      -- key exists
data ?| ARRAY['k1','k2']          -- any key exists
data ?& ARRAY['k1','k2']          -- all keys exist
data @> '{"status":"active"}'     -- contains JSON
data <@ '{"a":1,"b":2}'           -- is contained by

-- Functions
JSONB_EXTRACT_PATH(data, 'a', 'b')          -- same as data#>'{a,b}'
JSONB_EXTRACT_PATH_TEXT(data, 'a', 'b')     -- as text
JSON_ARRAY_LENGTH(data->'items')
JSONB_ARRAY_LENGTH(data->'items')
JSONB_OBJECT_KEYS(data)                     -- set of top-level keys
JSONB_EACH(data)                            -- expand to (key, value) rows
JSONB_EACH_TEXT(data)                       -- values as text
JSONB_ARRAY_ELEMENTS(data->'arr')           -- expand array to rows
JSONB_ARRAY_ELEMENTS_TEXT(data->'arr')      -- as text
JSONB_TYPEOF(data)                          -- 'object','array','string','number','boolean','null'
JSON_BUILD_OBJECT('name', name, 'age', age)
JSON_BUILD_ARRAY(1, 2, name)
ROW_TO_JSON(employees)                      -- whole row as JSON
JSON_AGG(row_to_json(employees))            -- aggregate rows to JSON array
JSONB_SET(data, '{key}', '"new_value"')     -- update a path (useful in SELECT)
JSONB_STRIP_NULLS(data)                     -- remove null-valued keys

-- Practical queries
SELECT data->>'name' AS name FROM users;
SELECT * FROM users WHERE data->>'status' = 'active';
SELECT * FROM users WHERE (data->>'age')::int > 30;
SELECT * FROM users WHERE data @> '{"role":"admin"}';
SELECT * FROM products WHERE data->'tags' ? 'sale';
SELECT * FROM products WHERE data->'tags' @> '["python","web"]';

-- Expand nested array of objects
SELECT id, JSONB_ARRAY_ELEMENTS(data->'items')->>'name' AS item_name
FROM orders;

-- Build JSON in SELECT
SELECT JSON_BUILD_OBJECT(
    'id', id,
    'name', name,
    'stats', JSON_BUILD_OBJECT('salary', salary, 'dept', department)
) FROM employees;
```

---

## Full-Text Search

```sql
-- to_tsvector: converts text to tsvector (lexemes)
-- to_tsquery:  converts search term to tsquery

-- Basic search
SELECT * FROM articles
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'postgresql');

-- Multiple terms: & = AND, | = OR, ! = NOT
WHERE to_tsvector('english', title || ' ' || body)
  @@ to_tsquery('english', 'database & (search | query) & !deprecated');

-- Phrase search
WHERE to_tsvector('english', body)
  @@ phraseto_tsquery('english', 'full text search');

-- websearch_to_tsquery — parses natural language query
WHERE to_tsvector('english', body)
  @@ websearch_to_tsquery('english', 'postgresql full text -deprecated');

-- Ranking results
SELECT title,
    ts_rank(to_tsvector('english', body), query) AS rank
FROM articles,
    to_tsquery('english', 'search & engine') AS query
WHERE to_tsvector('english', body) @@ query
ORDER BY rank DESC;

-- ts_rank_cd — cover density ranking (rewards closer matches)
SELECT title,
    ts_rank_cd(to_tsvector('english', body), query) AS rank
FROM articles,
    to_tsquery('english', 'full & text') AS query
WHERE to_tsvector('english', body) @@ query
ORDER BY rank DESC;

-- Highlight / headline
SELECT ts_headline('english', body,
    to_tsquery('english', 'search'),
    'StartSel=<b>, StopSel=</b>, MaxWords=50, MinWords=15'
) FROM articles WHERE ...;

-- Prefix search
WHERE to_tsvector('english', name) @@ to_tsquery('english', 'postgre:*');

-- Using a generated tsvector column (after CREATE INDEX)
WHERE search_vector @@ to_tsquery('english', 'query');

-- Strip lexemes to see what gets indexed
SELECT to_tsvector('english', 'The quick brown fox jumps over lazy dogs');
-- 'brown':3 'dog':9 'fox':4 'jump':5 'lazi':8 'quick':2
```

---

## Set Operations

```sql
-- UNION — combine and deduplicate
SELECT name FROM employees
UNION
SELECT name FROM contractors;

-- UNION ALL — combine keeping duplicates (faster)
SELECT name, 'employee' AS type FROM employees
UNION ALL
SELECT name, 'contractor'        FROM contractors;

-- INTERSECT — rows in BOTH result sets
SELECT product_id FROM orders_2023
INTERSECT
SELECT product_id FROM orders_2024;

-- EXCEPT — rows in first but NOT second
SELECT product_id FROM all_products
EXCEPT
SELECT product_id FROM discontinued;

-- INTERSECT ALL / EXCEPT ALL — with duplicates
SELECT product_id FROM orders INTERSECT ALL SELECT product_id FROM returns;
```

---

## Useful Utility Queries

### Information about data

```sql
-- Quick stats
SELECT COUNT(*), MIN(salary), MAX(salary), AVG(salary), STDDEV(salary)
FROM employees;

-- Distribution of a column
SELECT department, COUNT(*) AS n,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 1) AS pct
FROM employees
GROUP BY department ORDER BY n DESC;

-- Find duplicates
SELECT email, COUNT(*) AS cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Find first / latest row per group
SELECT DISTINCT ON (department)
    department, name, salary
FROM employees
ORDER BY department, salary DESC;   -- DISTINCT ON keeps first row after ORDER BY

-- Nth highest value
SELECT salary FROM employees
ORDER BY salary DESC OFFSET 2 LIMIT 1;   -- 3rd highest

-- Running total
SELECT date, amount,
    SUM(amount) OVER (ORDER BY date) AS running_total
FROM transactions;

-- Gaps in a sequence
SELECT id + 1 AS gap_start
FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM employees WHERE id = e.id + 1
) AND id < (SELECT MAX(id) FROM employees);

-- Pivot (manual using CASE)
SELECT
    department,
    SUM(CASE WHEN EXTRACT(QUARTER FROM hired_at) = 1 THEN salary END) AS q1,
    SUM(CASE WHEN EXTRACT(QUARTER FROM hired_at) = 2 THEN salary END) AS q2,
    SUM(CASE WHEN EXTRACT(QUARTER FROM hired_at) = 3 THEN salary END) AS q3,
    SUM(CASE WHEN EXTRACT(QUARTER FROM hired_at) = 4 THEN salary END) AS q4
FROM employees
GROUP BY department;
```

### GENERATE_SERIES (generate rows)

```sql
SELECT GENERATE_SERIES(1, 10) AS n;
SELECT GENERATE_SERIES(0, 100, 10) AS n;
SELECT GENERATE_SERIES(
    '2024-01-01'::date,
    '2024-12-01'::date,
    '1 month'::interval
) AS month;

-- Fill in missing dates (calendar join)
WITH calendar AS (
    SELECT d::date AS day
    FROM GENERATE_SERIES('2024-01-01', '2024-12-31', '1 day'::interval) d
)
SELECT c.day, COALESCE(SUM(o.amount), 0) AS revenue
FROM calendar c
LEFT JOIN orders o ON o.order_date = c.day
GROUP BY c.day ORDER BY c.day;
```

### Type casting

```sql
CAST(value AS int)
value::int                  -- PostgreSQL shorthand
value::text
value::numeric(10,2)
value::date
value::timestamp
value::boolean
value::json
value::jsonb
'2024-01-01'::date
'3.14'::float
```

---

## Query Patterns Cheat Sheet

```sql
-- Top N per group (window function)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
) t WHERE rn <= 3;

-- Median per group
SELECT department,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees GROUP BY department;

-- Self-comparison (% change from prev row)
SELECT date, revenue,
    LAG(revenue) OVER (ORDER BY date) AS prev_rev,
    ROUND((revenue - LAG(revenue) OVER (ORDER BY date))
          * 100.0 / NULLIF(LAG(revenue) OVER (ORDER BY date), 0), 2) AS pct_change
FROM daily_sales;

-- Deduplicate keeping latest row
SELECT DISTINCT ON (email) *
FROM users
ORDER BY email, created_at DESC;

-- Count + percentage in one query
SELECT status,
    COUNT(*) AS n,
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2) AS pct
FROM orders GROUP BY status;

-- Boolean flags as counts
SELECT
    COUNT(*) FILTER (WHERE salary > 80000)  AS high_earners,
    COUNT(*) FILTER (WHERE active = false)  AS inactive,
    COUNT(*) FILTER (WHERE bonus IS NULL)   AS no_bonus
FROM employees;

-- Conditional aggregation
SELECT department,
    COUNT(*) AS total,
    SUM(CASE WHEN gender='F' THEN 1 END) AS female,
    AVG(CASE WHEN active THEN salary END) AS active_avg_salary
FROM employees GROUP BY department;
```

---

## Quick Reference Card

```
Basic           SELECT cols FROM tbl WHERE cond ORDER BY col LIMIT n OFFSET m
Filter ops      =  !=  <  >  <=  >=  BETWEEN  IN  LIKE  ILIKE  SIMILAR TO  ~ ~* !~ !~*
NULL            IS NULL  IS NOT NULL  COALESCE  NULLIF
Aggregates      COUNT SUM AVG MIN MAX STDDEV PERCENTILE_CONT STRING_AGG ARRAY_AGG
Grouping        GROUP BY  HAVING  GROUPING SETS  ROLLUP  CUBE
Joins           INNER  LEFT  RIGHT  FULL OUTER  CROSS  SELF  LATERAL
Subqueries      WHERE col IN (...)   EXISTS (...)   NOT EXISTS   scalar in SELECT
CTE             WITH name AS (...) SELECT ... — WITH RECURSIVE for trees
Window          OVER (PARTITION BY ... ORDER BY ... ROWS BETWEEN ...)
                ROW_NUMBER  RANK  DENSE_RANK  NTILE  LAG  LEAD  FIRST_VALUE  LAST_VALUE
Set ops         UNION [ALL]   INTERSECT [ALL]   EXCEPT [ALL]
String          UPPER LOWER TRIM LPAD RPAD CONCAT_WS SUBSTRING POSITION REPLACE
                REGEXP_REPLACE SPLIT_PART STRING_TO_ARRAY LEVENSHTEIN
Date/time       EXTRACT  DATE_TRUNC  AGE  NOW  DATE_PART  TO_CHAR  AT TIME ZONE
JSON            -> ->> #> #>> ? ?| ?& @> <@ JSONB_EACH JSONB_ARRAY_ELEMENTS
Array           @> <@ && ANY ALL UNNEST ARRAY_AGG ARRAY_LENGTH ARRAY_APPEND
Full-text       to_tsvector @@ to_tsquery phraseto_tsquery websearch_to_tsquery ts_rank
Utility         GENERATE_SERIES  DISTINCT ON  FILTER(WHERE...)  CAST / ::
```

---

*Covers PostgreSQL 14–16. Docs: https://www.postgresql.org/docs/current/*
```
