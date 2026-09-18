# PostgreSQL Functions — Detailed Exam Notes

## Topics Covered

- **Numeric Functions:** `ROUND`, `TRUNC`, `ROUND()` vs `TRUNC()`, `FLOOR`, `POWER`, `CEIL`, `ABS`
- **Date & Time Functions:** `NOW`, `EXTRACT`, `INTERVAL`, `DATE_TRUNC()`, `TO_CHAR()`
- **String Functions:** `UPPER()` / `LOWER()`, `LENGTH()`, `SUBSTRING()`, `CONCAT()` / `||`, `REPLACE()`, `TRIM()`, `LIKE` / `ILIKE`, `SPLIT_PART()`
- **NULL Handling:** `COALESCE()`, `IS NULL`, `IS NOT NULL`, `NULLIF()`, `IS DISTINCT FROM`, Function Nesting

---

# Introduction

This set is important because these functions show up constantly in **`SELECT` queries, calculated columns, filtering, data cleaning, date analysis, and NULL handling**.

The explanations below use simple English + Hinglish and one consistent student dataset so that the concepts connect with each other.

---

# Our Example Table

Imagine we have:

### `students`

| student_id | name | city | marks | fee | email | joined_at |
|---:|---|---|---:|---:|---|---|
| 101 | Arnav | Pune | 85.67 | 25000.75 | arnav@gmail.com | 2026-07-15 10:30:45 |
| 102 | Rahul | Jaipur | 72.34 | 22000.50 | rahul@gmail.com | 2026-07-20 14:15:20 |
| 103 | Priya | Pune | 91.89 | NULL | priya@gmail.com | 2026-08-02 09:10:05 |
| 104 | Amit | Udaipur | 68.45 | 18000.25 | NULL | 2026-08-10 16:45:30 |
| 105 | Ananya | Jaipur | 88.12 | 24000.00 | ananya@gmail.com | 2026-08-15 11:20:10 |

---

# PART 1 — NUMERIC FUNCTIONS

These functions are used when working with numbers such as:

- marks
- salary
- fees
- percentages
- prices
- calculations

The functions in the syllabus are:

```text
ROUND
TRUNC
FLOOR
POWER
CEIL
ABS
```

---

# 1. `ROUND()`

## What is ROUND()?

`ROUND()` is used to **round a number to the nearest integer or to a specified number of decimal places**.

PostgreSQL supports:

```sql
ROUND(number)
```

and:

```sql
ROUND(number, decimal_places)
```

For `numeric`, PostgreSQL rounds ties away from zero.

---

## Basic Syntax

```sql
ROUND(number);
```

or:

```sql
ROUND(number, decimal_places);
```

---

## Example 1 — Round to nearest integer

```sql
SELECT ROUND(85.67);
```

Result:

```text
86
```

Because:

```text
85.67 → 86
```

---

## Example 2

```sql
SELECT ROUND(85.21);
```

Result:

```text
85
```

Because:

```text
85.21 → 85
```

### Hinglish

`ROUND()` decimal ko dekhkar decide karta hai ki number ko nearest value par le jana hai.

Simple rule:

```text
decimal part < .5 → lower integer
decimal part >= .5 → upper integer
```

For PostgreSQL's `numeric`, exact tie cases such as `.5` round away from zero.

---

# ROUND with Decimal Places

Suppose:

```text
85.67891
```

We want 2 decimal places.

```sql
SELECT ROUND(85.67891, 2);
```

Result:

```text
85.68
```

---

## 1 decimal place

```sql
SELECT ROUND(85.67891, 1);
```

Result:

```text
85.7
```

---

## 3 decimal places

```sql
SELECT ROUND(85.67891, 3);
```

Result:

```text
85.679
```

---

# ROUND in a Table

```sql
SELECT name, marks, ROUND(marks, 1) AS rounded_marks
FROM students;
```

Example:

| name | marks | rounded_marks |
|---|---:|---:|
| Arnav | 85.67 | 85.7 |
| Rahul | 72.34 | 72.3 |
| Priya | 91.89 | 91.9 |

---

# ROUND with Negative Precision

This is a slightly advanced but useful case.

```sql
SELECT ROUND(1234.56, -1);
```

Result:

```text
1230
```

Here `-1` means round to the nearest **10**.

Similarly:

```sql
SELECT ROUND(1234.56, -2);
```

Result:

```text
1200
```

So:

```text
2 decimal places → 1234.56
1 decimal place  → 1234.6
0                → 1235
-1               → 1230
-2               → 1200
```

---

# 2. `TRUNC()`

## What is TRUNC()?

`TRUNC()` means **truncate**.

It removes the fractional portion instead of rounding.

PostgreSQL provides:

```sql
TRUNC(number)
```

and:

```sql
TRUNC(number, decimal_places)
```

The numeric form truncates toward zero.

---

## Basic Example

```sql
SELECT TRUNC(85.67);
```

Result:

```text
85
```

It simply cuts off:

```text
85.67
   ↓
85
```

---

## TRUNC with decimal places

```sql
SELECT TRUNC(85.67891, 2);
```

Result:

```text
85.67
```

Notice:

```text
ROUND → 85.68
TRUNC → 85.67
```

---

# TRUNC Doesn't Round

This is very important.

Suppose:

```text
85.999
```

### ROUND

```sql
SELECT ROUND(85.999, 2);
```

Result:

```text
86.00
```

### TRUNC

```sql
SELECT TRUNC(85.999, 2);
```

Result:

```text
85.99
```

### Hinglish

`ROUND()` bolta hai:

> "Decimal ko dekh kar value change kar sakta hoon."

`TRUNC()` bolta hai:

> "Bas extra decimal digits kaat do."

---

# TRUNC with Negative Number

This is a very important concept.

```sql
SELECT TRUNC(-42.8);
```

Result:

```text
-42
```

It moves **toward zero**, not toward negative infinity.

---

# 3. ROUND() vs TRUNC()

This is a **must-learn exam comparison**.

| Number | ROUND | TRUNC |
|---:|---:|---:|
| 42.8 | 43 | 42 |
| 42.3 | 42 | 42 |
| 42.56 with 1 decimal | 42.6 | 42.5 |
| -42.8 | -43 | -42 |

### Memory Trick

> `ROUND` → **nearest**  
> `TRUNC` → **cut**

---

# 4. `FLOOR()`

## What is FLOOR()?

`FLOOR()` returns the **largest integer less than or equal to the given number**.

Simple words:

> Number ko neeche wali integer value par le jao.

---

## Example

```sql
SELECT FLOOR(42.8);
```

Result:

```text
42
```

---

### Another example

```sql
SELECT FLOOR(42.1);
```

Result:

```text
42
```

### Hinglish

`FLOOR()` number ko neeche le jata hai.

---

# FLOOR with Negative Numbers

This is where students commonly make mistakes.

```sql
SELECT FLOOR(-42.8);
```

Result:

```text
-43
```

Why?

Because `-43` is the greatest integer that is **less than or equal to** `-42.8`.

Think number line:

```text
-43 ---- -42.8 ---- -42
```

