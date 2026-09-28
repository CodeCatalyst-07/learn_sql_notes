# PostgreSQL Window Functions
## Deep Exam Notes — English + Hinglish

This set of notes explains why `GROUP BY` falls short, what window functions do, their syntax, partitions, ordering, frames, ranking functions, `LAG()` / `LEAD()`, rolling metrics, duplicate detection, running averages, SQL logical execution order, and the key exam questions.

PostgreSQL defines a window function as a calculation across rows related to the current row while still preserving the individual rows in the result.

---

# Our Example Table

We will use this table throughout.

### `students`

| student_id | name | course | marks |
|---:|---|---|---:|
| 101 | Arnav | DBMS | 95 |
| 102 | Rahul | DBMS | 85 |
| 103 | Priya | DBMS | 85 |
| 104 | Amit | Python | 92 |
| 105 | Ananya | Python | 78 |
| 106 | Riya | Python | 72 |

---

# 1. Why Does `GROUP BY` Fall Short?

Before understanding window functions, understand the problem they solve.

Suppose we want the average marks of each course.

`GROUP BY` is perfect:

```sql
SELECT
    course,
    AVG(marks) AS avg_marks
FROM students
GROUP BY course;
```

Result:

| course | avg_marks |
|---|---:|
| DBMS | 88.33 |
| Python | 80.67 |

We started with:

```text
6 student rows
```

and got:

```text
2 result rows
```

Why?

Because `GROUP BY` **collapses rows into groups**.

## The Real Problem

Suppose we want BOTH:

| name | course | marks | course_average |
|---|---|---:|---:|
| Arnav | DBMS | 95 | 88.33 |
| Rahul | DBMS | 85 | 88.33 |
| Priya | DBMS | 85 | 88.33 |
| Amit | Python | 92 | 80.67 |
| Ananya | Python | 78 | 80.67 |
| Riya | Python | 72 | 80.67 |

We want every student's original row **plus** the average of that student's course.

`GROUP BY` alone isn't designed for this because grouping collapses the rows.

A window function solves exactly this problem:

```sql
SELECT
    name,
    course,
    marks,
    AVG(marks) OVER (
        PARTITION BY course
    ) AS course_average
FROM students;
```

Result:

| name | course | marks | course_average |
|---|---|---:|---:|
| Arnav | DBMS | 95 | 88.33 |
| Rahul | DBMS | 85 | 88.33 |
| Priya | DBMS | 85 | 88.33 |
| Amit | Python | 92 | 80.67 |
| Ananya | Python | 78 | 80.67 |
| Riya | Python | 72 | 80.67 |

### BIG Difference

`GROUP BY`:

```text
Many rows
   ↓
Group
   ↓
One result row per group
```

Window function:

```text
Many rows
   ↓
Calculate across related rows
   ↓
Keep every original row
```

### Hinglish

> `GROUP BY` mein rows merge/collapse ho jaati hain.

> Window function mein rows **remain** karti hain; bas har row ke saath analytical calculation attach hoti hai.

---

# 2. What Does a Window Function Do?

A **window function** performs a calculation across a set of rows that are related to the current row.

The key phrase is:

> **"Related to the current row."**

Think:

```text
Current row
    ↕
Related rows
    ↓
Calculation
    ↓
Result attached to current row
```

### Hinglish

Window function ko aise samjho:

> "Mere current row ko delete/collapse mat karo. Bas uske aas-paas ke related rows ko dekh kar ek calculation batao."

This is why window functions are also commonly called **analytic functions**.

---

# 3. Window Function Syntax

The basic syntax is:

```sql
function_name()
OVER (
    PARTITION BY column
    ORDER BY column
);
```

Example:

```sql
ROW_NUMBER() OVER (
    PARTITION BY course
    ORDER BY marks DESC
)
```

Break it into pieces:

```text
ROW_NUMBER()
     ↓
What calculation?

OVER(...)
     ↓
Which rows are available to the calculation?
```

Inside `OVER`:

```text
PARTITION BY
     ↓
Which rows belong to the same group?

ORDER BY
     ↓
In what order should the window calculation see them?
```

### Memory Trick

```text
function()
   ↓
What?

OVER()
   ↓
Where?

PARTITION BY
   ↓
Which group?

ORDER BY
   ↓
Which order?
```

---

# 4. `OVER()` — Defines the Window

`OVER()` is the most important part.

Example:

```sql
SELECT
    name,
    marks,
    AVG(marks) OVER () AS overall_average
FROM students;
```

Because there is no `PARTITION BY`:

```text
Entire result
     ↓
One window
```

So every student sees the **same overall average**.

Result conceptually:

| name | marks | overall_average |
|---|---:|---:|
| Arnav | 95 | 84.5 |
| Rahul | 85 | 84.5 |
| Priya | 85 | 84.5 |
| Amit | 92 | 84.5 |
| Ananya | 78 | 84.5 |
| Riya | 72 | 84.5 |

### Hinglish

`OVER()` without anything inside means:

> "Saare rows ko ek hi window maan lo."

---

# 5. `PARTITION BY` — Group Without Collapsing

This is probably the most important window concept.

Consider:

```sql
AVG(marks) OVER (
    PARTITION BY course
)
```

This means:

```text
DBMS students → one partition
Python students → another partition
```

But unlike:

```sql
GROUP BY course
```

the rows **don't disappear**.

### GROUP BY

```sql
SELECT course, AVG(marks)
FROM students
GROUP BY course;
```

Result:

```text
DBMS     → 88.33
Python   → 80.67
```

### Window Function

```sql
SELECT
    name,
    course,
    marks,
    AVG(marks) OVER (
        PARTITION BY course
    )
FROM students;
```

Result conceptually:

```text
Arnav   → DBMS → 95 → 88.33
Rahul   → DBMS → 85 → 88.33
Priya   → DBMS → 85 → 88.33

Amit    → Python → 92 → 80.67
Ananya  → Python → 78 → 80.67
Riya    → Python → 72 → 80.67
```

### Memory Trick

> `GROUP BY` = **group and collapse**

