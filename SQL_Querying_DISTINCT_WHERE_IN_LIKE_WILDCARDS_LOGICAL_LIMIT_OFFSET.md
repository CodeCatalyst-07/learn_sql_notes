# SQL Querying — Detailed Exam Notes

## Topics Covered

1. `DISTINCT`
2. `WHERE`
3. `IN`
4. `LIKE`
5. Wildcards `%` and `_`
6. Difference between `%` and `_`
7. Pattern examples: `%A`, `%a`, `%it%`, `A__`
8. `AND`, `OR`, `NOT`
9. `LIMIT`
10. `OFFSET`
11. Combined examples, comparisons, common mistakes, and quick revision

---

# Introduction

These notes explain SQL querying concepts in **English + Hinglish** using a consistent `students` table. The aim is to understand not only syntax, but also **what each command does, when to use it, how it behaves, and common exam/practical mistakes**.

We will use this table throughout:

## `students`

| student_id | name | age | city | course | marks |
|---:|---|---:|---|---|---:|
| 101 | Arnav | 20 | Pune | DBMS | 85 |
| 102 | Rahul | 21 | Jaipur | Python | 72 |
| 103 | Priya | 20 | Pune | DBMS | 91 |
| 104 | Amit | 19 | Udaipur | Java | 68 |
| 105 | Ananya | 21 | Jaipur | Python | 88 |
| 106 | Riya | 20 | Pune | Java | 76 |
| 107 | Aakash | 22 | Kota | DBMS | 95 |
| 108 | Aman | 19 | Jaipur | Java | 64 |
| 109 | Anita | 20 | Udaipur | Python | 82 |
| 110 | Raj | 21 | Pune | DBMS | 79 |

The repeated cities such as Pune, Jaipur, and Udaipur are intentional so that `DISTINCT` is easy to understand.

---

# 1. DISTINCT

## What is `DISTINCT`?

`DISTINCT` is used to **remove duplicate rows from the result of a `SELECT` query**.

### Simple definition

> **DISTINCT = duplicate values hata do.**

It affects the result returned by the query. It does **not** change the stored data in the table.

---

## Without DISTINCT

Suppose we want to see all cities:

```sql
SELECT city
FROM students;
```

Possible result:

| city |
|---|
| Pune |
| Jaipur |
| Pune |
| Udaipur |
| Jaipur |
| Pune |
| Kota |
| Jaipur |
| Udaipur |
| Pune |

Here `Pune`, `Jaipur`, and `Udaipur` appear multiple times.

---

## With DISTINCT

```sql
SELECT DISTINCT city
FROM students;
```

Result:

| city |
|---|
| Pune |
| Jaipur |
| Udaipur |
| Kota |

Now each city appears once.

### Hinglish

`DISTINCT` bol raha hai:

> "Mujhe result mein har unique value sirf ek baar chahiye."

---

## DISTINCT with Multiple Columns

This is important.

```sql
SELECT DISTINCT city, course
FROM students;
```

Here PostgreSQL checks the **combination**:

```text
(city, course)
```

It does **not** remove duplicates separately from `city` and `course`.

For example:

| city | course |
|---|---|
| Pune | DBMS |
| Pune | Java |
| Jaipur | Python |
| Jaipur | Java |

These are all different combinations.

### Exam point

> `DISTINCT` removes duplicate **combinations of the selected columns**, not duplicates from each column independently.

---

## DISTINCT vs UNIQUE

Do not confuse these.

### `DISTINCT`

Used in a query:

```sql
SELECT DISTINCT city
FROM students;
```

It affects the result returned by the query.

### `UNIQUE`

Usually used as a constraint:

```sql
email VARCHAR(100) UNIQUE
```

It controls what values can be stored in the table.

### Memory trick

> `DISTINCT` → duplicate result values remove
>
> `UNIQUE` → duplicate stored values prevent

---

## PostgreSQL-specific: `DISTINCT ON`

PostgreSQL also supports:

```sql
DISTINCT ON
```

It keeps one row for each distinct value/expression specified. The row considered the "first" is only predictable when the query uses a suitable `ORDER BY`.

Example:

```sql
SELECT DISTINCT ON (city)
       city, name, marks
FROM students
ORDER BY city, marks DESC;
```