`FLOOR()` goes left/down:

```text
-43
```

---

# 5. `CEIL()`

## What is CEIL()?

`CEIL()` returns the **smallest integer greater than or equal to the number**.

PostgreSQL also supports:

```sql
CEILING()
```

which is equivalent to `CEIL()`.

---

## Example

```sql
SELECT CEIL(42.1);
```

Result:

```text
43
```

---

```sql
SELECT CEIL(42.8);
```

Result:

```text
43
```

---

# CEIL with Negative Number

```sql
SELECT CEIL(-42.8);
```

Result:

```text
-42
```

Because:

```text
-42 > -42.8
```

---

# FLOOR vs CEIL

| Number | FLOOR | CEIL |
|---:|---:|---:|
| 42.8 | 42 | 43 |
| 42.1 | 42 | 43 |
| -42.8 | -43 | -42 |
| -42.1 | -43 | -42 |

### Memory Trick

```text
FLOOR → down
CEIL  → up
```

But remember with negative numbers:

```text
down = more negative
up   = less negative
```

---

# 6. `POWER()`

## What is POWER()?

`POWER()` raises one number to the power of another.

Syntax:

```sql
POWER(base, exponent)
```

---

## Example

```sql
SELECT POWER(2, 3);
```

Result:

```text
8
```

Because:

```text
2 × 2 × 2 = 8
```

---

## More Examples

```sql
SELECT POWER(5, 2);
```

```text
25
```

```sql
SELECT POWER(10, 3);
```

```text
1000
```

---

# Square

```sql
SELECT POWER(7, 2);
```

Result:

```text
49
```

---

# Cube

```sql
SELECT POWER(4, 3);
```

Result:

```text
64
```

---

# Negative Exponent

```sql
SELECT POWER(2, -2);
```

Mathematically:

```text
1 / 2²
= 1/4
= 0.25
```

---

# POWER with Table Data

Suppose we want square of marks:

```sql
SELECT name, marks, POWER(marks, 2) AS marks_squared
FROM students;
```

This is more common in mathematical/statistical calculations than ordinary student queries.

---

# 7. `ABS()`

## What is ABS()?

`ABS()` returns the **absolute value** of a number.

Meaning:

> Negative sign hata do and positive magnitude rakho.

---

## Examples

```sql
SELECT ABS(-50);
```

Result:

```text
50
```

```sql
SELECT ABS(50);
```

Result:

```text
50
```

```sql
SELECT ABS(-17.4);
```

Result:

```text
17.4
```

---

# When is ABS Useful?

Suppose actual marks = `85`

Expected marks = `90`

Difference:

```sql
SELECT ABS(85 - 90);
```

Result:

```text
5
```

Without `ABS()`:

```text
85 - 90 = -5
```

But often we care about the **size of the difference**, not its direction.

---

# Numeric Functions — Quick Revision

```text
ROUND  → nearest value
TRUNC  → cut decimal digits
FLOOR  → largest integer ≤ number
CEIL   → smallest integer ≥ number
POWER  → raise to power
ABS    → remove negative sign
```

---

# PART 2 — DATE & TIME FUNCTIONS

PostgreSQL has rich date/time support.

Important topics:

```text
NOW()
EXTRACT()
INTERVAL
DATE_TRUNC()
TO_CHAR()
```

A very important distinction:

> `INTERVAL` is a **date/time data type/value**, not an ordinary function.

---

# 8. `NOW()`

## What is NOW()?

`NOW()` returns the **current date and time**.

Example:

```sql
SELECT NOW();
```

You may get something like:

```text
2026-09-18 05:58:32.123+05:30
```

The exact value naturally changes.

---

# VERY IMPORTANT PostgreSQL Behavior

In PostgreSQL:

```sql
NOW()
```

is equivalent to:

```sql
transaction_timestamp()
```

and returns the **start time of the current transaction**. Therefore, repeated calls inside the same transaction normally return the same timestamp.

PostgreSQL also provides `clock_timestamp()` when you need the actual current clock time at each call.

### Hinglish

Ye thoda surprising hai.

`NOW()` ka matlab:

> "Current transaction ke start ka time"

rather than:

> "Har baar exact real-time clock time."

For normal application work, this distinction often doesn't matter, but for exams it's good to know.

---

# NOW() for Default Values

Example:

```sql
CREATE TABLE attendance (
    id INT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

When a row is inserted without specifying `created_at`, PostgreSQL can use the current transaction timestamp at insertion.

---

# NOW() with Table

```sql
SELECT name, NOW()
FROM students;
```

This returns the current timestamp alongside each student.

More practically:

```sql
SELECT name,
       joined_at,
       NOW() - joined_at AS time_since_joining
FROM students;
```

This uses interval arithmetic.

---

# 9. `EXTRACT()`

## What is EXTRACT()?

`EXTRACT()` is used to **take a specific part/subfield from a date, timestamp, time, or interval**.

Syntax:

```sql
EXTRACT(field FROM source)
```

PostgreSQL returns a numeric value. Supported source types include `timestamp`, `date`, `time`, and `interval`, though not every field is valid for every type.

---

# Easy Meaning

Suppose:

```text
2026-09-18 05:30:45
```

You can ask:

```text
Year?
Month?
Day?
Hour?
Minute?
Second?
```

That's what `EXTRACT()` does.

---

# EXTRACT Year

```sql
SELECT EXTRACT(YEAR FROM TIMESTAMP '2026-09-18 05:30:45');
```

Result:

```text
2026
```

---

# EXTRACT Month

```sql
SELECT EXTRACT(MONTH FROM TIMESTAMP '2026-09-18 05:30:45');
```

Result:

```text
9
```

---

# EXTRACT Day

```sql
SELECT EXTRACT(DAY FROM TIMESTAMP '2026-09-18 05:30:45');
```

Result:

```text
18
```

---

# EXTRACT Hour

```sql
SELECT EXTRACT(HOUR FROM TIMESTAMP '2026-09-18 05:30:45');
```

Result:

```text
5
```

---

# EXTRACT Minute

```sql
SELECT EXTRACT(MINUTE FROM TIMESTAMP '2026-09-18 05:30:45');
```

Result:

```text
30
```

---

# EXTRACT Second

```sql
SELECT EXTRACT(SECOND FROM TIMESTAMP '2026-09-18 05:30:45');
```

Result:

```text
45
```

---

# EXTRACT in Table Queries

Suppose:

```sql
SELECT name,
       EXTRACT(YEAR FROM joined_at) AS joining_year