> `PARTITION BY` = **group but don't collapse**

### Hinglish

> `PARTITION BY` grouping jaisa hai, but output ki original rows ko remove nahi karta.

---

# 6. `ORDER BY` Inside `OVER()`

This is a very important distinction.

You have seen:

```sql
ORDER BY marks DESC
```

at the end of a query.

But window functions can have their **own** `ORDER BY`:

```sql
ROW_NUMBER() OVER (
    ORDER BY marks DESC
)
```

This controls:

> **The order used by the window function.**

It does **not** automatically mean:

> **The final output is sorted that way.**

## Example

```sql
SELECT
    name,
    marks,
    ROW_NUMBER() OVER (
        ORDER BY marks DESC
    ) AS position
FROM students;
```

You can get:

| name | marks | position |
|---|---:|---:|
| Arnav | 95 | 1 |
| Amit | 92 | 2 |
| Rahul | 85 | 3 |
| Priya | 85 | 4 |
| Ananya | 78 | 5 |
| Riya | 72 | 6 |

The window sees students in marks-descending order.

But if you want the **final output** sorted by name:

```sql
SELECT
    name,
    marks,
    ROW_NUMBER() OVER (
        ORDER BY marks DESC
    ) AS position
FROM students
ORDER BY name;
```

Now:

```text
Final output order → name
Ranking order      → marks
```

### Important

> Window `ORDER BY` controls the calculation.

> Top-level `ORDER BY` controls the displayed result order.

---

# 7. `PARTITION BY` + `ORDER BY`

This is the most common window pattern.

```sql
function()
OVER (
    PARTITION BY course
    ORDER BY marks DESC
)
```

Think:

```text
First:
divide students by course

Then:
sort students inside each course

Then:
perform the window calculation
```

Example:

```sql
SELECT
    name,
    course,
    marks,
    ROW_NUMBER() OVER (
        PARTITION BY course
        ORDER BY marks DESC
    ) AS rank_in_course
FROM students;
```

Result conceptually:

| name | course | marks | row_number |
|---|---|---:|---:|
| Arnav | DBMS | 95 | 1 |
| Rahul | DBMS | 85 | 2 |
| Priya | DBMS | 85 | 3 |
| Amit | Python | 92 | 1 |
| Ananya | Python | 78 | 2 |
| Riya | Python | 72 | 3 |

Notice:

```text
DBMS → numbering starts from 1
Python → numbering starts from 1
```

because each course is a separate partition.

---

# 8. The Frame Clause

A **window frame** tells us:

> For this particular current row, which rows inside the partition should the frame-sensitive calculation actually use?

So there are two levels:

```text
PARTITION
   ↓
Large group of related rows

FRAME
   ↓
Smaller/current subset used for this row's calculation
```

### Simple Hinglish

> `PARTITION BY` batata hai **kaun kaun related rows hain**.

> Frame batata hai **un related rows mein se current row ke calculation mein kitni rows use hongi**.

---

# 9. Why Do We Need a Frame?

Consider:

```sql
SUM(marks) OVER (
    ORDER BY student_id
)
```

With an `ORDER BY`, an aggregate window function uses a default frame that is essentially:

```text
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

with the current row's peers included.

So an ordered aggregate often behaves like a running calculation rather than a full-partition calculation.

---

# 10. Three Frame Ideas You Actually Need

For your exam, these three are the most useful.

## Frame 1 — Current Row Only

```sql
ROWS BETWEEN CURRENT ROW AND CURRENT ROW
```

Meaning:

> Only the current row belongs to the frame.

Example:

```sql
SELECT
    student_id,
    marks,
    SUM(marks) OVER (
        ORDER BY student_id
        ROWS BETWEEN CURRENT ROW AND CURRENT ROW
    ) AS current_marks
FROM students;
```

This basically gives the current row's `marks`.

### Hinglish

> Window mein sirf **"main khud"** hoon.

---

## Frame 2 — Bounded Window

Example:

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

Meaning:

```text
2 previous rows
+
current row
```

For a row in the middle:

```text
[previous previous] [previous] [CURRENT]
```

That's a **3-row sliding window**.

---

## Frame 3 — Unbounded Window

Example:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

Meaning:

```text
First row of partition
        ↓
...
Current row
```

So the window keeps growing.

This is the classic **running total/running average** frame.

### Memory Trick

> `CURRENT ROW` → only me

> `2 PRECEDING` → previous two + me

> `UNBOUNDED PRECEDING` → beginning se me tak everything

---

# 11. `ROWS`, `RANGE`, and `GROUPS`

PostgreSQL supports three frame modes:

```text
ROWS
RANGE
GROUPS
```

For your exam, understand them conceptually.

## `ROWS`

Counts actual physical rows.

Example:

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

means:

> Current row + previous 2 rows.

## `RANGE`

Works based on the value(s) in the window `ORDER BY` and peer groups.

This matters when duplicate ordering values exist.

## `GROUPS`

Works in terms of **peer groups** rather than individual rows.

A peer group means rows considered equivalent by the window's `ORDER BY`.

### Basic Memory

```text
ROWS   → individual rows
RANGE  → ordering values / peers
GROUPS → peer groups
```

---

# 12. Default Frame — Very Important

If you write:

```sql
SUM(marks) OVER (
    ORDER BY marks
)
```

without specifying a frame, PostgreSQL uses the default frame:

```sql
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

More precisely, the endpoint includes the current row's last peer.

This matters when duplicate ordering values exist.

## Example

Suppose salaries:

```text
3500
3900
4200
4800
4800
5000
```

With:

```sql
SUM(salary) OVER (
    ORDER BY salary
)
```

both `4800` rows can get the same running total because they are peers under the default `RANGE` frame.

---

# 13. `ROWS` vs Default `RANGE`

Suppose:

| id | marks |
|---:|---:|
| 1 | 10 |
| 2 | 20 |
| 3 | 20 |
| 4 | 30 |

## Query A

```sql
SELECT
    id,
    marks,
    SUM(marks) OVER (
        ORDER BY marks
    ) AS running_sum
FROM t;
```

