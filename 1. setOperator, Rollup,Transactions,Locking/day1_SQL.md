sql-01\day01\day01.md
                
# Advanced SQL Primer

## Agenda
* Transactions

* SET sql_mode='STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ONLY_FULL_GROUP_BY';
* SELECT @@sql_mode;

## Getting Started

```SQL
-- root login
SELECT user, host FROM mysql.user;

CREATE USER sunbeam@localhost IDENTIFIED BY 'sunbeam';

SHOW DATABASES;

-- CREATE DATABASE classwork;
-- if not created.

GRANT ALL ON classwork.* TO sunbeam@localhost;

FLUSH PRIVILEGES;

EXIT;
```

```SQL
-- sunbeam user login

SHOW DATABASES;

USE classwork;

SHOW TABLES;

SOURCE D:/onlinecourses/sql-01/db/classwork-db.sql

SHOW TABLES;

SELECT * FROM books;

SELECT * FROM emp;
```

## Set Operators
* UNION operator is to combine results of two SELECT queries. Both queries must have same number of columns.
* It automatically delete duplicate records.
* To retain duplicate records use UNION ALL operator.

```SQL
SELECT deptno, SUM(sal) FROM emp
GROUP BY deptno;

SELECT SUM(sal) FROM emp;

SELECT NULL, SUM(sal) FROM emp;

(SELECT deptno, SUM(sal) FROM emp
GROUP BY deptno)
UNION
(SELECT NULL, SUM(sal) FROM emp);
```

```SQL
SELECT deptno, SUM(sal) FROM emp
GROUP BY deptno
WITH ROLLUP;

SELECT deptno, job, COUNT(empno), SUM(sal) FROM emp
GROUP BY deptno, job;

SELECT deptno, job, COUNT(empno), SUM(sal) FROM emp
GROUP BY deptno, job
WITH ROLLUP;

SELECT deptno, job, COUNT(empno), SUM(sal) FROM emp
GROUP BY job, deptno
WITH ROLLUP;

(SELECT deptno, job, COUNT(empno), SUM(sal) FROM emp
GROUP BY deptno, job
WITH ROLLUP)
UNION
(SELECT deptno, job, COUNT(empno), SUM(sal) FROM emp
GROUP BY job, deptno
WITH ROLLUP);

SELECT deptno, job, COUNT(empno), SUM(sal) FROM emp
GROUP BY CUBE(deptno, job);
-- shows all possible combinations of sub-totals & grand-totals.
-- not supported in MySQL. Supported in Oracle.

SELECT deptno, job, COUNT(empno), SUM(sal), GROUPING(deptno), GROUPING(job) FROM emp
GROUP BY deptno, job
WITH ROLLUP;

SELECT deptno, job, COUNT(empno), SUM(sal), GROUPING(deptno), GROUPING(job) FROM emp
GROUP BY deptno, job
WITH ROLLUP
HAVING  GROUPING(job) = 1;
```

## Transactions

```SQL
CREATE TABLE accounts (id INT, type CHAR(20), amount DOUBLE);

INSERT INTO accounts VALUES(1, 'Saving', 10000);
INSERT INTO accounts VALUES(2, 'Saving', 2000);
INSERT INTO accounts VALUES(3, 'Saving', 5000);
INSERT INTO accounts VALUES(4, 'Saving', 3000);

SELECT * FROM accounts;
```

```SQL
START TRANSACTION;

UPDATE accounts SET amount = amount - 1000 WHERE id = 1;

UPDATE accounts SET amount = amount + 1000 WHERE id = 2;

SELECT * FROM accounts;

COMMIT;
```

```SQL
START TRANSACTION;

UPDATE accounts SET amount = amount - 1000 WHERE id = 1;

UPDATE accounts SET amount = amount + 1000 WHERE id = 3;

SELECT * FROM accounts;

ROLLBACK;

SELECT * FROM accounts;
```

```SQL
START TRANSACTION;

UPDATE accounts SET amount = amount - 1000 WHERE id = 1;

EXIT;

-- relogin to classwork 
-- mysql -u sunbeam -psunbeam classwork

SELECT * FROM accounts;
```

```SQL
START TRANSACTION;

UPDATE accounts SET amount = amount - 1000 WHERE id = 1;

UPDATE accounts SET amount = amount + 1000 WHERE id = 3;

DROP TABLE dummy;
-- auto-commit the current transaction.

SELECT * FROM accounts;
-- see updated results

ROLLBACK;
-- nothing to discard (changes already committed).

SELECT * FROM accounts;
```