FROM students;
```

This is useful for questions such as:

> Which year did students join?

---

# EXTRACT with WHERE

```sql
SELECT *
FROM students
WHERE EXTRACT(MONTH FROM joined_at) = 8;
```

Meaning:

> August mein join karne wale students.

---

# Useful EXTRACT Fields

Some important PostgreSQL-supported fields are:

```text
YEAR
MONTH
DAY
HOUR
MINUTE
SECOND
QUARTER
WEEK
DOW
ISODOW
DOY
EPOCH
```

PostgreSQL also supports additional fields such as decade, century, millennium, microseconds, milliseconds, ISO year, etc.

---

# `DOW` vs `ISODOW`

This can appear in exams.

### `DOW`

```text
Sunday = 0
Monday = 1
...
Saturday = 6
```

### `ISODOW`

```text
Monday = 1
Tuesday = 2
...
Sunday = 7
```

---

# EPOCH

`EPOCH` is useful when you want seconds from the Unix epoch for timestamps, or total seconds for intervals.

Example:

```sql
SELECT EXTRACT(EPOCH FROM TIMESTAMP '1970-01-02 00:00:00');
```

Conceptually:

```text
86400 seconds
```

The exact interpretation differs slightly depending on whether the source is `timestamp with time zone`, `timestamp`, `date`, or `interval`.

---

# 10. `INTERVAL`

## What is INTERVAL?

`INTERVAL` represents a **duration of time**.

Examples:

```text
1 day
3 hours
20 minutes
2 months
1 year
```

Again:

> `INTERVAL` is a data type/value used to represent time durations.

---

# Creating an Interval

```sql
SELECT INTERVAL '5 days';
```

Result:

```text
5 days
```

---

# More Examples

```sql
SELECT INTERVAL '3 hours';
```

```sql
SELECT INTERVAL '2 months';
```

```sql
SELECT INTERVAL '1 year 3 months 5 days';
```

---

# INTERVAL + Date

This is one of the most important uses.

```sql
SELECT DATE '2026-09-18' + INTERVAL '7 days';
```

Result:

```text
2026-09-25
```

PostgreSQL supports adding intervals to timestamps/dates and subtracting intervals as well.

---

# Subtract Interval

```sql
SELECT TIMESTAMP '2026-09-18 10:00'
       - INTERVAL '3 hours';
```

Result:

```text
2026-09-18 07:00
```

---

# Add Months

```sql
SELECT DATE '2026-09-18' + INTERVAL '2 months';
```

Result conceptually:

```text
2026-11-18
```

---

# Interval in Table Query

Suppose we want to find the date 30 days after joining:

```sql
SELECT name,
       joined_at,
       joined_at + INTERVAL '30 days' AS after_30_days
FROM students;
```

Very useful.

---

# Interval Multiplication

PostgreSQL also lets you multiply intervals by a number.

```sql
SELECT INTERVAL '1 day' * 7;
```

Result:

```text
7 days
```

PostgreSQL supports interval multiplication/division by a numeric scalar.

---

# Interval Difference

```sql
SELECT TIMESTAMP '2026-09-18'
       - TIMESTAMP '2026-09-10';
```

Result:

```text
8 days
```

Timestamp subtraction produces an `interval`.

---

# 11. `DATE_TRUNC()`

## What is DATE_TRUNC()?

`DATE_TRUNC()` is used to **truncate a timestamp/date to a specified precision**, such as:

```text
hour
day
week
month
quarter
year
```

It sets less-significant fields to zero (or the appropriate first value, such as the beginning of a month).

---

# Basic Syntax

```sql
DATE_TRUNC('precision', source)
```

---

# DATE_TRUNC Hour

Suppose:

```text
2026-09-18 05:47:32
```

Query:

```sql
SELECT DATE_TRUNC(
    'hour',
    TIMESTAMP '2026-09-18 05:47:32'
);
```

Result:

```text
2026-09-18 05:00:00
```

Minutes and seconds are removed.

---

# DATE_TRUNC Day

```sql
SELECT DATE_TRUNC(
    'day',
    TIMESTAMP '2026-09-18 05:47:32'
);
```

Result:

```text
2026-09-18 00:00:00
```

---

# DATE_TRUNC Month

```sql
SELECT DATE_TRUNC(
    'month',
    TIMESTAMP '2026-09-18 05:47:32'
);
```

Result:

```text
2026-09-01 00:00:00
```

---

# DATE_TRUNC Year

```sql
SELECT DATE_TRUNC(
    'year',
    TIMESTAMP '2026-09-18 05:47:32'
);
```

Result:

```text
2026-01-01 00:00:00
```

---

# DATE_TRUNC Week

```sql
SELECT DATE_TRUNC(
    'week',
    TIMESTAMP '2026-09-18 05:47:32'
);
```

This gives the beginning of the relevant week according to PostgreSQL's date/time behavior.

---

# Why DATE_TRUNC is Useful

Suppose you have thousands of transactions:

```text
2026-09-01 10:21
2026-09-01 13:42
2026-09-02 09:10
2026-09-05 16:33
...
```

You want sales **month-wise**.

You can group by:

```sql
DATE_TRUNC('month', transaction_date)
```

Example:

```sql
SELECT DATE_TRUNC('month', joined_at) AS month,
       COUNT(*) AS students
FROM students
GROUP BY DATE_TRUNC('month', joined_at);
```

This is a very important real-world use.

---

# DATE_TRUNC vs EXTRACT

This difference is very important.

### `EXTRACT`

Gets one specific piece:

```sql
EXTRACT(MONTH FROM joined_at)
```

Result:

```text
9
```

### `DATE_TRUNC`

Returns the date/time truncated to that level:

```sql
DATE_TRUNC('month', joined_at)
```

Result:

```text
2026-09-01 00:00:00
```

### Memory Trick

> `EXTRACT` → **take the part**
>
> `DATE_TRUNC` → **move timestamp to beginning of that part**

---

# DATE_TRUNC Time Zone Case

For `timestamp with time zone`, PostgreSQL performs truncation with respect to a time zone; an optional time-zone argument can be supplied.

For exam level:

> `DATE_TRUNC()` can truncate timestamps according to the requested precision, and timezone-aware timestamps can be truncated with timezone considerations.

---

# 12. `TO_CHAR()`

## What is TO_CHAR()?

`TO_CHAR()` converts a date/time or numeric value into a **formatted text string** according to a format pattern.

PostgreSQL supports `to_char()` for timestamp, timestamp with time zone, interval, and numeric values.

---

# Important

`TO_CHAR()` is mainly for **display formatting**.

For example:

Database value:

```text
2026-09-18 05:30:45
```

You may want:

```text
18-Sep-2026
```

---

# Basic Syntax

```sql
TO_CHAR(value, 'format')
```

---

# Date Example

```sql
SELECT TO_CHAR(
    TIMESTAMP '2026-09-18 05:30:45',
    'DD-MM-YYYY'
);
```

Result:

```text
18-09-2026
```

---

# Common Date Format Codes

| Pattern | Meaning |
|---|---|
| `YYYY` | 4-digit year |
| `YY` | 2-digit year |
| `MM` | month number |
| `DD` | day |
| `HH24` | hour 00–23 |
| `HH12` | 12-hour clock |
| `MI` | minute |
| `SS` | second |
| `Month` | month name |
| `Mon` | abbreviated month |

---

# Example

```sql
SELECT TO_CHAR(
    TIMESTAMP '2026-09-18 05:30:45',
    'DD Mon YYYY'
);
```

Result similar to:

```text
18 Sep 2026
```

---

# Time Example

```sql
SELECT TO_CHAR(
    TIMESTAMP '2026-09-18 17:30:45',
    'HH24:MI:SS'
);
```

Result:

```text
17:30:45
```

---

# 12-Hour Format

```sql
SELECT TO_CHAR(
    TIMESTAMP '2026-09-18 17:30:45',
    'HH12:MI:SS AM'
);
```

Result conceptually:

```text
05:30:45 PM
```

---

# Formatting a Table Column

```sql
SELECT name,
       TO_CHAR(joined_at, 'DD Mon YYYY') AS joined_date