Default `RANGE` can treat the two `20` rows as peers.

Conceptually:

| id | marks | sum |
|---:|---:|---:|
| 1 | 10 | 10 |
| 2 | 20 | 50 |
| 3 | 20 | 50 |
| 4 | 30 | 80 |

## Query B

```sql
SELECT
    id,
    marks,
    SUM(marks) OVER (
        ORDER BY marks, id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_sum
FROM t;
```

Now we're explicitly working with rows:

| id | marks | sum |
|---:|---:|---:|
| 1 | 10 | 10 |
| 2 | 20 | 30 |
| 3 | 20 | 50 |
| 4 | 30 | 80 |

### Key Idea

> `RANGE` can treat equal ordering values together.

> `ROWS` counts actual rows.

### Exam Tip

When you specifically want a row-by-row running calculation, an explicit `ROWS` frame is often the clearest choice.

---

# PART 2 — RANKING FUNCTIONS

The three ranking functions you need are:

```text
ROW_NUMBER()
RANK()
DENSE_RANK()
```

Let's use:

| name | marks |
|---|---:|
| Arnav | 95 |
| Amit | 92 |
| Rahul | 85 |
| Priya | 85 |
| Ananya | 78 |
| Riya | 72 |

---

# 14. `ROW_NUMBER()`

## What does it do?

`ROW_NUMBER()` gives a **unique sequential number** to each row inside its partition.

Example:

```sql
SELECT
    name,
    marks,
    ROW_NUMBER() OVER (
        ORDER BY marks DESC
    ) AS row_num
FROM students;
```

Result:

| name | marks | row_num |
|---|---:|---:|
| Arnav | 95 | 1 |
| Amit | 92 | 2 |
| Rahul | 85 | 3 |
| Priya | 85 | 4 |
| Ananya | 78 | 5 |
| Riya | 72 | 6 |

Even though Rahul and Priya have the same marks, they get different row numbers.

## Important Tie Point

If the window `ORDER BY` doesn't uniquely identify rows, the relative numbering of tied rows is unspecified.

So:

```sql
ROW_NUMBER() OVER (
    ORDER BY marks DESC
)
```

doesn't guarantee whether Rahul gets 3 and Priya gets 4, or vice versa.

For deterministic row numbering:

```sql
ROW_NUMBER() OVER (
    ORDER BY marks DESC, student_id
)
```

Now `student_id` breaks the tie.

### When is ROW_NUMBER useful?

Very useful for:

- removing duplicates
- picking first row
- pagination
- top-N per group
- assigning unique sequence numbers
- selecting latest record per customer

---

# 15. `RANK()`

`RANK()` gives rankings with ties receiving the same rank.

Example:

```sql
SELECT
    name,
    marks,
    RANK() OVER (
        ORDER BY marks DESC
    ) AS rank
FROM students;
```

Result:

| name | marks | rank |
|---|---:|---:|
| Arnav | 95 | 1 |
| Amit | 92 | 2 |
| Rahul | 85 | 3 |
| Priya | 85 | 3 |
| Ananya | 78 | 5 |
| Riya | 72 | 6 |

Notice:

```text
95 → rank 1
92 → rank 2
85 → rank 3
85 → rank 3
78 → rank 5
```

Why did we jump from 3 to 5?

Because two people occupied rank 3.

That's the **gap**.

### Hinglish

> Same marks → same rank.

> Tie ke baad next rank mein gap aata hai.

---

# 16. `DENSE_RANK()`

`DENSE_RANK()` also gives equal rank to ties, but it **does not create gaps**.

```sql
SELECT
    name,
    marks,
    DENSE_RANK() OVER (
        ORDER BY marks DESC
    ) AS dense_rank
FROM students;
```

Result:

| name | marks | dense_rank |
|---|---:|---:|
| Arnav | 95 | 1 |
| Amit | 92 | 2 |
| Rahul | 85 | 3 |
| Priya | 85 | 3 |
| Ananya | 78 | 4 |
| Riya | 72 | 5 |

No gap.

### Hinglish

> Same rank for ties, but next rank continuous rehta hai.

---

# 17. ROW_NUMBER vs RANK vs DENSE_RANK

This is a **must-memorize table**.

| Function | Ties | Gaps? |
|---|---|---|
| `ROW_NUMBER()` | Different numbers | No |
| `RANK()` | Same rank | Yes |
| `DENSE_RANK()` | Same rank | No |

With:

```text
95
92
85
85
78
```

we get:

```text
ROW_NUMBER
1
2
3
4
5

RANK
1
2
3
3
5

DENSE_RANK
1
2
3
3
4
```

---

# 18. RANK vs DENSE_RANK — Direct Answer

### `RANK()`

Ties share rank, and the next rank **skips numbers**.

```text
1
2
2
4
```

### `DENSE_RANK()`

Ties share rank, but the next rank **doesn't skip numbers**.

```text
1
2
2
3
```

### Memory

> `RANK` → **gap**

> `DENSE_RANK` → **dense/no gap**

---

# 19. Ranking Within Each Course

Use `PARTITION BY`:

```sql
SELECT
    name,
    course,
    marks,
    RANK() OVER (
        PARTITION BY course
        ORDER BY marks DESC
    ) AS course_rank
FROM students;
```

Now rankings restart for every course.

Example conceptually:

```text
DBMS:
Arnav  → 1
Rahul  → 2
Priya  → 2

Python:
Amit   → 1
Ananya → 2
Riya   → 3
```

---

# PART 3 — `LAG()`

Now we move from ranking to **looking backward**.

# 20. What is `LAG()`?

`LAG()` allows the current row to access a value from an **earlier row** in the window ordering.

Basic syntax:

```sql
LAG(value) OVER (
    ORDER BY something
)
```

Default offset is 1.

---

# Example

Suppose:

| month | sales |
|---|---:|
| Jan | 100 |
| Feb | 120 |
| Mar | 110 |
| Apr | 150 |

Query:

```sql
SELECT
    month,
    sales,
    LAG(sales) OVER (
        ORDER BY month
    ) AS previous_sales
FROM sales;
```

