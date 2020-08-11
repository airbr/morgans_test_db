Learning database technologies like MySQL can be helped along with a little imagination. In this Repo, we follow the life of "**The Company**". Its where I imagine the Employees sample database as a living breathing entity, undergoing some changes, successes and challenges.

What am I talking about? Basically, I wanted to spice up learning mysql, so I decided to imagine some scenarios how they might come to change the database of this fictional company.





**Explore the company and start backups** 

`select count(*) from employees` 

| count(*) |
| :------- |
| 300026   |

`describe employees` 

| Field      | Type          | Null | Key  | Default | Extra |
| ---------- | ------------- | ---- | ---- | ------- | ----- |
| emp_no     | int           | NO   | PRI  | NULL    |       |
| birth_date | date          | NO   |      | NULL    |       |
| first_name | varchar(14)   | NO   |      | NULL    |       |
| last_name  | varchar(16)   | NO   |      | NULL    |       |
| gender     | enum('M','F') | YES  |      | NULL    |       |
| hire_date  | date          | NO   |      | NULL    |       |

**Check the foreign keys**

```sql
SELECT TABLE_NAME,COLUMN_NAME,CONSTRAINT_NAME, REFERENCED_TABLE_NAME,REFERENCED_COLUMN_NAME
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE REFERENCED_TABLE_SCHEMA = 'employees'
```

| TABLE_NAME   | COLUMN_NAME | CONSTRAINT_NAME     | REFERENCED_TABLE_NAME | REFERENCED_COLUMN_NAME |
| ------------ | ----------- | ------------------- | --------------------- | ---------------------- |
| dept_emp     | emp_no      | dept_emp_ibfk_1     | employees             | emp_no                 |
| dept_emp     | dept_no     | dept_emp_ibfk_2     | departments           | dept_no                |
| dept_manager | emp_no      | dept_manager_ibfk_1 | employees             | emp_no                 |
| dept_manager | dept_no     | dept_manager_ibfk_2 | departments           | dept_no                |
| salaries     | emp_no      | salaries_ibfk_1     | employees             | emp_no                 |
| titles       | emp_no      | titles_ibfk_1       | employees             | emp_no                 |

**Start backups of the database**

```
crontab -e
```

Added this to run every night at ten o' clock. Use absolute paths:

```0 22 * * * /usr/local/bin/mysqldump -uroot -p[password] --all-databases > /absolute/path/to/where/you/want/the/backups/$(date +\%F)_fullcronbackup.sql```

**Add a CEO and VP, pay them large amounts of money**

```sql
INSERT INTO  (emp_no,birth_date,first_name,last_name,gender,hire_date) VALUES (1,'1945-01-01','Monty','Burns','M','1965-01-01');
INSERT INTO  (emp_no,birth_date,first_name,last_name,gender,hire_date) VALUES (2,'1950-01-01','Evil ','VP','M','2020-03-01');

INSERT INTO  (emp_no,salary,from_date,to_date) VALUES (1,99999000,'1965-01-01','9999-01-01');
INSERT INTO  (emp_no,salary,from_date,to_date) VALUES (2,90000000,'2000-01-01','9999-01-01');
```

**Make the company more inclusive - add non binary as a gender option**

`alter table employees modify gender ENUM('M','F','NB');`

`update employees set gender = NB where RAND() < .03;`