Conceptually, this can be used to get the highest-scoring student from each city:

```text
Pune     → highest marks student
Jaipur   → highest marks student
Udaipur  → highest marks student
Kota     → highest marks student
```

For a basic SQL exam, focus mainly on normal `DISTINCT`.

---

# 2. WHERE

## What is `WHERE`?

`WHERE` is used to **filter rows according to a condition**.

### Simple definition

> `WHERE` selects only those rows for which the condition is true.

### Hinglish

`WHERE` ek **filter** hai.

Imagine database mein 10,000 rows hain. Tum kehte ho:

> "Sirf age 20 wale students chahiye."

Then:

```sql
WHERE age = 20
```

---

## Basic Syntax

```sql
SELECT column_name
FROM table_name
WHERE condition;
```

---

## Basic Example

```sql
SELECT *
FROM students
WHERE age = 20;
```

Result:

| student_id | name | age | city | course | marks |
|---:|---|---:|---|---|---:|
| 101 | Arnav | 20 | Pune | DBMS | 85 |
| 103 | Priya | 20 | Pune | DBMS | 91 |
| 106 | Riya | 20 | Pune | Java | 76 |
| 109 | Anita | 20 | Udaipur | Python | 82 |

---

## WHERE with Comparison Operators

Common operators include:

```text
=
<>
!=
>
<
>=
<=
```

### Equal

```sql
SELECT *
FROM students
WHERE age = 20;
```

### Greater than

```sql
SELECT *
FROM students
WHERE marks > 80;
```

### Less than

```sql
SELECT *
FROM students
WHERE marks < 70;
```

### Greater than or equal

```sql
SELECT *
FROM students
WHERE marks >= 80;
```

### Not equal

```sql
SELECT *
FROM students
WHERE city <> 'Pune';
```

---

## WHERE with Text

Text values are normally written inside single quotes:

```sql
SELECT *
FROM students
WHERE city = 'Pune';
```

Correct:

```sql
WHERE city = 'Pune'
```

Not:

```sql
WHERE city = Pune
```

---

## WHERE with Numbers

Numbers do not normally need quotes:

```sql
WHERE marks > 80
```

For clean SQL, use values with the appropriate data type instead of unnecessarily writing numeric literals as strings.

---

## WHERE with NULL

This is an important SQL rule.

Do **not** write:

```sql
WHERE email = NULL;
```

Use:

```sql
WHERE email IS NULL;
```

And:

```sql
WHERE email IS NOT NULL;
```

### Why?

`NULL` represents an unknown/missing value. Normal equality comparison with `NULL` does not return `TRUE`.

SQL uses three-valued logic:

```text
TRUE
FALSE
UNKNOWN / NULL
```

### Exam rule

> Use `IS NULL` and `IS NOT NULL`, not `= NULL` or `<> NULL`.

---

## WHERE + DISTINCT

You can combine them:

```sql
SELECT DISTINCT city
FROM students
WHERE marks >= 80;
```

Meaning:

1. Filter students with marks `>= 80`.
2. Return unique cities from those rows.

---

# 3. IN

## What is `IN`?

`IN` is used when you want to check whether a value matches **any one of multiple values**.

### Simple definition

> `IN` checks whether a value belongs to a given list of values.

---

## Without IN

Suppose we want students from Pune, Jaipur, or Udaipur.

We could write:

```sql
SELECT *
FROM students
WHERE city = 'Pune'
   OR city = 'Jaipur'
   OR city = 'Udaipur';
```

This works, but is longer.

---

## With IN

```sql
SELECT *
FROM students
WHERE city IN ('Pune', 'Jaipur', 'Udaipur');
```

Conceptually, this means:

```text
city = Pune
OR
city = Jaipur
OR
city = Udaipur
```

### Hinglish

`IN` bolta hai:

> "Kya ye value meri list ke andar hai?"

---

## IN with Numbers

```sql
SELECT *
FROM students
WHERE marks IN (64, 72, 85, 95);
```

Meaning:

> Marks exactly `64` OR `72` OR `85` OR `95` hone chahiye.

---

## IN with Course

```sql
SELECT *
FROM students
WHERE course IN ('DBMS', 'Java');
```

Only students from DBMS or Java are selected.

---

## NOT IN

Use `NOT IN` to exclude listed values.