Result:

| month | sales | previous_sales |
|---|---:|---:|
| Jan | 100 | NULL |
| Feb | 120 | 100 |
| Mar | 110 | 120 |
| Apr | 150 | 110 |

### Hinglish

For February:

```text
Current = February = 120
Previous = January = 100
```

For March:

```text
Current = March = 110
Previous = February = 120
```

First row:

```text
No previous row
    ↓
LAG → NULL
```

---

# 21. LAG with Offset

Default:

```sql
LAG(sales)
```

means:

```text
previous 1 row
```

But:

```sql
LAG(sales, 2)
```

means:

> Two rows before.

Example:

```sql
SELECT
    month,
    sales,
    LAG(sales, 2) OVER (
        ORDER BY month
    ) AS sales_two_periods_ago
FROM sales;
```

---

# 22. LAG with Default Value

Syntax:

```sql
LAG(value, offset, default)
```

Example:

```sql
LAG(sales, 1, 0)
OVER (
    ORDER BY month
)
```

For the first row, instead of NULL:

```text
0
```

can be returned.

---

# 23. Business Uses of LAG

The most important use is:

```text
Current value
     -
Previous value
     =
Change
```

Example:

```sql
SELECT
    month,
    sales,
    sales - LAG(sales) OVER (
        ORDER BY month
    ) AS change
FROM sales;
```

Result:

| month | sales | change |
|---|---:|---:|
| Jan | 100 | NULL |
| Feb | 120 | 20 |
| Mar | 110 | -10 |
| Apr | 150 | 40 |

---

# 24. Percentage Change with LAG

A useful pattern is:

```sql
SELECT
    month,
    sales,
    (
        sales - LAG(sales) OVER (ORDER BY month)
    )
    /
    NULLIF(
        LAG(sales) OVER (ORDER BY month),
        0
    ) * 100 AS pct_change
FROM sales;
```

Conceptually:

```text
(Current - Previous)
-------------------- × 100
Previous
```

`NULLIF()` helps protect against division by zero.

### Business Uses

`LAG()` is useful for:

- sales growth
- month-over-month growth
- revenue changes
- user growth
- traffic changes
- score comparison
- salary comparison
- previous status
- previous transaction

---

# 25. LAG + PARTITION BY

Suppose you have multiple products:

| month | product | sales |
|---|---|---:|
| Jan | A | 100 |
| Feb | A | 120 |
| Jan | B | 80 |
| Feb | B | 90 |

You don't want Product A's previous value to mix with Product B.

Use:

```sql
LAG(sales) OVER (
    PARTITION BY product
    ORDER BY month
)
```

Now:

```text
Product A → its own history
Product B → its own history
```

---

# 26. When is LAG genuinely useful?

Use `LAG()` whenever the question sounds like:

> **"Compared with the previous..."**

Examples:

```text
Previous month sales?
Previous day's revenue?
Previous transaction?
Previous score?
Previous salary?
Previous status?
```

### Memory Trick

> Question mein **previous / last period / pichla** aaye → `LAG()` ke baare mein socho.

---

# PART 4 — `LEAD()`

# 27. What is `LEAD()`?

`LEAD()` is basically the opposite of `LAG()`.

```text
LAG:
Current ← Previous

LEAD:
Current → Next
```

Example:

```sql
SELECT
    month,
    sales,
    LEAD(sales) OVER (
        ORDER BY month
    ) AS next_sales
FROM sales;
```

Result:

| month | sales | next_sales |
|---|---:|---:|
| Jan | 100 | 120 |
| Feb | 120 | 110 |
| Mar | 110 | 150 |
| Apr | 150 | NULL |

---

# 28. LAG vs LEAD

| Function | Looks |
|---|---|
| `LAG()` | Backward |
| `LEAD()` | Forward |

### Memory

> **LAG = previous/back**

> **LEAD = next/forward**

---

# 29. Business Uses of LEAD

`LEAD()` is useful when you want:

- Next month's sales
- Next event
- Next transaction
- Next login
- Next status change
- Time until next event
- Difference between current and next record

## Example — Next Sales Difference

```sql
SELECT
    month,
    sales,
    LEAD(sales) OVER (
        ORDER BY month
    ) - sales AS change_to_next_month
FROM sales;
```

Meaning:

> "Next month ke sales se current month kitna different hai?"

## Example — Time Until Next Event

Suppose:

| customer | event_time |
|---|---|
| A | 10:00 |
| A | 10:30 |
| A | 11:45 |

Query:

```sql
SELECT
    customer,
    event_time,

    LEAD(event_time) OVER (
        PARTITION BY customer
        ORDER BY event_time
    ) - event_time AS time_to_next_event

FROM events;
```

Now you can find the time until each customer's next event.

---

# PART 5 — DETECTING DUPLICATES

One of the most useful applications of `ROW_NUMBER()` is duplicate detection.

Suppose:

### `users`

| id | email |
|---:|---|
| 1 | a@gmail.com |
| 2 | b@gmail.com |
| 3 | a@gmail.com |
| 4 | c@gmail.com |
| 5 | a@gmail.com |

`a@gmail.com` appears three times.

---

# 30. ROW_NUMBER for Duplicate Detection

```sql
SELECT
    id,
    email,

    ROW_NUMBER() OVER (
        PARTITION BY email
        ORDER BY id
    ) AS occurrence

FROM users;
```

Result:

| id | email | occurrence |
|---:|---|---:|
| 1 | a@gmail.com | 1 |
| 3 | a@gmail.com | 2 |
| 5 | a@gmail.com | 3 |
| 2 | b@gmail.com | 1 |
| 4 | c@gmail.com | 1 |

Think:

```text
a@gmail.com
    ↓
id 1 → occurrence 1
id 3 → occurrence 2
id 5 → occurrence 3

b@gmail.com
    ↓
id 2 → occurrence 1

c@gmail.com
    ↓
id 4 → occurrence 1
```

Therefore:

```text
occurrence = 1
→ first occurrence

occurrence > 1
→ duplicate occurrence
```

---

# 31. Find Only Duplicates

