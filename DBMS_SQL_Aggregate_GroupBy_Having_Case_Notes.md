# PostgreSQL Aggregate Functions, GROUP BY, HAVING, SQL Running Order & CASE
## Detailed Exam Notes — English + Hinglish

These notes explain aggregate functions, `GROUP BY`, `HAVING`, SQL logical processing order, and `CASE` using one consistent student dataset. The focus is simple understanding, exam preparation, practical SQL, and common mistakes.

PostgreSQL documentation describes aggregate functions as functions that take a set of input values and produce a single result, while `GROUP BY` creates groups over which aggregates are calculated. PostgreSQL's documented `SELECT` processing explains why `WHERE` and `HAVING` behave differently.

---

# Our Example Table

We will use this table throughout the notes.

## `students`

| student_id | name | city | course | marks | age |
|---:|---|---|---|---:|---:|
| 101 | Arnav | Pune | DBMS | 85 | 20 |
| 102 | Rahul | Jaipur | Python | 72 | 21 |
| 103 | Priya | Pune | DBMS | 91 | 20 |
| 104 | Amit | Udaipur | Java | 68 | 19 |
| 105 | Ananya | Jaipur | Python | 88 | 21 |
| 106 | Riya | Pune | Java | 76 | 20 |
| 107 | Aakash | Kota | DBMS | 95 | 22 |
| 108 | Aman | Jaipur | Java | 64 | 19 |
| 109 | Anita | Udaipur | Python | 82 | 20 |
| 110 | Raj | Pune | DBMS | 79 | 21 |

---

# PART 1 — AGGREGATE FUNCTIONS

## What is an Aggregate Function?

An **aggregate function** takes multiple rows/values and calculates **one result**.

Example:

```sql
SELECT SUM(marks)
FROM students;
```

There are 10 students, but the query gives one total.

Conceptually:

```text
Many rows
   ↓
Aggregate function
   ↓
One result
```

PostgreSQL's built-in general-purpose aggregates include `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX`.

### Hinglish

Aggregate function ka simple meaning:

> **Bahut saari rows/values ko process karke ek summarized result dena.**

---

# 1. `COUNT()`

## What is COUNT()?

`COUNT()` tells you **how many rows or non-NULL values exist**.

This is one of the most important aggregate functions.

There are several forms you need to understand.

---

## A. `COUNT(*)`

```sql
SELECT COUNT(*)
FROM students;
```

Result:

```text
10
```

### Meaning

> Count **all rows**.

`COUNT(*)` counts rows regardless of whether particular columns contain NULL.

### Hinglish

`COUNT(*)` bolta hai:

> **"Table mein total kitni rows hain?"**

---

## B. `COUNT(column)`

Now:

```sql
SELECT COUNT(email)
FROM students;
```

This counts **non-NULL values of that column**, not necessarily every row.

This distinction is extremely important.

Suppose:

| student_id | email |
|---:|---|
| 101 | arnav@gmail.com |
| 102 | rahul@gmail.com |
| 103 | NULL |
| 104 | amit@gmail.com |

Then:

```text
COUNT(*)
```

= `4`

while:

```text
COUNT(email)
```

= `3`

because the NULL email isn't counted.

### Memory trick

```text
COUNT(*)       → rows
COUNT(column)  → non-NULL column values
```

PostgreSQL documents `count(*)` as counting input rows, while `count(expression)` counts non-null input values.

---

## C. `COUNT(DISTINCT column)`

Suppose cities are:

```text
Pune
Jaipur
Pune
Udaipur
Jaipur
Pune
Kota
Jaipur
Udaipur
Pune
```

Query:

```sql
SELECT COUNT(DISTINCT city)
FROM students;
```

Result:

```text
4
```

Because unique cities are:

```text
Pune
Jaipur
Udaipur
Kota
```

### Hinglish

> `COUNT(DISTINCT city)` = **"Kitni unique cities hain?"**

---

## D. COUNT with WHERE

```sql
SELECT COUNT(*)
FROM students
WHERE marks >= 80;
```

Meaning:

> Pehle 80+ marks wale students select karo, then count karo.

This is an important example of how `WHERE` affects an aggregate.

---

## E. COUNT with GROUP BY

```sql
SELECT city, COUNT(*)
FROM students
GROUP BY city;
```

Result:

| city | count |
|---|---:|
| Jaipur | 3 |
| Kota | 1 |
| Pune | 4 |
| Udaipur | 2 |

Now `COUNT(*)` is calculated **separately for each city**.

---

# 2. `SUM()`

## What is SUM()?

`SUM()` adds numeric values together.

Example:

```sql
SELECT SUM(marks)
FROM students;
```

All marks are added.

Conceptually:

```text
85 + 72 + 91 + 68 + ...
```

---

## SUM with WHERE

```sql
SELECT SUM(marks)
FROM students
WHERE course = 'DBMS';
```

Meaning:

> Sirf DBMS students ke marks add karo.

---

## SUM with GROUP BY

```sql
SELECT course,
       SUM(marks) AS total_marks
FROM students
GROUP BY course;
```

Example result:

| course | total_marks |
|---|---:|
| DBMS | 350 |
| Java | 208 |
| Python | 242 |

The rows are grouped by course, and the sum is calculated separately inside each group.

---

## Important SUM + NULL Behavior

Suppose:

```text
100
200
NULL
300
```

Then:

```text
100 + 200 + 300 = 600
```

NULL input isn't treated as a numeric zero for the aggregate; aggregate functions such as `SUM` generally ignore NULL inputs.

Also, if there are **no input rows**, PostgreSQL returns `NULL` for `SUM`, not `0`.

Example:

```sql
SELECT COALESCE(SUM(marks), 0)
FROM students
WHERE course = 'SomethingThatDoesNotExist';
```

Result:

```text
0
```

### Hinglish

`SUM()` mein NULL ko generally ignore kiya jata hai, aur empty input set ke case mein `SUM()` `NULL` de sakta hai. Zero chahiye to `COALESCE()` use karo.

---

# 3. `AVG()`

## What is AVG()?

`AVG()` calculates the **arithmetic average**.

Formula:

```text
Average = Sum of values / Number of non-NULL values
```

Example:

```sql
SELECT AVG(marks)
FROM students;
```

PostgreSQL computes average over the non-NULL input values.

---

## AVG with GROUP BY