FROM students;
```

Result might look like:

| name | joined_date |
|---|---|
| Arnav | 15 Jul 2026 |
| Rahul | 20 Jul 2026 |
| Priya | 02 Aug 2026 |

---

# IMPORTANT: TO_CHAR Returns TEXT

This is an important conceptual point.

```sql
TO_CHAR(joined_at, 'YYYY-MM-DD')
```

returns a **text string**, not a date.

So:

```text
DATE/TIMESTAMP
      ↓
 TO_CHAR()
      ↓
   TEXT
```

### Hinglish

`TO_CHAR()` ko data calculation ke liye nahi, **presentation/formatting** ke liye socho.

---

# TO_CHAR with Numbers

Although the topic is under date/time functions, PostgreSQL also supports:

```sql
TO_CHAR(number, format)
```

Example:

```sql
SELECT TO_CHAR(12500.5, '99999.99');
```

This can be used to format numeric output. PostgreSQL supports numeric formatting patterns such as `9`, `0`, decimal points, commas, signs, etc.

---

# PART 3 — STRING FUNCTIONS

String functions are useful in:

- cleaning data
- searching names
- formatting output
- extracting parts of strings
- combining text

Topics:

```text
UPPER()
LOWER()
LENGTH()
SUBSTRING()
CONCAT()
||
REPLACE()
TRIM()
LIKE / ILIKE
SPLIT_PART()
```

---

# 13. `UPPER()`

## What is UPPER()?

Converts a string to uppercase.

```sql
SELECT UPPER('arnav');
```

Result:

```text
ARNAV
```

---

# Table Example

```sql
SELECT UPPER(name)
FROM students;
```

Result:

```text
ARNAV
RAHUL
PRIYA
AMIT
ANANYA
```

---

# 14. `LOWER()`

Exactly the opposite.

```sql
SELECT LOWER('ARNAV');
```

Result:

```text
arnav
```

---

# Practical Use

Suppose users entered:

```text
Arnav
ARNAV
arnav
ArNaV
```

For normalized display or comparison, you might use:

```sql
SELECT LOWER(email)
FROM students;
```

or:

```sql
WHERE LOWER(name) = 'arnav'
```

---

# UPPER vs LOWER

| Function | Purpose |
|---|---|
| `UPPER()` | Convert to uppercase |
| `LOWER()` | Convert to lowercase |

---

# 15. `LENGTH()`

## What is LENGTH()?

`LENGTH()` returns the number of characters in a string.

Example:

```sql
SELECT LENGTH('Arnav');
```

Result:

```text
5
```

PostgreSQL's character length counts characters rather than raw bytes for text.

---

# Examples

```sql
SELECT LENGTH('Hello');
```

```text
5
```

```sql
SELECT LENGTH('Database');
```

```text
8
```

---

# LENGTH for Validation

Suppose student ID should have exactly 5 characters:

```sql
SELECT *
FROM students
WHERE LENGTH(CAST(student_id AS TEXT)) = 5;
```

This converts the numeric ID to text and checks the character length.

---

# 16. `SUBSTRING()`

## What is SUBSTRING()?

`SUBSTRING()` extracts a portion of a string.

PostgreSQL supports forms such as:

```sql
SUBSTRING(string FROM start FOR count)
```

and more advanced pattern/regular-expression forms.

---

# Basic Syntax

```sql
SUBSTRING(string FROM start FOR count)
```

---

# Example

```sql
SELECT SUBSTRING('Arnav' FROM 1 FOR 3);
```

Result:

```text
Arn
```

---

# Important: Position Starts from 1

PostgreSQL's substring positions are character positions starting from `1`.

So:

```text
A r n a v
1 2 3 4 5
```

Therefore:

```sql
SUBSTRING('Arnav' FROM 2 FOR 3)
```

gives:

```text
rna
```

---

# Example Without FOR

```sql
SELECT SUBSTRING('Arnav' FROM 3);
```

Result:

```text
nav
```

It starts at position 3 and continues to the end.

---

# Example With FOR Only

```sql
SELECT SUBSTRING('Arnav' FOR 2);
```

Result:

```text
Ar
```

---

# Practical Example

Suppose emails are:

```text
arnav@gmail.com
```

You could extract the first few characters:

```sql
SELECT SUBSTRING(email FROM 1 FOR 5)
FROM students;
```

---

# Advanced SUBSTRING — Pattern

PostgreSQL also supports substring extraction based on regular expressions:

```sql
SUBSTRING(string FROM pattern)
```

For example, PostgreSQL documents:

```sql
SELECT substring('Thomas' FROM '...$');
```

which returns:

```text
mas
```

This is an advanced use, so for basic exams concentrate primarily on start/count extraction.

---

# 17. `CONCAT()`

## What is CONCAT()?

`CONCAT()` combines multiple values into a single string.

Syntax:

```sql
CONCAT(value1, value2, value3, ...)
```

PostgreSQL's `concat()` converts argument values to text and **ignores NULL arguments**.

---

# Example

```sql
SELECT CONCAT('Hello ', 'Arnav');
```

Result:

```text
Hello Arnav
```

---

# Multiple Values

```sql
SELECT CONCAT('Arnav', ' ', 'Maheshwari');
```

Result:

```text
Arnav Maheshwari
```

---

# CONCAT with Numbers

```sql
SELECT CONCAT('Marks: ', 85);
```

Result:

```text
Marks: 85
```

PostgreSQL allows non-string values because `concat()` converts them to text.

---

# CONCAT with NULL

This is important.

```sql
SELECT CONCAT('Hello ', NULL);
```

Result:

```text
Hello
```

`NULL` is ignored by `CONCAT()`.

---

# 18. String Concatenation Operator `||`

You can also combine strings with:

```sql
||
```

Example:

```sql
SELECT 'Hello ' || 'Arnav';
```

Result:

```text
Hello Arnav
```

PostgreSQL's `||` concatenates strings, and it can also accept a non-string value when the other side is a string.

---

# Column Example

```sql
SELECT name || ' - ' || city
FROM students;
```

Result:

```text
Arnav - Pune
Rahul - Jaipur
Priya - Pune
```

---

# CONCAT vs `||`

This is an important difference.

Consider:

```sql
SELECT CONCAT('Hello ', NULL);
```

Returns:

```text
Hello
```

But:

```sql
SELECT 'Hello ' || NULL;
```

produces:

```text
NULL
```

because ordinary string concatenation with a NULL operand produces NULL.

### Memory Trick

> `CONCAT()` → NULL arguments are ignored
>
> `||` → NULL can make the whole concatenation NULL

For nullable database columns, this difference matters.

---

# 19. `REPLACE()`

The syllabus listed `REPLACE()` twice. It is the same function, so it only needs to be learned once properly.

## What is REPLACE()?

`REPLACE()` replaces **all occurrences** of a particular substring with another substring.

Syntax:

```sql
REPLACE(string, from, to)
```

---

# Example

```sql
SELECT REPLACE('I love SQL', 'SQL', 'PostgreSQL');
```

Result:

```text
I love PostgreSQL
```

---

# Multiple Occurrences

```sql
SELECT REPLACE(
    'abc abc abc',
    'abc',
    'XYZ'
);
```

Result:

```text
XYZ XYZ XYZ
```

All occurrences are replaced.

---

# Practical Example — Fixing City Names

Suppose data contains:

```text
New Delhi
New Delhi
New Delhi
```

and you want:

```text
Delhi
```

```sql
SELECT REPLACE(city, 'New ', '')
FROM students;
```

---

# Replace Characters

```sql
SELECT REPLACE('123-456-789', '-', '');
```

Result:

```text
123456789
```

Useful for cleaning phone numbers, IDs, etc.

---

# REPLACE Doesn't Modify the Stored Value Automatically

Important.

This:

```sql
SELECT REPLACE(name, 'A', 'X')
FROM students;
```

only changes the **query result**.

It does not permanently change the table.

For permanent modification:

```sql
UPDATE students
SET name = REPLACE(name, 'A', 'X');
```

### Hinglish

`SELECT + REPLACE`:

> result change

`UPDATE + REPLACE`:

> actual table data change

---

# 20. `TRIM()`

## What is TRIM()?

`TRIM()` removes unwanted characters, usually spaces, from the beginning and/or end of a string.

PostgreSQL supports:

```text
LEADING
TRAILING
BOTH
```

with `BOTH` being the default direction when the standard syntax is used.

---

# Basic Example

```sql
SELECT TRIM('   Arnav   ');
```

Result:

```text
Arnav
```

Leading and trailing spaces removed.

---

# TRIM LEADING

```sql
SELECT TRIM(LEADING FROM '   Arnav   ');
```

Result:

```text
Arnav   
```

Only beginning side cleaned.

---

# TRIM TRAILING

```sql
SELECT TRIM(TRAILING FROM '   Arnav   ');
```

Result:

```text
   Arnav
