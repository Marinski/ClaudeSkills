---
name: sql-expert
description: "Expert SQL query writing, optimization, and database schema design with support for PostgreSQL, MySQL, SQLite, and SQL Server. Use when working with databases for: (1) Writing complex SQL queries with joins, subqueries, and window functions, (2) Optimizing slow queries and analyzing execution plans, (3) Designing database schemas with proper normalization, (4) Creating indexes and improving query performance, (5) Writing migrations and handling schema changes, (6) Debugging SQL errors and query issues"
---

# SQL Expert Skill

A comprehensive guide for writing, optimizing, and managing SQL databases across multiple database systems (PostgreSQL, MySQL, SQLite, SQL Server).

## Table of Contents

1. [Core Capabilities](#core-capabilities)
2. [Supported Database Systems](#supported-database-systems)
3. [Installation](#installation)
4. [Query Writing](#query-writing)
5. [Query Optimization](#query-optimization)
6. [Schema Design](#schema-design)
7. [Indexes and Performance](#indexes-and-performance)
8. [Migrations](#migrations)
9. [Advanced SQL Patterns](#advanced-sql-patterns)
10. [Best Practices](#best-practices)
11. [Common Pitfalls](#common-pitfalls)

---

## Core Capabilities

This skill enables you to:

- **Write complex SQL queries** with JOINs, subqueries, CTEs, and window functions
- **Optimize slow queries** using EXPLAIN plans and index recommendations
- **Design database schemas** with proper normalization (1NF, 2NF, 3NF, BCNF)
- **Create effective indexes** for query performance
- **Write database migrations** safely with rollback support
- **Debug SQL errors** and understand error messages
- **Handle transactions** with proper isolation levels
- **Work with JSON/JSONB** data types
- **Generate sample data** for testing
- **Convert between database dialects** (PostgreSQL ↔ MySQL ↔ SQLite)

---

## Supported Database Systems

### PostgreSQL
**Best for**: Complex queries, JSON data, advanced features, ACID compliance

```bash
pip install psycopg2-binary sqlalchemy
```

### MySQL/MariaDB
**Best for**: Web applications, WordPress, high-read workloads

```bash
pip install mysql-connector-python sqlalchemy
```

### SQLite
**Best for**: Local development, embedded databases, testing

```bash
pip install sqlite3  # Built into Python
```

### SQL Server
**Best for**: Enterprise applications, Windows environments

```bash
pip install pyodbc sqlalchemy
```

---

## Query Writing

### Basic SELECT Queries

```sql
-- Simple SELECT with filtering
SELECT
    column1,
    column2,
    column3
FROM
    table_name
WHERE
    condition = 'value'
    AND another_condition > 100
ORDER BY
    column1 DESC
LIMIT 10;
```

### JOINs

```sql
-- INNER JOIN
SELECT
    users.name,
    orders.order_date,
    orders.total_amount
FROM
    users
INNER JOIN
    orders ON users.id = orders.user_id
WHERE
    orders.status = 'completed';

-- LEFT JOIN (include all users, even without orders)
SELECT
    users.name,
    COUNT(orders.id) as order_count,
    COALESCE(SUM(orders.total_amount), 0) as total_spent
FROM
    users
LEFT JOIN
    orders ON users.id = orders.user_id
GROUP BY
    users.id, users.name
ORDER BY
    total_spent DESC;

-- SELF JOIN (for hierarchical data)
SELECT
    e.name as employee_name,
    m.name as manager_name
FROM
    employees e
LEFT JOIN
    employees m ON e.manager_id = m.id;
```

### Subqueries

```sql
-- Subquery in WHERE clause
SELECT
    name,
    salary
FROM
    employees
WHERE
    salary > (SELECT AVG(salary) FROM employees);

-- Subquery in FROM clause (derived table)
SELECT
    dept_stats.department,
    dept_stats.avg_salary
FROM (
    SELECT
        department,
        AVG(salary) as avg_salary,
        COUNT(*) as employee_count
    FROM
        employees
    GROUP BY
        department
) dept_stats
WHERE
    dept_stats.employee_count > 5;

-- Correlated subquery
SELECT
    e1.name,
    e1.department,
    e1.salary
FROM
    employees e1
WHERE
    e1.salary > (
        SELECT AVG(e2.salary)
        FROM employees e2
        WHERE e2.department = e1.department
    );
```

### Common Table Expressions (CTEs)

```sql
-- Basic CTE
WITH high_value_customers AS (
    SELECT
        user_id,
        SUM(total_amount) as lifetime_value
    FROM
        orders
    GROUP BY
        user_id
    HAVING
        SUM(total_amount) > 1000
)
SELECT
    users.name,
    users.email,
    hvc.lifetime_value
FROM
    users
INNER JOIN
    high_value_customers hvc ON users.id = hvc.user_id;

-- Recursive CTE (for hierarchical data)
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor member: top-level employees
    SELECT
        id,
        name,
        manager_id,
        1 as level
    FROM
        employees
    WHERE
        manager_id IS NULL

    UNION ALL

    -- Recursive member: employees reporting to previous level
    SELECT
        e.id,
        e.name,
        e.manager_id,
        eh.level + 1
    FROM
        employees e
    INNER JOIN
        employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy ORDER BY level, name;
```

### Window Functions

```sql
-- ROW_NUMBER (assign unique row numbers)
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as salary_rank
FROM
    employees;

-- RANK and DENSE_RANK
SELECT
    name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dense_rank
FROM
    employees;

-- Running totals with SUM
SELECT
    order_date,
    total_amount,
    SUM(total_amount) OVER (ORDER BY order_date) as running_total
FROM
    orders;

-- Moving averages
SELECT
    order_date,
    total_amount,
    AVG(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7days
FROM
    daily_sales;

-- LAG and LEAD (access previous/next row values)
SELECT
    order_date,
    total_amount,
    LAG(total_amount, 1) OVER (ORDER BY order_date) as prev_day_amount,
    LEAD(total_amount, 1) OVER (ORDER BY order_date) as next_day_amount,
    total_amount - LAG(total_amount, 1) OVER (ORDER BY order_date) as day_over_day_change
FROM
    daily_sales;
```

---

## Query Optimization

### Using EXPLAIN

```sql
-- PostgreSQL EXPLAIN
EXPLAIN ANALYZE
SELECT
    users.name,
    COUNT(orders.id) as order_count
FROM
    users
LEFT JOIN
    orders ON users.id = orders.user_id
GROUP BY
    users.id, users.name;

-- Look for:
-- - Seq Scan (bad) vs Index Scan (good)
-- - High cost numbers
-- - Large row counts being processed
```

### Key Performance Indicators

- **Seq Scan**: Table scan without index (slow for large tables)
- **Index Scan**: Using an index (fast)
- **Index Only Scan**: Best case - all data from index
- **Nested Loop**: Good for small datasets
- **Hash Join**: Good for larger datasets
- **Merge Join**: Good for sorted data

### Optimization Techniques

```sql
-- BAD: Using OR with different columns
SELECT * FROM users WHERE first_name = 'John' OR last_name = 'Smith';

-- GOOD: Use UNION if possible
SELECT * FROM users WHERE first_name = 'John'
UNION
SELECT * FROM users WHERE last_name = 'Smith';

-- BAD: Function on indexed column prevents index usage
SELECT * FROM users WHERE LOWER(email) = 'user@example.com';

-- GOOD: Use functional index or store lowercase
SELECT * FROM users WHERE email = LOWER('user@example.com');
-- Create index: CREATE INDEX idx_email_lower ON users(LOWER(email));

-- BAD: SELECT *
SELECT * FROM large_table WHERE id = 123;

-- GOOD: Select only needed columns
SELECT id, name, email FROM large_table WHERE id = 123;

-- BAD: Subquery in SELECT (executes for each row)
SELECT
    name,
    (SELECT COUNT(*) FROM orders WHERE user_id = users.id) as order_count
FROM
    users;

-- GOOD: Use JOIN instead
SELECT
    users.name,
    COUNT(orders.id) as order_count
FROM
    users
LEFT JOIN
    orders ON users.id = orders.user_id
GROUP BY
    users.id, users.name;
```

---

## Schema Design

### Normalization

#### First Normal Form (1NF)
- Eliminate repeating groups
- Each field contains atomic values

```sql
-- BAD: Repeating groups
CREATE TABLE orders_bad (
    order_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    product1 VARCHAR(100),
    product2 VARCHAR(100),
    product3 VARCHAR(100)
);

-- GOOD: Separate table for products
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_name VARCHAR(100)
);

CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY,
    order_id INT REFERENCES orders(order_id),
    product_name VARCHAR(100)
);
```

#### Second Normal Form (2NF)
- Meet 1NF
- All non-key attributes depend on the entire primary key

```sql
-- BAD: Product info depends only on product_id, not the composite key
CREATE TABLE order_items_bad (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100),
    product_price DECIMAL(10, 2),
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- GOOD: Separate product information
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    product_price DECIMAL(10, 2)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

#### Third Normal Form (3NF)
- Meet 2NF
- No transitive dependencies

```sql
-- BAD: city_state depends on city, not customer_id
CREATE TABLE customers_bad (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    city VARCHAR(100),
    city_state VARCHAR(2)  -- Depends on city, not customer_id
);

-- GOOD: Separate cities table
CREATE TABLE cities (
    city_id INT PRIMARY KEY,
    city_name VARCHAR(100),
    state VARCHAR(2)
);

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    city_id INT REFERENCES cities(city_id)
);
```

### Common Schema Patterns

#### One-to-Many

```sql
CREATE TABLE authors (
    author_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

CREATE TABLE books (
    book_id INT PRIMARY KEY,
    title VARCHAR(200),
    author_id INT NOT NULL,
    published_date DATE,
    FOREIGN KEY (author_id) REFERENCES authors(author_id)
);
```

#### Many-to-Many

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100)
);

-- Junction table
CREATE TABLE enrollments (
    enrollment_id INT PRIMARY KEY,
    student_id INT NOT NULL,
    course_id INT NOT NULL,
    enrollment_date DATE,
    grade CHAR(2),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id),
    UNIQUE (student_id, course_id)
);
```

#### Self-Referencing (Hierarchical)

```sql
CREATE TABLE categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(100),
    parent_category_id INT,
    FOREIGN KEY (parent_category_id) REFERENCES categories(category_id)
);
```

---

## Indexes and Performance

### Creating Indexes

```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);

-- Composite index (order matters!)
CREATE INDEX idx_orders_user_date ON orders(user_id, order_date);

-- Unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Partial index (PostgreSQL)
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';

-- Functional index
CREATE INDEX idx_users_email_lower ON users(LOWER(email));

-- Full-text search index (PostgreSQL)
CREATE INDEX idx_posts_search ON posts USING GIN(to_tsvector('english', title || ' ' || content));
```

### Index Guidelines

**When to create indexes:**
- ✅ Columns used in WHERE clauses
- ✅ Columns used in JOIN conditions
- ✅ Columns used in ORDER BY
- ✅ Foreign key columns
- ✅ Columns with high selectivity (many unique values)

**When NOT to create indexes:**
- ❌ Small tables (< 1000 rows)
- ❌ Columns with low selectivity (few unique values like boolean)
- ❌ Columns frequently updated
- ❌ Too many indexes on one table (slows INSERTs/UPDATEs)

### Index Maintenance

```sql
-- PostgreSQL: Rebuild index
REINDEX INDEX idx_users_email;

-- MySQL: Optimize table
OPTIMIZE TABLE users;

-- Check index usage (PostgreSQL)
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM
    pg_stat_user_indexes
ORDER BY
    idx_scan ASC;
```

---

## Migrations

### Migration Best Practices

```sql
-- Migration: Add new column with default
-- Step 1: Add column as nullable
ALTER TABLE users ADD COLUMN status VARCHAR(20);

-- Step 2: Populate existing rows
UPDATE users SET status = 'active' WHERE status IS NULL;

-- Step 3: Make it NOT NULL
ALTER TABLE users ALTER COLUMN status SET NOT NULL;

-- Step 4: Add default for new rows
ALTER TABLE users ALTER COLUMN status SET DEFAULT 'active';

-- Rollback plan
ALTER TABLE users DROP COLUMN status;
```

### Zero-Downtime Migrations

```sql
-- BAD: This locks the table
ALTER TABLE large_table ADD COLUMN new_column VARCHAR(100) NOT NULL DEFAULT 'value';

-- GOOD: Add column as nullable first, then backfill
ALTER TABLE large_table ADD COLUMN new_column VARCHAR(100);

-- Backfill in batches
UPDATE large_table SET new_column = 'value' WHERE new_column IS NULL LIMIT 1000;
-- Repeat until complete

-- Then make it NOT NULL
ALTER TABLE large_table ALTER COLUMN new_column SET NOT NULL;
```

### Renaming Columns Safely

```sql
-- Step 1: Add new column
ALTER TABLE users ADD COLUMN email_address VARCHAR(100);

-- Step 2: Copy data
UPDATE users SET email_address = email;

-- Step 3: Update application to use both columns
-- (Deploy application code)

-- Step 4: Drop old column (after verification)
ALTER TABLE users DROP COLUMN email;
```

---

## Advanced SQL Patterns

### UPSERT (Insert or Update)

```sql
-- PostgreSQL: ON CONFLICT
INSERT INTO users (user_id, name, email, updated_at)
VALUES (1, 'John Doe', 'john@example.com', NOW())
ON CONFLICT (user_id)
DO UPDATE SET
    name = EXCLUDED.name,
    email = EXCLUDED.email,
    updated_at = NOW();

-- MySQL: ON DUPLICATE KEY UPDATE
INSERT INTO users (user_id, name, email, updated_at)
VALUES (1, 'John Doe', 'john@example.com', NOW())
ON DUPLICATE KEY UPDATE
    name = VALUES(name),
    email = VALUES(email),
    updated_at = NOW();

-- SQLite: ON CONFLICT
INSERT INTO users (user_id, name, email, updated_at)
VALUES (1, 'John Doe', 'john@example.com', datetime('now'))
ON CONFLICT(user_id) DO UPDATE SET
    name = excluded.name,
    email = excluded.email,
    updated_at = datetime('now');
```

### Bulk Operations

```sql
-- Bulk INSERT
INSERT INTO users (name, email) VALUES
    ('Alice', 'alice@example.com'),
    ('Bob', 'bob@example.com'),
    ('Charlie', 'charlie@example.com');

-- Bulk UPDATE from another table
UPDATE products p
SET price = new_prices.price
FROM (
    VALUES
        (1, 19.99),
        (2, 29.99),
        (3, 39.99)
) AS new_prices(product_id, price)
WHERE p.product_id = new_prices.product_id;

-- Bulk DELETE with JOIN
DELETE FROM orders
WHERE order_id IN (
    SELECT order_id
    FROM orders o
    INNER JOIN users u ON o.user_id = u.user_id
    WHERE u.status = 'deleted'
);
```

### Pivot Tables

```sql
-- Transform rows to columns
SELECT
    product_name,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 1 THEN quantity ELSE 0 END) as jan,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 2 THEN quantity ELSE 0 END) as feb,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 3 THEN quantity ELSE 0 END) as mar
FROM
    order_items oi
INNER JOIN
    products p ON oi.product_id = p.product_id
GROUP BY
    product_name;

-- PostgreSQL crosstab (requires tablefunc extension)
CREATE EXTENSION IF NOT EXISTS tablefunc;

SELECT * FROM crosstab(
    'SELECT product_name, month, total_quantity
     FROM monthly_sales
     ORDER BY 1, 2',
    'SELECT DISTINCT month FROM monthly_sales ORDER BY 1'
) AS ct(product_name TEXT, jan INT, feb INT, mar INT);
```

### JSON Operations (PostgreSQL)

```sql
-- Query JSON data
SELECT
    user_id,
    preferences->>'theme' as theme,
    preferences->>'language' as language
FROM
    users
WHERE
    preferences->>'notifications' = 'true';

-- Update JSON field
UPDATE users
SET preferences = jsonb_set(
    preferences,
    '{theme}',
    '"dark"'
)
WHERE user_id = 123;

-- JSON aggregation
SELECT
    department,
    jsonb_agg(jsonb_build_object(
        'name', name,
        'salary', salary
    )) as employees
FROM
    employees
GROUP BY
    department;
```

---

## Best Practices

### 1. Always Use Parameterized Queries
```python
# BAD: SQL injection vulnerable
query = f"SELECT * FROM users WHERE email = '{user_input}'"

# GOOD: Parameterized query
query = "SELECT * FROM users WHERE email = %s"
cursor.execute(query, (user_input,))
```

### 2. Use Transactions for Related Operations
```sql
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;
-- Or ROLLBACK if something goes wrong
```

### 3. Add Appropriate Constraints
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT NOT NULL,
    order_date DATE NOT NULL DEFAULT CURRENT_DATE,
    total_amount DECIMAL(10, 2) CHECK (total_amount >= 0),
    status VARCHAR(20) CHECK (status IN ('pending', 'completed', 'cancelled')),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

### 4. Use VARCHAR Instead of CHAR for Variable-Length Strings
```sql
-- BAD: Wastes space
CREATE TABLE users (name CHAR(100));

-- GOOD: Only uses needed space
CREATE TABLE users (name VARCHAR(100));
```

### 5. Include Timestamps
```sql
CREATE TABLE posts (
    post_id INT PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### 6. Use Meaningful Names
```sql
-- BAD
CREATE TABLE t1 (id INT, n VARCHAR(100));

-- GOOD
CREATE TABLE customers (customer_id INT, customer_name VARCHAR(100));
```

---

## Common Pitfalls

### 1. N+1 Query Problem
```python
# BAD: N+1 queries (1 + N queries total)
users = db.query("SELECT * FROM users")
for user in users:
    orders = db.query("SELECT * FROM orders WHERE user_id = ?", user.id)
    # This executes a query for EACH user

# GOOD: Single query with JOIN
result = db.query("""
    SELECT
        users.*,
        orders.*
    FROM users
    LEFT JOIN orders ON users.id = orders.user_id
""")
```

### 2. Not Using LIMIT
```sql
-- BAD: Returns all rows (could be millions)
SELECT * FROM large_table WHERE status = 'active';

-- GOOD: Limit results for exploratory queries
SELECT * FROM large_table WHERE status = 'active' LIMIT 100;
```

### 3. Implicit Type Conversions
```sql
-- BAD: String comparison on INT column prevents index usage
SELECT * FROM users WHERE user_id = '123';

-- GOOD: Use correct type
SELECT * FROM users WHERE user_id = 123;
```

### 4. Using COUNT(*) When You Just Need EXISTS
```sql
-- BAD: Counts all rows
SELECT COUNT(*) FROM orders WHERE user_id = 123;

-- GOOD: Just check existence
SELECT EXISTS(SELECT 1 FROM orders WHERE user_id = 123);
```

### 5. Not Handling NULLs Properly
```sql
-- BAD: NULL comparisons always return NULL (not TRUE or FALSE)
SELECT * FROM users WHERE deleted_at = NULL;  -- Returns no rows!

-- GOOD: Use IS NULL / IS NOT NULL
SELECT * FROM users WHERE deleted_at IS NULL;
```

### 6. Using SELECT DISTINCT Instead of Fixing the Query
```sql
-- BAD: Band-aid solution
SELECT DISTINCT user_id, name FROM users
JOIN orders ON users.id = orders.user_id;

-- GOOD: Fix the underlying issue
SELECT users.id, users.name FROM users
WHERE EXISTS (SELECT 1 FROM orders WHERE orders.user_id = users.id);
```

---

## Helper Scripts

See `scripts/sql_helper.py` for utility functions including:
- Query builder with parameterization
- Schema introspection
- Index analysis
- Query execution timing
- Migration helpers
- Sample data generation

## Examples

See `examples/` directory for:
- Complex query examples
- Schema design patterns
- Migration scripts
- Performance optimization examples