```sql
SELECT *
FROM students
WHERE city NOT IN ('Pune', 'Jaipur');
```

Meaning:

> Pune aur Jaipur ke alawa baaki cities.

---

## Important NOT IN + NULL Case

Be careful when `NULL` can appear in a `NOT IN` list or the value being tested can itself be `NULL`.

For example:

```sql
WHERE city NOT IN ('Pune', 'Jaipur', NULL)
```

Because `NULL` introduces an unknown result, this may produce unexpected filtering behavior rather than simply meaning "not Pune and not Jaipur".

### Exam takeaway

> Be careful with `NOT IN` when `NULL` may be present.

---

## IN with a Subquery

`IN` can also use the result of another query:

```sql
SELECT *
FROM students
WHERE course IN (
    SELECT course_name
    FROM courses
    WHERE duration > 6
);
```

The inner query generates a list of values and `IN` checks membership in that list.

---

# 4. LIKE

## What is `LIKE`?

`LIKE` is used for **pattern matching**, especially with text.

Instead of asking:

> "Is the name exactly `Arnav`?"

we can ask:

> "Does the name start with `A`?"

or:

> "Does the name contain `it`?"

### Important rule

`LIKE` matches the **entire string against the pattern**.

Two main wildcards are used:

```text
% → zero or more characters
_ → exactly one character
```

---

## Basic Syntax

```sql
SELECT *
FROM students
WHERE name LIKE 'pattern';
```

---

## Exact Match with LIKE

You can write:

```sql
SELECT *
FROM students
WHERE name LIKE 'Arnav';
```

Because there are no wildcards, the whole value must match the pattern.

This is conceptually similar to:

```sql
WHERE name = 'Arnav'
```

---

# 5. Wildcards `%` and `_`

These are the two most important SQL `LIKE` wildcards.

---

## `%` Wildcard

`%` means:

> **Any sequence of zero or more characters.**

Examples:

```text
%
A%
%A
%A%
%it%
```

---

## `_` Wildcard

`_` means:

> **Exactly one character.**

Examples:

```text
A_
A__
_A_
____
```

---

# 6. Difference Between `%` and `_`

This is a **must-remember exam point**.

| `%` | `_` |
|---|---|
| Matches zero or more characters | Matches exactly one character |
| Variable length | Exactly one character position |
| `A%` = A followed by anything | `A__` = A + exactly 2 more characters |
| `%it%` = `it` can occur anywhere | `_it_` = exactly 4-character string with `it` in the middle |

### Memory trick

> `%` = **many / any number**
>
> `_` = **one**

---

# 7. Important LIKE Patterns: `%A`, `%a`, `%it%`, `A__`

Suppose the names include:

```text
Arnav
Amit
Ananya
Riya
Priya
Ritik
Anita
Aman
```

---

## Pattern 1: `%A`

```sql
WHERE name LIKE '%A'
```

Meaning:

> Ends with uppercase `A`.

So it does **not** mean "contains A".

It means the final character must match uppercase `A`.

---

## Pattern 2: `%a`

```sql
WHERE name LIKE '%a'
```

Meaning:

> Ends with lowercase `a`.

Under normal PostgreSQL `LIKE`, case matters.

For example, names ending in lowercase `a` such as these can match:

```text
Riya
Priya
Ananya
Anita
```

provided their stored text has that exact case.

---

## Pattern 3: `%it%`

```sql
WHERE name LIKE '%it%'
```

Meaning:

> The sequence `it` can occur anywhere in the name.

Possible matches include names such as:

```text
Amit
Ritik
```

when the exact case and character sequence match.

---

## Pattern 4: `A__`

```sql
WHERE name LIKE 'A__'
```

Meaning:

```text
A + one character + one character
```

Total length = **3 characters**.

Matches could include:

```text
Ari → ✅
Ava → ✅
Ali → ✅
```

Does not match:

```text
Amit  → ❌
Arnav → ❌
```

because those contain more than three characters.

---

# More `%` Examples

## `%`

```sql
LIKE '%'
```

Matches any string, including an empty string, when applied to a non-NULL value.

---

## `A%`

```sql
WHERE name LIKE 'A%'
```

Meaning:

> Starts with `A`.

Examples:

```text
Arnav
Amit
Ananya
Aakash
Aman
Anita
```