```

---

# TRIM BOTH

```sql
SELECT TRIM(BOTH FROM '   Arnav   ');
```

Both sides cleaned.

---

# Removing Specific Characters

PostgreSQL also lets you specify characters.

```sql
SELECT TRIM(BOTH 'x' FROM 'xxxArnavxxx');
```

Result:

```text
Arnav
```

More precisely, PostgreSQL removes the longest leading/trailing sequence consisting only of the specified characters.

---

# IMPORTANT

`TRIM('xxxArnavxxx', 'x')` does **not** mean:

> Remove every `x` anywhere.

It removes qualifying characters from the **ends**.

So:

```text
xxxArnavxxx
```

becomes:

```text
Arnav
```

But internal characters aren't removed.

---

# Practical Data Cleaning

Suppose a database accidentally contains:

```text
'  Arnav  '
' Rahul '
'    Priya'
```

You can clean:

```sql
SELECT TRIM(name)
FROM students;
```

---

# 21. `LIKE` / `ILIKE`

`LIKE` and `ILIKE` are string pattern-matching operators.

### `LIKE`

Pattern matching:

```sql
SELECT *
FROM students
WHERE name LIKE 'A%';
```

Meaning:

> Name `A` se start ho.

### `ILIKE`

Case-insensitive pattern matching in PostgreSQL:

```sql
SELECT *
FROM students
WHERE name ILIKE 'a%';
```

This can match `A...` and `a...` without case sensitivity in the normal PostgreSQL behavior.

---

# `%` and `_`

### `%`

Matches:

> zero or more characters

```sql
'A%'
```

means:

```text
A + anything
```

### `_`

Matches:

> exactly one character

```sql
'A__'
```

means:

```text
A + one character + one character
```

---

# LIKE vs ILIKE Example

Suppose:

```text
Arnav
arnav
ARNAV
```

### LIKE

```sql
WHERE name LIKE 'arnav'
```

normally only matches the exact case form.

### ILIKE

```sql
WHERE name ILIKE 'arnav'
```

matches irrespective of letter case in normal case-insensitive behavior.

---

# `NOT LIKE`

```sql
WHERE name NOT LIKE 'A%'
```

means:

> Names that don't start with A.

---

# `NOT ILIKE`

PostgreSQL also supports:

```sql
WHERE name NOT ILIKE 'a%'
```

which performs the negative case-insensitive pattern check.

---

# 22. `SPLIT_PART()`

## What is SPLIT_PART()?

`SPLIT_PART()` breaks a string using a delimiter and returns a particular field.

Syntax:

```sql
SPLIT_PART(string, delimiter, n)
```

PostgreSQL counts fields from `1`; PostgreSQL 18 also supports a negative `n` to count from the end.

---

# Example

Suppose:

```text
arnav@gmail.com
```

We want the username:

```text
arnav
```

Use:

```sql
SELECT SPLIT_PART(
    'arnav@gmail.com',
    '@',
    1
);
```

Result:

```text
arnav
```

---

# Get Domain

```sql
SELECT SPLIT_PART(
    'arnav@gmail.com',
    '@',
    2
);
```

Result:

```text
gmail.com
```

---

# Table Example

```sql
SELECT email,
       SPLIT_PART(email, '@', 1) AS username,
       SPLIT_PART(email, '@', 2) AS domain
FROM students
WHERE email IS NOT NULL;
```

This is an excellent practical example.

---

# SPLIT_PART with Comma

Suppose:

```text
Pune, Maharashtra, India
```

Then:

```sql
SELECT SPLIT_PART(
    'Pune, Maharashtra, India',
    ', ',
    1
);
```

Result:

```text
Pune
```

Second:

```sql
SELECT SPLIT_PART(
    'Pune, Maharashtra, India',
    ', ',
    2
);
```

Result:

```text
Maharashtra
```

Third:

```sql
SELECT SPLIT_PART(
    'Pune, Maharashtra, India',
    ', ',
    3
);
```

Result:

```text
India
```

---

# Negative Position

PostgreSQL 18 supports:

```sql
SPLIT_PART(string, delimiter, -n)
```

to count from the end.

Example:

```sql
SELECT SPLIT_PART(
    'a,b,c,d',
    ',',
    -1
);
```

Result:

```text
d
```

And:

```sql
SELECT SPLIT_PART(
    'a,b,c,d',
    ',',
    -2
);
```

Result:

```text
c
```

---

# PART 4 — NULL HANDLING

This is one of the **most important SQL concepts**.

Topics:

```text
COALESCE()
IS NULL
IS NOT NULL
NULLIF()
IS DISTINCT FROM
Function Nesting
```

Before learning these, you must understand:

# What is NULL?

`NULL` does **not** mean:

```text
0
```

It does not mean:

```text
''
```

It does not mean:

```text
false
```

It means approximately:

> **Missing / unknown / unavailable value**

---

# Example

Suppose:

| name | fee |
|---|---:|
| Arnav | 25000 |
| Priya | NULL |
| Amit | 18000 |

For Priya:

```text
fee = NULL
```

means:

> We don't have a fee value.

It does **not** necessarily mean fee = 0.

---

# 23. `IS NULL`

## What is IS NULL?

Used to check whether a value is NULL.

Correct:

```sql
SELECT *
FROM students
WHERE email IS NULL;
```

This returns students whose email is missing.

---

# WRONG

Do NOT write:

```sql
WHERE email = NULL
```

Why?

Because ordinary SQL comparison with NULL doesn't produce TRUE.

Use:

```sql
IS NULL
```

instead.

---

# Example

```sql
SELECT name
FROM students
WHERE email IS NULL;
```

Result:

```text
Amit
```

---

# 24. `IS NOT NULL`

Used to find values that are present.

```sql
SELECT *
FROM students
WHERE email IS NOT NULL;
```

Meaning:

> Jin students ka email available hai.

---

# IS NULL vs IS NOT NULL

```text
IS NULL
→ missing value