```sql
SELECT course,
       AVG(marks) AS average_marks
FROM students
GROUP BY course;
```

Possible result:

| course | average_marks |
|---|---:|
| DBMS | 87.5 |
| Java | 69.33 |
| Python | 80.67 |

---

## Important AVG + NULL Concept

Suppose values are:

```text
80
90
NULL
```

Average is:

```text
(80 + 90) / 2
= 85
```

It isn't divided by 3.

Why?

Because `AVG()` ignores NULL input values.

### Hinglish

NULL ko student ka zero mark mat samajhna.

`NULL` means:

> **marks available nahi hain.**

So `AVG()` generally excludes that value from the average calculation.

---

# 4. `MIN()`

## What is MIN()?

`MIN()` returns the **smallest value** in the input.

```sql
SELECT MIN(marks)
FROM students;
```

Result:

```text
64
```

---

## MIN with GROUP BY

```sql
SELECT course,
       MIN(marks) AS lowest_marks
FROM students
GROUP BY course;
```

Meaning:

> Har course ka lowest marks.

---

## MIN Works Beyond Numbers

In PostgreSQL, `MIN()` is not restricted to numeric values. It can also work with sortable types such as strings and date/time values.

Example with strings:

```sql
SELECT MIN(name)
FROM students;
```

This uses the relevant ordering rules for the text values.

Example with dates:

```sql
SELECT MIN(joined_at)
FROM students;
```

would give the earliest date/time.

---

# 5. `MAX()`

`MAX()` returns the **largest value**.

```sql
SELECT MAX(marks)
FROM students;
```

Result:

```text
95
```

---

## MAX with GROUP BY

```sql
SELECT course,
       MAX(marks) AS highest_marks
FROM students
GROUP BY course;
```

Meaning:

> Har course ka highest marks.

Like `MIN()`, PostgreSQL supports `MAX()` for various sortable data types, not just numbers.

---

# MIN vs MAX

```text
MIN → smallest
MAX → largest
```

Example:

```text
64, 68, 72, 76, 79, 82, 85, 88, 91, 95

MIN → 64
MAX → 95
```

---

# Aggregate Functions — Quick Table

| Function | Meaning |
|---|---|
| `COUNT()` | Count |
| `SUM()` | Total |
| `AVG()` | Average |
| `MIN()` | Smallest |
| `MAX()` | Largest |

Memory trick:

```text
COUNT → How many?
SUM   → How much total?
AVG   → What is the average?
MIN   → What is the smallest?
MAX   → What is the largest?
```

---

# PART 2 — `GROUP BY`

## What is GROUP BY?

`GROUP BY` combines rows having the same value(s) in one or more specified columns into groups.

PostgreSQL describes it as grouping rows with the same values in the listed grouping columns and producing one group row per group.

### Hinglish

Imagine 10 students hain.

Tum bolo:

> **"Same course wale students ko ek group bana do."**

That's:

```sql
GROUP BY course
```

---

# Without GROUP BY

```sql
SELECT COUNT(*)
FROM students;
```

Result:

```text
10
```

One total.

---

# With GROUP BY

```sql
SELECT course, COUNT(*)
FROM students
GROUP BY course;
```

Result:

| course | count |
|---|---:|
| DBMS | 4 |
| Java | 3 |
| Python | 3 |

Now we get **one row per course**.

---

# GROUP BY Mental Model

Suppose:

```text
Student rows
      ↓
GROUP BY course
      ↓
DBMS group
Java group
Python group
      ↓
Aggregate each group
      ↓
One result per group
```

---

# GROUP BY Multiple Columns

You can group using multiple columns.

Example:

```sql
SELECT city,
       course,
       COUNT(*) AS student_count
FROM students
GROUP BY city, course;
```

Now groups are based on the **combination**:

```text
(city, course)
```

For example:

```text
(Pune, DBMS)
(Pune, Java)
(Jaipur, Python)
(Jaipur, Java)
...
```

PostgreSQL groups using the values of all listed grouping expressions.

---

# Important GROUP BY Rule

Suppose:

```sql
SELECT course, name, COUNT(*)
FROM students
GROUP BY course;
```

This is problematic because:

```text
course → grouped
name   → not grouped
```

For a course group containing several students, which `name` should PostgreSQL return?

There isn't one unambiguous answer.

So generally, a selected column must either:

1. Be included in `GROUP BY`, or
2. Be inside an aggregate expression.

---

# Correct

```sql
SELECT course,
       COUNT(*) AS student_count
FROM students
GROUP BY course;
```

Or:

```sql
SELECT course,
       MIN(name),
       COUNT(*)
FROM students
GROUP BY course;
```

Here `MIN(name)` is an aggregate.

---

# GROUP BY without Aggregate

This is interesting.

```sql
SELECT city
FROM students
GROUP BY city;
```

This produces one row per unique city.

So it behaves similarly to:

```sql
SELECT DISTINCT city
FROM students;
```

PostgreSQL explicitly notes that grouping without aggregate expressions effectively calculates distinct values.

---

# PART 3 — `HAVING`

## What is HAVING?

`HAVING` filters **groups after grouping and aggregate calculation**.

This is one of the most important concepts in SQL.

### Hinglish

Think:

```text
WHERE  → rows ko filter karo
HAVING → groups ko filter karo
```

---

# Basic Example

Suppose:

```sql
SELECT course,
       COUNT(*) AS student_count
FROM students
GROUP BY course;
```

Result:

| course | student_count |
|---|---:|
| DBMS | 4 |
| Java | 3 |
| Python | 3 |

Now suppose we only want courses having at least 4 students:

```sql
SELECT course,
       COUNT(*) AS student_count
FROM students
GROUP BY course
HAVING COUNT(*) >= 4;
```

Result:

| course | student_count |
|---|---:|
| DBMS | 4 |

---

# Why WHERE Can't Do This

This is invalid:

```sql
SELECT course,
       COUNT(*)
FROM students
WHERE COUNT(*) >= 4
GROUP BY course;
```

Why?

Because `WHERE` filters rows **before the groups and aggregates are formed**. `COUNT(*)` doesn't exist yet at that stage.

So:

```text
WHERE
  ↓
comes before aggregate calculation
```

`HAVING` is the correct place to filter based on group aggregates.

---

# WHERE vs HAVING

This is a **must-learn exam question**.

