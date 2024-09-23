Guide on SQL Syntax

SQL (Structured Query Language) is the standard language for interacting with relational databases. SQL enables users to query, update, insert, and delete data within a database. Below is a comprehensive guide to understanding key SQL syntax for a self-paced learning journey.

1. **SQL Basics: SELECT Statement**

   The `SELECT` statement is used to query data from a database. It's the most frequently used SQL command.

   ```sql
   SELECT column1, column2, ...
   FROM table_name;
   ```

  - **Example:**
  ```sql
  SELECT first_name, last_name
  FROM employees;
  ```

This will retrieve the `first_name` and `last_name` columns from the `employees` table.

#### Common Clauses with SELECT:

- **WHERE**: Filters rows that satisfy a given condition.
  ```sql
  SELECT column1, column2 
  FROM table_name
  WHERE condition;
  ```

- **ORDER BY**: Sorts the result set in ascending (ASC) or descending (DESC) order.
  ```sql
  SELECT column1, column2
  FROM table_name
  ORDER BY column1 DESC;
  ```

- **LIMIT**: Restricts the number of rows returned.
  ```sql
  SELECT column1, column2
  FROM table_name
  LIMIT 10;
  ```

---

2. **SQL Filtering: WHERE Clause**

The `WHERE` clause is used to filter records based on conditions.

```sql
SELECT column1, column2
FROM table_name
WHERE condition;
```

- **Example:**
  ```sql
  SELECT first_name, last_name
  FROM employees
  WHERE department = 'Sales';
  ```

**Operators in WHERE:**
- `=`: Equal
- `!=` or `<>`: Not equal
- `>`: Greater than
- `<`: Less than
- `LIKE`: Pattern matching (e.g., `LIKE 'J%'` for values starting with "J")
- `IN`: Matches any of a set of values.
  ```sql
  SELECT first_name, last_name
  FROM employees
  WHERE department IN ('Sales', 'Marketing');
  ```

---

3. **SQL Aggregations: GROUP BY & HAVING**

SQL provides aggregate functions to perform calculations on data, such as `SUM()`, `COUNT()`, `AVG()`, `MIN()`, and `MAX()`.

- **GROUP BY**: Groups rows that have the same values into summary rows, often used with aggregate functions.

```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department;
```

This query will return the count of employees in each department.

- **HAVING**: Acts like `WHERE` but is used to filter groups after aggregating.
  ```sql
  SELECT department, COUNT(*)
  FROM employees
  GROUP BY department
  HAVING COUNT(*) > 5;
  ```
4. **SQL Joins**

Joins are used to combine rows from two or more tables based on related columns.

- **INNER JOIN**: Returns rows when there is a match in both tables.

  ```sql
  SELECT employees.first_name, departments.department_name
  FROM employees
  INNER JOIN departments
  ON employees.department_id = departments.id;
  ```

- **LEFT JOIN**: Returns all rows from the left table, even if there are no matches in the right table.

  ```sql
  SELECT employees.first_name, departments.department_name
  FROM employees
  LEFT JOIN departments
  ON employees.department_id = departments.id;
  ```

- **RIGHT JOIN**: Returns all rows from the right table, and the matched rows from the left table.
  ```sql
  SELECT employees.first_name, departments.department_name
  FROM employees
  RIGHT JOIN departments
  ON employees.department_id = departments.id;
  ```
5. **Inserting Data: INSERT INTO**

The `INSERT INTO` statement is used to add new rows into a table.

```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```

- **Example:**
  ```sql
  INSERT INTO employees (first_name, last_name, department_id)
  VALUES ('John', 'Doe', 3);
  ```
6. **Updating Data: UPDATE**

The `UPDATE` statement is used to modify existing rows in a table.

```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

- **Example:**
  ```sql
  UPDATE employees
  SET department_id = 4
  WHERE employee_id = 1;
  ```

---

7. **Deleting Data: DELETE**

The `DELETE` statement is used to remove rows from a table.

```sql
DELETE FROM table_name
WHERE condition;
```

- **Example:**
  ```sql
  DELETE FROM employees
  WHERE employee_id = 1;
  ```

8. **Creating and Altering Tables**

- **CREATE TABLE**: Used to create a new table in the database.

  ```sql
  CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    department_id INT
  );
  ```

- **ALTER TABLE**: Used to modify an existing table (e.g., add/drop columns).
  ```sql
  ALTER TABLE employees
  ADD salary DECIMAL(10, 2);
  ```


## Conclusion

Understanding SQL syntax is foundational for working with relational databases. Regular practice of querying, updating, and managing databases using SQL will improve your proficiency over time.