IS NOT NULL
→ value exists
```

---

# 25. `COALESCE()`

## What is COALESCE()?

`COALESCE()` returns the **first non-NULL value** from the list of arguments.

Syntax:

```sql
COALESCE(value1, value2, value3, ...)
```

PostgreSQL returns the first argument that is not NULL; it returns NULL only when all arguments are NULL.

---

# Simplest Example

```sql
SELECT COALESCE(NULL, 'Hello');
```

Result:

```text
Hello
```

Because first value is NULL, so PostgreSQL moves to the next one.

---

# Another Example

```sql
SELECT COALESCE(NULL, NULL, 'Arnav');
```

Result:

```text
Arnav
```

---

# All NULL

```sql
SELECT COALESCE(NULL, NULL, NULL);
```

Result:

```text
NULL
```

---

# Practical Example

Our table has:

```text
fee
25000
22000
NULL
18000
24000
```

Instead of displaying NULL:

```sql
SELECT name,
       COALESCE(fee, 0) AS fee
FROM students;
```

Now Priya gets:

```text
0
```

Instead of NULL.

---

# Multiple Fallbacks

Suppose:

```text
description
short_description
default text
```

You can do:

```sql
SELECT COALESCE(
    description,
    short_description,
    'No description'
);
```

Meaning:

```text
description available?
   ↓ yes → use it
   ↓ no
short description available?
   ↓ yes → use it
   ↓ no
use 'No description'
```

This is exactly the behavior of `COALESCE()`.

---

# Hinglish Memory Trick

> `COALESCE` = **"Pehla non-NULL value de do."**

---

# 26. `NULLIF()`

## What is NULLIF()?

`NULLIF(a, b)` returns:

- `NULL` if `a = b`
- otherwise `a`

PostgreSQL defines it this way.

---

# Basic Example

```sql
SELECT NULLIF(10, 10);
```

Result:

```text
NULL
```

because both values are equal.

---

```sql
SELECT NULLIF(10, 20);
```

Result:

```text
10
```

because they are different.

---

# Easy Way to Remember

> `NULLIF()` = **"Agar dono same hain, NULL kar do."**

---

# Practical Use: Avoid Division by Zero

Suppose:

```text
total_sales = 1000
number_of_orders = 0
```

This can create a division-by-zero problem.

You can use:

```sql
SELECT 1000 / NULLIF(0, 0);
```

Since:

```text
NULLIF(0,0) → NULL
```

you avoid dividing by zero.

A common pattern is:

```sql
SELECT total_sales / NULLIF(number_of_orders, 0)
FROM sales;
```

### Concept

```text
orders = 0
   ↓
NULLIF(orders, 0)
   ↓
NULL
   ↓
division doesn't divide by zero
```

This is one of the most useful practical applications of `NULLIF()`.

---

# COALESCE vs NULLIF

This is an excellent exam comparison.

### COALESCE

```text
NULL → replace with first available value
```

Example:

```sql
COALESCE(fee, 0)
```

### NULLIF

```text
specific value → convert to NULL
```

Example:

```sql
NULLIF(fee, 0)
```

### Memory Trick

```text
COALESCE → NULL ko replace karo
NULLIF   → value ko NULL banao
```

---

# 27. `IS DISTINCT FROM`

This is a very useful SQL null-safe comparison.

## The problem with `=`

Consider:

```sql
NULL = NULL
```

The result is not `TRUE`; it is `NULL`/unknown under SQL's normal three-valued logic.

But sometimes we need:

> "Are these two values different, while treating NULL as a comparable value?"

That's where:

```sql
IS DISTINCT FROM
```

comes in.

PostgreSQL guarantees that `IS DISTINCT FROM` returns either `TRUE` or `FALSE`, never NULL. A NULL is considered distinct from a non-NULL, while two NULLs are considered not distinct.

---

# Basic Examples

```sql
SELECT 10 IS DISTINCT FROM 20;
```

Result:

```text
TRUE
```

Different.

---

```sql
SELECT 10 IS DISTINCT FROM 10;
```

Result:

```text
FALSE
```

Same.

---

# NULL Examples

```sql
SELECT NULL IS DISTINCT FROM NULL;
```

Result:

```text
FALSE
```

Because both are considered **not distinct**.

---

```sql
SELECT NULL IS DISTINCT FROM 10;
```

Result:

```text
TRUE
```

Because NULL and 10 are different.

---

```sql
SELECT 10 IS DISTINCT FROM NULL;
```

Result:

```text
TRUE
```

---

# Opposite: `IS NOT DISTINCT FROM`

There is also:

```sql
IS NOT DISTINCT FROM
```

Think of it as a NULL-safe equality test.

Examples:

```sql
SELECT NULL IS NOT DISTINCT FROM NULL;
```

Result:

```text
TRUE
```

And:

```sql
SELECT 10 IS NOT DISTINCT FROM 10;
```

Result:

```text
TRUE
```

But:

```sql
SELECT 10 IS NOT DISTINCT FROM NULL;
```

Result:

```text
FALSE
```

---

# Why is IS DISTINCT FROM Useful?

Suppose you need to compare database values where NULL should behave like a real comparable state.

Normal:

```sql
a = b
```

can result in NULL when either side is NULL.

But:

```sql
a IS NOT DISTINCT FROM b
```

always gives TRUE or FALSE.

### Hinglish

Normal `=` ke saath NULL thoda tricky hai.

`IS DISTINCT FROM` bolta hai:

> "NULL ko bhi properly compare karo, aur mujhe definite TRUE/FALSE do."

---

# 28. Function Nesting

## What is Function Nesting?

**Function nesting** means using one function **inside another function**.

Example:

```sql
UPPER(TRIM(name))
```

Here:

```text
TRIM()
  ↓