| WHERE | HAVING |
|---|---|
| Filters rows | Filters groups |
| Before `GROUP BY` | After grouping |
| Used for row-level conditions | Used for aggregate/group conditions |
| Normally cannot contain aggregate functions | Can use aggregate functions |

---

# Example — WHERE

```sql
SELECT *
FROM students
WHERE marks >= 80;
```

Question:

> Which individual students have marks >= 80?

---

# Example — HAVING

```sql
SELECT course,
       AVG(marks)
FROM students
GROUP BY course
HAVING AVG(marks) >= 80;
```

Question:

> Which courses have an average mark of at least 80?

Notice the difference:

```text
WHERE  → student-level
HAVING → course-level
```

---

# WHERE + HAVING Together

This is extremely important.

```sql
SELECT course,
       AVG(marks) AS avg_marks
FROM students
WHERE marks >= 70
GROUP BY course
HAVING AVG(marks) >= 80;
```

Read it carefully.

### WHERE

```text
Remove students whose marks < 70
```

### GROUP BY

```text
Group remaining students by course
```

### HAVING

```text
Keep only courses whose average is >= 80
```

---

# HAVING Does Not Always Need Aggregate

For example:

```sql
SELECT course,
       COUNT(*)
FROM students
GROUP BY course
HAVING course = 'DBMS';
```

This can work because `course` is the grouped column.

But if the condition doesn't need an aggregate, using `WHERE` is generally more efficient because it can eliminate rows before grouping.

So prefer:

```sql
WHERE course = 'DBMS'
```

rather than unnecessarily:

```sql
HAVING course = 'DBMS'
```

when filtering individual source rows.

---

# PART 4 — SQL RUNNING ORDER

You should understand this logical processing model:

```text
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

A more complete PostgreSQL logical processing model is approximately:

```text
FROM
  ↓
WHERE
  ↓
GROUP BY / aggregate computation
  ↓
HAVING
  ↓
SELECT
  ↓
DISTINCT
  ↓
ORDER BY
  ↓
LIMIT / OFFSET
```

This ordering is the **logical processing order**, not the order in which we type the query.

---

# Written Order vs Running Order

We normally write:

```sql
SELECT ...
FROM ...
WHERE ...
GROUP BY ...
HAVING ...
ORDER BY ...;
```

But logically, PostgreSQL processes it approximately as:

```text
FROM
↓
WHERE
↓
GROUP BY / aggregate
↓
HAVING
↓
SELECT
↓
DISTINCT
↓
ORDER BY
↓
LIMIT / OFFSET
```

This is one of the most important concepts in SQL.

---

# Why Does Running Order Matter?

Because it explains **what can be referenced where**.

For example:

```sql
SELECT name
FROM students
WHERE COUNT(*) > 2;
```

doesn't work because:

```text
WHERE
 ↓
comes before aggregate computation
```

`COUNT(*)` isn't available yet.

---

# Another Important Example — SELECT Alias

Suppose:

```sql
SELECT marks * 2 AS double_marks
FROM students
ORDER BY double_marks;
```

This works because `ORDER BY` is processed after the `SELECT` output expressions, so the alias is available there.

---

# Query Running Order Example

Query:

```sql
SELECT course,
       AVG(marks) AS average_marks
FROM students
WHERE marks >= 70
GROUP BY course
HAVING AVG(marks) >= 80
ORDER BY average_marks DESC;
```

Let's execute it mentally.

### Step 1 — FROM

Get:

```text
students
```

---

### Step 2 — WHERE

Remove:

```text
marks < 70
```

---

### Step 3 — GROUP BY

Create:

```text
DBMS group
Python group
Java group
```

---

### Step 4 — Aggregate

Calculate:

```text
AVG(marks)
```

for every group.

---

### Step 5 — HAVING

Keep groups:

```text
average_marks >= 80
```

---

### Step 6 — SELECT

Create output:

```text
course
average_marks
```

---

### Step 7 — ORDER BY

Sort:

```text
average_marks DESC
```

---

# Very Important Complete Order

For your notes, memorize:

```text
FROM
  ↓
WHERE
  ↓
GROUP BY
  ↓
HAVING
  ↓
SELECT
  ↓
DISTINCT
  ↓
ORDER BY
  ↓
LIMIT / OFFSET
```

This is the simplified logical order documented by PostgreSQL.

---

# PART 5 — CASE

## What is CASE?

`CASE` is SQL's way of expressing **conditional logic**.

You can think of it as:

```text
if
else if
else
```

from normal programming languages.

PostgreSQL calls `CASE` a **conditional expression** and supports both searched and simple forms. It can be used wherever an expression is valid.

---

# Real-Life Analogy

In programming:

```text
if marks >= 90:
    Grade A
else if marks >= 75:
    Grade B
else:
    Grade C
```

In SQL:

```sql
CASE
    WHEN marks >= 90 THEN 'A'
    WHEN marks >= 75 THEN 'B'
    ELSE 'C'
END
```

---

# Basic CASE Syntax

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE result3
END
```

---

# Simple Example

```sql
SELECT name,
       marks,
       CASE
           WHEN marks >= 90 THEN 'Excellent'
           WHEN marks >= 75 THEN 'Good'
           WHEN marks >= 60 THEN 'Average'
           ELSE 'Needs Improvement'
       END AS performance
FROM students;
```

Output conceptually:

| name | marks | performance |
|---|---:|---|
| Arnav | 85 | Good |
| Rahul | 72 | Average |
| Priya | 91 | Excellent |
| Amit | 68 | Average |
| Aakash | 95 | Excellent |

---

# CASE with Multiple WHEN

This is like:

```text
if
else if
else if
else
```

Example:

```sql
CASE
    WHEN marks >= 90 THEN 'A'
    WHEN marks >= 80 THEN 'B'
    WHEN marks >= 70 THEN 'C'
    WHEN marks >= 60 THEN 'D'
    ELSE 'F'
END
```

---

# IMPORTANT: CASE Stops at the First Match

Suppose marks = `95`.

```sql
CASE
    WHEN marks >= 60 THEN 'D'
    WHEN marks >= 90 THEN 'A'
    ELSE 'F'
END
```

Result:

```text
D
```

Why?

Because:

```text
marks >= 60
```

is already true.

The later condition isn't used.

### Very important rule

> **Put more specific conditions before broader conditions.**

Correct:

```sql
CASE
    WHEN marks >= 90 THEN 'A'
    WHEN marks >= 80 THEN 'B'
    WHEN marks >= 70 THEN 'C'
    ELSE 'F'
END
```

---

# CASE Without ELSE

You can omit `ELSE`.

Example:

```sql
CASE
    WHEN marks >= 90 THEN 'Excellent'
END
```

If no condition is true, the result is:

```text
NULL
```

### Best practice for exams

Usually write:

```sql
ELSE 'Other'
```

or another suitable fallback.

---

# CASE Result Data Types

All `THEN`/`ELSE` results need to be convertible to a common data type.

Good:

```sql
CASE
    WHEN marks >= 80 THEN 'Pass'
    ELSE 'Fail'
END
```

Both results are text.

Another valid example:

```sql
CASE
    WHEN marks >= 80 THEN 1
    ELSE 0
END
```

Both results are numeric.

---

# PART 6 — CASE WITH SELECT

This is the most common use.

```sql
SELECT
    name,
    marks,
    CASE
        WHEN marks >= 90 THEN 'Excellent'
        WHEN marks >= 80 THEN 'Very Good'
        WHEN marks >= 70 THEN 'Good'
        ELSE 'Needs Improvement'
    END AS result
FROM students;
```

### Hinglish

`CASE` SELECT ke andar ek **calculated column** bana raha hai.

Database mein koi actual column `result` nahi tha.

Query runtime par `result` calculate kar rahi hai.

---

# CASE with City

```sql
SELECT
    name,
    city,
    CASE
        WHEN city = 'Pune' THEN 'Maharashtra'
        WHEN city = 'Jaipur' THEN 'Rajasthan'
        ELSE 'Other State'
    END AS state_group
FROM students;
```

This is useful for:

- categorization
- labeling
- reporting
- derived columns

---

# CASE with AND / OR

```sql
SELECT
    name,
    marks,
    age,
    CASE
        WHEN marks >= 80 AND age <= 20 THEN 'Young High Scorer'
        WHEN marks >= 80 THEN 'High Scorer'
        ELSE 'Other'
    END AS category
FROM students;
```

`WHEN` can contain normal Boolean logic.

---

# CASE with NULL

Suppose we have a nullable `email` column.

```sql
SELECT
    name,
    CASE
        WHEN email IS NULL THEN 'Email Missing'
        ELSE 'Email Available'
    END AS email_status
FROM students;
```

This is a very common data-quality use.

---

# CASE with IS NULL

Remember:

```sql
email IS NULL
```

not:

```sql
email = NULL
```

So:

```sql
CASE
    WHEN email IS NULL THEN 'Missing'
    ELSE 'Present'
END
```

---

# PART 7 — SIMPLE CASE FORM

There are two broad forms of `CASE`.

## A. Searched CASE

This is what we've mostly used:

```sql
CASE
    WHEN marks >= 90 THEN 'A'
    WHEN marks >= 80 THEN 'B'
    ELSE 'C'
END
```

Each `WHEN` contains a complete condition.

---

## B. Simple CASE

Here one expression is compared against possible values.

Syntax:

```sql
CASE expression
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ELSE result3
END
```

Example:

```sql
SELECT
    name,
    course,
    CASE course
        WHEN 'DBMS' THEN 'Database Course'
        WHEN 'Python' THEN 'Programming Course'
        WHEN 'Java' THEN 'Programming Course'
        ELSE 'Other'
    END AS course_type
FROM students;
```

### Hinglish

Here PostgreSQL asks:

```text
course kya hai?
```

Then:

```text
DBMS    → Database Course
Python  → Programming Course
Java    → Programming Course
```

The searched form is usually more flexible because `WHEN` can use conditions such as `>`, `<`, `BETWEEN`, `LIKE`, etc.

---

# PART 8 — CASE WITH ORDER BY

This is a very powerful use.

Suppose you want a **custom sorting order**:

```text
DBMS first
Python second
Java third
Everything else after
```

Normal alphabetical order won't give that.

Use:

```sql
SELECT name, course
FROM students
ORDER BY
    CASE
        WHEN course = 'DBMS' THEN 1
        WHEN course = 'Python' THEN 2
        WHEN course = 'Java' THEN 3
        ELSE 4
    END;
```

---

# How It Works

The CASE generates a temporary ordering value:

```text
DBMS   → 1
Python → 2
Java   → 3
Other  → 4
```

Then `ORDER BY` sorts using those numbers.

### Hinglish

CASE yahan directly result column nahi bana raha.

It is creating a **sorting priority**.

---

# CASE + ORDER BY Marks

You can create custom ranking:

```sql
SELECT name,
       marks
FROM students
ORDER BY
    CASE
        WHEN marks >= 90 THEN 1
        WHEN marks >= 80 THEN 2
        WHEN marks >= 70 THEN 3
        ELSE 4
    END,
    marks DESC;
```

Meaning:

```text
Excellent students first
Then very good
Then good
Then others
```

and within each category:

```text
marks DESC
```

sorts highest first.

---

# PART 9 — CASE INSIDE COUNT()

This is extremely important.

You can use `CASE` inside an aggregate function.

Example:

> Count how many students scored at least 80.

```sql
SELECT
    COUNT(
        CASE
            WHEN marks >= 80 THEN 1
        END
    ) AS high_scorers
FROM students;
```

How does it work?

For every row:

```text
marks >= 80?
    ↓
YES → 1
NO  → NULL
```

Then:

```text
COUNT(...)
```

counts only non-NULL results.

So effectively it counts students satisfying the condition.

---

# Alternative — COUNT with ELSE

You can write:

```sql
COUNT(
    CASE
        WHEN marks >= 80 THEN 1
        ELSE NULL
    END
)
```

Same basic idea.

But:

```text
ELSE NULL
```

is unnecessary because if `ELSE` is omitted and no `WHEN` matches, `CASE` returns NULL.

---

# COUNT Multiple Categories

Example:

```sql
SELECT
    COUNT(
        CASE WHEN marks >= 80 THEN 1 END
    ) AS above_80,

    COUNT(
        CASE WHEN marks < 80 THEN 1 END
    ) AS below_80

FROM students;
```

This gives you separate counts in one query.

---

# CASE + COUNT + GROUP BY

This becomes even more powerful.

```sql
SELECT
    course,

    COUNT(*) AS total_students,

    COUNT(
        CASE
            WHEN marks >= 80 THEN 1
        END
    ) AS students_above_80

FROM students
GROUP BY course;
```