---

## `%A`

```sql
WHERE name LIKE '%A'
```

Meaning:

> Ends with uppercase `A`.

---

## `%A%`

```sql
WHERE name LIKE '%A%'
```

Meaning:

> Contains uppercase `A` somewhere.

---

## `%it%`

```sql
WHERE name LIKE '%it%'
```

Meaning:

> Contains the exact sequence `it` somewhere.

---

# More `_` Examples

## `A_`

```sql
WHERE name LIKE 'A_'
```

Means:

```text
A + exactly 1 more character
```

Total length = 2 characters.

Examples:

```text
Am → ✅
An → ✅
Ap → ✅
```

`Amit` does not match because it is longer than 2 characters.

---

## `A__`

```sql
WHERE name LIKE 'A__'
```

Means:

```text
A + exactly 2 more characters
```

Total length = 3 characters.

Examples:

```text
Ari → ✅
Ava → ✅
Ali → ✅
```

---

## `____`

```sql
WHERE name LIKE '____'
```

Means:

> Exactly 4 characters.

---

# Important LIKE Rule: Whole String Matching

This is a common confusion.

Suppose:

```sql
WHERE name LIKE 'it'
```

This does **not** mean:

> Find `it` anywhere.

It means the entire value must match exactly `it`.

So:

```text
it    → ✅
Amit  → ❌
Ritik → ❌
```

To find `it` anywhere, use:

```sql
WHERE name LIKE '%it%'
```

---

# Case Sensitivity: LIKE vs ILIKE in PostgreSQL

PostgreSQL provides both:

```text
LIKE
ILIKE
```

### LIKE

Normal pattern matching where case matters.

Example:

```sql
SELECT *
FROM students
WHERE name LIKE 'a%';
```

This normally looks for lowercase `a` at the start.

### ILIKE

PostgreSQL's case-insensitive alternative:

```sql
SELECT *
FROM students
WHERE name ILIKE 'a%';
```

Now names starting with `A` or `a` can match.

### Memory

```text
LIKE  → case-sensitive pattern matching
ILIKE → case-insensitive pattern matching (PostgreSQL)
```

---

# NOT LIKE

You can negate a pattern:

```sql
SELECT *
FROM students
WHERE name NOT LIKE 'A%';
```

Meaning:

> Names jo `A` se start nahi hote.

---

# Matching Literal `%` or `_`

Sometimes `%` or `_` is actual data, not a wildcard.

For example, suppose a value is:

```text
50%
```

To search for a literal `%`, you need to escape the wildcard behavior.

Example:

```sql
WHERE value LIKE '50\%'
```

Or use a custom escape character:

```sql
WHERE value LIKE '50#%' ESCAPE '#'
```

Here `#` is used as the escape character, so `%` is treated as a literal character.

This is an advanced case but useful for understanding wildcard escaping.

---

# 8. AND

## What is `AND`?

`AND` is used when **all conditions must be true**.

Example:

```sql
SELECT *
FROM students
WHERE age = 20
AND city = 'Pune';
```

The student must satisfy both:

```text
age = 20
AND
city = Pune
```

### Hinglish

> `AND` → dono/all conditions satisfy honi chahiye.

---

## Example

```sql
SELECT *
FROM students
WHERE marks > 80
AND age = 20;
```

Both conditions must be true.

---

# 9. OR

## What is `OR`?

`OR` means:

> **At least one condition should be true.**

Example:

```sql
SELECT *
FROM students
WHERE city = 'Pune'
OR city = 'Jaipur';
```

This accepts a row if it is from either Pune or Jaipur.

### Hinglish

> `OR` → koi bhi ek condition true ho jaye.

---

# 10. NOT

## What is `NOT`?

`NOT` reverses a Boolean condition.

Example:

```sql
SELECT *
FROM students
WHERE NOT city = 'Pune';
```

Meaning:

> Pune nahi chahiye.

In this simple case, it is equivalent to:

```sql
WHERE city <> 'Pune'
```

---

## NOT with IN

```sql
SELECT *
FROM students
WHERE city NOT IN ('Pune', 'Jaipur');
```

Meaning:

> Pune aur Jaipur ko exclude karo.

---

## NOT with LIKE

```sql
SELECT *
FROM students
WHERE name NOT LIKE 'A%';
```