Use a subquery:

```sql
SELECT *
FROM (
    SELECT
        id,
        email,
        ROW_NUMBER() OVER (
            PARTITION BY email
            ORDER BY id
        ) AS occurrence
    FROM users
) t
WHERE occurrence > 1;
```

### Important Concept

Window functions are calculated before the outer query filters them, so a subquery/CTE is commonly used to filter a window result.

---

# PART 6 — MOVING AVERAGE

Now we reach one of the most important applications of window frames.

Suppose:

| day | sales |
|---|---:|
| Mon | 100 |
| Tue | 120 |
| Wed | 140 |
| Thu | 160 |
| Fri | 180 |

We want a:

> **3-day moving average**

For Wednesday:

```text
Mon + Tue + Wed
```

For Thursday:

```text
Tue + Wed + Thu
```

For Friday:

```text
Wed + Thu + Fri
```

This is a **sliding window**.

---

# 32. Moving Average Query

```sql
SELECT
    day,
    sales,

    AVG(sales) OVER (
        ORDER BY day
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg

FROM daily_sales;
```

---

# How the Frame Moves

For Monday:

```text
[Mon]
```

For Tuesday:

```text
[Mon Tue]
```

For Wednesday:

```text
[Mon Tue Wed]
```

For Thursday:

```text
[Tue Wed Thu]
```

For Friday:

```text
[Wed Thu Fri]
```

The frame has a fixed maximum size and **slides forward**.

---

# 33. Business Uses of Rolling Metrics

Moving/rolling calculations are useful for:

- 7-day average sales
- 30-day average revenue
- Rolling website traffic
- Rolling active users
- Rolling defect rate
- Rolling temperature
- Rolling expenses
- Smoothing noisy data

Example:

```sql
AVG(revenue) OVER (
    ORDER BY date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
)
```

This means:

> Current row + previous 6 rows = up to 7 rows.

### Important

That's **7 rows**, not necessarily seven calendar days if your dataset has missing dates or multiple records per day.

---

# PART 7 — RUNNING AVERAGE

Now compare that with a **running average**.

A running average keeps including everything from the beginning.

Frame:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

---

# 34. Running Average

```sql
SELECT
    day,
    sales,

    AVG(sales) OVER (
        ORDER BY day
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_avg

FROM daily_sales;
```

For:

```text
100
120
140
160
180
```

we get:

### Day 1

```text
100
```

### Day 2

```text
(100 + 120) / 2 = 110
```

### Day 3

```text
(100 + 120 + 140) / 3 = 120
```

### Day 4

```text
(100 + 120 + 140 + 160) / 4 = 130
```

### Day 5

```text
(100 + 120 + 140 + 160 + 180) / 5 = 140
```

Result:

| day | sales | running_avg |
|---|---:|---:|
| Mon | 100 | 100 |
| Tue | 120 | 110 |
| Wed | 140 | 120 |
| Thu | 160 | 130 |
| Fri | 180 | 140 |

---

# 35. Running Average vs Sliding Window

## Running Average

Frame:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

Window:

```text
[1]
[1,2]
[1,2,3]
[1,2,3,4]
[1,2,3,4,5]
```

It keeps getting larger.

## 3-Row Sliding Average

Frame:

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

Window:

```text
[1]
[1,2]
[1,2,3]
[2,3,4]
[3,4,5]
```

It stays bounded.

### Memory Trick

> **Running = grows**

> **Sliding = moves**

---

# 36. Running SUM

The same idea isn't limited to `AVG`.

```sql
SELECT
    day,
    sales,

    SUM(sales) OVER (
        ORDER BY day
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total

FROM daily_sales;
```

This gives cumulative sales.

For:

```text
100
120
140
160
180
```

the running total is:

```text
100
220
360
520
700
```

---

# 37. Running MIN / MAX

Window functions can also use aggregates such as:

```text
MIN(...)
MAX(...)
SUM(...)
AVG(...)
COUNT(...)
```

Example:

```sql
SELECT
    day,
    sales,

    MAX(sales) OVER (
        ORDER BY day
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS highest_so_far

FROM daily_sales;
```

This gives:

> Highest sales seen so far.

---

# PART 8 — THE THREE FRAMES YOU SHOULD KNOW

## Frame A — Current Row Only

```sql
ROWS BETWEEN CURRENT ROW AND CURRENT ROW
```

Meaning:

```text
[current]
```

Useful when you need a calculation over only the current row.

---

## Frame B — Bounded Window

Example:

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

Meaning:

```text
previous 2
+
current
```

Useful for:

- moving average
- rolling sum
- recent-N-row metrics

---

## Frame C — Unbounded Window

Example:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

Meaning:

```text
first row
   ↓
all previous
   ↓
current row
```

Useful for:

- running total
- running average
- cumulative count
- cumulative maximum

---

# 38. Other Frame Boundaries You Should Recognize

PostgreSQL allows frame boundaries such as:

```text
UNBOUNDED PRECEDING
n PRECEDING
CURRENT ROW
n FOLLOWING
UNBOUNDED FOLLOWING
```

Examples:

```sql
ROWS BETWEEN 3 PRECEDING AND CURRENT ROW
```

```sql
ROWS BETWEEN CURRENT ROW AND 3 FOLLOWING
```

```sql
ROWS BETWEEN UNBOUNDED PRECEDING
     AND UNBOUNDED FOLLOWING
```

The last one means:

> Entire partition.

---

# 39. Entire Partition vs Running Window

Compare:

### Entire partition

```sql
SUM(marks) OVER (
    PARTITION BY course
)
```

Every student gets the same course total.

### Running total inside course

```sql
SUM(marks) OVER (
    PARTITION BY course
    ORDER BY marks DESC
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

Now the value changes as you move through the course's ordered rows.

### Important

> `PARTITION BY` tells you the full group.

> The frame tells you how much of that group is used for the current calculation.

---

# PART 9 — GROUP BY VS WINDOW FUNCTIONS

# 40. GROUP BY

```sql
SELECT
    course,
    AVG(marks)
