# DBMS-Queries

## Table of Contents

1. [Basic SQL Concepts](#basic-sql-concepts)
2. [General Query Examples](#general-query-examples)
3. [Company Database Queries](#company-database-queries)
4. [Sailor-Boat-Reserve Database Queries](#sailor-boat-reserve-database-queries)
5. [SQL Functions](#sql-functions)
6. [Stored Procedures](#stored-procedures)
7. [Triggers](#triggers)
8. [Views](#views)
9. [User Roles and Permissions](#user-roles-and-permissions)

---

## Basic SQL Concepts

### Database Management

```sql
-- Display all databases
show databases;

-- Know the current database
select database();

-- Create a database
create database classi;

-- Get into the database
use classi;
```

### Table Creation and Basic DDL

```sql
-- Create table commands
create table faculty(
    fid integer,
    fname varchar(10) not null,
    salary numeric(10,2) default 1000,
    primary key(fid)
);

create table student(
    sid integer,
    sname varchar(10) not null,
    sage integer,
    smarks integer not null,
    fam integer,
    primary key(sid),
    foreign key(fam) references faculty(fid)
);
```

### Basic DML Operations

```sql
-- Insert few values
insert into faculty values(10,'ram',20202);
insert into faculty values(11,'ramya',21102);
insert into faculty values(12,'shruthi',7102);

-- Know the structure of the table
desc student; -- or describe student;
show create table student;

-- Note: Duplicate primary key 10, this will fail if executed after the first insert
insert into faculty values(10,'rama',29292);
insert into faculty values(11,'rahul',1295);

-- Select operations
select * from faculty;

insert into student values(100,'ronit',21,23,10);
insert into student values(101,'madhu',22,32,11);
insert into student values(102,'sia',22,40,10);
insert into student values(103,'alice',23,23,10);
insert into student values(104,'ramya',20,24,11);

select * from student;
select * from student,faculty; -- cartesian product

select sname from student;

-- Equi join
select * from student,faculty where fam=fid;
```

### Table Operations (AS and LIKE)

```sql
-- Create table as select
create table new_student as select * from student;
create table new1_student like student;

-- Describe and explore both new_student and new1_student to differentiate between as and like
-- Try to insert/del/modify data into original table and check if its reflected in new table
```

### Drop and Truncate Tables

```sql
-- To drop the table--delete's the entire content along with structure
drop table tablename;
drop table new_student;
```

### ALTER TABLE Operations

```sql
-- To add column--email_id to faculty table
alter table faculty add column email_id varchar(10);

-- To rename column smarks to isa_marks in student table
alter table student rename column smarks to isa_marks;

-- To change the datatype of a column
alter table student modify isa_marks numeric(10,2);

-- To add a column
alter table student add temp integer;
-- To drop a column
alter table student drop temp;

-- Rename a table
rename table student to students;

-- Truncate table and drop table
truncate table new_student; -- Assuming new_temp was a typo for new_student
drop table new1_student; -- Assuming temp was a typo for new1_student
```

### Advanced ALTER TABLE Operations

```sql
-- ADD one column in the table
alter table faculty add column emailid varchar(10);
alter table faculty add column emailid varchar(10) first;
alter table faculty add column emailid varchar(10) after fid;

-- ADD multiple column in the table
alter table faculty add column phone integer,add column address varchar(10);

-- Modify column definition
alter table faculty modify column emailid varchar(30);
alter table faculty modify emailid varchar(35) after phone;//both def and position

-- Drop column
alter table faculty drop column phone;
-- or
alter table faculty drop phone;

-- Rename column--need to specify col definiton also
alter table faculty change column emailid email_id varchar(30); -- emamilid was a typo

-- To change default value
alter table faculty alter fname set default 'xxx';

-- To drop default value
alter table faculty alter fname drop default;

-- To set/add default value
alter table faculty alter fname set default 'abc';

-- Cannot drop primary key because it's referenced by student table
alter table faculty drop primary key;

-- To drop primary key
alter table students drop primary key;
-- To add
alter table students add primary key(sid);

-- To drop foreign key
alter table students drop foreign key student_ibfk_1;--- will not work--use constraint name from show create table statement
```

### INSERT Operations

```sql
-- To insert values
insert into faculty values(15,'abc','dfsf',3434);--all values
insert into faculty(fid,fname) values(16,'xyz');--few values

-- To delete
delete from faculty;---all rows;
delete from faculty where fid=101;
```

### Database Creation for Company Schema

```sql
-- To create company database
create database company;
use company;
source 'c:/path/to/the/file/new_company.sql';
```

### Basic Company Database Queries

```sql
select fname,lname from employee;
select fname,lname from employee where dno=5;
select fname,lname,dname from employee,department;
select fname,lname,dname from employee,department where dno=dnumber;
select fname,lname,dname from employee,department where dno=dnumber and dname='Research';
```

### Queries with Dependents

```sql
-- List the names of the employees who have dependents
select fname from employee,dependent where ssn=essn;
select distinct fname from employee,dependent where ssn=essn;

-- List the names of the employees who have dependents along with their dependents name and relationship
select fname,dependent_name,relationship from employee,dependent where ssn=essn;
```

### Arithmetic Operations and NULL Handling

```sql
-- Arithmetic operators
select ssn,salary,salary*2 from employee;
select ssn,salary,salary+1000 from employee;

-- Check how null behaves when arithmetic operation is used
select fid,fname,salary,salary+100 as inc_sal from faculty;

mysql> select fid,fname,salary,salary+100 as inc_sal from faculty;
+-----+-------+---------+---------+
| fid | fname | salary  | inc_sal |
+-----+-------+---------+---------+
|  10 | fam   | 1000.00 | 1100.00 |
|  11 | fam   | 2000.00 | 2100.00 |
|  12 | fam2  |    NULL |    NULL |
+-----+-------+---------+---------+
3 rows in set (0.00 sec)

mysql> update faculty set salary=salary+100;
Query OK, 2 rows affected (0.01 sec)
Rows matched: 3  Changed: 2  Warnings: 0

mysql> select fid,fname,salary,salary+100 as inc_sal from faculty;
+-----+-------+---------+---------+
| fid | fname | salary  | inc_sal |
+-----+-------+---------+---------+
|  10 | fam   | 1100.00 | 1200.00 |
|  11 | fam   | 2100.00 | 2200.00 |
|  12 | fam2  |    NULL |    NULL |
+-----+-------+---------+---------+
```

### Complex Joins

```sql
-- Retrieve fname,lname,dname of employees
select fname,lname,dname from employee,department where dno=dnumber;

-- Retrieve details of the dept along with their manager details
select dnumber,dname,mgr_ssn,fname from employee,department where dno=dnumber;
-- The above query is incorrect
-- Correct query is:
select dnumber,dname,mgr_ssn,fname from employee,department where mgr_ssn=ssn;

-- Retrieve ssn,fname,pno,pname of the employees
select ssn,fname,pno,pname
from employee,works_on,project
where ssn=essn and pno=pnumber;

-- Retrieve the project details along with their controlling department
select pno,pname,dnumber,dname from project,department where dnum=dnumber;

-- For every project located in 'Stafford', list the project number, the controlling department number,
-- and the department manager's last name, address, and birth date.
SELECT Pnumber, Dnum, Lname, Address, Bdate
FROM PROJECT, DEPARTMENT, EMPLOYEE
WHERE Dnum = Dnumber AND Mgr_ssn = Ssn AND Plocation = 'Stafford';

-- Above query with dept location
SELECT Pnumber, Dnum, Lname, Address, Bdate, dlocation
FROM PROJECT, DEPARTMENT, EMPLOYEE,dept_locations
WHERE Dnum = Dnumber AND Mgr_ssn = Ssn
AND dept_locations.dnumber=department.dnumber and Plocation = 'Stafford';

SELECT Pnumber, Dnum, Lname, Address, Bdate, dlocation,plocation
FROM PROJECT p, DEPARTMENT d, EMPLOYEE e,dept_locations ds
WHERE
    Dnum = d.dnumber AND Mgr_ssn = Ssn
AND ds.dnumber=d.dnumber and Plocation = 'Stafford';

-- Above query with dept location
SELECT Pnumber, Dnum, Lname,Bdate, ds.dlocation,plocation
FROM PROJECT p, DEPARTMENT d, EMPLOYEE e,dept_locations ds
WHERE
    Dnum = d.dnumber AND Mgr_ssn = Ssn
AND ds.dnumber=d.dnumber and Plocation = 'Sugarland';
```

### String Comparisons and NULL Handling

```sql
-- String comparison
select fname,lname,bdate from employee where bdate like '196_______';

-- Issue: null cannot be compared with = instead use is
select ssn,fname,super_ssn from employee where super_ssn = null;
select ssn,fname,super_ssn from employee where super_ssn is null;

select ssn,super_ssn from employee where super_ssn is not null;
-- The above query lists all the employees with supervisor.
```

### Aggregate Functions

```sql
-- Aggregate functions
select sum(salary) as total,min(salary),avg(salary) from employee where dno=5;

-- Count demo
select count(ssn) from employee;
select count(*) from employee;
select count(super_ssn) from employee;

-- Count of employees working for research dept
select count(*) from employee join department on dno=dnumber where dname='Research';

-- Find the second highest salary
SELECT MAX(salary)
FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);

-- Find employees who started managing the dept in the year 1988
select * from department where year(mgr_start_date)=1988;

-- Display department-wise average salary
SELECT dno, AVG(salary) AS avg_salary FROM employee group by dno;
```

### JOIN Operations

```sql
-- Inner Join - Equi join
select * from students join faculty on fam=fid;
+-----+-------+------+--------+-----------+------+-----+--------+----------+
| sid | sname | sage | sgrade | isa_marks | fam  | fid | fname  | salary   |
+-----+-------+------+--------+-----------+------+-----+--------+----------+
| 104 | ra    |   22 | a      |     22.00 |   11 |  11 | sowmya | 22244.00 |
| 105 | ssra  |   22 | a      |     22.00 |   12 |  12 | fam2   |     NULL |
+-----+-------+------+--------+-----------+------+-----+--------+----------+

-- Left Outer join
select * from students left outer join faculty on fam=fid;
+-----+-------+------+--------+-----------+------+------+--------+----------+
| sid | sname | sage | sgrade | isa_marks | fam  | fid  | fname  | salary   |
+-----+-------+------+--------+-----------+------+------+--------+----------+
| 102 | radha |   22 | a      |     22.00 | NULL | NULL | NULL   |     NULL |
| 103 | ramya |   22 | a      |     22.00 | NULL | NULL | NULL   |     NULL |
| 104 | ra    |   22 | a      |     22.00 |   11 |   11 | sowmya | 22244.00 |
| 105 | ssra  |   22 | a      |     22.00 |   12 |  12 | fam2   |     NULL |
+-----+-------+------+--------+-----------+------+------+--------+----------+
4 rows in set (0.00 sec)

-- Right outer join
mysql> select * from students right outer join faculty on fam=fid;
+------+-------+------+--------+-----------+------+-----+--------+----------+
| sid  | sname | sage | sgrade | isa_marks | fam  | fid | fname  | salary   |
+------+-------+------+--------+-----------+------+-----+--------+----------+
| NULL | NULL  | NULL | NULL   |      NULL | NULL |  10 | ramya  | 22244.00 |
|  104 | ra    |   22 | a      |     22.00 |   11 |  11 | sowmya | 22244.00 |
|  105 | ssra  |   22 | a      |     22.00 |   12 |  12 | fam2   |     NULL |
| NULL | NULL  | NULL | NULL   |      NULL | NULL |  13 | pushpa | 22244.00 |
+------+-------+------+--------+-----------+------+-----+--------+----------+

-- Full outer join
select * from students left outer join faculty on fam=fid union select * from students right outer join faculty on fam=fid;
+------+-------+------+--------+-----------+------+------+--------+----------+
| sid  | sname | sage | sgrade | isa_marks | fam  | fid  | fname  | salary   |
+------+-------+------+--------+-----------+------+------+--------+----------+
|  102 | radha |   22 | a      |     22.00 | NULL | NULL | NULL   |     NULL |
|  103 | ramya |   22 | a      |     22.00 | NULL | NULL | NULL   |     NULL |
|  104 | ra    |   22 | a      |     22.00 |   11 |   11 | sowmya | 22244.00 |
|  105 | ssra  |   22 | a      |     22.00 |   12 |  12 | fam2   |     NULL |
| NULL | NULL  | NULL | NULL   |      NULL | NULL |   10 | ramya  | 22244.00 |
| NULL | NULL  | NULL | NULL   |      NULL | NULL |   13 | pushpa | 22244.00 |
+------+-------+------+--------+-----------+------+------+--------+----------+

-- Retrieve employees who earn more than their supervisor
SELECT e.ssn, e.fname FROM employee e JOIN employee m ON e.super_ssn = m.ssn
WHERE e.salary > m.salary;

-- Get top 3 salaries from each department
SELECT dno, fname, salary
FROM (
    SELECT dno, fname, salary,
        RANK() OVER (PARTITION BY dno ORDER BY salary DESC) AS rnk
    FROM employee
) t
WHERE rnk <= 3;

-- Retrieve the names of all employees who have two or more dependents
SELECT Lname, Fname FROM EMPLOYEE
WHERE ( SELECT COUNT(*) FROM DEPENDENT WHERE Ssn = Essn ) >= 2;

-- Alternative query for employees with 2+ dependents
SELECT e.fname, e.lname
FROM employee e
JOIN dependent d ON e.ssn = d.essn
GROUP BY e.ssn, e.fname
HAVING COUNT(d.dependent_name) >= 2;
```

---

## General Query Examples

### Basic Aggregate Queries

```sql
-- Retrieve max salary without max function
select salary from employee where salary>=all(select salary from employee);

-- Retrieve ssn of the employees who are working for one of the proj as '123456789'
select distinct essn from works_on where pno in(select pno from works_on where essn='123456789');
```

### Project Involvement Queries

```sql
-- Retrieve the list of project numbers of projects that have an employee with the last name 'Smith'
-- involved either as a manager or as a worker

-- Q1: SELECT Pno FROM WORKS_ON, EMPLOYEE WHERE Essn = Ssn AND Lname = 'Smith';

-- Q2: SELECT Pnumber
--     FROM PROJECT, DEPARTMENT, EMPLOYEE
--     WHERE Dnum = Dnumber AND Mgr_ssn = Ssn AND Lname = 'Smith';

-- Final query:
SELECT DISTINCT Pnumber FROM PROJECT WHERE Pnumber IN (SELECT Pno FROM WORKS_ON, EMPLOYEE WHERE Essn = Ssn AND Lname = 'Smith')
or pnumber in
(SELECT Pnumber FROM PROJECT, DEPARTMENT, EMPLOYEE WHERE Dnum = Dnumber AND Mgr_ssn = Ssn AND Lname = 'Smith');
```

### Workload Comparison Queries

```sql
-- Retrieve the ssn of all employees who work on the same (project, hours)
-- combination on some project that employee 'John Smith' (ssn = '123456789') works on.

SELECT DISTINCT Essn FROM WORKS_ON
WHERE (Pno, Hours) IN
( SELECT Pno, Hours FROM WORKS_ON
WHERE Essn = '123456789' );

-- Add data so that any other employee has same workload
```

### Salary Comparison Queries

```sql
-- Retrieve the last and first names of all employees whose salary is greater than
-- the salary of all the employees in department 5

SELECT Lname, Fname,salary FROM EMPLOYEE WHERE Salary > ALL(SELECT Salary FROM EMPLOYEE WHERE Dno = 5 );
-- or
SELECT Lname, Fname,salary FROM EMPLOYEE WHERE Salary >(SELECT max(salary) FROM EMPLOYEE WHERE Dno = 5 );

-- Retrieve the last and first names of all employees whose salary is greater than
-- the salary of at least one employee in department 5

SELECT Lname, Fname FROM EMPLOYEE WHERE Salary >SOME(SELECT Salary FROM EMPLOYEE WHERE Dno = 5 );

or use any:
SELECT Lname, Fname FROM EMPLOYEE WHERE Salary >any(SELECT Salary FROM EMPLOYEE WHERE Dno = 5 );
```

### Dependent-Related Queries

```sql
-- Retrieve the name of each employee who has a dependent of the same gender as the employee

SELECT E.Fname, E.Lname FROM EMPLOYEE AS E WHERE E.Ssn IN
 ( SELECT D.Essn FROM DEPENDENT AS D  WHERE E.Gender = D.Gender );

-- Equivalent query using join
SELECT E.Fname, E.Lname FROM EMPLOYEE AS E, DEPENDENT AS D WHERE E.Ssn = D.Essn AND E.Gender = D.Gender;

-- Using exists
SELECT Fname, Lname FROM EMPLOYEE AS E
WHERE EXISTS
( SELECT * FROM DEPENDENT AS D
WHERE E.Ssn = D.Essn AND E.Gender = D.Gender);
```

### Employee-Dependent Relationship Queries

```sql
-- Retrieve the names of employees who do not have dependents
select fname from employee where ssn not in (select essn from dependent);

-- or

select fname, lname from employee where ssn in(select ssn from employee except select essn from dependent);

-- or

SELECT Fname, Lname FROM EMPLOYEE WHERE NOT EXISTS
( SELECT * FROM DEPENDENT WHERE Ssn = Essn );

-- List the names of managers who have at least one dependent
select fname,lname from employee,department where ssn=mgr_ssn and ssn in (select essn from dependent);

-- or

SELECT Fname, Lname FROM EMPLOYEE WHERE
EXISTS ( SELECT * FROM DEPENDENT WHERE Ssn = Essn )
AND EXISTS ( SELECT * FROM DEPARTMENT WHERE Ssn = Mgr_ssn );

-- For each EMPLOYEE tuple, the first nested query selects all DEPENDENT tuples related to the EMPLOYEE tuple.
-- The second nested query selects all DEPARTMENT tuples managed by EMPLOYEE tuple.
-- The EMPLOYEE tuple is selected if both the nested queries return at least one tuple each.

-- or

select distinct fname,lname from employee,department,dependent where ssn=mgr_ssn and ssn=essn;
```

### Universal Quantification Queries

```sql
-- Retrieve the name of each employee who works on all the projects controlled by department number 4

SELECT Fname, Lname FROM EMPLOYEE WHERE NOT EXISTS
 ( ( SELECT Pnumber FROM PROJECT WHERE Dnum = 4)
EXCEPT
   ( SELECT Pno FROM WORKS_ON WHERE Ssn = Essn) );

-- or
SELECT Fname, Lname FROM EMPLOYEE
WHERE NOT EXISTS
( SELECT * FROM WORKS_ON B WHERE ( B.Pno IN ( SELECT Pnumber FROM PROJECT WHERE Dnum = 4 ) 	AND NOT EXISTS ( SELECT * FROM WORKS_ON C
WHERE C.Essn = Ssn AND C.Pno = B.Pno )));
```

### Basic Selection Queries

```sql
-- Retrieve the Social Security numbers of all employees who work on project numbers 1, 2, or 3.
SELECT DISTINCT Essn FROM WORKS_ON WHERE Pno IN (1, 2, 3);
```

### Aggregate and Grouping Queries

```sql
-- Find the average employee salary of those departments where the average salary is greater than $32,000.

SELECT Dno, AVG(Salary) as avg_salary
FROM EMPLOYEE
GROUP BY Dno
HAVING AVG(Salary) > 32000;

-- With round function
SELECT Dno, ROUND(AVG(Salary),2)
FROM EMPLOYEE
GROUP BY Dno
HAVING AVG(Salary) > 32000;

-- or

select dno, avg_salary from(select dno, avg(salary) as avg_salary from employee group by dno) as dept_avg where avg_salary>32000.00;

-- Find the maximum of total employee salaries of each department across all departments.

SELECT MAX(tot_salary)
FROM (SELECT Dno, SUM(Salary) FROM EMPLOYEE GROUP BY Dno)
AS dept_total(Dnumber, tot_salary);
-- or

SELECT dno, SUM(salary) FROM employee GROUP BY dno ORDER BY SUM(salary) DESC LIMIT 1;
```

### Complex Aggregate Queries

```sql
-- Find the ssn of the employee who works for the highest number of hours on a particular project.
-- Display the project number and the number of hours the employee has worked on that project.

select essn,pno,hours from works_on where hours=(select max(hours) from works_on);

WITH max_work(max_hours) AS
( SELECT MAX(Hours) FROM WORKS_ON)
SELECT Essn, Pno, Hours
FROM WORKS_ON, max_work
WHERE Hours = max_hours;

-- Without using with--a nested query for above
SELECT Essn, Pno, Hours
FROM WORKS_ON
WHERE Hours = (SELECT MAX(Hours) FROM WORKS_ON);

-- Find all departments where the total salary is greater than the average of the total salary at all departments.

WITH
dept_total(Dno,tot_salary)
AS(SELECT Dno, SUM(Salary) FROM EMPLOYEE GROUP BY Dno),
dept_total_avg(avg_tot_salary) AS (SELECT AVG(tot_salary) FROM dept_total)
SELECT Dname, Dnumber FROM dept_total, dept_total_avg, DEPARTMENT
WHERE tot_salary > avg_tot_salary AND Dnumber = Dno;

-- Above query split up:

-- SELECT Dno, SUM(Salary) FROM EMPLOYEE GROUP BY Dno;
-- output table is dept_total with attributes Dno,tot_salary

-- SELECT AVG(tot_salary) FROM dept_total;
-- output table dept_total_avg with attribute avg_tot_salary -Uses dept_total-table of previous query

-- SELECT Dname, Dnumber FROM dept_total, dept_total_avg, DEPARTMENT
-- WHERE tot_salary > avg_tot_salary AND Dnumber = Dno; --final query uses the above 2 tables
```

### CTE (Common Table Expression) Examples

```sql
-- Sample query for with clause
-- Find employees without dependents using "with-clause"

with xyz as (select ssn from employee except select essn from dependent) select ssn,fname from xyz natural join employee;

WITH high_salary_employees
AS
(
    SELECT * FROM employee WHERE salary > 50000
)
SELECT * FROM high_salary_employees;


WITH dept_total(Dno,tot_salary)
AS
(SELECT Dno, SUM(Salary) FROM EMPLOYEE GROUP BY Dno),
dept_total_avg(avg_tot_salary) AS (SELECT AVG(tot_salary) FROM dept_total)
SELECT Dname, Dnumber FROM dept_total, dept_total_avg, DEPARTMENT
WHERE tot_salary > avg_tot_salary AND Dnumber = Dno;
```

### JOIN Operations and Examples

```sql
-- Create sample tables for join examples
create table student(sid integer, sname varchar(8),fam integer,primary key(sid));
create table faculty(fid integer,fname varchar(10),primary key(fid));
insert into faculty values(101,'A'),(102,'B'),(103,'C');
alter table student add foreign key(fam) references faculty(fid);
insert into student values(1,'AA',101),(2,'BB',102),(3,'CC',null),(4,'DD',101),(5,'EE',102);

-- Cartesian product
select * from student,faculty;

-- Cross join (same as cartesian)
select * from student join faculty;

-- Inner join with condition
select * from student,faculty where fam=fid;

-- Explicit inner join
select * from student join faculty on fam=fid;

-- Join variations
select * from student join faculty on fam=fid;
select * from student join faculty on fam<>fid;

-- Left outer join - can be used to get all employee's info along with their dependent info
select * from student left outer join faculty on fam=fid;

-- Right outer join
select * from student right outer join faculty on fam=fid;

-- Full outer join (union of left and right outer joins)
select * from student left outer join faculty on fam=fid
union
select * from student right outer join faculty on fam=fid;

-- Natural join--can be applied if column name is same
alter table student rename column fam to fid;
select * from student natural join faculty;

-- Difference between natural join and equijoin???

-- Is the below query replacement for cartesian????
select * from student join faculty on fam=fid
union
select * from student join faculty on fam<>fid;
```

### GROUP_CONCAT Function

```sql
-- GROUP_CONCAT examples
select essn,dependent_name from dependent;

select essn,group_concat(dependent_name) as dependents from dependent group by essn;

-- Output example:
-- +-----------+-------------------------+
-- | essn      | dependents              |
-- +-----------+-------------------------+
-- | 123456789 | Alice,Elizabeth,Michael |
-- | 333445555 | Alice,Joy,Theodore      |
-- | 987654321 | Abner                   |
-- +-----------+-------------------------+

-- Using CTE with GROUP_CONCAT
with temp as (select essn,group_concat(dependent_name) as dependents from dependent group by essn)
select ssn,fname,dependents from temp,employee where temp.essn=employee.ssn;

-- or

select essn,fname, group_concat(dependent_name) as dependents from dependent,employee where essn=ssn group by essn, fname;

-- Output example:
-- +-----------+----------+-------------------------+
-- | ssn       | fname    | dependents              |
-- +-----------+----------+-------------------------+
-- | 123456789 | John     | Alice,Elizabeth,Michael |
-- | 333445555 | Franklin | Alice,Joy,Theodore      |
-- | 987654321 | Jennifer | Abner                   |
-- +-----------+----------+-------------------------+
```

### Recursive CTE Example

```sql
-- Recursive CTE for organizational hierarchy
with recursive orghierarchy as(
 select ssn,fname,lname,super_ssn, 1 as level
 from employee
 where super_ssn is null
 union all
 select e.ssn,e.fname,e.lname,e.super_ssn, OH.level+1
 from employee e
 join orghierarchy OH on e.super_ssn=OH.ssn
 )
 select * from orghierarchy;
```

---

## Company Database Queries

### Pattern Matching and Filtering

```sql
select fname,address from employee where address like '%Houston%';
select fname,address from employee where ssn like '___4____9';
select fname from employee where bdate like '1965%';
select fname from employee WHERE(Salary BETWEEN 30000 AND 40000) AND Dno = 5;
select fname,salary*1.1 as increased_salary from employee where dno=5;
select * from employee where dno=5 order by fname asc;
```

### Supervisor-Subordinate Queries

```sql
select fname, ssn, super_ssn from employee;
select fname from employee where super_ssn is null;
select fname from employee where super_ssn is not null;
```

### Complex Project-Employee Queries

```sql
-- Projects managed by Smith or worked on by Smith
(select distinct pnumber from project,department,employee where dnum=dnumber and mgr_ssn=ssn and lname='Smith') union (select distinct pnumber from project,works_on,employee where essn=ssn and lname='Smith');
select distinct pnumber from project where pnumber in(select pnumber from project,department,employee where dnum=dnumber and mgr_ssn=ssn and lname='Smith') or pnumber in(select pno from works_on,employee where essn=ssn and lname='Smith');
select distinct essn from works_on where (pno,hours) = any (select pno,hours from works_on where essn='123456789');
```

### Salary and Ranking Queries

```sql
select fname, salary from employee where salary in(select max(salary) from employee);
select ssn,fname,salary from employee where salary >=all(select salary from employee);
select lname,fname from employee where salary>all (select salary from employee where dno=5);
```

### Employee-Dependent Queries

```sql
-- Employees with dependents of same gender
select e.fname,e.lname from employee as e where e.ssn in (select essn from dependent as d where e.fname=d.dependent_name and e.gender=d.gender);

select e.fname ,e.lname from employee as e, dependent as d where e.ssn=d.essn and e.gender=d.gender and e.fname=d.dependent_name;

select e.fname,e.lname from employee as e where exists (select * from dependent as d where e.ssn=d.essn and e.fname=d.dependent_name and e.gender=d.gender);

-- Employees without dependents
select fname,lname from employee where ssn in((select ssn from employee except select essn from dependent));
select fname,lname from employee where not exists (select * from dependent where essn=ssn);

-- Managers with dependents
select distinct fname,lname from employee,department,dependent where ssn=essn and ssn=mgr_ssn;

select ssn,fname,lname from employee where ssn in(select mgr_ssn from department) and ssn in (select essn from dependent);

select fname,lname from employee where exists (select * from dependent where essn=ssn) and exists (select * from department where ssn=mgr_ssn);

-- Employees working on all projects of department 4
select fname,lname from employee where not exists ((select pnumber from project where dnum=4)except (select pno from works_on where essn=ssn));

select distinct essn from works_on where pno in(1,2,3);
```

### Supervisor Name Query

```sql
select e.lname as employee_name,s.lname as supervisor_name from employee e,employee s where e.super_ssn=s.ssn;
```

### JOIN Variations in Company Database

```sql
select fname,lname,address from employee join department on dno=dnumber;

select fname,lname,address from employee join department on dno=dnumber where dname='Research';

-- Natural join without common attribute gives cartesian product
select fname,dname from employee natural join department;

-- Left join for supervisor relationships
select e.lname,s.lname from employee as e left join employee as s on e.super_ssn=s.ssn;
```

### Aggregation and Grouping

```sql
select pnumber,pname,count(*) from project,works_on where pnumber=pno group by pnumber;
select pnumber, pname, count(*) from project, works_on where pnumber=pno group by pnumber having count(*)>2;

select dnumber,count(*) from department, employee where dnumber=dno and salary>40000 and dnumber in(select dno from employee group by dno having count(*)>5) group by dnumber;

-- Will get empty set for the above query

select dnumber, count(*) from department,employee where dnumber=dno and salary>=30000 and dnumber in(select dno from employee group by dno having count(*)>=4) group by dnumber;

select ssn,dno,salary from employee join department on dno=dnumber where dno=5;

select super_ssn,count(*) from employee group by super_ssn;

-- Employees without dependents using "with"
with xyz as (select ssn from employee except select essn from dependent) select ssn,fname from xyz natural join employee;
```

### Department-wise Statistics

```sql
select dno, avg(salary) from employee group by dno having count(*)>10 order by dno desc

select ssn,fname,essn,dependent_name from employee left join dependent on ssn=essn;
```

### JOIN Types Explanation

```sql
-- Left join
-- Right join
-- Cross join or just join without on clause for cartesian product
-- Natural join without condition
-- Full outer join - not supported -use left join union right join

-- Full outer join simulation
(select ssn,essn from employee left join dependent on ssn=essn) union (select ssn,essn from employee right join dependent on ssn=essn);
```

### CASE Statement Examples

```sql
-- Sample student table structure
desc student;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| sid    | int(11)     | NO   | PRI | NULL    |       |
| sname  | varchar(10) | NO   |     | NULL    |       |
| smarks | int(11)     | YES  |     | 24      |       |
| fam    | int(11)     | YES  | MUL | NULL    |       |
+--------+-------------+------+-----+---------+-------+

+-----+--------+--------+------+
| sid | sname  | smarks | fam  |
+-----+--------+--------+------+
|   1 | a      |     10 |  101 |
|   2 | b      |     10 |  102 |
|   3 | c      |     10 |  101 |
|   4 | d      |     10 |  102 |
|   5 | e      |     99 |  101 |
|   8 | pp     |    120 |  101 |
|  10 | rama   |    120 | NULL |
|  11 | ramaya |    120 | NULL |
+-----+--------+--------+------+
8 rows in set (0.002 sec)

-- CASE statement for grading
select sname,smarks, case when smarks>90 then 'excellent' when smarks>9 then 'good' end as remarks from student;

-- Adding columns and updating with CASE
alter table student add column totalmarks integer default 0;

update student set totalmarks = case when smarks>90 then totalmarks+10 when smarks>9 then totalmarks+5 end;

alter table student add column remarks varchar(10);

update student set remarks = case when smarks>100 then 'excellent' when smarks>10 then 'good' end;
```

---

## Sailor-Boat-Reserve Database Queries

### Database Schema Setup

```sql
create table sailors(sid integer primary key, sname varchar(20),srating integer,sage integer);
create table boats(bid integer primary key, bname varchar(20),color varchar(10));
create table reserves(sid integer,bid integer,bdate date,primary key(sid,bid,bdate),foreign key(sid) references sailors(sid), foreign key(bid) references boats(bid));

insert into sailors values(22,'dustin',7,45);
insert into sailors values(29,'brutus',1,33);
insert into sailors values(31,'lubber',8,55);
insert into sailors values(32,'andy',8,25);
insert into sailors values(58,'rusty',10,35);
insert into sailors values(64,'horatio',7,35);
insert into sailors values(71,'zorba',10,16);
insert into sailors values(75,'horatio',9,35);
insert into sailors values(85,'art',3,35);
insert into sailors values(95,'bob',3,63);

insert into boats values(101,'interlake','blue');
insert into boats values(102,'interlake','red');
insert into boats values(103,'clipper','green');
insert into boats values(104,'marine','red');

insert into reserves values(22,101,'1998-10-10');
insert into reserves values(22,102,'1998-10-10');
insert into reserves values(22,103,'1998-10-08');
insert into reserves values(22,104,'1998-10-07');
insert into reserves values(31,102,'1998-11-10');
insert into reserves values(31,103,'1998-11-06');
insert into reserves values(31,104,'1998-11-12');
insert into reserves values(64,101,'1998-09-05');
insert into reserves values(64,102,'1998-09-08');
insert into reserves values(74,103,'1998-09-08');
```

### Basic Queries

```sql
-- Find the names of the sailors who have reserved at least two boats
select s.sid,count(r.bid) from sailors s, reserves r where s.sid=r.sid group by s.sid having count(r.bid)>= 2;

-- Find the names and ages of all sailors
select s.sname ,s.age from sailors s;

-- Find all sailors with a rating above 7
select s.sid,s.sname,s.srating,s.age from sailors s where s.srating>7;

-- Find the names of sailors who have reserved boat number 103
select distinct s.sid,s.sname from sailors s,reserves r where s.sid=r.sid and r.bid=103;

-- Find the names of sailors who have reserved a Red boat
select distinct s.sid,s.sname,b.color from sailors s,reserves r,boats b where s.sid=r.sid and r.bid=b.bid and b.color='red';

-- Find the colors of boats reserved by Lubber
select distinct b.color from sailors s,boats b , reserves r where s.sid=r.sid and r.bid=b.bid and s.sname='Lubber';

-- Find the names of sailors who have reserved at least one boat
select distinct s.sname from sailors s,reserves r where s.sid=r.sid;

-- Compute increments for the ratings of persons who have sailed two different boats on the same day
select distinct s.sname,s.srating+1 as inc from sailors s,reserves r1,reserves r2
where s.sid=r1.sid and s.sid=r2.sid and r1.bdate=r2.bdate and r1.bid <> r2.bid;

-- Find the ages of sailors whose name begins and ends with B and has at least three characters
select distinct s.sname,s.age from sailors s where s.sname like 'B_%B';

-- Find the names of sailors who have reserved a red or a green boat
select distinct s.sname from sailors s,boats b, reserves r where s.sid=r.sid
and r.bid=b.bid and (b.color='red' or b.color='green');

-- Alternative using UNION
select s1.sname from sailors s1,reserves r1,boats b1 where s1.sid=r1.sid and r1.bid=b1.bid and b1.color='red'
union
select s2.sname from sailors s2,reserves r2,boats b2 where s2.sid=r2.sid and r2.bid=b2.bid and b2.color='green';

-- Find the names of sailors who have reserved both a red and a green boat
select  distinct s.sname from sailors s,boats b1,boats b2, reserves r1,reserves r2 where s.sid=r1.sid
and s.sid=r2.sid and r1.bid=b1.bid and r2.bid=b2.bid and r2.bid=b2.bid
and (b1.color='red' AND b2.color='green');

-- Alternative using INTERSECT
select s1.sname from sailors s1,reserves r1,boats b1 where s1.sid=r1.sid and r1.bid=b1.bid and b1.color='red'
intersect
select s2.sname from sailors s2,reserves r2,boats b2 where s2.sid=r2.sid and r2.bid=b2.bid and b2.color='green';

-- Find the sids of sailors who have reserved red boats but not green boats
select s1.sname from sailors s1,reserves r1,boats b1 where s1.sid=r1.sid and r1.bid=b1.bid and b1.color='red'
except
select s2.sname from sailors s2,reserves r2,boats b2 where s2.sid=r2.sid and r2.bid=b2.bid and b2.color='green';

-- Find all sids of sailors who have a rating of 10 or reserved boat 104
select s1.sid from sailors s1,reserves r1,boats b1 where s1.srating=10
union
select s2.sid from sailors s2,reserves r2 where s2.sid=r2.sid and r2.bid=104;
```

### Nested Queries

```sql
-- Find the names of sailors who have reserved boat 103
select distinct s1.sname from sailors s1 where s1.sid in(select r1.sid from reserves r1 where r1.bid=103);

-- Find the names of sailors who have reserved a red boat
select s1.sname from sailors s1 where s1.sid
in (select r1.sid from reserves r1 where r1.bid in(select b1.bid from boats b1 where b1.color='red'));

-- Find the names of sailors who have not reserved a red boat
select s1.sname from sailors s1 where s1.sid
not in (select r1.sid from reserves r1 where r1.bid in(select b1.bid from boats b1 where b1.color='red'));
```

### Correlational Queries

```sql
-- Find the names of sailors who have reserved boat number 103
select s1.sname from sailors s1 where exists (select * from reserves r1 where r1.bid=103 and s1.sid=r1.sid);

-- Find sailors whose rating is better than some sailor called Horatio
select s1.sname from sailors s1 where s1.srating>any (select s2.srating from sailors s2 where s2.sname='Horatio');

-- Find sailors whose rating is better than every sailor called Horatio
select s1.sname,s1.srating from sailors s1 where s1.srating>=all (select s2.srating from sailors s2 where s2.sname='Horatio');

-- Find the sailors with the highest rating
select s1.sname,s1.srating from sailors s1 where s1.srating>=all (select s2.srating from sailors s2);
```

### More Nested Queries

```sql
-- Find the names of sailors who have reserved both a red and a green boat
select distinct s1.sname,s1.sid from sailors s1,reserves r1,boats b1 where s1.sid=r1.sid and r1.bid=b1.bid and b1.color='red'
and s1.sid in (select s2.sid from sailors s2,reserves r2,boats b2 where s2.sid=r2.sid and r2.bid=b2.bid and b2.color='green');

-- Find the names of sailors who have reserved all boats
SELECT S.sname
FROM Sailors S
WHERE NOT EXISTS (( SELECT B.bid
FROM Boats B )
EXCEPT
(SELECT R. bid
FROM Reserves R
WHERE R.sid = S.sid ));
```

### Aggregate Operations

```sql
-- Find the average age of all sailors
select avg(s.age) from sailors s;

-- Find the average age of sailors with a rating of 10
select avg(s.age) from sailors s where s.srating=10;

-- Find the name and age of the oldest sailor
select s.sname,s.age from sailors s where s.age=(select max(s1.age) from sailors s1);

-- Count the number of sailors
select count(*) from sailors s;

-- Count the number of different sailor names
select count(distinct sname) from sailors s;

-- Find the names of sailors who are older than the oldest sailor with a rating of 10
select s1.sname,s1.age from sailors s1 where s1.age>(select max(s2.age) from sailors s2 where s2.srating=10);
```

### GROUP BY and HAVING Clauses

```sql
-- Find the age of the youngest sailor for each rating level
select s1.srating,min(s1.age) from sailors s1 group by s1.srating ;

-- Find the age of the youngest sailor who is eligible to vote (i.e., is at least 18 years old)
-- for each rating level with at least two such sailors
select s1.srating, min(s1.age) from sailors s1 where s1.age > 18 group by s1.srating having count(*)>=2 ;
```

### More Aggregate Queries

```sql
-- For each red boat; find the number of reservations for this boat
select b.bid,count(*) as NoOfReservations from reserves r,boats b where r.bid=b.bid and b.color='red' group by b.bid;

-- Find the average age of sailors for each rating level that has at least two sailors
select s.srating,avg(s.age) as AvgAge from sailors s group by s.srating having count(*)>1;

-- Find the average age of sailors who are of voting age (i.e., at least 18 years old)
-- for each rating level that has at least two sailors
select s.srating,avg(s.age) as AvgAge from sailors s where s.age>=18 group by s.srating having count(*)>1;

-- Find those ratings for which the average age of sailors is the minimum over all ratings
```

### Complex Integrity Constraints

```sql
-- To ensure that rating must be an integer in the range 1 to 10
CREATE TABLE Sailors ( sid INTEGER, sname CHAR(10), rating INTEGER, age REAL, PRIMARY KEY (sid),
CHECK (srating >= 1 AND srating <= 10 ))

-- A new domain using the CREATE DOMAIN statement, which uses CHECK constraints

-- Assertions: ICs over Several Tables
-- To enforce the constraint that the number of boats plus the number of sailors should be less than 100
-- (This condition might be required, say, to qualify as a 'small' sailing club.)
```

---

## SQL Functions

### User-Defined Functions (UDFs)

```sql
-- General function
DELIMITER $$
CREATE FUNCTION CalculateAge(bdate DATE)
  RETURNS INT
  DETERMINISTIC
BEGIN
  DECLARE age INT;

  SET age = TIMESTAMPDIFF(YEAR, bdate, CURDATE());

  RETURN age;
END$$
DELIMITER ;

SELECT ssn,fname,bdate,CalculateAge(bdate) AS age FROM employee;

select CalculateAge('2002-06-16');

-- Command to drop a function
drop function functionname;

-- Department size function
DELIMITER $$
CREATE FUNCTION Dept_size1(deptno INT)
	RETURNS VARCHAR(7)
DETERMINISTIC
	BEGIN
		DECLARE No_of_emps INT;
		SELECT COUNT(*) INTO No_of_emps FROM EMPLOYEE WHERE Dno = deptno;
		IF No_of_emps > 3 THEN    RETURN 'HUGE';
		ELSEIF No_of_emps > 2 THEN    RETURN 'LARGE';
		ELSEIF No_of_emps > 1 THEN    RETURN 'MEDIUM';
		ELSE    RETURN 'SMALL';
		END IF;
END$$
DELIMITER ;

select dept_size1(5);

-- Using user-defined functions (UDFs) in INSERT
-- sample--INSERT INTO employee (name, salary)
-- VALUES ('Alice', calculate_bonus(50000));

-- Cannot create a function without return value--ie a function must return a value
```

### Examples from functions.txt

```sql
-- Example 1
DELIMITER $$
CREATE FUNCTION DeptSize(deptno INT)
	RETURNS VARCHAR(7) DETERMINISTIC
	BEGIN
		DECLARE NoEMP INT;
		SELECT COUNT(*) INTO NoEMP FROM EMPLOYEE WHERE Dno = deptno;
		IF NoEMP > 3 THEN    RETURN 'HUGE';
		ELSEIF NoEMP > 2 THEN  RETURN 'LARGE';
		ELSEIF NoEMP > 1 THEN  RETURN 'MEDIUM';
		ELSE    RETURN 'SMALL';
		END IF;
END;$$

-- OUTPUT
mysql> select Dname,Dnumber,DeptSize(Dnumber) from department;
+----------------+---------+-------------------+
| Dname          | Dnumber | DeptSize(Dnumber) |
+----------------+---------+-------------------+
| Administration |       4 | LARGE             |
| Headquarters   |       1 | HUGE              |
| Research       |       5 | HUGE              |
+----------------+---------+-------------------+
3 rows in set (0.01 sec)

-- Example 2
DELIMITER $$
CREATE FUNCTION no_of_years(date1 date)
	RETURNS INTEGER deterministic
		BEGIN
			DECLARE date2 DATE;
			Select current_date()into date2;
			RETURN year(date2)-year(date1);
		END$$

-- OUTPUT
mysql> select Fname,no_of_years(Bdate) from employee;
+----------+--------------------+
| Fname    | no_of_years(Bdate) |
+----------+--------------------+
| Richard  |                 53 |
| advika   |                 37 |
| John     |                 60 |
| Richard  |                 53 |
| Franklin |                 70 |
| Joyce    |                 53 |
| Ramesh   |                 63 |
| James    |                 88 |
| Jennifer |                 84 |
| Ahmed    |                 56 |
| Alicia   |                 57 |
+----------+--------------------+
11 rows in set (0.00 sec)
```

### Additional Functions from f_p_t.txt

```sql
DELIMITER $$
CREATE FUNCTION display_load(eid1 INTEGER)
	RETURNS VARCHAR(7)
DETERMINISTIC
	BEGIN
		DECLARE LOAD1 INTEGER;
		SELECT DURATION INTO LOAD1 FROM WORKS_ON_PROJ WHERE EID=EID1 LIMIT 1; -- Added LIMIT 1 as the query might return multiple rows without it
		IF LOAD1 > 20 THEN    RETURN 'HUGE';
		ELSEIF LOAD1 > 10 THEN    RETURN 'LARGE';
		ELSEIF LOAD1 > 5 THEN    RETURN 'MEDIUM';
		ELSE    RETURN 'SMALL';
		END IF;
END$$
DELIMITER ;

DELIMITER $$
CREATE FUNCTION display_load1(eid1 INTEGER)
	RETURNS INTEGER
DETERMINISTIC
	BEGIN
		DECLARE LOAD1 INTEGER;
		SELECT SUM(DURATION) INTO LOAD1 FROM WORKS_ON_PROJ WHERE EID=EID1 GROUP BY EID;
		RETURN LOAD1;
END$$
DELIMITER ;

-- Query using sailor database
SELECT sname FROM sailors
  WHERE ( SELECT COUNT(*) FROM reserves WHERE reserves.sid = sailors.sid ) >= 2;
```

---

## Stored Procedures

### Basic Stored Procedures

```sql
-- List all stored procedures in the current database
SHOW PROCEDURE STATUS WHERE Db = 'your_database_name';

-- List all stored procedures across all databases
SHOW PROCEDURE STATUS;

-- Show the definition (code) of a procedure
SHOW CREATE PROCEDURE procedure_name\G
```

### Procedure Examples

```sql
DELIMITER $$
CREATE PROCEDURE inform_supervisor111(
    IN superssn1 char(9),
    IN sal decimal(10,2)
)
BEGIN
declare empsal decimal(10,2);
declare supersal decimal(10,2);
set empsal=sal;
set supersal=(select salary from employee where ssn=superssn1);

IF empsal>supersal THEN
   SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT="employee salary cannot be more than supervisior";
END IF;
END$$
DELIMITER;


### Procedures with Different Parameter Types

```sql
-- Zero argument procedure
DELIMITER $$
CREATE PROCEDURE get_high_paid_employee1()
BEGIN
	SELECT ssn,fname,salary FROM employee WHERE salary> 30000;
	 select count(*) from employee;
	select ssn from employee where dno=4;
END$$
DELIMITER ;
call get_high_paid_employee1();

drop procedure get_high_paid_employee1;

-- IN parameter procedure
DELIMITER &&
CREATE PROCEDURE get_emps(IN var1 INT)
BEGIN
    SELECT fname,ssn FROM employee LIMIT var1;
END &&
DELIMITER ;

call get_emps(3);

drop procedure get_emps;

-- OUT parameter procedure
DELIMITER &&
CREATE PROCEDURE display_max_sal (OUT highestsal INTEGER)
BEGIN
    SELECT max(salary) INTO highestsal FROM employee;
END&&
DELIMITER ;

call display_max_sal(@output);
select @output;

-- INOUT parameter procedure
DELIMITER &&
CREATE PROCEDURE display_sal(INOUT var1 char(9))
BEGIN
    SELECT salary INTO var1 FROM employee WHERE ssn = var1;
END&&
DELIMITER ;

set @m='123456789';
call display_sal(@m);
select @m;

drop procedure proc_name;
```

---

## Triggers

### Trigger Concepts

A trigger is a stored database object that is automatically executed in response to a data modification event on a table.

The data modification events include insert, update, and delete caused by the insert, update, and delete statements.

MariaDB stored triggers as named objects in the data dictionary. A trigger is always associated with a table.

Six trigger types:
- before insert
- after insert
- before update
- after update
- before delete
- after delete

### Trigger Characteristics

| Characteristics | Triggers | Stored Procedures |
|------------------|----------|-------------------|
| Event-driven | Yes | No |
| Associated with a table | Yes | No |
| Return a result | No | Yes |

### Trigger Examples

```sql
CREATE TABLE employees(id INT,fname VARCHAR(10) NOT NULL,lname VARCHAR(10) NOT NULL,PRIMARY KEY(id));

CREATE TABLE employee_audits (id integer auto_increment,employee_id INT NOT NULL,last_name VARCHAR(10) NOT NULL, changed_on TIMESTAMP,primary key(id));

INSERT INTO employees VALUES (1,'John', 'Doe');
INSERT INTO employees VALUES (2,'Lily', 'Bush');

delimiter $$
CREATE TRIGGER last_name_changes
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
	-- The logic from the procedure has been moved here.
	-- Stored procedures called from triggers cannot access NEW/OLD values.
	IF NEW.lname <> OLD.lname THEN
		INSERT INTO employee_audits(employee_id,last_name, changed_on)
		VALUES(OLD.id, OLD.lname, NOW());
	END IF;
END;
$$
delimiter ;

UPDATE employees SET last_name = 'Brown' WHERE ID = 2;

-- To drop the trigger: DROP TRIGGER trigger_name;
-- To change the trigger: alter trigger trigger_name....
-- Note: Disabling triggers is not supported in standard MySQL.
-- The following are examples from other SQL dialects (e.g., SQL Server).
-- To disable the trigger: ALTER TABLE employees DISABLE TRIGGER log_last_name_changes;
-- To disable all triggers: ALTER TABLE employees DISABLE TRIGGER ALL;
-- To enable: ALTER TABLE table_name ENABLE TRIGGER trigger_name | ALL;
-- To enable certain trigger: ALTER TABLE employees ENABLE TRIGGER salary_before_update;
```

### Trigger Examples from TRIGGERS.txt

```sql
-- Creating triggers
DELIMITER $$

CREATE TRIGGER trg_MinSalary
BEFORE INSERT ON Employee
FOR EACH ROW
BEGIN
    IF NEW.Salary < 20000 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Salary must be at least 20000!';
    END IF;
END$$

DELIMITER ;

-- Employee table before inserting new values
mysql> select *from employee;
+----------+-------+---------+-----------+------------+--------------------------+--------+----------+-----------+-----+
| Fname    | Minit | Lname   | Ssn       | Bdate      | Address                  | Gender | Salary   | Super_ssn | Dno |
+----------+-------+---------+-----------+------------+--------------------------+--------+----------+-----------+-----+
| John     | B     | Smith   | 123456789 | 1965-01-09 | 731 Fondren,Houston,TX   | M      | 30000.00 | 888665555 |   5 |
| Franklin | T     | Wong    | 333445555 | 1955-12-08 | 638 voss,Houston,TX      | M      | 40000.00 | 888665555 |   5 |
| Joyce    | A     | English | 453453453 | 1972-07-31 | 5631 Rice,Houston,TX     | F      | 25000.00 | 333445555 |   5 |
| Ramesh   | K     | Narayan | 666884444 | 1962-09-15 | 975 Fire Oak, Humble, TX | M      | 38000.00 | 333445555 |   5 |
| James    | E     | Borg    | 888665555 | 1937-11-10 | 450 Stone, Houston,TX    | M      | 55000.00 | NULL      |   1 |
| Jennifer | S     | Wallace | 987654321 | 1941-06-20 | 291 Berry, Bellaire,Tx   | F      | 43000.00 | 333445555 |   4 |
| Ahmed    | V     | Jabbar  | 987987987 | 1969-03-29 | 980 Dallas, Houston,TX   | M      | 25000.00 | 987654321 |   4 |
| Alicia   | J     | Zelaya  | 999887777 | 1968-01-19 | 3321 Castle,Spring,Tx    | F      | 25000.00 | 333445555 |   4 |
+----------+-------+---------+-----------+------------+--------------------------+--------+----------+-----------+-----+
8 rows in set (0.01 sec)

-- Now insert wrong values
mysql> INSERT INTO EMPLOYEE VALUES ( 'Richard', 'M', 'BENIN', '333333333', '1972-12-30', '198 Oak Forest, Katy, TX', 'M', 17000, '333445555', 1 );
ERROR 1644 (45000): Salary must be at least 20000!

-- NULL value in that place
INSERT INTO EMPLOYEE VALUES ( 'Richard', 'M', 'BENIN', '111111111', '1972-12-30', '198 Oak Forest, Katy, TX', 'M', NULL, '333445555', 1 );
Query OK, 1 row affected (0.03 sec)

-- Drop trigger and try
mysql> DROP TRIGGER IF EXISTS trg_MinSalary;
Query OK, 0 rows affected (0.03 sec)

mysql> INSERT INTO EMPLOYEE VALUES ( 'Richard', 'M', 'BENIN', '222222222', '1972-12-30', '198 Oak Forest, Katy, TX', 'M', 17000, '333445555', 1 );
Query OK, 1 row affected (0.03 sec)

-- Final table after insert
mysql> select *from employee;
+----------+-------+---------+-----------+------------+--------------------------+--------+----------+-----------+-----+
| Fname    | Minit | Lname   | Ssn       | Bdate      | Address                  | Gender | Salary   | Super_ssn | Dno |
+----------+-------+---------+-----------+------------+--------------------------+--------+----------+-----------+-----+
| Richard  | M     | BENIN   | 111111111 | 1972-12-30 | 198 Oak Forest, Katy, TX | M      |     NULL | 333445555 |   1 |
| John     | B     | Smith   | 123456789 | 1965-01-09 | 731 Fondren,Houston,TX   | M      | 30000.00 | 888665555 |   5 |
| Richard  | M     | BENIN   | 222222222 | 1972-12-30 | 198 Oak Forest, Katy, TX | M      | 17000.00 | 333445555 |   1 |
| Franklin | T     | Wong    | 333445555 | 1955-12-08 | 638 voss,Houston,TX      | M      | 40000.00 | 888665555 |   5 |
| Joyce    | A     | English | 453453453 | 1972-07-31 | 5631 Rice,Houston,TX     | F      | 25000.00 | 333445555 |   5 |
| Ramesh   | K     | Narayan | 666884444 | 1962-09-15 | 975 Fire Oak, Humble, TX | M      | 38000.00 | 333445555 |   5 |
| James    | E     | Borg    | 888665555 | 1937-11-10 | 450 Stone, Houston,TX    | M      | 55000.00 | NULL      |   1 |
| Jennifer | S     | Wallace | 987654321 | 1941-06-20 | 291 Berry, Bellaire,Tx   | F      | 43000.00 | 333445555 |   4 |
| Ahmed    | V     | Jabbar  | 987987987 | 1969-03-29 | 980 Dallas, Houston,TX   | M      | 25000.00 | 987654321 |   4 |
| Alicia   | J     | Zelaya  | 999887777 | 1968-01-19 | 3321 Castle,Spring,Tx    | F      | 25000.00 | 333445555 |   4 |
+----------+-------+---------+-----------+------------+--------------------------+--------+----------+-----------+-----+
10 rows in set (0.00 sec)
```

### More Trigger Examples from f_p_t.txt

```sql
CREATE TABLE Student_sample ( SRN INT, Name VARCHAR(10), PRIMARY KEY(SRN) );
CREATE TABLE Marks_sample ( SRN INT, Course VARCHAR(10), Marks INT, PRIMARY KEY(SRN, Course), FOREIGN KEY(SRN) REFERENCES Student_sample(SRN) );
CREATE TABLE marks_history( SRN INT, Course VARCHAR(10), Marks INT, PRIMARY KEY(SRN, Course));

DELIMITER //
CREATE TRIGGER CheckMarks
BEFORE INSERT ON marks_sample
FOR EACH ROW
BEGIN
	IF NEW.marks < 0 OR NEW.marks > 100 THEN
		SIGNAL SQLSTATE '45000'
		SET MESSAGE_TEXT = 'Invalid marks: Marks must be between 0 and 100';
           	END IF;
END //
DELIMITER ;

insert into marks_sample values(10,'cs02',100);--valid
insert into marks_sample values(10,'cs02',101);--Invalid
insert into marks_sample values(10,'cs02',-1);--Invalid

-- Insert sample data
insert into student_sample values(10,'rahul'),(11,'ria'),(12,'rina'),(13,'roopa'),(14,'shriya');
insert into marks_sample values(10,'cs1',98),(10,'cs2',89),(11,'cs1',100),(11,'cs2',98),(11,'cs3',80),(13,'cs1',95),(13,'cs2',98);

DELIMITER //
CREATE TRIGGER update_marks_on_cascade
BEFORE UPDATE ON Student_sample
FOR EACH ROW
BEGIN
	SET FOREIGN_KEY_CHECKS = 0;
	UPDATE Marks_sample    SET SRN = NEW.SRN    WHERE SRN = OLD.SRN;
	SET FOREIGN_KEY_CHECKS = 1;
END;//
DELIMITER ;

DELIMITER //
CREATE TRIGGER backup_marks_info1
BEFORE DELETE
ON Marks_sample
FOR EACH ROW
BEGIN
-- A more direct way: INSERT INTO Marks_history(SRN, Course, Marks) VALUES(OLD.SRN, OLD.Course, OLD.Marks);
INSERT INTO Marks_history SELECT * FROM Marks_sample WHERE SRN = OLD.SRN AND COURSE = OLD.COURSE;
END//
DELIMITER ;

DELETE FROM Marks_sample WHERE SRN = 10;

DELIMITER //
CREATE TRIGGER del_works_on3
before DELETE ON employee
FOR EACH ROW
BEGIN
	SET FOREIGN_KEY_CHECKS = 0;
    DELETE FROM works_on_proj
    WHERE eid= old.eid;
SET FOREIGN_KEY_CHECKS = 1;
END //
DELIMITER ;
```

### Trigger Calling Stored Procedure

```sql
DELIMITER $$
CREATE TRIGGER SALARY_VIOLATION11
before INSERT
ON EMPLOYEE FOR EACH ROW
BEGIN
 IF (NEW.salary > ( select salary from employee
                     where ssn=new.superssn)) THEN
 CALL inform_supervisor111(new.superssn,new.salary);
 END IF;
END$$
DELIMITER;

-- Test the trigger
insert into employee values('advika','n','k',
'111111889','1988-08-08','bangalore','F',66000,
'888665555',1);

-- Output:
mysql> insert into employee values('advika','n','k',
    -> '111111889','1988-08-08','bangalore','F',66000,
    -> '888665555',1);
ERROR 1644 (45000): employee salary cannot be more than supervisior
```

## Views

### Creating and Using Views

```sql
-- Authors Table
CREATE TABLE Authors (
    Author_ID VARCHAR(20) PRIMARY KEY,
    Name VARCHAR(20) NOT NULL,
    Age INT,
    Email VARCHAR(30) UNIQUE,
    Bio TEXT,
    Password VARCHAR(20) NOT NULL
);

-- Insert values into Authors table
INSERT INTO Authors (Author_ID, Name, Age, Email, Bio, Password) VALUES
('A001', 'Alice Johnson', 35, 'alice.johnson@example.com', 'Expert in AI and ML', 'password123'),
('A002', 'Bob Smith', 40, 'bob.smith@example.com', 'Data Scientist', 'password456'),
('A003', 'Carol White', 28, 'carol.white@example.com', 'Researcher in NLP', 'password789'),
('A004', 'David Brown', 50, 'david.brown@example.com', 'Professor in Quantum Computing', 'password321'),
('A005', 'Eve Davis', 32, 'eve.davis@example.com', 'Cloud Computing Specialist', 'password654'),
('A006', 'Frank Martin', 45, 'frank.martin@example.com', 'Expert in Computer Vision', 'password987'),
('A007', 'Grace Lee', 30, 'grace.lee@example.com', 'Cryptography Enthusiast', 'password147'),
('A008', 'Henry Wilson', 27, 'henry.wilson@example.com', 'Cybersecurity Analyst', 'password258'),
('A009', 'Ivy Scott', 34, 'ivy.scott@example.com', 'Data Engineering Expert', 'password369');

show tables;
desc authors;
select * from authors;

-- View to get Experienced Authors(Age>=30 and Bio has the word "Expert")
create view experienced_authors as
select * from authors
where age>=30 and Bio like '%Expert%';

select * from experienced_authors;

-- Inserting a new experienced author
insert into experienced_authors (Author_ID, Name, Age, Email, Bio, Password)
values ("A010", "John Wick", 52, "john.wick@example.com", "Expert in Pencils", "password567");

-- Updating Alice's Age to 40 from 35
update experienced_authors set Age=40 where Author_ID='A001';

-- Updated Table with changes
select * from experienced_authors;

-- Deleting Ivy Scott (A009) from the view
delete from experienced_authors where Author_ID="A009" and Name="Ivy Scott";
select * from experienced_authors;

-- All CRUD operations are reflected on the original table as well
select * from authors;
```

---

## User Roles and Permissions

### Creating and Managing Roles

```sql
-- Connect to MySQL as root user
-- mysql -u root -p

-- 1. Create a role
CREATE ROLE 'app_readonly';

-- After creating we can see
mysql> select user from mysql.user;
+------------------+
| user             |
+------------------+
| app_readonly     |
| mysql.infoschema |
| mysql.session    |
| mysql.sys        |
| root             |
+------------------+
5 rows in set (0.00 sec)

-- 2. Grant privileges to role
GRANT SELECT ON companydb.* TO 'app_readonly';

-- 3. Create a user
CREATE USER 'student'@'localhost' IDENTIFIED BY 'password123';

-- 4. Assign role to user
GRANT 'app_readonly' TO 'student'@'localhost';

-- 5. Enable the role by default for the user (optional)
SET DEFAULT ROLE 'app_readonly' TO 'student'@'localhost';

-- 6. Removing the permission
mysql> revoke select on companydb.* from 'app_readonly';
Query OK, 0 rows affected (0.04 sec)

-- Open another terminal and connect as student user


-- mysql -u student -p
Enter password: ***********

-- Below commands need to be executed at student user

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| companydb          |
| information_schema |
| performance_schema |
+--------------------+
3 rows in set (0.019 sec)

mysql> use companydb;
Database changed
mysql> show tables;
+---------------------+
| Tables_in_companydb |
+---------------------+
| authors             |
| department          |
| dependent           |
| dept_locations      |
| employee            |
| experienced_authors |
| project             |
| works_on            |
+---------------------+
8 rows in set (0.03 sec)

mysql> select *from employee;
+----------+-------+---------+-----------+------------+--------------------------+--------+----------+-----------+-----+
| Fname    | Minit | Lname   | Ssn       | Bdate      | Address                  | Gender | Salary   | Super_ssn | Dno |
+----------+-------+---------+-----------+------------+--------------------------+--------+----------+-----------+-----+
| John     | B     | Smith   | 123456789 | 1965-01-09 | 731 Fondren,Houston,TX   | M      | 30000.00 | 888665555 |   5 |
| Franklin | T     | Wong    | 333445555 | 1955-12-08 | 638 voss,Houston,TX      | M      | 40000.00 | 888665555 |   5 |
| Joyce    | A     | English | 453453453 | 1972-07-31 | 5631 Rice,Houston,TX     | F      | 25000.00 | 333445555 |   5 |
| Ramesh   | K     | Narayan | 666884444 | 1962-09-15 | 975 Fire Oak, Humble, TX | M      | 38000.00 | 333445555 |   5 |
| James    | E     | Borg    | 888665555 | 1937-11-10 | 450 Stone, Houston,TX    | M      | 55000.00 | NULL      |   1 |
| Jennifer | S     | Wallace | 987654321 | 1941-06-20 | 291 Berry, Bellaire,Tx   | F      | 43000.00 | 333445555 |   4 |
| Ahmed    | V     | Jabbar  | 987987987 | 1969-03-29 | 980 Dallas, Houston,TX   | M      | 25000.00 | 987654321 |   4 |
| Alicia   | J     | Zelaya  | 999887777 | 1968-01-19 | 3321 Castle,Spring,Tx    | F      | 25000.00 | 333445555 |   4 |
+----------+-------+---------+-----------+------------+--------------------------+--------+----------+-----------+-----+
8 rows in set (0.02 sec)

mysql> delete from employee where ssn= '999887777';
ERROR 1142 (42000): DELETE command denied to user 'student'@'localhost' for table 'employee'

-- Execute below commands in root login
-- Create one more role and user with limited access to tables
CREATE ROLE 'app_department';

GRANT SELECT ON companydb.department TO 'app_department';

CREATE USER 'student1'@'localhost' IDENTIFIED BY 'password123';

GRANT 'app_department' TO 'student1'@'localhost';

SET DEFAULT ROLE 'app_department' TO 'student1'@'localhost';

-- Open another terminal and connect as student user1


-- mysql -u student1 -p
Enter password: ***********

-- Type below commands and check you have access to only one table
-- i.e department
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| companydb          |
| information_schema |
| performance_schema |
+--------------------+
3 rows in set (0.019 sec)

mysql> use companydb;
Database changed
mysql> show tables;
+---------------------+
| Tables_in_companydb |
+---------------------+
| department          |
+---------------------+
1 row in set (0.023 sec)
```