UPPER()
```

The result of `TRIM()` becomes input to `UPPER()`.

---

# Example 1

Suppose:

```text
'   arnav   '
```

Query:

```sql
SELECT UPPER(TRIM('   arnav   '));
```

Step 1:

```text
TRIM('   arnav   ')
→ 'arnav'
```

Step 2:

```text
UPPER('arnav')
→ 'ARNAV'
```

Final:

```text
ARNAV
```

---

# Example 2 — LOWER + TRIM

```sql
SELECT LOWER(TRIM('   ARNAV   '));
```

Step by step:

```text
'   ARNAV   '
      ↓
TRIM
      ↓
'ARNAV'
      ↓
LOWER
      ↓
'arnav'
```

---

# Example 3 — REPLACE + UPPER

```sql
SELECT UPPER(
    REPLACE('hello sql', 'sql', 'postgresql')
);
```

First:

```text
hello sql
↓
hello postgresql
```

Then:

```text
HELLO POSTGRESQL
```

---

# Example 4 — COALESCE + UPPER

Suppose:

```text
email = NULL
```

You can do:

```sql
SELECT UPPER(
    COALESCE(email, 'NO EMAIL')
)
FROM students;
```

For NULL email:

```text
NULL
 ↓
COALESCE
 ↓
NO EMAIL
 ↓
UPPER
 ↓
NO EMAIL
```

---

# Example 5 — TRIM + LOWER + REPLACE

```sql
SELECT LOWER(
           REPLACE(
               TRIM(name),
               ' ',
               '_'
           )
       )
FROM students;
```

Suppose the input is:

```text
" Arnav Maheshwari "
```

Step 1:

```text
TRIM
↓
"Arnav Maheshwari"
```

Step 2:

```text
REPLACE spaces with _
↓
"Arnav_Maheshwari"
```

Step 3:

```text
LOWER
↓
"arnav_maheshwari"
```

This is a great example of **data cleaning using nested functions**.

---

# Example 6 — ROUND + ABS

Suppose we have:

```text
Actual = 85.67
Expected = 90.00
```

We want rounded absolute difference:

```sql
SELECT ROUND(ABS(85.67 - 90), 2);
```

Step 1:

```text
85.67 - 90
= -4.33
```

Step 2:

```text
ABS(-4.33)
= 4.33
```

Step 3:

```text
ROUND(4.33, 2)
= 4.33
```

---

# Example 7 — COALESCE + ROUND

Suppose fee can be NULL:

```sql
SELECT ROUND(
           COALESCE(fee, 0),
           2
       )
FROM students;
```

Flow:

```text
fee
 ↓
NULL?
 ↓ yes
0
 ↓
ROUND(...,2)
```

---

# Example 8 — DATE_TRUNC + TO_CHAR

This is a very useful combination.

```sql
SELECT TO_CHAR(
           DATE_TRUNC('month', joined_at),
           'Mon YYYY'
       )
FROM students;
```

Step by step:

```text
joined_at
   ↓
DATE_TRUNC('month', ...)
   ↓
2026-08-01 00:00:00
   ↓
TO_CHAR(..., 'Mon YYYY')
   ↓
Aug 2026
```

This is extremely useful for monthly reports.

---

# Example 9 — EXTRACT inside CONCAT

```sql
SELECT CONCAT(
    'Year: ',
    EXTRACT(YEAR FROM joined_at)
)
FROM students;
```

Result:

```text
Year: 2026
```

---

# Example 10 — COALESCE + CONCAT

```sql
SELECT CONCAT(
    name,
    ' - ',
    COALESCE(email, 'No Email')
)
FROM students;
```

Output:

```text
Arnav - arnav@gmail.com
Amit - No Email
```

---

# Function Nesting Rule

When you see:

```sql
A(B(C(value)))
```

read it **from inside to outside**.

Example:

```sql
UPPER(TRIM(name))
```

Read:

```text
1. TRIM(name)
2. UPPER(the result)
```

### Hinglish

Inner function pehle mentally solve karo, then outer function.

---

# FUNCTION CHEAT SHEET

## Numeric

```text
ROUND(x)
→ nearest integer

ROUND(x, 2)
→ 2 decimal places

TRUNC(x)
→ remove fractional part

TRUNC(x, 2)
→ cut after 2 decimal places

FLOOR(x)
→ greatest integer <= x

CEIL(x)
→ smallest integer >= x

POWER(x, y)
→ x^y

ABS(x)
→ absolute value
```

---

## Date & Time

```text
NOW()
→ current transaction timestamp

EXTRACT(YEAR FROM date)
→ year part

EXTRACT(MONTH FROM date)
→ month part

INTERVAL '7 days'
→ time duration

date + INTERVAL '7 days'
→ add duration

DATE_TRUNC('month', date)
→ beginning of month

TO_CHAR(date, 'DD-MM-YYYY')
→ formatted text
```

---

## String

```text
UPPER(text)
→ uppercase

LOWER(text)
→ lowercase

LENGTH(text)
→ number of characters

SUBSTRING(text FROM 2 FOR 3)
→ extract portion

CONCAT(a,b,c)
→ join values

a || b
→ concatenate

REPLACE(text, old, new)
→ replace all occurrences

TRIM(text)
→ remove surrounding spaces

LIKE
→ case-sensitive pattern matching

ILIKE
→ case-insensitive pattern matching

SPLIT_PART(text, delimiter, n)
→ split and get specific part
```

---

## NULL Handling

```text
IS NULL
→ check missing value

IS NOT NULL
→ check existing value

COALESCE(a,b,c)
→ first non-NULL value

NULLIF(a,b)
→ NULL if a = b

IS DISTINCT FROM
→ NULL-safe difference check

IS NOT DISTINCT FROM
→ NULL-safe equality check
```

---

# MOST IMPORTANT COMPARISONS

## ROUND vs TRUNC

```text
ROUND → rounds
TRUNC → cuts
```

Example:

```text
42.89
ROUND → 43
TRUNC → 42
```

---

## FLOOR vs CEIL

```text
FLOOR → downward
CEIL  → upward
```

Example:

```text
42.8
FLOOR → 42
CEIL  → 43
```

For negative numbers:

```text
-42.8
FLOOR → -43
CEIL  → -42
```

---

## EXTRACT vs DATE_TRUNC

```text
EXTRACT
→ gives a part

DATE_TRUNC
→ gives the beginning of that part
```

Example:

```sql
EXTRACT(MONTH FROM timestamp)
```

→ `9`

while:

```sql
DATE_TRUNC('month', timestamp)
```

→ `2026-09-01 00:00:00`

---

## CONCAT vs `||`

```text
CONCAT('A', NULL)
→ A

'A' || NULL
→ NULL
```

This difference is very useful when nullable columns are involved.

---

## LIKE vs ILIKE

```text
LIKE
→ case-sensitive pattern matching

ILIKE
→ case-insensitive pattern matching
```

PostgreSQL-specific `ILIKE` is particularly useful for user-entered text searches.

---

## COALESCE vs NULLIF

```text
COALESCE
→ NULL ko usable value se replace