FROM students
GROUP BY course;
```

Output:

```text
DBMS     → 88.33
Python   → 80.67
```

Rows collapse.

---

# 41. Window Function

```sql
SELECT
    name,
    course,
    marks,
    AVG(marks) OVER (
        PARTITION BY course
    ) AS course_avg
FROM students;
```

Output:

```text
Arnav  → DBMS → 95 → 88.33
Rahul  → DBMS → 85 → 88.33
Priya  → DBMS → 85 → 88.33
...
```

Rows remain.

---

# 42. When Should You Use GROUP BY vs a Window Function?

### Use `GROUP BY` when:

You want:

> **One result per group.**

Examples:

```text
Average marks per course
Total sales per city
Number of students per course
Maximum salary per department
```

Pattern:

```sql
SELECT group_column, AGGREGATE(...)
FROM table
GROUP BY group_column;
```

### Use a Window Function when:

You want:

> **Every original row plus a calculation involving related rows.**

Examples:

```text
Each student's course average
Rank each student
Previous month's sales
Running total
Moving average
Percentage compared with previous period
Top N within each category
```

Pattern:

```sql
SELECT
    columns,
    function(...) OVER (...)
FROM table;
```

### Exam Line

> **`GROUP BY` collapses rows into groups; window functions calculate across related rows while preserving row-level results.**

---

# PART 10 — SQL LOGICAL EXECUTION ORDER

You previously learned:

```text
FROM
→ WHERE
→ GROUP BY
→ HAVING
→ SELECT
→ ORDER BY
```

Now window functions add another important stage.

A useful mental model is:

```text
FROM
  ↓
WHERE
  ↓
GROUP BY
  ↓
HAVING
  ↓
Window Functions
  ↓
SELECT / output expressions
  ↓
DISTINCT
  ↓
ORDER BY
  ↓
LIMIT / OFFSET
```

The key fact for the exam is:

> **Window functions are evaluated after grouping, aggregation, and HAVING filtering.**

Window functions are allowed in the `SELECT` list and `ORDER BY`, not directly in `WHERE`, `GROUP BY`, or `HAVING`.

---

# 43. Grouped Result + Window Function

Consider:

```sql
SELECT
    course,
    COUNT(*) AS student_count,
    RANK() OVER (
        ORDER BY COUNT(*) DESC
    ) AS course_rank
FROM students
GROUP BY course;
```

Why is this possible?

Conceptually:

```text
FROM
 ↓
GROUP BY
 ↓
COUNT per course
 ↓
Window ranking of those grouped results
```

So you are effectively ranking the aggregated course rows.

---

# 44. Window Functions Cannot Normally Go in WHERE

This is invalid:

```sql
SELECT
    name,
    marks,
    ROW_NUMBER() OVER (
        ORDER BY marks DESC
    ) AS rn
FROM students
WHERE rn <= 3;
```

Why?

Because:

```text
WHERE
 ↓
comes before the window calculation
```

## Correct Method

Use a subquery:

```sql
SELECT *
FROM (
    SELECT
        name,
        marks,
        ROW_NUMBER() OVER (
            ORDER BY marks DESC
        ) AS rn
    FROM students
) t
WHERE rn <= 3;
```

Now:

```text
Inner query
→ calculate row number

Outer query
→ filter row number
```

---

# PART 11 — TOP N PER GROUP

This is one of the biggest real-world uses of window functions.

Suppose:

> Give me the top 2 students from every course.

Use:

```sql
SELECT *
FROM (
    SELECT
        name,
        course,
        marks,

        ROW_NUMBER() OVER (
            PARTITION BY course
            ORDER BY marks DESC, student_id
        ) AS rn

    FROM students
) t

WHERE rn <= 2;
```

### Flow

```text
Partition by course
       ↓
Rank students within each course
       ↓
Keep rn <= 2
```

---

# What if Ties Should Be Included?

If you want:

> Top 2 ranks, including ties

you might use:

```sql
RANK() OVER (
    PARTITION BY course
    ORDER BY marks DESC
)
```

instead of `ROW_NUMBER()`.

Because `ROW_NUMBER()` forces unique positions, while `RANK()` preserves ties.

---

# PART 12 — DUPLICATE DETECTION RECAP

Suppose:

| id | email |
|---:|---|
| 1 | a@gmail.com |
| 2 | b@gmail.com |
| 3 | a@gmail.com |
| 4 | c@gmail.com |
| 5 | a@gmail.com |

Query:

```sql
SELECT
    id,
    email,
    ROW_NUMBER() OVER (
        PARTITION BY email
        ORDER BY id
    ) AS occurrence
FROM users;
```

Think:

```text
a@gmail.com
    ↓
id 1 → occurrence 1
id 3 → occurrence 2
id 5 → occurrence 3

b@gmail.com
    ↓
id 2 → occurrence 1

c@gmail.com
    ↓
id 4 → occurrence 1
```

Therefore:

```text
occurrence > 1
→ duplicate occurrence
```

---

# PART 13 — LAG / LEAD Business Examples

## Example 1 — Month-over-Month Sales

```sql
SELECT
    month,
    sales,
    sales - LAG(sales) OVER (
        ORDER BY month
    ) AS change
FROM monthly_sales;
```

## Example 2 — Previous Login

```sql
SELECT
    user_id,
    login_time,

    LAG(login_time) OVER (
        PARTITION BY user_id
        ORDER BY login_time
    ) AS previous_login

FROM logins;
```

## Example 3 — Time Between Logins

```sql
SELECT
    user_id,
    login_time,

    login_time
    - LAG(login_time) OVER (
        PARTITION BY user_id
        ORDER BY login_time
    ) AS time_since_previous_login

FROM logins;
```

## Example 4 — Next Appointment

```sql
SELECT
    patient_id,
    appointment_time,

    LEAD(appointment_time) OVER (
        PARTITION BY patient_id
        ORDER BY appointment_time
    ) AS next_appointment

FROM appointments;
```

## Example 5 — Next Event Gap

```sql
SELECT
    user_id,
    event_time,

    LEAD(event_time) OVER (
        PARTITION BY user_id
        ORDER BY event_time
    ) - event_time AS time_to_next_event