[See this link about usage of Rand() here](https://stackoverflow.com/questions/11087059/mysql-how-do-i-update-50-of-the-rows-randomly-selected);

**Penalize the CEO, presumably for poor performance:**

`UPDATE salaries SET salary = salary - 1000 WHERE emp_no = '1'`

We have other options:

* Run SQL script as a Cron Job
* Event scheduler 
* Stored Procedure. + event

**Attrition: Recruiters snag your least paid senior talent- event schedule to delete employees**

SQL to find the Senior Engineer who is currently paid the least:

```SELECT 
SELECT 
    salaries.emp_no, salary, title
FROM
    employees
        INNER JOIN
    salaries ON employees.emp_no = salaries.emp_no
        INNER JOIN
    titles ON employees.emp_no = titles.emp_no
WHERE
    title = 'Senior Engineer'
        AND salaries.to_date = '9999-01-01'
ORDER BY salary ASC
LIMIT 1
```

Add a Stored Procedure which deletes this selected Engineer (dang recruiters!)

``` sql
DELIMITER //

CREATE PROCEDURE dang_recruiter()
BEGIN
	SELECT 
    salaries.emp_no
INTO    
    @empno
FROM
    employees
        INNER JOIN
    salaries ON employees.emp_no = salaries.emp_no
    INNER JOIN
    titles ON employees.emp_no = titles.emp_no
    WHERE title = 'Senior Engineer' AND salaries.to_date = '9999-01-01'
    ORDER BY salary ASC
    limit 1;
    
DELETE FROM employees where emp_no = @empno;
END //

DELIMITER ;
```

Add a event which calls this stored procedure every hour (yikes!)

```sql
DROP EVENT IF EXISTS `every_hour_a_recruiter_strikes`;
DELIMITER $$
CREATE EVENT `every_hour_a_recruiter_strikes`
  ON SCHEDULE EVERY 60 MINUTE STARTS '2020-08-09 00:00:00'
  ON COMPLETION PRESERVE
DO BEGIN
  CALL dang_recruiter();
END 
$$
DELIMITER ;
```

**Need to generate new employees. Use existing names in the database ?**

I need to create new employees, ideally with a salary, title and department and dept.

There are probably a few ways I could approach this.

Its a pretty big database, so I could just use a lot of existing values randomized:

```
select first_name into @randomfirstname from employees order by rand() limit 1;
select last_name into @randomlastname from employees order by rand() limit 1;
select @randomfirstname, @randomlastname;
```



**TODO:**

Create Stored Procedure:

Insert Employee record, generate random first and last name combo

Insert Title for new employee

Insert Salary for new Employee

Insert Dept_employee record entry

**TODO:**

Investigate gender pay disparities - initial results are promising there is not a big gap but it remains yet to be solved in detail

Monthly turnover of 1.25% of the company - event schedule to delete employees from all departments

Hiring: Hire to maintain current levels - regaining that lost 1.25% . Difficulty: medium-hard.

Script code for some events:

Event: Layoff of a particular department

Event: Company-wide Strike

Event: Covid-19 impact





# Original ReadMe:

# test_db

A sample database with an integrated test suite, used to test your applications and database servers

This repository was migrated from [Launchpad](https://launchpad.net/test-db).

See usage in the [MySQL docs](https://dev.mysql.com/doc/employee/en/index.html)


## Where it comes from

The original data was created by Fusheng Wang and Carlo Zaniolo at 
Siemens Corporate Research. The data is in XML format.
http://timecenter.cs.aau.dk/software.htm

Giuseppe Maxia made the relational schema and Patrick Crews exported
the data in relational format.

The database contains about 300,000 employee records with 2.8 million 
salary entries. The export data is 167 MB, which is not huge, but
heavy enough to be non-trivial for testing.

The data was generated, and as such there are inconsistencies and subtle
problems. Rather than removing them, we decided to leave the contents
untouched, and use these issues as data cleaning exercises.

## Prerequisites

You need a MySQL database server (5.0+) and run the commands below through a 
user that has the following privileges:

    SELECT, INSERT, UPDATE, DELETE, 
    CREATE, DROP, RELOAD, REFERENCES, 
    INDEX, ALTER, SHOW DATABASES, 
    CREATE TEMPORARY TABLES, 
    LOCK TABLES, EXECUTE, CREATE VIEW

## Installation:

1. Download the repository
2. Change directory to the repository

Then run

    mysql < employees.sql


If you want to install with two large partitioned tables, run

    mysql < employees_partitioned.sql


## Testing the installation

After installing, you can run one of the following

    mysql -t < test_employees_md5.sql
    # OR
    mysql -t < test_employees_sha.sql

For example:

    mysql  -t < test_employees_md5.sql
    +----------------------+
    | INFO                 |
    +----------------------+
    | TESTING INSTALLATION |
    +----------------------+
    +--------------+------------------+----------------------------------+
    | table_name   | expected_records | expected_crc                     |
    +--------------+------------------+----------------------------------+
    | employees    |           300024 | 4ec56ab5ba37218d187cf6ab09ce1aa1 |
    | departments  |                9 | d1af5e170d2d1591d776d5638d71fc5f |
    | dept_manager |               24 | 8720e2f0853ac9096b689c14664f847e |
    | dept_emp     |           331603 | ccf6fe516f990bdaa49713fc478701b7 |
    | titles       |           443308 | bfa016c472df68e70a03facafa1bc0a8 |
    | salaries     |          2844047 | fd220654e95aea1b169624ffe3fca934 |
    +--------------+------------------+----------------------------------+
    +--------------+------------------+----------------------------------+
    | table_name   | found_records    | found_crc                        |
    +--------------+------------------+----------------------------------+
    | employees    |           300024 | 4ec56ab5ba37218d187cf6ab09ce1aa1 |
    | departments  |                9 | d1af5e170d2d1591d776d5638d71fc5f |
    | dept_manager |               24 | 8720e2f0853ac9096b689c14664f847e |
    | dept_emp     |           331603 | ccf6fe516f990bdaa49713fc478701b7 |
    | titles       |           443308 | bfa016c472df68e70a03facafa1bc0a8 |
    | salaries     |          2844047 | fd220654e95aea1b169624ffe3fca934 |
    +--------------+------------------+----------------------------------+
    +--------------+---------------+-----------+
    | table_name   | records_match | crc_match |
    +--------------+---------------+-----------+
    | employees    | OK            | ok        |
    | departments  | OK            | ok        |
    | dept_manager | OK            | ok        |
    | dept_emp     | OK            | ok        |
    | titles       | OK            | ok        |
    | salaries     | OK            | ok        |
    +--------------+---------------+-----------+


## DISCLAIMER

To the best of my knowledge, this data is fabricated and
it does not correspond to real people. 
Any similarity to existing people is purely coincidental.


## LICENSE
This work is licensed under the 
Creative Commons Attribution-Share Alike 3.0 Unported License. 
To view a copy of this license, visit 
http://creativecommons.org/licenses/by-sa/3.0/ or send a letter to 
Creative Commons, 171 Second Street, Suite 300, San Francisco, 
California, 94105, USA.