NULLIF
→ specific value ko NULL mein convert
```

---

# NULL — The Biggest Exam Concept

Remember this sequence:

```text
NULL
│
├── Check it
│     ├── IS NULL
│     └── IS NOT NULL
│
├── Replace/fallback
│     └── COALESCE()
│
├── Convert specific value to NULL
│     └── NULLIF()
│
└── Compare safely
      ├── IS DISTINCT FROM
      └── IS NOT DISTINCT FROM
```

---

# A Full Real Query Using Multiple Functions

Let's build one query that combines many concepts:

```sql
SELECT
    UPPER(TRIM(name)) AS student_name,

    ROUND(marks, 1) AS rounded_marks,

    COALESCE(
        ROUND(fee, 2),
        0
    ) AS fee,

    EXTRACT(YEAR FROM joined_at) AS join_year,

    TO_CHAR(
        DATE_TRUNC('month', joined_at),
        'Mon YYYY'
    ) AS join_month,

    CONCAT(
        LOWER(SPLIT_PART(email, '@', 1)),
        ' @ ',
        COALESCE(
            SPLIT_PART(email, '@', 2),
            'unknown'
        )
    ) AS email_info

FROM students;
```

Let's understand it.

### Student name

```sql
UPPER(TRIM(name))
```

First:

```text
TRIM
```

Then:

```text
UPPER
```

---

### Marks

```sql
ROUND(marks, 1)
```

Round to one decimal place.

---

### Fee

```sql
COALESCE(ROUND(fee, 2), 0)
```

First:

```text
ROUND fee
```

Then if NULL:

```text
COALESCE → 0
```

---

### Join year

```sql
EXTRACT(YEAR FROM joined_at)
```

Gets the year.

---

### Join month

```sql
DATE_TRUNC('month', joined_at)
```

moves it to month beginning.

Then:

```sql
TO_CHAR(..., 'Mon YYYY')
```

formats it for display.

---

### Email

```sql
SPLIT_PART(email, '@', 1)
```

gets username.

Then:

```sql
LOWER(...)
```

normalizes it.

Then:

```sql
CONCAT(...)
```

builds the final display.

---

# EXAM-ORIENTED QUESTIONS TO PRACTICE

## Numeric Functions

**Q1.** Difference between `ROUND()` and `TRUNC()`?

**Q2.** Find `FLOOR(15.9)`, `CEIL(15.1)`, `ABS(-25)`, `POWER(2,5)`.

**Q3.** Round `1234.5678` to 2 decimal places.

---

## Date/Time

**Q4.** Extract month from a date.

```sql
EXTRACT(MONTH FROM joined_at)
```

**Q5.** Find the date 7 days after a date.

```sql
date_column + INTERVAL '7 days'
```

**Q6.** Get beginning of month.

```sql
DATE_TRUNC('month', date_column)
```

**Q7.** Format a date as `DD-MM-YYYY`.

```sql
TO_CHAR(date_column, 'DD-MM-YYYY')
```

---

## String

**Q8.** Convert name to uppercase.

```sql
UPPER(name)
```

**Q9.** Get first three characters.

```sql
SUBSTRING(name FROM 1 FOR 3)
```

**Q10.** Remove spaces.

```sql
TRIM(name)
```

**Q11.** Extract email domain.

```sql
SPLIT_PART(email, '@', 2)
```

**Q12.** Replace `-` with nothing.

```sql
REPLACE(phone, '-', '')
```

---

## NULL

**Q13.** Find students with missing email.

```sql
WHERE email IS NULL
```

**Q14.** Replace NULL fee with zero.

```sql
COALESCE(fee, 0)
```

**Q15.** Turn zero into NULL.

```sql
NULLIF(fee, 0)
```

**Q16.** Compare values safely when NULL is possible.

```sql
a IS DISTINCT FROM b
```

---

# FINAL MEMORY MAP

```text
NUMERIC
│
├── ROUND   → nearest
├── TRUNC   → cut
├── FLOOR   → down
├── CEIL    → up
├── POWER   → exponent
└── ABS     → positive magnitude


DATE/TIME
│
├── NOW()        → current transaction time
├── EXTRACT()    → take a date/time part
├── INTERVAL     → duration
├── DATE_TRUNC() → beginning of time unit
└── TO_CHAR()    → format as text


STRING
│
├── UPPER()      → uppercase
├── LOWER()      → lowercase
├── LENGTH()     → character count
├── SUBSTRING()  → extract portion
├── CONCAT()     → join values
├── ||           → concatenate
├── REPLACE()    → replace occurrences
├── TRIM()       → clean ends
├── LIKE         → pattern match
├── ILIKE        → case-insensitive pattern match
└── SPLIT_PART() → split + select part


NULL
│
├── IS NULL
├── IS NOT NULL
├── COALESCE()
├── NULLIF()
├── IS DISTINCT FROM
└── IS NOT DISTINCT FROM


NESTING
│
└── Function inside another function
    ↓
    solve inner → then outer
```

---

# The 10 Lines to Memorize Before the Exam

```text
ROUND(x)       → nearest integer
TRUNC(x)       → cut decimals
FLOOR(x)       → integer below/equal
CEIL(x)        → integer above/equal
POWER(x,y)     → x^y
ABS(x)         → absolute value

EXTRACT(...)   → get part of date/time
INTERVAL       → time duration
DATE_TRUNC     → beginning of date/time unit
TO_CHAR        → formatted text

UPPER/LOWER    → change case
LENGTH         → count characters
SUBSTRING      → extract text
CONCAT / ||    → join text
REPLACE        → replace text
TRIM           → remove outer characters/spaces
LIKE/ILIKE     → pattern search
SPLIT_PART     → split string and take part

IS NULL        → check NULL
COALESCE       → first non-NULL
NULLIF         → convert matching value to NULL
IS DISTINCT FROM → NULL-safe difference
```

---

# Core Takeaway

The most important conceptual idea across this entire topic is that **SQL functions can be chained together**.

Once you understand:

```text
TRIM → LOWER → REPLACE → COALESCE → TO_CHAR / ROUND
```

you can handle a large number of practical SQL data-cleaning, reporting, formatting, filtering, and analysis questions.

---

# Quick One-Page Revision

## Numeric

```text
ROUND  → nearest
TRUNC  → cut
FLOOR  → down
CEIL   → up
POWER  → exponent
ABS    → positive magnitude
```

## Date & Time

```text
NOW        → current transaction timestamp
EXTRACT    → get part
INTERVAL   → duration
DATE_TRUNC → beginning of time unit
TO_CHAR    → display formatting as text
```

## String

```text
UPPER      → uppercase
LOWER      → lowercase
LENGTH     → character count
SUBSTRING  → part of string
CONCAT     → combine text
||         → combine text
REPLACE    → substitute text
TRIM       → remove outer spaces/chars
LIKE       → case-sensitive pattern
ILIKE      → case-insensitive pattern
SPLIT_PART → split and select piece
```

## NULL

```text
IS NULL              → check missing
IS NOT NULL          → check present
COALESCE              → first non-NULL
NULLIF                → equal values become NULL
IS DISTINCT FROM      → NULL-safe difference
IS NOT DISTINCT FROM  → NULL-safe equality
```