FROM events;
```

---

# PART 14 — Window Function vs GROUP BY — Side-by-Side

Suppose we want total marks per course.

### GROUP BY

```sql
SELECT
    course,
    SUM(marks)
FROM students
GROUP BY course;
```

Result:

```text
DBMS → total
Python → total
```

One row per course.

### Window Function

```sql
SELECT
    name,
    course,
    marks,
    SUM(marks) OVER (
        PARTITION BY course
    ) AS course_total
FROM students;
```

Result:

```text
Arnav  → DBMS → 95 → DBMS total
Rahul  → DBMS → 85 → DBMS total
Priya  → DBMS → 85 → DBMS total
...
```

### Memory

```text
GROUP BY:
group → calculate → collapse

WINDOW:
partition → calculate → preserve
```

---

# PART 15 — Why `ORDER BY` Inside `OVER()` Matters So Much

Compare:

## No window ORDER BY

```sql
SUM(marks) OVER (
    PARTITION BY course
)
```

Meaning:

> Entire course partition.

Every DBMS student gets the same DBMS total.

## With window ORDER BY

```sql
SUM(marks) OVER (
    PARTITION BY course
    ORDER BY marks DESC
)
```

Now you're asking for an ordered window calculation.

With the default frame, this behaves like a running-style calculation through the current row and its peers.

---

# PART 16 — A Very Important Frame Example

Let's use:

| id | marks |
|---:|---:|
| 1 | 10 |
| 2 | 20 |
| 3 | 30 |
| 4 | 40 |
| 5 | 50 |

## Full partition

```sql
SUM(marks) OVER ()
```

Every row:

```text
150
```

## Running total

```sql
SUM(marks) OVER (
    ORDER BY id
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

Results:

```text
10
30
60
100
150
```

## 3-row sliding total

```sql
SUM(marks) OVER (
    ORDER BY id
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)
```

Results:

```text
10
30
60
90
120
```

Breakdown:

```text
Row 1 → 10

Row 2 → 10+20 = 30

Row 3 → 10+20+30 = 60

Row 4 → 20+30+40 = 90

Row 5 → 30+40+50 = 120
```

---

# 45. How Does the Frame Clause Change the Result?

The frame determines **which rows are actually included in the calculation for each current row**.

Same data:

```text
10
20
30
40
50
```

## Frame 1

```sql
ROWS BETWEEN CURRENT ROW AND CURRENT ROW
```

Frame for row 4:

```text
40
```

Result with SUM:

```text
40
```

## Frame 2

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

Frame for row 4:

```text
20
30
40
```

Result:

```text
90
```

## Frame 3

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

Frame for row 4:

```text
10
20
30
40
```

Result:

```text
100
```

### Core Idea

```text
Current row only
→ very small frame

Bounded frame
→ sliding/rolling metric

Unbounded preceding
→ running/cumulative metric
```

---

# PART 17 — `LAG` / `LEAD` and Frames

One subtle but important point:

`LAG()` and `LEAD()` look at rows relative to the current row in the partition's ordering; they are not ordinary frame-based aggregates.

So don't assume:

```sql
LAG(...)
```

is controlled by the same frame logic as:

```sql
SUM(...) OVER (...)
```

`LAG` / `LEAD` use a row offset before/after the current row, while frame boundaries are especially important for aggregate window functions and functions such as `first_value`, `last_value`, and `nth_value`.

---

# PART 18 — Common Exam Traps

## Trap 1: `GROUP BY` and `PARTITION BY` are not the same

### GROUP BY

```text
Groups + collapses rows
```

### PARTITION BY

```text
Groups logically + keeps rows
```

---

## Trap 2: `ORDER BY` inside OVER is not final sorting

This:

```sql
ROW_NUMBER() OVER (
    ORDER BY marks DESC
)
```

controls ranking.

It doesn't guarantee final result ordering.

For final ordering:

```sql
ORDER BY marks DESC
```

at query level.

---

## Trap 3: RANK ≠ ROW_NUMBER

```text
ROW_NUMBER
1
2
3
4

RANK
1
2
2
4
```

---

## Trap 4: RANK ≠ DENSE_RANK

```text
RANK
1
2
2
4
```

```text
DENSE_RANK
1
2
2
3
```

---

## Trap 5: Window functions cannot normally be used in WHERE

Wrong:

```sql
WHERE ROW_NUMBER() OVER (...) <= 3
```

Use a subquery/CTE and filter outside.

---

## Trap 6: Default frame has peer behavior

If you write:

```sql
SUM(x) OVER (
    ORDER BY x
)
```

the default `RANGE` frame includes the current row's peers, so duplicate ordering values can produce the same running result.

When you specifically want row-by-row behavior, consider:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

---

## Trap 7: `ROW_NUMBER()` with ties

This:

```sql
ROW_NUMBER() OVER (
    ORDER BY marks DESC
)
```

doesn't give equal row numbers to equal marks.

For equal marks:

```text
85 → 3
85 → 4
```

If you want ties to share rank:

```text
RANK()
```

or:

```text
DENSE_RANK()
```

---

## Trap 8: Missing `PARTITION BY`

Suppose:

```sql
ROW_NUMBER() OVER (
    ORDER BY marks DESC
)
```

All students compete together.

But:

```sql
ROW_NUMBER() OVER (
    PARTITION BY course
    ORDER BY marks DESC
)
```

ranking restarts for every course.

---

# 46. A Complete Advanced Example

Suppose we have:

### `sales`

| sale_date | region | sales |
|---|---|---:|
| Jan 1 | North | 100 |
| Jan 2 | North | 120 |
| Jan 3 | North | 90 |
| Jan 1 | South | 80 |
| Jan 2 | South | 100 |
| Jan 3 | South | 110 |

We want:

- Previous day's sales
- Change from previous day
- Running total
- 3-row moving average
- Rank within region

Query:

```sql
SELECT
    sale_date,
    region,
    sales,

    LAG(sales) OVER (
        PARTITION BY region
        ORDER BY sale_date
    ) AS previous_sales,

    sales
    - LAG(sales) OVER (
        PARTITION BY region
        ORDER BY sale_date
    ) AS change_from_previous,

    SUM(sales) OVER (
        PARTITION BY region
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total,

    AVG(sales) OVER (
        PARTITION BY region
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_average,

    RANK() OVER (
        PARTITION BY region
        ORDER BY sales DESC
    ) AS sales_rank

FROM sales;
```

---

# 47. How to Read That Query

### `PARTITION BY region`

```text
North → its own window
South → its own window
```

### `LAG`

```text
Look backward inside that region.
```

### `SUM`

```text
Start from the first date
and keep accumulating.
```

### `AVG`

```text
Look at current + previous 2 rows.
```

### `RANK`

```text
Rank sales values within that region.
```

This is the power of window functions:

> Multiple analytical calculations can be performed without collapsing the rows.

---

# PART 19 — Direct Exam Questions

## Q1. When should you use GROUP BY vs a window function?

### Use `GROUP BY` when:

You want:

> **One row per group.**

Example:

```sql
SELECT course, AVG(marks)
FROM students
GROUP BY course;
```

---

### Use a window function when:

You want:

> **Every original row plus a calculation over related rows.**

Example:

```sql
SELECT
    name,
    course,
    marks,
    AVG(marks) OVER (
        PARTITION BY course
    )
FROM students;
```

### Exam Line

> **`GROUP BY` collapses rows into groups; window functions calculate across related rows while preserving row-level results.**

---

# Q2. RANK vs DENSE_RANK — what is the difference?

Suppose:

```text
100
90
90
80
```

### `RANK()`

```text
1
2
2
4
```

There is a gap after the tie.

### `DENSE_RANK()`

```text
1
2
2
3
```

No gap.

### Memory

> `RANK` → **gap**

> `DENSE_RANK` → **no gap**

---

# Q3. When is LAG genuinely useful?

When you need:

> **Previous row's value.**

Examples:

```text
Current sales vs previous sales
Current month vs previous month
Current login vs previous login
Current score vs previous score
Current transaction vs previous transaction
```

Most common pattern:

```sql
current_value - LAG(current_value)
```

For percentage growth:

```sql
(
    current_value - LAG(current_value)
)
/
LAG(current_value)
* 100
```

Use `NULLIF()` where appropriate to protect a zero denominator.

---

# Q4. How does the frame clause change the result?

The frame controls:

> **Which rows are actually included in a frame-sensitive window calculation for the current row.**

Example:

```sql
ROWS BETWEEN CURRENT ROW AND CURRENT ROW
```

→ only current row

```sql
ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
```

→ current + previous 2 rows

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

→ everything from partition beginning to current row

Therefore:

```text
Current only
→ current value

Bounded
→ sliding/rolling metric

Unbounded preceding
→ running/cumulative metric
```

---

# PART 20 — MASTER MEMORY MAP

Keep this whole topic in your head like this:

```text
WINDOW FUNCTIONS
│
├── OVER()
│   └── Defines the window
│
├── PARTITION BY
│   └── Group without collapsing
│
├── ORDER BY inside OVER()
│   └── Defines window ordering
│
├── FRAME
│   ├── CURRENT ROW
│   ├── n PRECEDING / FOLLOWING
│   └── UNBOUNDED PRECEDING / FOLLOWING
│
├── RANKING
│   ├── ROW_NUMBER()
│   ├── RANK()
│   └── DENSE_RANK()
│
├── LOOKING AROUND
│   ├── LAG()  → previous
│   └── LEAD() → next
│
└── ANALYTICS
    ├── Running total
    ├── Running average
    ├── Moving average
    ├── Duplicate detection
    └── Top-N per group
```

---

# 🧠 THE 15 THINGS TO MEMORIZE

```text
1. GROUP BY collapses rows.

2. Window functions preserve rows.

3. OVER() defines the window.

4. PARTITION BY groups rows without collapsing them.

5. ORDER BY inside OVER controls window calculation order.

6. Top-level ORDER BY controls final output order.

7. ROW_NUMBER gives unique sequential numbers.

8. RANK gives tied ranks with gaps.

9. DENSE_RANK gives tied ranks without gaps.

10. LAG looks backward.

11. LEAD looks forward.

12. Running average:
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

13. Sliding 3-row average:
    ROWS BETWEEN 2 PRECEDING AND CURRENT ROW

14. Window functions happen after WHERE/GROUP BY/HAVING.

15. To filter a window result, use a subquery or CTE.
```

---

# 🚀 Final Mental Model

When you see a SQL question, ask these questions:

### Question 1

> **Do I want to collapse rows?**

Yes:

```text
GROUP BY
```

No:

```text
Window Function
```

---

### Question 2

> **Do I want to divide rows into independent groups?**

Use:

```text
PARTITION BY
```

---

### Question 3

> **Do I care about previous/next rows?**

Use:

```text
LAG / LEAD
```

---

### Question 4

> **Do I need ranking?**

Choose:

```text
ROW_NUMBER → unique position
RANK       → ties + gaps
DENSE_RANK → ties + no gaps
```

---

### Question 5

> **Do I need a rolling/running calculation?**

Think:

```sql
OVER (
    ORDER BY ...
    ROWS BETWEEN ...
)
```

---

### Question 6

> **Do I need current row + previous N rows?**

Think:

```sql
ROWS BETWEEN N PRECEDING AND CURRENT ROW
```

---

### Question 7

> **Do I need everything from the beginning until now?**

Think:

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

---

# ⭐ One Final Comparison

```text
GROUP BY
    ↓
"Same course ke students ko combine karo."
    ↓
1 row per course


WINDOW FUNCTION
    ↓
"Same course ke students ko dekh kar
har student ki row par result likho."
    ↓
1 row per student
```

That single distinction is the foundation of the entire topic.

And the easiest way to remember window functions is:

> **`PARTITION BY` tells you WHO is related, `ORDER BY` tells you IN WHAT ORDER, and the FRAME tells you HOW MUCH of that ordered partition the current row can see.**