Meaning:

> Name A se start nahi hona chahiye.

---

# AND + OR Together

This is important because mixed logical conditions can become confusing.

Consider:

```sql
SELECT *
FROM students
WHERE city = 'Pune'
AND marks > 80
OR age = 21;
```

It is safer and clearer to use parentheses:

```sql
SELECT *
FROM students
WHERE (city = 'Pune' AND marks > 80)
   OR age = 21;
```

### Logical precedence to remember

SQL evaluates logical operators according to precedence. For these operators, the practical order is:

```text
NOT
↓
AND
↓
OR
```

### Exam tip

Whenever you have:

```text
AND + OR
```

use parentheses to make your intention clear.

---

# Truth Tables

## AND

| A | B | A AND B |
|---|---|---|
| TRUE | TRUE | TRUE |
| TRUE | FALSE | FALSE |
| FALSE | TRUE | FALSE |
| FALSE | FALSE | FALSE |

---

## OR

| A | B | A OR B |
|---|---|---|
| TRUE | TRUE | TRUE |
| TRUE | FALSE | TRUE |
| FALSE | TRUE | TRUE |
| FALSE | FALSE | FALSE |

---

## NOT

| A | NOT A |
|---|---|
| TRUE | FALSE |
| FALSE | TRUE |

### SQL extra point

SQL also has `NULL`/unknown, so SQL's complete logical behavior is based on three-valued logic:

```text
TRUE
FALSE
UNKNOWN
```

---

# 11. LIMIT

## What is `LIMIT`?

`LIMIT` controls the **maximum number of rows returned by a query**.

### Simple definition

> `LIMIT` = maximum number of rows to return.

### Hinglish

`LIMIT` bol raha hai:

> "Maximum itni rows hi dikhana."

---

## Basic Example

```sql
SELECT *
FROM students
LIMIT 5;
```

At most 5 rows will be returned.

---

# LIMIT with ORDER BY

Suppose we want the top 3 marks:

```sql
SELECT *
FROM students
ORDER BY marks DESC
LIMIT 3;
```

Meaning:

1. Sort students by marks descending.
2. Take only the first 3 rows.

### Important point

Do not assume:

```sql
SELECT *
FROM students
LIMIT 3;
```

means "top 3" or "first three meaningful rows".

Without an appropriate `ORDER BY`, the specific rows returned are not guaranteed to represent a meaningful order.

For top 3 marks, use:

```sql
SELECT *
FROM students
ORDER BY marks DESC
LIMIT 3;
```

---

# LIMIT with WHERE

```sql
SELECT *
FROM students
WHERE city = 'Pune'
LIMIT 2;
```

Meaning:

> Pune ke students mein se maximum 2 rows.

---

# LIMIT with DISTINCT

```sql
SELECT DISTINCT city
FROM students
LIMIT 3;
```

Meaning:

> Unique cities nikalo, then maximum 3 results return karo.

---

# LIMIT ALL

PostgreSQL also supports:

```sql
LIMIT ALL
```

This means effectively no limit. Omitting `LIMIT` has the same effect. `LIMIT NULL` also behaves as no limit.

This is usually not necessary for a basic exam, but is useful to know.

---

# 12. OFFSET

## What is `OFFSET`?

`OFFSET` tells PostgreSQL to **skip a specified number of rows before returning results**.

### Simple definition

> `OFFSET` = number of rows to skip before starting the returned result.

### Hinglish

`OFFSET` bol raha hai:

> "Starting mein itni rows skip karo, uske baad result dena start karo."

---

## Basic Example

```sql
SELECT *
FROM students
OFFSET 3;
```

Conceptually:

```text
Rows:
1 → skip
2 → skip
3 → skip
4 → start returning
```

---

# OFFSET + LIMIT

This is where `OFFSET` becomes very useful.

```sql
SELECT *
FROM students
LIMIT 3
OFFSET 3;
```

Meaning:

```text
Skip first 3 rows
↓
Return next 3 rows
```

`OFFSET` is applied before counting rows toward `LIMIT`.

---

# Pagination Example

Suppose a website shows:

```text
5 students per page
```

### Page 1

```sql
SELECT *
FROM students
ORDER BY student_id
LIMIT 5
OFFSET 0;
```