Now each course gets:

```text
total students
+
students scoring >= 80
```

---

# Modern Alternative: FILTER

PostgreSQL also supports aggregate `FILTER`, which can make conditional aggregates clearer:

```sql
SELECT
    course,
    COUNT(*) AS total_students,
    COUNT(*) FILTER (WHERE marks >= 80) AS students_above_80
FROM students
GROUP BY course;
```

`FILTER` restricts input to the particular aggregate it is attached to.

For an exam, however, **CASE inside COUNT** is worth learning because it tests whether you understand both `CASE` and aggregates.

---

# PART 10 — CASE INSIDE SUM()

Another very common pattern.

Suppose:

```text
marks >= 80 → count as 1
otherwise → count as 0
```

Then:

```sql
SELECT
    SUM(
        CASE
            WHEN marks >= 80 THEN 1
            ELSE 0
        END
    ) AS high_scorers
FROM students;
```

How does it work?

Suppose marks are:

```text
85
72
91
68
```

CASE produces:

```text
1
0
1
0
```

Then:

```text
SUM = 2
```

So:

> `SUM(CASE...)` can be used as a conditional counter.

---

# COUNT(CASE) vs SUM(CASE)

### COUNT(CASE)

```sql
COUNT(
    CASE
        WHEN marks >= 80 THEN 1
    END
)
```

counts non-NULL CASE results.

### SUM(CASE)

```sql
SUM(
    CASE
        WHEN marks >= 80 THEN 1
        ELSE 0
    END
)
```

adds 1 for each matching row and 0 otherwise.

For a simple yes/no condition, they commonly produce the same count.

---

# Why `ELSE 0` Matters in SUM

Pattern:

```sql
SUM(
    CASE
        WHEN marks >= 80 THEN 1
        ELSE 0
    END
)
```

Explicitly producing `0` for non-matching rows is useful when you want a numeric conditional total.

If there are no input rows at all, `SUM` can still return NULL for an empty input set, so use `COALESCE` when a report specifically needs `0`.

Example:

```sql
COALESCE(
    SUM(
        CASE
            WHEN marks >= 80 THEN 1
            ELSE 0
        END
    ),
    0
)
```

---

# PART 11 — CASE WITH HAVING

Now combine:

```text
CASE
+
GROUP BY
+
HAVING
```

Suppose we want:

> Courses where at least 2 students scored 80 or above.

```sql
SELECT
    course,
    COUNT(
        CASE
            WHEN marks >= 80 THEN 1
        END
    ) AS high_scorers
FROM students
GROUP BY course
HAVING COUNT(
    CASE
        WHEN marks >= 80 THEN 1
    END
) >= 2;
```

---

# How It Works

### Step 1 — GROUP BY

Students are separated into:

```text
DBMS
Python
Java
```

### Step 2 — CASE

For each student:

```text
marks >= 80?
```

→ `1` if yes

→ `NULL` if no

### Step 3 — COUNT

Count the `1`s.

### Step 4 — HAVING

Keep only:

```text
high_scorers >= 2
```

This is an excellent example of why understanding query processing order matters.

---

# CASE + HAVING Another Example

Suppose we want courses where the average of only high-scoring students is at least 85.

A CASE-based approach:

```sql
SELECT
    course,
    AVG(
        CASE
            WHEN marks >= 80 THEN marks
        END
    ) AS high_score_average
FROM students
GROUP BY course
HAVING AVG(
    CASE
        WHEN marks >= 80 THEN marks
    END
) >= 85;
```

For each course:

```text
marks < 80 → NULL
marks >= 80 → marks
```

Then `AVG` works over the non-NULL values.

That gives you a conditional average.

---

# PART 12 — CASE WITH SELECT + GROUP BY

You can also classify first and group based on the classification.

Example:

```sql
SELECT
    CASE
        WHEN marks >= 80 THEN 'High'
        ELSE 'Low'
    END AS performance_group,

    COUNT(*) AS students

FROM students

GROUP BY
    CASE
        WHEN marks >= 80 THEN 'High'
        ELSE 'Low'
    END;
```

Result conceptually:

| performance_group | students |
|---|---:|
| High | 6 |
| Low | 4 |

The exact counts depend on the data.

---

# Why repeat CASE in GROUP BY?

Because the query needs to group by the same expression.

Depending on the exact SQL/PostgreSQL context, positional grouping can sometimes be used:

```sql
GROUP BY 1
```

But for exam clarity, writing the full expression is often easier to understand.

PostgreSQL allows grouping by expressions and supports positional references in `GROUP BY`.

---

# PART 13 — CASE WITH WHERE

A useful combination for understanding what CASE can do.

Example:

```sql
SELECT *
FROM students
WHERE
    CASE
        WHEN course = 'DBMS' THEN marks
        ELSE 0
    END >= 80;
```

Meaning:

```text
If course is DBMS:
    compare marks with 80

Otherwise:
    use 0
```

This is valid conceptually, but often a direct Boolean condition is clearer:

```sql
WHERE course = 'DBMS'
  AND marks >= 80
```

### Important lesson

> `CASE` is not always the best replacement for normal Boolean logic.

Use it when you actually need **conditional values**, categories, custom ordering, or conditional aggregation.

---

# PART 14 — CASE for Grade Calculation

A classic exam question.

```sql
SELECT
    name,
    marks,
    CASE
        WHEN marks >= 90 THEN 'A'
        WHEN marks >= 80 THEN 'B'
        WHEN marks >= 70 THEN 'C'
        WHEN marks >= 60 THEN 'D'
        ELSE 'F'
    END AS grade
FROM students;
```

---

# CASE for Pass/Fail

```sql
SELECT
    name,
    CASE
        WHEN marks >= 40 THEN 'PASS'
        ELSE 'FAIL'
    END AS status
FROM students;
```

---

# CASE for Age Category

```sql
SELECT
    name,
    age,
    CASE
        WHEN age < 18 THEN 'Minor'
        WHEN age BETWEEN 18 AND 21 THEN 'Young Adult'
        ELSE 'Adult'
    END AS age_group
FROM students;
```

---

# CASE for NULL Status

```sql
SELECT
    name,
    CASE
        WHEN email IS NULL THEN 'Missing'
        ELSE 'Available'
    END AS email_status
FROM students;
```

