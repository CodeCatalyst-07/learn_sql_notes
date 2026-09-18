# DBMS, Data Engineering & SQL — Complete Exam Recap Notes

## Lecture 1 + Introduction to SQL + PostgreSQL

> **Purpose:** Detailed exam-recap notes in English + Hinglish, with definitions, explanations, examples, SQL syntax, comparisons, memory tricks, and common exam traps.

---

# Table of Contents

1. [Data Fundamentals](#1-data-fundamentals)
   - [Introduction to Data](#11-introduction-to-data)
   - [Data → Information → Insight](#12-data--information--insight)
   - [Structured, Semi-Structured & Unstructured Data](#13-structured-semi-structured--unstructured-data)
   - [Database & DBMS](#14-database--dbms)
   - [DBMS vs File-Based Systems](#15-dbms-vs-file-based-systems)
   - [Advantages of DBMS](#16-advantages-of-dbms)
2. [Relational Database Concepts](#2-relational-database-concepts)
   - [Relational Database](#21-relational-database)
   - [Tables, Rows, Columns & Attributes](#22-tables-rows-columns--attributes)
   - [Entities & Attributes](#23-entities--attributes)
   - [Database Schema](#24-database-schema)
   - [Super Key](#25-super-key)
   - [Candidate Key](#26-candidate-key)
   - [Primary Key](#27-primary-key)
   - [Composite Key](#28-composite-key)
   - [Constraints](#29-constraints)
   - [Foreign Key](#210-foreign-key)
   - [Referential Integrity](#211-referential-integrity)
   - [Relationships Between Tables](#212-relationships-between-tables)
3. [Introduction to SQL](#3-introduction-to-sql)
   - [SQL and Relational Databases](#31-sql-and-relational-databases)
   - [Declarative Nature of SQL](#32-declarative-nature-of-sql)
4. [PostgreSQL Basics](#4-postgresql-basics)
   - [What is PostgreSQL?](#41-what-is-postgresql)
   - [SQL vs PostgreSQL](#42-sql-vs-postgresql)
   - [Basic PostgreSQL Architecture](#43-basic-postgresql-architecture)
5. [Database → Schema → Table](#5-database--schema--table)
   - [Database](#51-database)
   - [Schema](#52-schema)
   - [Table](#53-table)
   - [public Schema](#54-public-schema)
   - [Fully Qualified Table Name](#55-fully-qualified-table-name)
6. [Creating and Connecting to PostgreSQL](#6-creating-and-connecting-to-postgresql)
   - [CREATE DATABASE](#61-create-database)
   - [psql Connection](#62-psql-connection)
   - [psql Meta-Commands](#63-psql-meta-commands)
7. [DDL](#7-ddl)
   - [CREATE](#71-create)
   - [ALTER](#72-alter)
   - [TRUNCATE](#73-truncate)
   - [DROP](#74-drop)
   - [DELETE vs TRUNCATE vs DROP](#75-delete-vs-truncate-vs-drop)
8. [DML](#8-dml)
   - [INSERT](#81-insert)
   - [SELECT](#82-select)
   - [UPDATE](#83-update)
   - [DELETE](#84-delete)
   - [UPDATE/DELETE Without WHERE](#85-updatedelete-without-where)
9. [TCL](#9-tcl)
   - [Transaction](#91-transaction)
   - [BEGIN](#92-begin)
   - [COMMIT](#93-commit)
   - [ROLLBACK](#94-rollback)
   - [SAVEPOINT](#95-savepoint)
   - [ACID](#96-acid-connection)
10. [DCL](#10-dcl)
    - [GRANT](#101-grant)
    - [REVOKE](#102-revoke)
11. [SQL Command Categories](#11-sql-command-categories)
12. [Integrated End-to-End Example](#12-integrated-end-to-end-example)
13. [Important Comparisons](#13-important-comparisons)
14. [Exam Traps & Quick Revision](#14-exam-traps--quick-revision)
15. [One-Page Memory Map](#15-one-page-memory-map)
16. [Likely Exam Questions](#16-likely-exam-questions)

---

# 1. Data Fundamentals

## 1.1 Introduction to Data

### Definition

**Data** is a collection of raw facts, values, observations, or measurements that may not have a complete meaning without context or processing.

> **Data = Raw facts**

### Example

```text
Arnav
20
Bhilwara
85
```

These are pieces of data. Without context, we do not automatically know whether `20` means age, `85` means marks, etc.

A college student table may contain:

| Student ID | Name | Marks |
|---:|---|---:|
| 101 | Arnav | 85 |
| 102 | Rahul | 72 |
| 103 | Priya | 91 |

Every value in this table is data.

### Common forms of data

- Numbers: `100`, `25.5`
- Text: `"Arnav"`
- Dates: `2026-08-20`
- Boolean: `TRUE`, `FALSE`
- Images
- Videos
- Audio
- Sensor readings

### Hinglish

**Data basically raw material hai.** Jaise factory mein raw material ko process karke useful product banta hai, waise hi raw data ko process karke information aur insight milti hai.

### Exam Definition

> **Data is a collection of raw facts and figures that may not have meaningful interpretation without processing or context.**

---

## 1.2 Data → Information → Insight

A very important chain:

```text
Data → Information → Insight
```

## Data

Raw facts.

Example:

```text
85, 72, 91, 63, 88
```

These numbers alone do not tell us much.

## Information

When data is **processed, organized, and given context**, it becomes information.

Example:

```text
Student marks:
Arnav → 85
Rahul → 72
Priya → 91
Aman → 63
Riya → 88
```

Now the meaning is clear.

## Insight

An insight is a **useful conclusion, pattern, or understanding derived from information**.

Example:

```text
Average marks = 79.8
Priya scored the highest.
Aman needs improvement.
```

### Real-life example: e-commerce

**Data:**

```text
₹999
₹1,499
₹799
₹2,999
```

**Information:**

> These are prices of products sold today.

**Insight:**

> Products in the ₹500–₹1,500 range are selling more frequently.

That insight can support decisions such as pricing or marketing.

### Easy difference

| Term | Meaning |
|---|---|
| Data | Raw facts |
| Information | Processed and organized data |
| Insight | Useful conclusion derived from information |

### Memory Trick

> **Data = What happened**  
> **Information = What does it mean**  
> **Insight = What should we understand/do**

### Hinglish

**Data raw hai → Information organized hai → Insight decision-making ke kaam aata hai.**

---

## 1.3 Structured, Semi-Structured & Unstructured Data

Data can be broadly classified into three forms:

```text
                 DATA
                   |
       -------------------------
       |           |           |
 Structured   Semi-Structured  Unstructured
```

---

### A. Structured Data

Structured data has a **fixed format or predefined schema**.

Typical form:

- Tables
- Rows
- Columns
- Relational databases

Example:

| ID | Name | Age | City |
|---:|---|---:|---|
| 101 | Arnav | 20 | Bhilwara |
| 102 | Rahul | 21 | Jaipur |

SQL example:

```sql
CREATE TABLE students (
    student_id INT,
    name VARCHAR(100),
    age INT,
    city VARCHAR(50)
);
```

### Advantages

- Easy to store
- Easy to search
- Easy to analyze
- SQL can be used effectively

### Hinglish

Structured data ka format pehle se fixed hota hai. **Column kya hoga aur datatype kya hoga, sab defined hota hai.**

---

### B. Semi-Structured Data

Semi-structured data does **not follow a strict tabular format**, but it contains organizational elements such as keys, tags, metadata, or nested structures.

Common formats:

- JSON
- XML
- YAML

Example JSON:

```json
{
  "student_id": 101,
  "name": "Arnav",
  "age": 20,
  "skills": ["SQL", "Python"]
}
```

This data has structure through keys like `student_id`, `name`, and `skills`, but it is not a conventional relational table.

### Hinglish

Semi-structured data mein **structure hota hai, but strict fixed table structure nahi hota**.

---

### C. Unstructured Data

Unstructured data has **no predefined tabular structure**.

Examples:

- Images
- Videos
- Audio
- PDFs
- Social media posts
- Emails
- Free-form documents

Example:

```text
IMG_2026_08_20.jpg
```

A photo contains visual information, but it is not naturally represented as rows and columns.

---

### Comparison

| Feature | Structured | Semi-Structured | Unstructured |
|---|---|---|---|
| Structure | Fixed | Flexible/partial | No fixed structure |
| Schema | Predefined | Flexible | Usually no predefined schema |
| Example | SQL table | JSON/XML | Image/video/audio |
| Querying | Easy | Moderate | More difficult |
| Common storage | RDBMS | Document/NoSQL systems etc. | File/object storage etc. |

### Memory Trick

> **Structured = Table**  
> **Semi-Structured = JSON/XML**  
> **Unstructured = Image/Video/Audio**

---

## 1.4 Database & DBMS

## Database

A **database** is an organized collection of related data that can be stored, accessed, managed, and updated efficiently.

Example of a college database:

```text
Students
Courses
Teachers
Attendance
Marks
Fees
```

## DBMS

**DBMS = Database Management System**

A DBMS is software used to:

- Create databases
- Store data
- Retrieve data
- Update data
- Delete data
- Manage security
- Maintain data integrity
- Support controlled access/concurrency
- Support backup and recovery

### Examples of DBMS

- PostgreSQL
- MySQL
- Oracle Database
- Microsoft SQL Server
- SQLite

### Simple analogy

Think of a library.

**Database:** the books and information stored in the library.

**DBMS:** the system that helps you find, add, remove, manage, and protect the books/data.

So:

> **Database = Data**  
> **DBMS = Software that manages the data**

### Hinglish

Database ek **data ka organized collection** hai, while DBMS ek **software/system hai jo database ko manage karta hai**.

---

## 1.5 DBMS vs File-Based Systems

Before DBMS, organizations often stored data in separate files:

```text
students.txt
fees.txt
attendance.txt
marks.txt
```

This is a **file-based system**.

### Problems with File-Based Systems

#### 1. Data Redundancy

Same information may be stored repeatedly.

```text
students.txt → Arnav, Bhilwara
fees.txt     → Arnav, Bhilwara
marks.txt    → Arnav, Bhilwara
```

#### 2. Data Inconsistency

Different files may contain different versions of the same value.

```text
students.txt → Arnav, Jaipur
fees.txt     → Arnav, Bhilwara
```

#### 3. Difficult Data Sharing

Different applications may have difficulty accessing and updating common data consistently.

#### 4. Poor Security

Fine-grained control over who can access what is harder.

#### 5. Difficult Data Management

Searching, updating, deleting, and maintaining relationships across files becomes complicated.

#### 6. Weak Integrity Enforcement

Rules such as "Student ID must be unique" or "Marks cannot be negative" are harder to enforce consistently.

### Comparison

| Feature | File-Based System | DBMS |
|---|---|---|
| Data redundancy | High | Reduced/minimized |
| Data consistency | Difficult | Better |
| Security | Limited | Stronger |
| Data sharing | Difficult | Better support |
| Integrity constraints | Difficult | Supported |
| Relationships | Difficult | Built-in relational support |
| Backup/recovery | Limited | Supported |
| Concurrent access | Difficult | Supported |

### Hinglish

File system mein data **alag-alag files mein scattered** ho sakta hai. DBMS data ko **centrally organize, control, secure, and manage** karne mein help karta hai.

---

## 1.6 Advantages of DBMS

### 1. Reduced Data Redundancy

Duplicate data can be minimized.

### 2. Improved Data Consistency

Data can remain more consistent across the system.

### 3. Better Data Security

Users/roles can receive different permissions.

Example:

```text
Admin   → broad access
Teacher → marks-related access
Student → own/permitted records
```

### 4. Data Integrity

Rules can ensure valid data.

```sql
age INT CHECK (age >= 0)
```

### 5. Data Sharing

Multiple users/applications can work with the same database.

### 6. Backup and Recovery

Mechanisms support recovery after failures.

### 7. Concurrency Control

Multiple users can access data while the DBMS coordinates concurrent operations.

### 8. Efficient Data Access

SQL makes querying and filtering efficient.

```sql
SELECT * FROM students;
```

### 9. Data Independence

Applications can often remain insulated from certain changes in internal storage/organization.

### Exam Answer

If asked **"State advantages of DBMS"**, write:

1. Reduced data redundancy
2. Improved data consistency
3. Better security
4. Data integrity
5. Data sharing
6. Backup and recovery
7. Concurrency control
8. Efficient data retrieval
9. Data independence

---

# 2. Relational Database Concepts

## 2.1 Relational Database

A **relational database** stores data in the form of **relations/tables**.

A relation is represented as a table with rows and columns.

Example:

### STUDENTS

| student_id | name | age | city |
|---:|---|---:|---|
| 101 | Arnav | 20 | Bhilwara |
| 102 | Rahul | 21 | Jaipur |
| 103 | Priya | 20 | Udaipur |

The relational model was introduced by **E. F. Codd**.

### Main concepts

```text
Database
   ↓
Tables
   ↓
Rows + Columns
   ↓
Keys + Constraints
   ↓
Relationships
```

---

## 2.2 Tables, Rows, Columns & Attributes

Consider:

| student_id | name | age | city |
|---:|---|---:|---|
| 101 | Arnav | 20 | Bhilwara |
| 102 | Rahul | 21 | Jaipur |

### Table

The complete relation, e.g. `STUDENTS`, is a **table**.

### Row

A horizontal record is a **row**, also called a **record** or **tuple**.

Example:

```text
101 | Arnav | 20 | Bhilwara
```

### Column

A vertical field is a **column**.

```text
student_id
name
age
city
```

### Attribute

An **attribute** is a property/characteristic of an entity.

In relational terms, an attribute generally corresponds to a column.

### Important equivalences

> **Row ≈ Record/Tuple**  
> **Column ≈ Attribute/Field**

---

## 2.3 Entities & Attributes

### Entity

An **entity** is a real-world object, person, place, thing, or concept about which data is stored.

Examples:

- Student
- Teacher
- Course
- Employee
- Product
- Customer

### Attributes

Attributes describe an entity.

Example:

```text
Student
├── Student_ID
├── Name
├── Age
├── Email
└── City
```

### Entity Instance

One particular occurrence of an entity is an **entity instance**.

```text
Entity → Student
Instance → Arnav
```

For example:

```text
101, Arnav, 20, arnav@gmail.com
```

is one student instance.

### Hinglish

**Entity = object/category**  
**Entity instance = category ka actual member/object**

Easy trick:

> **Entity = Noun**  
> **Attribute = Description of the noun**

---

## 2.4 Database Schema

A **database schema** is the logical structure/design of a database.

It may define:

- Tables
- Columns
- Data types
- Keys
- Constraints
- Relationships
- Other database objects

Example:

```text
STUDENT
---------
student_id INT
name VARCHAR(100)
age INT

COURSE
---------
course_id INT
course_name VARCHAR(100)
```

### Schema vs Data

**Schema:** describes how the database is structured.

**Data:** describes what is actually stored.

```text
Schema:
student_id INT
name VARCHAR(100)

Data:
101, Arnav
102, Rahul
```

### Memory Trick

> **Schema = Design / Blueprint**  
> **Data = Actual content/values**

### Hinglish

Schema ko database ka **blueprint/design** samajh sakte ho.

---

## 2.5 Super Key

### Definition

A **Super Key** is any set of one or more attributes that can **uniquely identify a row** in a table.

Example table:

| student_id | email | name |
|---:|---|---|
| 101 | arnav@gmail.com | Arnav |
| 102 | rahul@gmail.com | Rahul |

Possible super keys include:

```text
{student_id}
{email}
{student_id, name}
{student_id, email}
{student_id, email, name}
```

If a combination uniquely identifies a row, it is a super key.

### Important Point

A super key **may contain extra/unnecessary attributes**.

For example:

```text
{student_id, name}
```

can be a super key if `student_id` alone is already unique.

### Hinglish

Super key ka main goal:

> **"Row ko uniquely identify kar paaye."**

Extra attributes allowed hain.

---

## 2.6 Candidate Key

### Definition

A **Candidate Key** is a **minimal super key**.

It:

1. Uniquely identifies the row.
2. Contains no unnecessary attribute.

Suppose `student_id` and `email` are individually unique.

Candidate keys:

```text
{student_id}
{email}
```

But:

```text
{student_id, name}
```

is not a candidate key because `name` is unnecessary.

### Relationship

```text
Super Key
   ↓ remove unnecessary attributes
Candidate Key
```

### Example

| student_id | email | name |
|---:|---|---|
| 101 | a@gmail.com | Arnav |
| 102 | b@gmail.com | Rahul |

Candidate keys:

```text
{student_id}
{email}
```

---

## 2.7 Primary Key

### Definition

A **Primary Key** is the candidate key selected to uniquely identify rows in a table.

Suppose candidate keys are:

```text
student_id
email
```

We choose `student_id` as the primary key.

### SQL

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);
```

### Properties

- Uniquely identifies each row.
- Primary key values cannot be `NULL`.
- There is one **primary key constraint** per table.
- The primary key can consist of multiple columns (then it is a composite primary key).

### Important difference

A table can have:

```text
Multiple candidate keys
```

but only:

```text
One selected primary key constraint
```

### Example

```text
Candidate Keys:
1. student_id
2. email

Chosen Primary Key:
student_id
```

The unchosen candidate key can still be enforced as `UNIQUE`.

---

## 2.8 Composite Key

### Definition

A **Composite Key** is a key consisting of **two or more attributes/columns** used together to uniquely identify a row.

Example: enrollment table.

| student_id | course_id | enrollment_date |
|---:|---:|---|
| 101 | 501 | 2026-08-01 |
| 101 | 502 | 2026-08-02 |
| 102 | 501 | 2026-08-03 |

Here:

```text
student_id alone → NOT unique
course_id alone → NOT unique
```

But:

```text
(student_id, course_id)
```

together can uniquely identify each enrollment.

### SQL

```sql
CREATE TABLE enrollment (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id)
);
```

### Hinglish

Jab **single column unique identification nahi kar paati**, tab multiple columns ko combine karke key banate hain.

---

## 2.9 Constraints

### Definition

**Constraints** are rules applied to columns/table data to restrict invalid values and maintain data quality/integrity.

Think:

> **Constraints = Data ke rules**

### Common SQL constraints

#### 1. PRIMARY KEY

Uniquely identifies each row.

```sql
student_id INT PRIMARY KEY
```

#### 2. NOT NULL

Column cannot contain `NULL`.

```sql
name VARCHAR(100) NOT NULL
```

#### 3. UNIQUE

Values must be unique under the constraint.

```sql
email VARCHAR(100) UNIQUE
```

#### 4. CHECK

Ensures a condition.

```sql
age INT CHECK (age >= 18)
```

#### 5. DEFAULT

Provides a default value when one is not supplied.

```sql
city VARCHAR(50) DEFAULT 'Bhilwara'
```

#### 6. FOREIGN KEY

References a key in another table.

```sql
course_id INT REFERENCES courses(course_id)
```

### Complete example

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    age INT CHECK (age >= 18),
    city VARCHAR(50) DEFAULT 'Bhilwara'
);
```

---

## 2.10 Foreign Key

### Definition

A **Foreign Key** is a column or set of columns in one table that refers to a key (commonly a primary key or a suitable `UNIQUE` key) in another table.

It helps establish relationships between tables and supports referential integrity.

### Parent table

```text
COURSES
```

| course_id | course_name |
|---:|---|
| 501 | DBMS |
| 502 | Python |
| 503 | Java |

`course_id` is the primary key.

### Child table

```text
STUDENTS
```

| student_id | name | course_id |
|---:|---|---:|
| 101 | Arnav | 501 |
| 102 | Rahul | 502 |

Here:

```text
students.course_id
        ↓
 courses.course_id
```

### SQL

```sql
CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100)
);

CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    course_id INT,
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

### Why?

A student record should not reference a course ID that does not exist in the parent table, unless the relationship is intentionally nullable and the value is `NULL`.

### Hinglish

Foreign key ka kaam tables ko **connect karna** hai.

---

## 2.11 Referential Integrity

### Definition

**Referential Integrity** means that relationships between tables remain valid and foreign key references are consistent with the referenced table.

Example:

### Parent

| course_id | course_name |
|---:|---|
| 501 | DBMS |
| 502 | Python |

### Child

| student_id | name | course_id |
|---:|---|---:|
| 101 | Arnav | 501 |

This is valid because `501` exists in the parent table.

But:

| student_id | name | course_id |
|---:|---|---:|
| 102 | Rahul | 999 |

is invalid when `999` is absent from the referenced table.

### Why is it important?

It prevents:

- Invalid references
- Orphan records
- Broken relationships
- Inconsistent data

### Hinglish

Agar `students.course_id = 501` hai, toh `courses` table mein **501 exist karna chahiye** (unless the foreign key is nullable and the value is `NULL`).

---

## 2.12 Relationships Between Tables

Main relationship types:

```text
1. One-to-One (1:1)
2. One-to-Many (1:N)
3. Many-to-Many (M:N)
```

### A. One-to-One (1:1)

One row in Table A corresponds to at most one row in Table B, and vice versa.

Example:

```text
Person ↔ Passport
```

Conceptual structure:

```text
PERSON
person_id
name

PASSPORT
passport_id
person_id
```

### B. One-to-Many (1:N)

One row in Table A can relate to many rows in Table B.

Example:

```text
Department → Employees
```

One department can have many employees.

Example tables:

**DEPARTMENT**

| department_id | department_name |
|---:|---|
| 10 | IT |
| 20 | HR |

**EMPLOYEE**

| employee_id | name | department_id |
|---:|---|---:|
| 101 | Arnav | 10 |
| 102 | Rahul | 10 |
| 103 | Priya | 20 |

Here:

```text
Department 10 → Employee 101
             → Employee 102
```

The foreign key is typically on the **many side** (`employee.department_id`).

### C. Many-to-Many (M:N)

Many rows in Table A can relate to many rows in Table B.

Example:

```text
Students ↔ Courses
```

One student may enroll in many courses, and one course may have many students.

A relational database commonly implements this using an **intermediate/junction/bridge table**.

### STUDENTS

| student_id | name |
|---:|---|
| 101 | Arnav |
| 102 | Rahul |

### COURSES

| course_id | course_name |
|---:|---|
| 501 | DBMS |
| 502 | Python |

### ENROLLMENT

| student_id | course_id |
|---:|---:|
| 101 | 501 |
| 101 | 502 |
| 102 | 501 |

The `ENROLLMENT` table resolves the many-to-many relationship.

### Relationship summary

| Relationship | Example |
|---|---|
| 1:1 | Person ↔ Passport |
| 1:N | Department → Employees |
| M:N | Students ↔ Courses |

---

# 3. Introduction to SQL

## 3.1 SQL and Relational Databases

### Definition

**SQL = Structured Query Language**.

SQL is the standard language used to define, query, manipulate, and control data in relational database systems.

Using SQL, we can:

- Create databases/tables and other objects
- Insert data
- Read/query data
- Update data
- Delete data
- Control transactions
- Manage permissions

### Simple explanation

SQL is the **language used to communicate with relational databases**.

Example:

```sql
SELECT * FROM students;
```

Meaning:

> "Students table ka saara data mujhe dikhao."

### SQL with relational tables

Example:

**Students**

| student_id | name | course_id |
|---:|---|---:|
| 101 | Arnav | 501 |
| 102 | Rahul | 502 |

**Courses**

| course_id | course_name |
|---:|---|
| 501 | DBMS |
| 502 | Python |

The relationship is represented through `course_id`.

SQL can query these related tables.

---

## 3.2 Declarative Nature of SQL

SQL is primarily a **declarative language**.

That means you usually specify **WHAT result/data you want**, rather than the exact procedural steps for how to obtain it.

Example:

```sql
SELECT name
FROM students
WHERE age > 18;
```

You do not tell PostgreSQL exactly which pages, indexes, or physical memory locations to scan. PostgreSQL chooses an execution strategy.

### Hinglish

Tum PostgreSQL ko **result kya chahiye** batate ho. Result ko internally kaise obtain karna hai, wo database engine/optimizer decide karta hai.

---

# 4. PostgreSQL Basics

## 4.1 What is PostgreSQL?

**PostgreSQL** is an open-source **object-relational database management system (ORDBMS)**.

It is commonly called:

```text
Postgres
```

### Important capabilities

PostgreSQL supports:

- Relational tables
- SQL
- Primary and foreign keys
- Constraints
- Transactions
- Indexes
- Views
- Functions
- Triggers
- JSON/JSONB
- Advanced data types
- Extensions

---

## 4.2 SQL vs PostgreSQL

This distinction is important.

### SQL

SQL is a **language**.

### PostgreSQL

PostgreSQL is a **DBMS/ORDBMS software system that supports SQL**.

Think:

```text
SQL → Language
PostgreSQL → Database management system that understands SQL
```

### Analogy

```text
English → Language
Human → Entity that understands/uses the language

SQL → Database language
PostgreSQL → Database system that understands/uses SQL
```

> Do not write: **"PostgreSQL is a SQL language."**
>
> Better: **"PostgreSQL is a DBMS/ORDBMS that supports SQL."**

---

## 4.3 Basic PostgreSQL Architecture

A simple conceptual view:

```text
PostgreSQL Server / Instance
      │
      ├── Database 1
      │      ├── Schemas
      │      │    ├── Tables
      │      │    └── Other objects
      │
      ├── Database 2
      │
      └── Database 3
```

A client connects to a PostgreSQL server and works with a selected database.

---

# 5. Database → Schema → Table

A useful PostgreSQL hierarchy is:

```text
Database
   ↓
Schema
   ↓
Table
   ↓
Columns + Rows
```

---

## 5.1 Database

A **database** is a logical container that stores related data and database objects.

Example:

```text
college_db
```

This database may contain data about students, courses, teachers, etc.

---

## 5.2 Schema

A **schema** in PostgreSQL is a namespace/container inside a database used to organize tables and other database objects.

Example:

```text
college_db
   ├── public
   │     ├── students
   │     └── courses
   │
   └── administration
         ├── teachers
         └── departments
```

Here:

```text
college_db → Database
public → Schema
students → Table
```

### Why schemas?

Schemas help organize objects and can separate namespaces, such as:

```text
engineering
management
administration
```

### Hinglish

Schema ko database ke andar ek **folder/namespace** ki tarah samajh sakte ho. Technically schema is a namespace, not merely a folder, but this analogy is useful for learning.

---

## 5.3 Table

A table stores structured records.

Example:

```text
students
```

| student_id | name | age |
|---:|---|---:|
| 101 | Arnav | 20 |
| 102 | Rahul | 21 |

So:

```text
Database
   ↓
Schema
   ↓
Table
   ↓
Rows + Columns
```

---

## 5.4 public Schema

PostgreSQL commonly creates a schema named:

```text
public
```

for a default setup.

If you create a table without explicitly naming a schema:

```sql
CREATE TABLE students (...);
```

it is normally created in the first suitable schema on your current `search_path`, commonly `public` in a default configuration.

You can also specify it explicitly:

```sql
CREATE TABLE public.students (...);
```

---

## 5.5 Fully Qualified Table Name

You may write:

```sql
SELECT * FROM public.students;
```

Here:

```text
public   → Schema
students → Table
```

This is useful when multiple schemas contain similarly named tables.

---

# 6. Creating and Connecting to PostgreSQL

Creating a database and connecting to it are separate concepts.

---

## 6.1 CREATE DATABASE

Create a database:

```sql
CREATE DATABASE college_db;
```

Meaning:

> "Create a new database named `college_db`."

### Important point

`CREATE DATABASE` creates the **database container**, not the tables inside it.

After creating it, you still need to connect to that database and create tables.

---

## 6.2 psql Connection

`psql` is PostgreSQL's command-line client.

Example:

```bash
psql -U postgres -d college_db
```

Meaning:

```text
psql          → PostgreSQL command-line client
-U postgres   → connect as user 'postgres'
-d college_db → connect to database 'college_db'
```

A more explicit form:

```bash
psql -h localhost -p 5432 -U postgres -d college_db
```

Where:

```text
-h → Host
-p → Port
-U → Username
-d → Database
```

PostgreSQL's commonly used default port is:

```text
5432
```

unless changed in the configuration.

---

## 6.3 psql Meta-Commands

Important: commands beginning with `\` in `psql` are generally **psql meta-commands**, not standard SQL statements.

### Show connection information

```text
\conninfo
```

### List databases

```text
\l
```

or

```text
\list
```

### Connect to another database

```text
\c college_db
```

or

```text
\connect college_db
```

### SQL vs psql command

SQL:

```sql
CREATE DATABASE college_db;
```

psql meta-command:

```text
\c college_db
```

They are different kinds of commands.

---

# 7. DDL

## Data Definition Language

**DDL = Data Definition Language**

DDL commands primarily define or modify the **structure/schema of database objects**.

Main commands in this syllabus:

```text
CREATE
ALTER
TRUNCATE
DROP
```

### Memory trick

> **DDL = Design/Definition = Structure**

---

## 7.1 CREATE

### Definition

`CREATE` is used to create database objects such as databases, tables, schemas, views, indexes, etc.

### Create Database

```sql
CREATE DATABASE college_db;
```

### Create Table

```sql
CREATE TABLE students (
    student_id INT,
    name VARCHAR(100),
    age INT
);
```

Breakdown:

- `CREATE TABLE` → create a table
- `students` → table name
- `student_id INT` → integer column
- `name VARCHAR(100)` → text column
- `age INT` → integer column

### Better table example

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INT CHECK (age >= 0),
    email VARCHAR(150) UNIQUE
);
```

Now:

```text
student_id → PRIMARY KEY
name       → NOT NULL
age        → CHECK
email      → UNIQUE
```

---

## 7.2 ALTER

### Definition

`ALTER` modifies the structure of an existing database object.

Suppose:

```sql
CREATE TABLE students (
    student_id INT,
    name VARCHAR(100)
);
```

### Add a column

```sql
ALTER TABLE students
ADD COLUMN email VARCHAR(150);
```

Hinglish:

> Existing table ko delete nahi karna; bas usmein new column add karna.

### Rename a column

```sql
ALTER TABLE students
RENAME COLUMN name TO full_name;
```

### Rename a table

```sql
ALTER TABLE students
RENAME TO college_students;
```

### Change a data type

```sql
ALTER TABLE students
ALTER COLUMN age TYPE SMALLINT;
```

This works only when PostgreSQL can perform the conversion appropriately.

### Set a default

```sql
ALTER TABLE students
ALTER COLUMN city SET DEFAULT 'Bhilwara';
```

### Drop a column

```sql
ALTER TABLE students
DROP COLUMN city;
```

This removes the column and its stored values.

### ALTER summary

```text
ALTER TABLE → existing table ki structure modify karo
```

---

## 7.3 TRUNCATE

### Definition

`TRUNCATE` removes **all rows from a table** while keeping the table structure.

```sql
TRUNCATE TABLE students;
```

After the command:

```text
students table → still exists
rows → removed
columns → remain
structure → remains
```

### Before

| id | name |
|---:|---|
| 101 | Arnav |
| 102 | Rahul |

### After

| id | name |
|---:|---|
| *(empty)* | |

### Important

`TRUNCATE` does not support a `WHERE` clause.

Invalid:

```sql
TRUNCATE TABLE students WHERE age < 18;
```

Use `DELETE` instead:

```sql
DELETE FROM students
WHERE age < 18;
```

### Important PostgreSQL note

`TRUNCATE` is transactional in PostgreSQL, meaning it can participate in a transaction and can be rolled back before commit, subject to normal transaction rules. It is nevertheless commonly taught in the DDL-style group because it changes the contents of the table as a whole and operates at table level.

---

## 7.4 DROP

### Definition

`DROP` removes a database object itself.

Example:

```sql
DROP TABLE students;
```

After this:

```text
students table → gone
data → gone
table definition → gone
```

The following will fail because the table no longer exists:

```sql
SELECT * FROM students;
```

### Hinglish

`DROP` ka matlab sirf data clear karna nahi hai — **table/object hi hata dena**.

---

## 7.5 DELETE vs TRUNCATE vs DROP

| Command | Removes Rows | Removes Table Structure | `WHERE` |
|---|---:|---:|---|
| DELETE | Yes | No | Yes |
| TRUNCATE | All rows | No | No |
| DROP | Yes | Yes | No |

### Memory Trick

```text
DELETE    → Data/selected rows remove
TRUNCATE  → All rows remove, structure remains
DROP      → Entire table/object remove
```

---

# 8. DML

## Data Manipulation Language

DML is used to manipulate data stored in tables.

Commonly taught commands:

```text
INSERT
SELECT
UPDATE
DELETE
```

> **Note:** In some textbooks, `SELECT` is classified separately as **DQL (Data Query Language)**. For an exam, follow the classification used in your course/teacher's notes.

---

## 8.1 INSERT

### Definition

`INSERT` adds new rows to a table.

Example table:

```sql
CREATE TABLE students (
    student_id INT,
    name VARCHAR(100),
    age INT
);
```

### Insert one row

```sql
INSERT INTO students
(student_id, name, age)
VALUES
(101, 'Arnav', 20);
```

Breakdown:

```text
INSERT INTO students → target table
(student_id, name, age) → columns being populated
VALUES (...) → actual values
```

### Insert multiple rows

```sql
INSERT INTO students
(student_id, name, age)
VALUES
(101, 'Arnav', 20),
(102, 'Rahul', 21),
(103, 'Priya', 20);
```

### Insert without specifying columns

```sql
INSERT INTO students
VALUES (101, 'Arnav', 20);
```

This relies on the table's column order and is more fragile.

### Better practice

Specify column names explicitly:

```sql
INSERT INTO students
(student_id, name, age)
VALUES
(101, 'Arnav', 20);
```

---

## 8.2 SELECT

### Definition

`SELECT` retrieves/query data.

### All columns, all rows

```sql
SELECT * FROM students;
```

Breakdown:

```text
SELECT → what data?
*      → all columns
FROM   → from which table?
students → table
```

### Specific columns

```sql
SELECT name, age
FROM students;
```

### With WHERE

Suppose:

| id | name | age |
|---:|---|---:|
| 101 | Arnav | 20 |
| 102 | Rahul | 21 |
| 103 | Priya | 17 |

Query:

```sql
SELECT *
FROM students
WHERE age >= 18;
```

Result:

| id | name | age |
|---:|---|---:|
| 101 | Arnav | 20 |
| 102 | Rahul | 21 |

### Hinglish

`WHERE` ek **filter** lagata hai.

> "Sirf woh rows do jo condition satisfy karti hain."

### ORDER BY

Ascending (default):

```sql
SELECT *
FROM students
ORDER BY age;
```

Descending:

```sql
SELECT *
FROM students
ORDER BY age DESC;
```

### LIMIT

```sql
SELECT *
FROM students
LIMIT 5;
```

Maximum 5 rows return karega.

---

## 8.3 UPDATE

### Definition

`UPDATE` modifies existing rows.

Suppose:

| id | name | age |
|---:|---|---:|
| 101 | Arnav | 20 |

Update age:

```sql
UPDATE students
SET age = 21
WHERE student_id = 101;
```

Breakdown:

```text
UPDATE students → which table?
SET age = 21    → what change?
WHERE ...       → which row(s)?
```

### Dangerous example

```sql
UPDATE students
SET age = 21;
```

Without `WHERE`, every row is updated.

### Hinglish

`UPDATE` ke saath `WHERE` bhoolna dangerous ho sakta hai.

---

## 8.4 DELETE

### Definition

`DELETE` removes existing rows.

```sql
DELETE FROM students
WHERE student_id = 101;
```

Only that row is removed.

### Delete every row

```sql
DELETE FROM students;
```

This removes all rows but keeps the table structure.

### DELETE vs DROP

```sql
DELETE FROM students;
```

> Table rakho, records delete karo.

```sql
DROP TABLE students;
```

> Table hi remove kar do.

---

## 8.5 UPDATE/DELETE Without WHERE

### UPDATE without WHERE

```sql
UPDATE students
SET age = 20;
```

**All rows are updated.**

### DELETE without WHERE

```sql
DELETE FROM students;
```

**All rows are deleted.**

### Exam/practical rule

Before executing an `UPDATE` or `DELETE`, double-check your `WHERE` condition unless you intentionally want every row affected.

---

# 9. TCL

## Transaction Control Language

**TCL = Transaction Control Language**

TCL controls **transactions**.

Common commands:

```text
COMMIT
ROLLBACK
SAVEPOINT
```

---

## 9.1 Transaction

### Definition

A **transaction** is a logical unit of work consisting of one or more database operations that should be treated together.

### Bank transfer example

Suppose Arnav sends ₹1000 to Rahul.

Two operations:

```text
1. Arnav balance -1000
2. Rahul balance +1000
```

These operations should behave as one logical unit.

If the first succeeds and the second fails without proper transaction handling, the database could become incorrect.

A transaction helps coordinate this work.

---

## 9.2 BEGIN

While not listed as a separate TCL item in every syllabus, `BEGIN` is commonly used to explicitly start a transaction in PostgreSQL.

Example:

```sql
BEGIN;
```

Then perform operations, and finally `COMMIT` or `ROLLBACK`.

---

## 9.3 COMMIT

### Definition

`COMMIT` makes the current transaction's changes permanent/visible as committed according to transaction rules.

Example:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE account_id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE account_id = 2;

COMMIT;
```

### Hinglish

`COMMIT` ka matlab:

> **"Jo changes maine transaction mein kiye hain, unhe final maan lo."**

---

## 9.4 ROLLBACK

### Definition

`ROLLBACK` undoes changes in the current transaction that have not yet been committed.

Example:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE account_id = 1;

ROLLBACK;
```

The update is undone.

### Hinglish

`ROLLBACK`:

> **"Oops, ye changes nahi chahiye. Previous transaction state par wapas jao."**

---

## 9.5 SAVEPOINT

### Definition

A `SAVEPOINT` creates an intermediate point inside a transaction. You can later roll back to that point without canceling the entire transaction.

Example:

```sql
BEGIN;

UPDATE students
SET age = 21
WHERE student_id = 101;

SAVEPOINT s1;

UPDATE students
SET age = 25
WHERE student_id = 102;

ROLLBACK TO SAVEPOINT s1;

COMMIT;
```

### What happens?

1. Student 101 age becomes 21.
2. `s1` checkpoint is created.
3. Student 102 age becomes 25.
4. `ROLLBACK TO SAVEPOINT s1` undoes changes made after `s1`, so the second update is undone.
5. The first update remains.
6. `COMMIT` commits the remaining transaction changes.

### Hinglish

`SAVEPOINT` = **transaction ke beech mein checkpoint**.

> "Agar aage kuch galat hua toh isi checkpoint tak rollback kar dena."

---

## 9.6 ACID Connection

Transactions are closely associated with the ACID properties:

```text
A → Atomicity
C → Consistency
I → Isolation
D → Durability
```

### Atomicity

All operations of the logical transaction happen or the transaction's effects are not partially committed.

### Consistency

A successful transaction preserves defined database rules/invariants.

### Isolation

Concurrent transactions should not improperly interfere with one another.

### Durability

Committed changes are designed to survive failures according to the database's durability guarantees.

### Simple Hinglish

```text
Atomicity  → All or nothing
Consistency → Valid state to valid state
Isolation   → Transactions interfere na karein in invalid ways
Durability  → COMMIT ke baad data survive kare
```

---

# 10. DCL

## Data Control Language

**DCL = Data Control Language**

DCL deals with **permissions/access control**.

Main commands:

```text
GRANT
REVOKE
```

---

## 10.1 GRANT

### Definition

`GRANT` gives privileges to a user/role.

Example:

```sql
GRANT SELECT ON students TO user1;
```

Meaning:

> `user1` ko `students` table ko read karne ki permission do.

### Multiple privileges

```sql
GRANT SELECT, INSERT, UPDATE
ON students
TO user1;
```

Now the role/user gets the specified privileges.

### Common privileges (depending on object)

```text
SELECT
INSERT
UPDATE
DELETE
TRUNCATE
REFERENCES
TRIGGER
```

---

## 10.2 REVOKE

### Definition

`REVOKE` removes privileges that have been granted through the relevant access-control mechanisms.

Example:

```sql
REVOKE INSERT
ON students
FROM user1;
```

Meaning:

> `user1` se `INSERT` permission hatao.

### Real-life example

A college database might conceptually have:

```text
Student role  → SELECT on permitted student data
Teacher role  → SELECT + UPDATE on permitted academic data
Admin role    → broader privileges
```

This is access control.

---

# 11. SQL Command Categories

The common introductory classification is:

```text
                         SQL
                          │
       ┌──────────────────┼──────────────────┐
       │                  │                  │
      DDL                DML                TCL
       │                  │                  │
 CREATE               INSERT             COMMIT
 ALTER                SELECT*            ROLLBACK
 TRUNCATE             UPDATE             SAVEPOINT
 DROP                 DELETE
                          │
                         DCL
                          │
                       GRANT
                       REVOKE
```

## DDL — Data Definition Language

Used for structure:

```text
CREATE
ALTER
TRUNCATE
DROP
```

Think:

> **Structure ke saath kaam.**

## DML — Data Manipulation Language

Used for table data:

```text
INSERT
UPDATE
DELETE
```

`SELECT` is often taught with DML in introductory notes, but more formally is often classified as DQL.

Think:

> **Data ke saath kaam.**

## TCL — Transaction Control Language

```text
COMMIT
ROLLBACK
SAVEPOINT
```

Think:

> **Transactions ko control karna.**

## DCL — Data Control Language

```text
GRANT
REVOKE
```

Think:

> **Who can do what?**

### Category comparison

| Category | Full Form | Main Purpose | Commands |
|---|---|---|---|
| DDL | Data Definition Language | Define/change structure | CREATE, ALTER, TRUNCATE, DROP |
| DML | Data Manipulation Language | Manipulate data | INSERT, SELECT*, UPDATE, DELETE |
| TCL | Transaction Control Language | Manage transactions | COMMIT, ROLLBACK, SAVEPOINT |
| DCL | Data Control Language | Manage permissions | GRANT, REVOKE |

`*` `SELECT` is frequently classified as DQL in formal classifications.

---

# 12. Integrated End-to-End Example

Let's build a small student database from scratch.

## Step 1 — Create Database

```sql
CREATE DATABASE college_db;
```

Category:

```text
CREATE → DDL
```

---

## Step 2 — Connect to Database

In `psql`:

```text
\c college_db
```

---

## Step 3 — Create Table

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INT CHECK (age >= 0),
    email VARCHAR(150) UNIQUE
);
```

---

## Step 4 — Insert Data

```sql
INSERT INTO students
(student_id, name, age, email)
VALUES
(101, 'Arnav', 20, 'arnav@example.com'),
(102, 'Rahul', 21, 'rahul@example.com'),
(103, 'Priya', 20, 'priya@example.com');
```

Category:

```text
INSERT → DML
```

---

## Step 5 — Read Data

```sql
SELECT * FROM students;
```

Example output:

| student_id | name | age | email |
|---:|---|---:|---|
| 101 | Arnav | 20 | arnav@example.com |
| 102 | Rahul | 21 | rahul@example.com |
| 103 | Priya | 20 | priya@example.com |

---

## Step 6 — Update Data

```sql
UPDATE students
SET age = 21
WHERE student_id = 101;
```

Arnav's age becomes 21.

---

## Step 7 — Delete Data

```sql
DELETE FROM students
WHERE student_id = 103;
```

Priya's row is removed.

---

## Step 8 — Transaction

```sql
BEGIN;

UPDATE students
SET age = 22
WHERE student_id = 101;

ROLLBACK;
```

The change is undone.

---

# 13. Important Comparisons

## 13.1 Super Key vs Candidate Key vs Primary Key

Suppose the table has:

```text
student_id
email
name
```

and both `student_id` and `email` are unique.

### Super Key

Any combination that uniquely identifies the row, e.g.:

```text
{student_id}
{email}
{student_id, name}
{email, name}
{student_id, email}
...
```

### Candidate Key

Minimal super keys:

```text
{student_id}
{email}
```

### Primary Key

One selected candidate key:

```text
{student_id}
```

### Memory Trick

```text
Super Key
   ↓ remove unnecessary attributes
Candidate Key
   ↓ choose one
Primary Key
```

---

## 13.2 Primary Key vs Foreign Key

| Primary Key | Foreign Key |
|---|---|
| Identifies rows in its own table | References a key in another table |
| Must be unique within its key constraint | Can repeat |
| Cannot be `NULL` | May be `NULL` if relationship is optional and column permits it |
| One primary key constraint per table | A table can have multiple foreign keys |
| Represents entity identity | Helps establish relationships |

Example:

```text
COURSE
course_id PK
```

```text
STUDENT
course_id FK
```

---

## 13.3 Entity vs Attribute

```text
Student → Entity
Name    → Attribute
Age     → Attribute
Email   → Attribute
```

Memory:

> **Entity = Noun**  
> **Attribute = Description of noun**

---

## 13.4 Schema vs Data

### Schema

```text
student_id INT
name VARCHAR(100)
age INT
```

### Data

```text
101 | Arnav | 20
102 | Rahul | 21
```

Memory:

> **Schema = Structure**  
> **Data = Values**

---

## 13.5 DBMS vs File System

| Aspect | File-Based | DBMS |
|---|---|---|
| Redundancy | Often high | Reduced |
| Consistency | Harder | Better |
| Security | Limited | Stronger |
| Integrity rules | Harder | Built-in constraints |
| Relationships | Difficult | Supported |
| Concurrency | Difficult | Supported |
| Backup/recovery | More limited | Supported |

---

## 13.6 SQL vs PostgreSQL

| SQL | PostgreSQL |
|---|---|
| Language | DBMS/ORDBMS |
| Defines/query/manipulates data | Executes/implements SQL and manages databases |
| Standard language | Specific open-source software system |

---

## 13.7 DELETE vs TRUNCATE vs DROP

| DELETE | TRUNCATE | DROP |
|---|---|---|
| Removes rows | Removes all rows | Removes table/object |
| Can use `WHERE` | No `WHERE` | No `WHERE` |
| Structure remains | Structure remains | Structure removed |
| Data removal is row-oriented | Table-level all-row removal | Object removal |

---

# 14. Exam Traps & Quick Revision

## Trap 1: DELETE vs TRUNCATE vs DROP

### DELETE

```sql
DELETE FROM students
WHERE age < 18;
```

Can remove selected rows.

### TRUNCATE

```sql
TRUNCATE TABLE students;
```

Removes all rows; no `WHERE`.

### DROP

```sql
DROP TABLE students;
```

Removes the table itself.

---

## Trap 2: PostgreSQL is not SQL

Wrong:

> PostgreSQL is a SQL language.

Correct:

> PostgreSQL is an open-source DBMS/ORDBMS that supports SQL.

---

## Trap 3: Database vs Schema vs Table

Remember:

```text
Database
   ↓
Schema
   ↓
Table
   ↓
Rows + Columns
```

Example:

```text
college_db
   ↓
public
   ↓
students
   ↓
101 | Arnav | 20
```

---

## Trap 4: UPDATE Without WHERE

```sql
UPDATE students
SET age = 20;
```

All rows are updated.

Targeted update:

```sql
UPDATE students
SET age = 20
WHERE student_id = 101;
```

---

## Trap 5: DELETE Without WHERE

```sql
DELETE FROM students;
```

All rows are deleted, but the table remains.

---

## Trap 6: SELECT Classification

You may see:

```text
DML:
INSERT
SELECT
UPDATE
DELETE
```

or:

```text
DML:
INSERT
UPDATE
DELETE

DQL:
SELECT
```

Both classification styles are encountered. For an exam, follow your teacher/course material.

---

## Trap 7: `psql` Commands vs SQL

```text
\l
\c college_db
\conninfo
```

are `psql` meta-commands, not standard SQL statements.

By contrast:

```sql
CREATE DATABASE college_db;
```

is SQL.

---

# 15. One-Page Memory Map

## Data flow

```text
DATA
  ↓
Information
  ↓
Insight
```

## Data types

```text
DATA
  ├── Structured → Tables
  ├── Semi-Structured → JSON/XML
  └── Unstructured → Images/Videos/Audio/etc.
```

## Relational DB flow

```text
Database
  ↓
Tables
  ↓
Rows + Columns
  ↓
Entities + Attributes
  ↓
Keys + Constraints
  ↓
Relationships
```

## Keys

```text
Super Key → any unique combination
Candidate Key → minimal super key
Primary Key → selected candidate key
Composite Key → 2+ columns together
Foreign Key → reference to another table
```

## SQL command categories

```text
SQL
│
├── DDL → Structure
│   ├── CREATE
│   ├── ALTER
│   ├── TRUNCATE
│   └── DROP
│
├── DML → Data
│   ├── INSERT
│   ├── SELECT*
│   ├── UPDATE
│   └── DELETE
│
├── TCL → Transactions
│   ├── BEGIN
│   ├── COMMIT
│   ├── ROLLBACK
│   └── SAVEPOINT
│
└── DCL → Permissions
    ├── GRANT
    └── REVOKE
```

`*` `SELECT` is also commonly classified as DQL.

---

# 16. Likely Exam Questions

## Theory questions

1. What is data? Explain Data → Information → Insight.
2. Differentiate structured, semi-structured, and unstructured data.
3. Define database and DBMS.
4. Explain advantages of DBMS.
5. Differentiate DBMS and file-based systems.
6. What is a relational database?
7. Define entity and attribute.
8. What is a database schema?
9. Define super key, candidate key, and primary key.
10. What is a composite key?
11. Explain different database constraints.
12. What is a foreign key?
13. Explain referential integrity.
14. Explain one-to-one, one-to-many, and many-to-many relationships.
15. What is SQL?
16. Differentiate SQL and PostgreSQL.
17. Explain Database → Schema → Table in PostgreSQL.
18. How do you create and connect to a PostgreSQL database?
19. Explain DDL commands.
20. Explain DML commands.
21. Explain TCL commands.
22. Explain DCL commands.
23. Differentiate DELETE, TRUNCATE, and DROP.
24. Explain COMMIT, ROLLBACK, and SAVEPOINT.
25. Explain GRANT and REVOKE.
26. Explain SQL command categories.

## Practical/SQL questions

### Identify constraints

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    age INT CHECK (age >= 18),
    city VARCHAR(50) DEFAULT 'Bhilwara'
);
```

Identify:

```text
student_id → Primary Key
name       → NOT NULL
email      → UNIQUE
age        → CHECK
city       → DEFAULT
```

### Foreign key

```sql
CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100)
);

CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    course_id INT,
    FOREIGN KEY (course_id)
        REFERENCES courses(course_id)
);
```

### DDL practice

```sql
CREATE TABLE book_store (
    book_id INT,
    book_title VARCHAR(100),
    price DECIMAL(7,2)
);
```

Add a column:

```sql
ALTER TABLE book_store
ADD COLUMN author VARCHAR(100);
```

Rename column:

```sql
ALTER TABLE book_store
RENAME COLUMN author TO book_author;
```

Change type:

```sql
ALTER TABLE book_store
ALTER COLUMN book_title TYPE VARCHAR(150);
```

Rename table:

```sql
ALTER TABLE book_store
RENAME TO books;
```

Remove column:

```sql
ALTER TABLE books
DROP COLUMN book_author;
```

---

# Final Ultra-Quick Recap

## Definitions

**Data:** Raw facts and figures.

**Information:** Processed and meaningful data.

**Insight:** Useful conclusion/pattern derived from information.

**Database:** Organized collection of related data.

**DBMS:** Software used to create, store, retrieve, update, secure, and manage databases.

**Relational database:** Database that stores data in related tables.

**Table:** Collection of related records.

**Row/Tuple:** One record.

**Column/Attribute:** A property/field.

**Entity:** Real-world object/concept being represented.

**Schema:** Logical design/structure/namespace of database objects, depending on context.

**Super Key:** Any set of attributes that uniquely identifies a row.

**Candidate Key:** Minimal super key.

**Primary Key:** Selected candidate key used as main identifier.

**Composite Key:** Key with two or more columns.

**Foreign Key:** Column(s) referencing a key in another table.

**Referential Integrity:** Ensures valid foreign-key references.

**Constraint:** Rule that restricts/validates data.

**SQL:** Language for interacting with relational database systems.

**PostgreSQL:** Open-source object-relational database management system that supports SQL.

---

## Command memory

```text
CREATE   → create object
ALTER    → modify structure
TRUNCATE → remove all rows, keep structure
DROP     → remove object

INSERT   → add rows
SELECT   → read/query rows
UPDATE   → modify rows
DELETE   → remove rows

BEGIN    → start transaction
COMMIT   → finalize transaction changes
ROLLBACK → undo uncommitted transaction changes
SAVEPOINT → create transaction checkpoint

GRANT    → give privileges
REVOKE   → remove privileges
```

## One-line Hinglish recap

> **DDL database ka structure banata/change karta hai, DML data ko manipulate karta hai, TCL transaction ko control karta hai, aur DCL permissions ko control karta hai.**

---

# Ultra-Important Memory Tricks

### Data pipeline

> **Data → Information → Insight**

### Database hierarchy

> **Database → Schema → Table → Rows/Columns**

### Key hierarchy

> **Super Key → Candidate Key → Primary Key**

### SQL categories

> **DDL = Structure**  
> **DML = Data**  
> **TCL = Transactions**  
> **DCL = Permissions**

### Destructive command ladder

> **DELETE → selected/all rows**  
> **TRUNCATE → all rows**  
> **DROP → table/object itself**

### Transaction memory

> **COMMIT = Save**  
> **ROLLBACK = Undo**  
> **SAVEPOINT = Checkpoint**

### Access control memory

> **GRANT = Give permission**  
> **REVOKE = Take permission back**

---

# End of Notes

This file combines the theory, definitions, examples, SQL syntax, comparisons, Hinglish explanations, memory tricks, exam traps, and practical patterns covered in the discussion.