Rows 1–5.

### Page 2

```sql
SELECT *
FROM students
ORDER BY student_id
LIMIT 5
OFFSET 5;
```

Rows 6–10.

### Page 3

```sql
SELECT *
FROM students
ORDER BY student_id
LIMIT 5
OFFSET 10;
```

Rows 11–15.

---

# Pagination Formula

If:

```text
page size = N
page number = P
```

then:

```text
OFFSET = (P - 1) × N
```

### Example

```text
N = 10
P = 4
```

Therefore:

```text
OFFSET = (4 - 1) × 10
       = 30
```

Query:

```sql
SELECT *
FROM students
ORDER BY student_id
LIMIT 10
OFFSET 30;
```

Meaning:

> First 30 rows skip karo, then next 10 rows do.

---

# OFFSET 0

```sql
OFFSET 0
```

means:

> Don't skip anything.

It has the same practical effect as omitting `OFFSET`.

---

# LIMIT vs OFFSET

| LIMIT | OFFSET |
|---|---|
| Controls how many rows are returned | Controls how many rows are skipped first |
| "How many?" | "From where should we start?" |
| `LIMIT 5` | `OFFSET 10` |

Think:

```text
OFFSET 10
    ↓
skip 10

LIMIT 5
    ↓
then take 5
```

---

# LIMIT + OFFSET + ORDER BY

For pagination, the common pattern is:

```sql
SELECT *
FROM students
ORDER BY student_id
LIMIT 10
OFFSET 20;
```

Meaning:

```text
1. Sort by student_id
2. Skip first 20 rows
3. Return next 10 rows
```

Using `ORDER BY` is important because it provides a defined order for pagination.

---

# 13. Combining the Concepts

Now combine everything.

---

## Example 1 — Unique Cities

```sql
SELECT DISTINCT city
FROM students;
```

Meaning:

> Duplicate cities remove karo.

---

## Example 2 — Students from Pune

```sql
SELECT *
FROM students
WHERE city = 'Pune';
```

Meaning:

> Pune ke students do.

---

## Example 3 — Students from Selected Cities

```sql
SELECT *
FROM students
WHERE city IN ('Pune', 'Jaipur');
```

Meaning:

> Pune ya Jaipur ke students.

---

## Example 4 — Names Starting with A

```sql
SELECT *
FROM students
WHERE name LIKE 'A%';
```

Meaning:

> A se start hone wale names.

---

## Example 5 — Names Ending with `a`

```sql
SELECT *
FROM students
WHERE name LIKE '%a';
```

Meaning:

> Lowercase `a` par end hone wale names, under normal case-sensitive `LIKE`.

---

## Example 6 — Names Containing `it`

```sql
SELECT *
FROM students
WHERE name LIKE '%it%';
```

Meaning:

> Jahan `it` sequence kahin bhi present ho.

---

## Example 7 — Exactly 3 Characters Starting with A

```sql
SELECT *
FROM students
WHERE name LIKE 'A__';
```

Meaning:

> A + exactly two characters.

---

## Example 8 — Marks Above 80 AND Pune

```sql
SELECT *
FROM students
WHERE marks > 80
AND city = 'Pune';
```

Meaning:

> Dono conditions true honi chahiye.

---

## Example 9 — Pune OR Jaipur

```sql
SELECT *
FROM students
WHERE city = 'Pune'
OR city = 'Jaipur';
```

Meaning:

> Kisi ek condition ka true hona enough hai.

---

## Example 10 — Not Pune

```sql
SELECT *
FROM students
WHERE NOT city = 'Pune';
```

or:

```sql
SELECT *
FROM students
WHERE city <> 'Pune';
```

Meaning:

> Pune ke alawa baaki cities.

---

## Example 11 — Top 3 Students

```sql
SELECT *
FROM students
ORDER BY marks DESC
LIMIT 3;
```

Meaning:

> Highest marks wale top 3 students.

---

## Example 12 — Second Page

Suppose 3 rows per page:

```sql
SELECT *
FROM students
ORDER BY student_id
LIMIT 3
OFFSET 3;
```

Meaning:

> First 3 skip → next 3 display.

---

# 14. One Complex Query — Understand the Whole Flow

Look at this query:

```sql
SELECT DISTINCT city
FROM students
WHERE marks >= 80
  AND city IN ('Pune', 'Jaipur', 'Udaipur')
ORDER BY city
LIMIT 2
OFFSET 1;
```

Break it down piece by piece.

### `SELECT DISTINCT city`

> Unique cities chahiye.

### `FROM students`

> Students table se.

### `WHERE marks >= 80`

> Sirf 80+ marks wale.

### `AND city IN (...)`

> Aur city Pune/Jaipur/Udaipur mein honi chahiye.

### `ORDER BY city`

> Cities sort karo.

### `LIMIT 2`

> Maximum 2 results.

### `OFFSET 1`

> First result skip karo.

### Exam strategy

Long SQL ko ek giant statement ki tarah mat padho. Isse pieces mein break karo.

---

# 15. Important Differences

## WHERE vs DISTINCT

### `WHERE`

Filters rows.

Example:

```sql
SELECT *
FROM students
WHERE marks > 80;
```

### `DISTINCT`

Removes duplicate rows from the selected result.

Example:

```sql
SELECT DISTINCT city
FROM students;
```

### Combined

```sql
SELECT DISTINCT city
FROM students
WHERE marks > 80;
```

Conceptually:

1. Filter rows with `marks > 80`.
2. Remove duplicate cities from the selected result.

---

# IN vs OR

For simple equality checks:

```sql
WHERE city IN ('Pune', 'Jaipur')
```

is conceptually equivalent to:

```sql
WHERE city = 'Pune'
   OR city = 'Jaipur'
```

`IN` is usually cleaner when checking many exact values.

---

# LIKE vs `=`

### `=`

Exact comparison:

```sql
WHERE name = 'Amit'
```

### `LIKE`

Pattern comparison:

```sql
WHERE name LIKE 'A%';
```

### Memory

```text
=     → exact value
LIKE  → pattern
```

---

# LIKE vs ILIKE

```text
LIKE  → normal case-sensitive pattern matching
ILIKE → case-insensitive pattern matching in PostgreSQL
```

---

# `%` vs `_`

```text
% → zero or more characters
_ → exactly one character
```

This is one of the most important rules in the topic.

---

# LIMIT vs OFFSET

```text
LIMIT  → kitni rows leni hain?
OFFSET → kitni rows skip karni hain?
```

---

# 16. Common Mistakes and Exam Traps

## Mistake 1 — `%A` vs `%A%`

Wrong interpretation:

```sql
WHERE name LIKE '%A'
```

thinking it means:

> Contains A.

Correct meaning:

> Ends with uppercase `A`.

For contains `A` anywhere:

```sql
WHERE name LIKE '%A%'
```

---

## Mistake 2 — `_` Means Unlimited Characters

Wrong.

```text
_ = exactly one character
```

---

## Mistake 3 — `A__` Means Any Length

Wrong.

```text
A__
```

means:

> Exactly 3 characters total, first one is `A`.

---

## Mistake 4 — `= NULL`

Wrong:

```sql
WHERE name = NULL
```

Correct:

```sql
WHERE name IS NULL
```

---

## Mistake 5 — `NOT IN` with NULL

Be careful if `NULL` may be present.

`NULL` can make the condition evaluate to unknown and cause surprising filtering results.

---

## Mistake 6 — Assuming `LIMIT 3` Means Top 3

This:

```sql
SELECT *
FROM students
LIMIT 3;
```

does not define what "top 3" means.

For top 3 marks:

```sql
SELECT *
FROM students
ORDER BY marks DESC
LIMIT 3;
```

---

## Mistake 7 — Pagination Without ORDER BY

Less reliable for pagination:

```sql
SELECT *
FROM students
LIMIT 10
OFFSET 20;
```

Better:

```sql
SELECT *
FROM students
ORDER BY student_id
LIMIT 10
OFFSET 20;
```

---

## Mistake 8 — Confusing `LIKE 'it'` with `LIKE '%it%'`

```sql
LIKE 'it'
```

requires the full string to match `it`.

```sql
LIKE '%it%'
```

allows `it` to appear anywhere in the string.

---

# 17. Quick Revision Table