---

# PART 15 — CASE + Aggregate Dashboard

Now let's create a useful report.

```sql
SELECT
    course,

    COUNT(*) AS total_students,

    SUM(
        CASE
            WHEN marks >= 80 THEN 1
            ELSE 0
        END
    ) AS high_scorers,

    SUM(
        CASE
            WHEN marks < 40 THEN 1
            ELSE 0
        END
    ) AS failed_students,

    AVG(marks) AS average_marks,

    MIN(marks) AS minimum_marks,

    MAX(marks) AS maximum_marks

FROM students

GROUP BY course;
```

This single query gives:

```text
Course
↓
Total students
↓
High scorers
↓
Failed students
↓
Average marks
↓
Minimum
↓
Maximum
```

This is a realistic **data analytics/reporting query**.

---

# PART 16 — WHERE vs HAVING — Deep Example

Let's compare these carefully.

---

## Query A

```sql
SELECT course,
       COUNT(*)
FROM students
WHERE marks >= 80
GROUP BY course;
```

Meaning:

> Only students with marks >= 80 enter the groups.

Then count them by course.

---

## Query B

```sql
SELECT course,
       COUNT(*)
FROM students
GROUP BY course
HAVING COUNT(*) >= 2;
```

Meaning:

> First create groups using all students. Then keep courses having at least 2 students.

---

## Query C

```sql
SELECT course,
       COUNT(*)
FROM students
WHERE marks >= 80
GROUP BY course
HAVING COUNT(*) >= 2;
```

Meaning:

```text
1. Remove students with marks < 80
2. Group remaining students by course
3. Keep courses with at least 2 remaining students
```

This is an extremely important pattern.

---

# PART 17 — The Difference in One Sentence

### WHERE

> **Which rows should participate in the calculation?**

### HAVING

> **Which groups/results should remain after the calculation?**

That sentence is worth memorizing.

---

# PART 18 — Aggregate Functions + NULL

This is a very important exam topic.

In general, PostgreSQL aggregate functions such as `SUM`, `AVG`, `MIN`, and `MAX` ignore NULL input values; `COUNT(column)` also counts only non-NULL values. `COUNT(*)`, however, counts input rows. Also, except for `COUNT`, aggregates generally return NULL for an empty input set.

Example:

```text
marks:
80
90
NULL
```

Then:

```text
COUNT(*)       → 3
COUNT(marks)   → 2
SUM(marks)     → 170
AVG(marks)     → 85
MIN(marks)     → 80
MAX(marks)     → 90
```

This is a **must-remember table**.

---

# PART 19 — Can We Use Aggregate in WHERE?

No, not directly at the same query level.

Wrong:

```sql
SELECT course
FROM students
WHERE AVG(marks) > 80
GROUP BY course;
```

Why?

Because:

```text
WHERE
 ↓
before aggregate calculation
```

Correct:

```sql
SELECT course,
       AVG(marks)
FROM students
GROUP BY course
HAVING AVG(marks) > 80;
```

PostgreSQL's aggregate tutorial explains this conceptual reason.

---

# PART 20 — Finding the Student with Maximum Marks

A classic conceptual problem.

You cannot simply do:

```sql
SELECT name
FROM students
WHERE marks = MAX(marks);
```

because `MAX()` cannot be used directly inside the same-level `WHERE`.

Instead, one approach is a subquery:

```sql
SELECT name, marks
FROM students
WHERE marks = (
    SELECT MAX(marks)
    FROM students
);
```

Result:

```text
Aakash | 95
```

This connects:

```text
Aggregate Functions
+
WHERE
+
Subqueries
```

---

# PART 21 — SQL Running Order with CASE

Consider:

```sql
SELECT
    course,
    CASE
        WHEN AVG(marks) >= 80 THEN 'Strong'
        ELSE 'Needs Improvement'
    END AS category
FROM students
GROUP BY course;
```

Let's see the logic:

### FROM

Get students.

### GROUP BY

Create course groups.

### AVG

Calculate average marks for each course.

### SELECT

Run the CASE using those aggregate results.

Result conceptually:

```text
DBMS   → Strong
Python → Strong
Java   → Needs Improvement
```

This demonstrates why knowing the running order matters.

---

# PART 22 — CASE + HAVING + Aggregate

Now:

```sql
SELECT
    course,
    AVG(marks) AS avg_marks,
    CASE
        WHEN AVG(marks) >= 80 THEN 'Strong'
        ELSE 'Weak'
    END AS category

FROM students

GROUP BY course

HAVING AVG(marks) >= 70;
```

Flow:

```text
FROM
 ↓
GROUP BY
 ↓
AVG
 ↓
HAVING avg >= 70
 ↓
SELECT
 ↓
CASE evaluates avg
```

---

# PART 23 — Custom ORDER BY with CASE

Suppose your business rule is:

```text
Pune first
Jaipur second
Udaipur third
Kota last
```

Use:

```sql
SELECT name, city
FROM students
ORDER BY
    CASE city
        WHEN 'Pune' THEN 1
        WHEN 'Jaipur' THEN 2
        WHEN 'Udaipur' THEN 3
        WHEN 'Kota' THEN 4
        ELSE 5
    END;
```

This is a classic use of `CASE` in `ORDER BY`.

---

# PART 24 — CASE + ORDER BY + DESC

You can also mix:

```sql
SELECT name, marks
FROM students
ORDER BY
    CASE
        WHEN marks >= 80 THEN 1
        ELSE 2
    END,
    marks DESC;
```

This means:

```text
First → 80+ students
Then  → everyone else

Within each category:
highest marks first
```

---

# PART 25 — CASE for Conditional SUM

Suppose you want total marks only for high scorers:

```sql
SELECT
    SUM(
        CASE
            WHEN marks >= 80 THEN marks
            ELSE 0
        END
    ) AS high_scorer_marks
FROM students;
```

Example logic:

```text
85 → 85
72 → 0
91 → 91
68 → 0
88 → 88
```

Then:

```text
85 + 91 + 88 + ...
```

This is called **conditional aggregation**.

---

# PART 26 — CASE for Conditional AVG

Suppose you want average marks only for students scoring at least 80:

```sql
SELECT
    AVG(
        CASE
            WHEN marks >= 80 THEN marks
        END
    ) AS average_high_scores
FROM students;
```

For rows below 80:

```text
CASE → NULL
```

`AVG` ignores those NULL values.

So only qualifying marks participate in the average.

---

# PART 27 — CASE for Multiple Counts

Very useful for dashboards.

```sql
SELECT
    COUNT(*) AS total,

    COUNT(
        CASE WHEN marks >= 90 THEN 1 END
    ) AS excellent,

    COUNT(
        CASE
            WHEN marks >= 80 AND marks < 90
            THEN 1
        END
    ) AS very_good,

    COUNT(
        CASE
            WHEN marks < 40
            THEN 1
        END
    ) AS failed

FROM students;
```

One query gives multiple statistics.

---

# PART 28 — GROUP BY + CASE

You don't have to group only by a table column.

You can group by a calculated category.

Example:

```sql
SELECT
    CASE
        WHEN marks >= 80 THEN 'High'
        WHEN marks >= 60 THEN 'Medium'
        ELSE 'Low'
    END AS performance,
    COUNT(*) AS students
FROM students
GROUP BY
    CASE
        WHEN marks >= 80 THEN 'High'
        WHEN marks >= 60 THEN 'Medium'
        ELSE 'Low'
    END;
```

Output conceptually:

| performance | students |
|---|---:|
| High | 6 |
| Medium | 4 |
| Low | 0 |

The exact counts depend on the data.

### Hinglish

Pehle har student ko CASE category milegi:

```text
85 → High
72 → Medium
91 → High
68 → Medium
...
```

Then `GROUP BY` same categories ko group karega.

---

# PART 29 — `CASE` Is an Expression, Not a Standalone Query Command

This is an important conceptual point.

You don't normally write:

```sql
CASE ...
```

by itself as a query.

You use it **inside an expression**, such as:

```sql
SELECT
    CASE ...
END
```

or:

```sql
ORDER BY
    CASE ...
END
```

or:

```sql
SUM(
    CASE ...
END
)
```

PostgreSQL explicitly describes `CASE` as an expression that can be used wherever an expression is valid.

---

# PART 30 — CASE vs IF

For exam understanding:

```text
CASE → SQL query expression
IF   → procedural programming/control structure
```

You may know `IF` from languages such as:

```text
Python
Java
C
JavaScript
```

SQL commonly uses:

```sql
CASE
```

for conditional query expressions.

PostgreSQL also has procedural `IF` and `CASE` statements inside PL/pgSQL, but that is a different context from the `CASE` expression used in ordinary SQL queries.

---

# 🔥 THE MOST IMPORTANT AGGREGATE PATTERNS

## Total

```sql
SELECT SUM(marks)
FROM students;
```

## Average

```sql
SELECT AVG(marks)
FROM students;
```

## Highest

```sql
SELECT MAX(marks)
FROM students;
```

## Lowest

```sql
SELECT MIN(marks)
FROM students;
```

## Number of rows

```sql
SELECT COUNT(*)
FROM students;
```

## Unique values count

```sql
SELECT COUNT(DISTINCT city)
FROM students;
```

---

# 🔥 GROUP BY Patterns

## Count per course

```sql
SELECT course, COUNT(*)
FROM students
GROUP BY course;
```

## Average per course

```sql
SELECT course, AVG(marks)
FROM students
GROUP BY course;
```

## Maximum per course

```sql
SELECT course, MAX(marks)
FROM students
GROUP BY course;
```

---

# 🔥 HAVING Patterns

## Courses with at least 3 students

```sql
SELECT course, COUNT(*)
FROM students
GROUP BY course
HAVING COUNT(*) >= 3;
```

## Courses with average >= 80

```sql
SELECT course, AVG(marks)
FROM students
GROUP BY course
HAVING AVG(marks) >= 80;
```

---

# 🔥 CASE Patterns

## Grade

```sql
CASE
    WHEN marks >= 90 THEN 'A'
    WHEN marks >= 80 THEN 'B'
    WHEN marks >= 70 THEN 'C'
    ELSE 'F'
END
```

## Pass/Fail

```sql
CASE
    WHEN marks >= 40 THEN 'PASS'
    ELSE 'FAIL'
END
```

## NULL handling

```sql
CASE
    WHEN email IS NULL THEN 'Missing'
    ELSE 'Available'
END
```

## Custom ordering

```sql
ORDER BY
    CASE
        WHEN course = 'DBMS' THEN 1
        WHEN course = 'Python' THEN 2
        WHEN course = 'Java' THEN 3
        ELSE 4
    END
```

## Conditional count

```sql
COUNT(
    CASE
        WHEN marks >= 80 THEN 1
    END
)
```

## Conditional sum

```sql
SUM(
    CASE
        WHEN marks >= 80 THEN 1
        ELSE 0
    END
)
```

---

# 🧠 THE BIGGEST CONCEPT: WHERE vs GROUP BY vs HAVING

Remember this picture:

```text
TABLE
 │
 │ FROM
 ▼
All rows
 │
 │ WHERE
 ▼
Filtered rows
 │
 │ GROUP BY
 ▼
Groups
 │
 │ Aggregate functions
 ▼
Group calculations
 │
 │ HAVING
 ▼
Filtered groups
 │
 │ SELECT
 ▼
Final columns/expressions
 │
 │ ORDER BY
 ▼
Sorted result
```

This mental model will solve a huge number of SQL questions.

---

# 🧠 CASE Mental Model

Think:

```text
CASE
  ↓
Check WHEN #1
  ↓
True? → return THEN result
  ↓ No
Check WHEN #2
  ↓
True? → return THEN result
  ↓ No
...
  ↓
ELSE
```

And:

> **First matching `WHEN` wins.**

---

# 🧠 Aggregate + CASE Mental Model

For:

```sql
SUM(
    CASE
        WHEN marks >= 80 THEN 1
        ELSE 0
    END
)
```

Think:

```text
Each row
   ↓
CASE
   ↓
80+ ? → 1
<80 ? → 0
   ↓
SUM
   ↓
Total number of 80+ students
```

For:

```sql
COUNT(
    CASE
        WHEN marks >= 80 THEN 1
    END
)
```

think:

```text
Each row
   ↓
80+ ? → 1
otherwise → NULL
   ↓
COUNT
   ↓
Count non-NULL results
```

---

# 🎯 EXAM TRAPS

## Trap 1 — COUNT(*) vs COUNT(column)