```SQL
START TRANSACTION;

INSERT INTO accounts VALUES (5, 'Saving', 2000);
INSERT INTO accounts VALUES (6, 'Saving', 2000);
SELECT * FROM accounts;

SAVEPOINT sa1;

INSERT INTO accounts VALUES (7, 'Saving', 2000);
INSERT INTO accounts VALUES (8, 'Saving', 2000);
SELECT * FROM accounts;

SAVEPOINT sa2;

INSERT INTO accounts VALUES (9, 'Saving', 2000);
INSERT INTO accounts VALUES (10, 'Saving', 2000);
SELECT * FROM accounts;

ROLLBACK TO sa1;

SELECT * FROM accounts;

INSERT INTO accounts VALUES (11, 'Saving', 2000);
INSERT INTO accounts VALUES (12, 'Saving', 2000);

COMMIT;

SELECT * FROM accounts;
```

```SQL
SELECT @@autocommit;
-- 1

SELECT * FROM accounts;

DELETE FROM accounts WHERE id > 4;
-- records are immediately deleted, because this query is executed in tx and it is committed.

SELECT * FROM accounts;
```

```SQL
SET autocommit = 0;
-- automatically one tx is created by the database

SELECT @@autocommit;
-- 0

DELETE FROM accounts WHERE id > 2;

SELECT * FROM accounts;

ROLLBACK;
-- discard DML operations in current transaction
-- automatically one new tx is created by the database

SELECT * FROM accounts;
-- get old state back

DELETE FROM accounts WHERE id > 3;

SELECT * FROM accounts;

COMMIT;
-- changes in tx are done perment
-- automatically one new tx is created by the database

SELECT * FROM accounts;

SET autocommit = 1;

SELECT @@autocommit;
-- 1
```

* Login into two different terminals (with different users or same users).
	* terminal1> mysql -u sunbeam -p classwork
	* terminal2> mysql -u root -p classwork

```SQL
-- sunbeam & root user
SELECT USER(), DATABASE();

SELECT id, name, price FROM books;

-- root user
START TRANSACTION;

UPDATE books SET price = 125 WHERE id = 1001;

SELECT  id, name, price FROM books WHERE id = 1001;

-- sunbeam user
SELECT  id, name, price FROM books WHERE id = 1001;

-- root user
COMMIT;

SELECT  id, name, price FROM books WHERE id = 1001;

-- sunbeam user
SELECT  id, name, price FROM books WHERE id = 1001;
```

```SQL
-- root user
START TRANSACTION;

UPDATE books SET price = 130 WHERE id = 1001;

SELECT  id, name, price FROM books WHERE id = 1001;

-- sunbeam user
SELECT  id, name, price FROM books WHERE id = 1001;

UPDATE books SET price = 140 WHERE id = 1001;
-- blocked

-- root user
COMMIT;

-- root & sunbeam user
SELECT  id, name, price FROM books WHERE id = 1001;
-- 140
```

```SQL
-- root user
START TRANSACTION;

UPDATE books SET price = 150 WHERE id = 1001;

SELECT  id, name, price FROM books WHERE id = 1001;

-- sunbeam user
SELECT  id, name, price FROM books WHERE id = 1001;

DELETE FROM books WHERE id = 1001;
-- blocked

-- root user
ROLLBACK;

-- sunbeam user tx will be unblocked and record deleted.

-- root & sunbeam user
SELECT  id, name, price FROM books WHERE id = 1001;
-- no row found
```

```SQL
-- root user
START TRANSACTION;

DELETE FROM books WHERE id = 1002;

SELECT  id, name, price FROM books WHERE id = 1002;

-- sunbeam user
SELECT  id, name, price FROM books WHERE id = 1002;

UPDATE books SET price = 300 WHERE id = 1002;
-- blocked

-- root user
COMMIT;

-- sunbeam user tx will be unblocked and record will not be updated, because it is already deleted in root tx.
-- note that, the record would have updated, if root user tx is ROLLBACK.
```

```SQL
DESCRIBE books;
-- table has PK and hence internally table is indexed.
```

```SQL
-- root user
START TRANSACTION;

SELECT * FROM books WHERE id = 2001 FOR UPDATE;
-- pessimistic locking

UPDATE books SET price = 300 WHERE id = 2001;

-- sunbeam user
SELECT * FROM books WHERE id = 2001 FOR UPDATE;
-- blocked until root tx is completed or time-out

-- root user
COMMIT;

```