| Feature | Simple Meaning | Example |
|---|---|---|
| `DISTINCT` | Remove duplicate result rows | `SELECT DISTINCT city` |
| `WHERE` | Filter rows | `WHERE age > 18` |
| `IN` | Match one of many values | `city IN ('Pune','Jaipur')` |
| `NOT IN` | Exclude listed values | `city NOT IN ('Pune')` |
| `LIKE` | Pattern matching | `name LIKE 'A%'` |
| `%` | 0 or more characters | `'A%'` |
| `_` | Exactly 1 character | `'A__'` |
| `AND` | All conditions true | `age > 18 AND marks > 80` |
| `OR` | At least one true | `city='Pune' OR city='Jaipur'` |
| `NOT` | Reverse condition | `NOT city='Pune'` |
| `LIMIT` | Maximum rows returned | `LIMIT 5` |
| `OFFSET` | Rows skipped first | `OFFSET 10` |

---

# 18. Super-Easy Memory Tricks

Remember:

```text
DISTINCT → duplicate hatao
WHERE    → filter lagao
IN       → list mein check karo
LIKE     → pattern search karo

%        → many / any number
_        → one character

AND      → dono/all conditions
OR       → koi ek
NOT      → ulta

LIMIT    → kitni rows leni hain
OFFSET   → kitni rows skip karni hain
```

### The four patterns to memorize

```text
%A    → ends with A
%a    → ends with a
%it%  → contains "it"
A__   → exactly 3 characters, starts with A
```

---

# 19. Best Mental Model for SELECT Queries

A very useful way to think about these clauses is:

```text
SELECT      → What do I want?
FROM        → Where is it?
WHERE       → Which rows?
DISTINCT    → Remove duplicates
ORDER BY    → In what order?
LIMIT       → How many?
OFFSET      → From which position?
```

Example:

```sql
SELECT DISTINCT city
FROM students
WHERE marks >= 80
ORDER BY city
LIMIT 3
OFFSET 1;
```

Read it as:

> Students table se 80+ marks wale records lo, unique cities nikalo, cities ko sort karo, first result skip karo, aur next maximum 3 results do.

---

# 20. Final Exam Recap

## DISTINCT

> Removes duplicate rows from the query result.

```sql
SELECT DISTINCT city FROM students;
```

## WHERE

> Filters rows according to a condition.

```sql
SELECT * FROM students WHERE age > 20;
```

## IN

> Checks whether a value belongs to a list of values.

```sql
SELECT * FROM students WHERE city IN ('Pune', 'Jaipur');
```

## LIKE

> Performs pattern matching.

```sql
SELECT * FROM students WHERE name LIKE 'A%';
```

## `%`

> Zero or more characters.

## `_`

> Exactly one character.

## AND

> All conditions must be true.

## OR

> At least one condition must be true.

## NOT

> Reverses a condition.

## LIMIT

> Limits how many rows are returned.

## OFFSET

> Skips a specified number of rows before returning the result.

---

# Final One-Page Memory Map

```text
                SQL QUERYING
                     |
        +------------+-------------+
        |                          |
     FILTERING                  RESULT
        |                          |
   +----+----+                  DISTINCT
   |    |    |                     |
 WHERE  IN   LIKE                  |
   |    |    |                     |
   |    |    +---- Wildcards ------+
   |    |         % / _
   |    |
   |    +---- multiple exact values
   |
   +---- conditions
         |
       AND
       OR
       NOT

       RESULT CONTROL
       |
       +---- ORDER BY
       |
       +---- LIMIT
       |
       +---- OFFSET
```

## Ultra-short revision

```text
DISTINCT = duplicate remove
WHERE    = filter
IN       = one of many values
LIKE     = pattern
%        = 0 or more
_        = exactly 1
AND      = all true
OR       = any one true
NOT      = reverse
LIMIT    = maximum rows
OFFSET   = skip rows
```

---

# Note on PostgreSQL-Specific Behavior

The notes above are written for PostgreSQL-oriented SQL learning. In particular:

- PostgreSQL supports `ILIKE` for case-insensitive pattern matching.
- PostgreSQL supports `DISTINCT ON`.
- PostgreSQL supports `LIMIT` / `OFFSET` syntax.
- PostgreSQL uses SQL three-valued logic involving `TRUE`, `FALSE`, and `UNKNOWN` for expressions involving `NULL`.

For exam preparation, always follow the exact classification and terminology used by your instructor when it differs from general SQL terminology.