```text
COUNT(*) → all rows
COUNT(column) → only non-NULL values
```

---

## Trap 2 — SUM of no rows

Don't assume:

```text
SUM → 0
```

PostgreSQL returns:

```text
SUM → NULL
```

for an empty input set.

Use:

```sql
COALESCE(SUM(marks), 0)
```

when you need zero.

---

## Trap 3 — AVG ignores NULL

```text
80
90
NULL
```

Average:

```text
85
```

not:

```text
56.67
```

---

## Trap 4 — Aggregate in WHERE

Wrong:

```sql
WHERE COUNT(*) > 2
```

Correct:

```sql
HAVING COUNT(*) > 2
```

---

## Trap 5 — Forgetting GROUP BY

Wrong:

```sql
SELECT course, COUNT(*)
FROM students;
```

You're asking for a course value plus a single aggregate result, without defining which course corresponds to that result.

Correct:

```sql
SELECT course, COUNT(*)
FROM students
GROUP BY course;
```

---

## Trap 6 — CASE order

Wrong:

```sql
CASE
    WHEN marks >= 60 THEN 'Pass'
    WHEN marks >= 90 THEN 'Excellent'
END
```

A 95 gets:

```text
Pass
```

because the first condition matches.

Correct:

```sql
CASE
    WHEN marks >= 90 THEN 'Excellent'
    WHEN marks >= 60 THEN 'Pass'
END
```

---

## Trap 7 — CASE without ELSE

If nothing matches:

```text
CASE without ELSE
→ NULL
```

---

# 🔥 MOST IMPORTANT COMPARISONS

## WHERE vs HAVING

```text
WHERE
→ filters rows

HAVING
→ filters groups
```

---

## COUNT(*) vs COUNT(column)

```text
COUNT(*)
→ count every row

COUNT(column)
→ count non-NULL values
```

---

## COUNT(CASE) vs SUM(CASE)

```text
COUNT(CASE WHEN condition THEN 1 END)
→ count matching rows using non-NULL results

SUM(CASE WHEN condition THEN 1 ELSE 0 END)
→ add 1 for matches and 0 otherwise
```

Both are common conditional-counting techniques.

---

## Simple CASE vs Searched CASE

### Simple

```sql
CASE course
    WHEN 'DBMS' THEN 'Database'
    WHEN 'Java' THEN 'Programming'
END
```

Compares one expression to values.

### Searched

```sql
CASE
    WHEN marks >= 90 THEN 'A'
    WHEN marks >= 80 THEN 'B'
END
```

Tests arbitrary Boolean conditions.

---

# 🚀 ONE HUGE QUERY

Here's a query that combines almost everything you've learned:

```sql
SELECT
    course,

    COUNT(*) AS total_students,

    COUNT(
        CASE
            WHEN marks >= 80 THEN 1
        END
    ) AS high_scorers,

    ROUND(AVG(marks), 2) AS average_marks,

    MIN(marks) AS minimum_marks,

    MAX(marks) AS maximum_marks,

    CASE
        WHEN AVG(marks) >= 85 THEN 'Excellent Course'
        WHEN AVG(marks) >= 75 THEN 'Good Course'
        ELSE 'Needs Improvement'
    END AS course_rating

FROM students

WHERE age >= 20

GROUP BY course

HAVING COUNT(*) >= 2

ORDER BY
    CASE
        WHEN AVG(marks) >= 85 THEN 1
        WHEN AVG(marks) >= 75 THEN 2
        ELSE 3
    END,
    average_marks DESC;
```

Don't try to memorize this whole query.

Understand the flow:

```text
FROM students
      ↓
WHERE age >= 20
      ↓
GROUP BY course
      ↓
COUNT / AVG / MIN / MAX
      ↓
HAVING COUNT(*) >= 2
      ↓
SELECT final columns
      ↓
CASE creates course rating
      ↓
ORDER BY CASE creates custom priority
```

This single query combines:

```text
Aggregate Functions
+
WHERE
+
GROUP BY
+
HAVING
+
CASE
+
ORDER BY
```

---

# 📌 FINAL EXAM REVISION SHEET

## Aggregate Functions

```text
COUNT → how many
SUM   → total
AVG   → average
MIN   → smallest
MAX   → largest
```

## GROUP BY

```text
Same values → make groups
```

## WHERE

```text
Filter rows before grouping
```

## HAVING

```text
Filter groups after grouping
```

## SQL Logical Processing Order

```text
FROM
↓
WHERE
↓
GROUP BY / Aggregate
↓
HAVING
↓
SELECT
↓
DISTINCT
↓
ORDER BY
↓
LIMIT / OFFSET
```

## CASE

```text
CASE
WHEN condition THEN result
WHEN condition THEN result
ELSE result
END
```

Think:

```text
IF
ELSE IF
ELSE
```

## CASE Rules

```text
First matching WHEN wins
No match + no ELSE → NULL
THEN/ELSE results need compatible types
```

## CASE Uses

```text
SELECT
→ create categories

ORDER BY
→ custom sorting

COUNT
→ conditional counting

SUM
→ conditional totals/counts

AVG
→ conditional average

HAVING
→ conditional group filtering

GROUP BY
→ group by calculated categories
```

---

# Quick Mental Model to Memorize

> **`WHERE` decides which rows enter the calculation, `GROUP BY` decides how those rows are grouped, aggregate functions calculate group summaries, `HAVING` decides which groups survive, `SELECT` creates the output expressions, and `ORDER BY` sorts the final result.**

And for `CASE`:

> **CASE is SQL's conditional expression: check conditions from top to bottom, return the result of the first true condition, and use `ELSE` as the fallback.**

---

# Official PostgreSQL References

The original notes were prepared with reference to PostgreSQL's official documentation, especially these sections:

- Aggregate Functions: https://www.postgresql.org/docs/18/functions-aggregate.html
- Tutorial — Aggregate Functions: https://www.postgresql.org/docs/18/tutorial-agg.html
- Table Expressions / `GROUP BY` / `HAVING`: https://www.postgresql.org/docs/18/queries-table-expressions.html
- `SELECT` processing order: https://www.postgresql.org/docs/18/sql-select.html
- Conditional Expressions / `CASE`: https://www.postgresql.org/docs/18/functions-conditional.html
- PostgreSQL 18 documentation PDF: https://www.postgresql.org/files/documentation/pdf/18/postgresql-18-US.pdf
