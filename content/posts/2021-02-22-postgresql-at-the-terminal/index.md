---
title: PostgreSQL at the terminal
author: Jack Quarm
date: '2021-02-22'
slug: []
categories:
  - Databases
tags:
  - postgres
  - terminal
  - linux
---

# Getting started

PostgreSQL is a fantastic DBMS, especially for data analysis and features loads of extensions that you can use that make life a lot easier. Often times data analysis with Python and R focus on reading and writing data from files in CSV or JSON format and SQL is taught as a separate entity, but lets try to bridge the gap and get started working with databases.

### Logging into POSTGRES in a terminal

After installing linux, switch to the user 'postgres'
``` go {linenos=inline}
su postgres
```

Then launch postgres using the `psql` command in the terminal.

To start off, change the password of this user by running the SQL command:
```go {linenos=inline}
ALTER USER postgres WITH PASSWORD 'passwordGoesHere';
```

Great, you're up and running and now have access to the postgres terminal. If your terminal window isn't wide enough to show all of the results you'd like to see, you can enable or disable the extended view mode by using the following command:

```go {linenos=inline}
\x
```

If your request returns many results, they'll be shown in a new window. This new window will have the word (END) at the bottom of the page. Don't worry, you can exit this window by pressing `q`. 

You can also exit the postgres terminal using:
```go {linenos=inline}
\q
```

### Basic commands to list contents

To list all available databases:
```go {linenos=inline}
\l
```

After connecting to a database, view all tables within a database:
```go {linenos=inline}
\dt
```

After connecting to a database, see a description of a specific table:
```go {linenos=inline}
\d <TableName>
```

## Connecting to a database though the terminal

To connect to a database from the terminal, make sure that you're either the user "postgres" or have the correct permissions to use `psql` then run the following:

```go {linenos=inline}
psql -h localhost -p 5432 -U postgres <DatabaseName> 
```
Then you'll be promted for the password for the user 'postgres'.

* `-h` Stands for "host"
* `-p` Stands for "port" - the default value is 5432
* `-U` Stands for "user"
* The last argument is obviously the name of the database.

Alternatively, you can connect to psql from the terminal then use this to connect directly to the database:
```go {linenos=inline}
\c <DatabaseName>
```

# Basics

### Creating databases

```go {linenos=inline}
CREATE DATABASE <DatabaseName>
```

### Removing databases

```go {linenos=inline}
DROP DATABASE <DatabaseName>
```

### Adding tables to a database

The basic syntax for adding new tables to a database follows this formula:
```go {linenos=inline}
CREATE TABLE table_name (
  Column name + data type + constraints if there are any
)
```

Here is an example of adding a table to the "test" database:

```go {linenos=inline}
\c test
```
```sql {linenos=inline}
CREATE TABLE person (
id INT,
first_name VARCHAR(50),
last_name VARCHAR(50),
date_of_birth DATE );
```

This is a good start, but this is not an example of best practices. The following code specifies constraints that enforce data validity:

```go {linenos=inline}
\c test
```
```sql {linenos=inline}
CREATE TABLE person (
id BIGSERIAL NOT NULL PRIMARY KEY,
first_name VARCHAR(50) NOT NULL,
last_name VARCHAR(50) NOT NULL,
date_of_birth DATE NOT NULL);
```

### Altering tables

Ah whoops, looks like I entered the table without the `gender` variable, lets see if we can add it after we've already created the table:

```go {linenos=inline}
ALTER TABLE person
ADD COLUMN gender VARCHAR(6) NOT NULL;
```

### Descriptions of tables

To see the column names in a table and their associated datatypes you can use the following query:

```go {linenos=inline}
SELECT column_name, data_type FROM information_schema.columns`</br>
WHERE table_name = '<Table_name>';
```

### Inserting records into tables

The basic syntax for adding records into tables follows this formula:

```go {linenos=inline}
INSERT INTO person (
first_name,
last_name,
gender,
date_of_birth)
VALUES ('Anne', 'Smith', 'FEMALE', DATE '1988-02-22');
```

Lets add some data to our table. As we don't have the personal data of 1000 people on hand, lets generate some data using the site [Mockaroo](https://mockaroo.com). First specify the names of the columns you want and how many entries you want generated, then save the file as `.sql`.

Instead of writing 1000 `INSERT` statements in the terminal, lets modify the `.sql` file that we generated to include our data validation steps and use this file to load in our new rows.

To run these instructions from a file in the psql console, use the `\i` command followed by the location of the file, which for me, was:

```go {linenos=inline}
\i /home/jack/Downloads/person.sql
```

### Deleting records

Normally you'll want to delete specific records, however you can also delete entries based on conditional logic using `WHERE`. To delete a specific element you use `DELETE`:
```go {linenos=inline}
DELETE FROM <TableName> WHERE id = <id>;
```

### Updating records

`UPDATE` allows us to update a record based on one or more columns: 

```go {linenos=inline}
UPDATE <TableName> SET <Column> = <Value>
WHERE <PrimaryKey> = <Value>;
```

In our table there is somone called `Augusto` who has a `NULL` email. You can update his record by using `UPDATE`, specifying the columns we want to update using `SET`, then specifying that we're updting only his record by selecting his primary key with `WHERE`:

```go {linenos=inline}
UPDATE person SET email = 'augusto42@gmail.com'
WHERE id = 234;
```
To do this with multiple columns, just specify more of them and separate them with a comma:

```go {linenos=inline}
UPDATE person SET email = 'felicia1992@hotmail.com', country_of_birth = 'Scotland'
WHERE id = 235;
```
# Queries

### Basic operators

* SELECT
* WHERE
* ORDER BY
* DISTINCT
* LIMIT

### Getting started

To return **all columns** from a table use the command:
```go {linenos=inline}
SELECT * FROM person;`
```

To select **specific columns**, just specify them followed by a comma:
```go {linenos=inline}
SELECT first_name, email FROM person;
```

To **order** the results (the default is ascending) specify the column that you want to order by:

```go {linenos=inline}
SELECT first_name, email FROM person
ORDER BY email;
```
If you want to view **only unique values** ordered alphabetically, use the DISTINCT keyword then order by:

```go {linenos=inline}
SELECT DISTINCT country FROM person
ORDER BY country;
```

If you want to view only the records that fit **specific levels**, you can specify these with the WHERE keyword:

```go {linenos=inline}
SELECT * FROM person
WHERE GENDER = 'MALE' AND country = 'Iceland';
```

To return only a **specific number of levels** specify the number of records you want returned using the LIMIT keyword:

```go {linenos=inline}
SELECT * FROM person
WHERE GENDER = 'Male' LIMIT 10;
```

If you want to do this starting from row 5 for example, you can specify this using the OFFSET keyword:

```go {linenos=inline}
SELECT * FROM person
WHERE GENDER = 'Male'
OFFSET 5 LIMIT 10;
```

# Queries with multiple conditions

Multiple conditions can be specified after **AND** statements, just wrap them with `()`.

```go {linenos=inline}
SELECT * FROM person
WHERE gender = 'FEMALE' AND (date_of_birth > '1980-01-01' OR date_of_birth < '1955-01-01')
ORDER BY date_of_birth;
```

To avoid having to write **OR** multiple times in statements like this:

```go {linenos=inline}
SELECT * FROM person
WHERE country_of_birth = 'China' OR country_of_birth = 'Brazil' OR country_of_birth = 'France'
ORDER BY country_of_birth, date_of_birth;
```

You can instead, use the **IN** keyword, which takes an array of values:

```go {linenos=inline}
SELECT * FROM person
WHERE country_of_birth IN ('China', 'Brazil', 'France')
ORDER BY country_of_birth, date_of_birth;
```

# More Keywords

### Limit, Offset and Fetch

```go {linenos=inline}
SELECT * FROM person
FETCH FIRST 10 ROWS ONLY;
```

### Between

The **BETWEEN** keyword allows you to filter out rows by specifying a start and an end:

```go {linenos=inline}
SELECT * FROM person
WHERE date_of_birth BETWEEN '1950-01-01' AND '1955-01-01'
ORDER BY date_of_birth;
```

### LIKE and ILIKE

`LIKE` and `ILIKE` can be used for pattern matching for returning values that match certain contions, pretty much like regular expressions.

The `LIKE` keyword is also **case sensitive**, whereas **ILIKE** is not:

```go {linenos=inline}
SELECT * FROM person
WHERE email LIKE '%mozilla%';
```

The query above will return any results that contain the characters **mozilla**. The wildcard `%` means that there can be anything in front or behind the string.

The `_` underscore character means that the query will have to match single characters, for example to get all of the three letter names from the database:

```go {linenos=inline}
SELECT first_name FROM person
WHERE first_name LIKE '___'
ORDER BY first_name;
```

### GROUP BY

The `GROUP BY` condition is used in conjunction with an **Aggregate function** to return sets of values. The following query will return a list of countries and the number of people from the database who come from those countries. This works by calling the **COUNT(*)** function: 

```go {linenos=inline}
SELECT country_of_birth, COUNT(*) FROM person
GROUP BY country_of_birth
ORDER BY COUNT(*) DESC;
```

Combining `GROUP BY` with `DISTINCT` can be a useful query. Here's an example:

```go {linenos=inline}
SELECT make, COUNT(DISTINCT model) AS n_models FROM car
GROUP BY make
ORDER BY n_models DESC;
```

### HAVING

This is used to add additional filters after making an aggregation. For example, with the query above, we aggregate by counting the number of results for each country in the table. Using the `HAVING` keyword, we can then filter the aggregated results using more conditions:

```go {linenos=inline}
SELECT country_of_birth, COUNT(*) FROM person
GROUP BY country_of_birth
HAVING COUNT(*) > 20
ORDER BY COUNT(*) DESC;
```

### Calculating MAX, MIN and SUM

To use the `max` and `min` functions, they take the column names as arguments. The following query will return the max price. Super simple.

```go {linenos=inline}
SELECT MAX(price) FROM car;
```

To get the average of all car prices, you can use the `AVG()` keyword in the exact same way. You'll want to use this with the `ROUND()` keyword otherwise you'll get a result with a weird number of sigfigs.

```go {linenos=inline}
SELECT ROUND(AVG(price)) FROM car;
```

To find the average price of each make, we can execute the following with `GROUP BY`:

```go {linenos=inline}
SELECT make, ROUND(AVG(price)) FROM car
GROUP BY make
ORDER BY AVG(price);
```

To find the sum of all of the cars, we can use the `SUM()` keyword:

```go {linenos=inline}
SELECT SUM(price) FROM car;
```

To find the sum the prices of all of the cars, we do the same thing but use the `GROUP BY` keyword again:

```go {linenos=inline}
SELECT make, SUM(price) FROM car
GROUP BY make
ORDER BY SUM(price);
```

### Naming variables with AS

Say we wanted to know what half of the price would be for every car in our table? We can return custom variables by creating them using `SELECT`. If we want to return everything but we also wanted to know half of the price of each car with 2 decimal places:

```go {linenos=inline}
SELECT *, ROUND(price * 0.5, 2) FROM car;
```

This works, but it returns all of our results under the column name `round` which isn't helpful. Lets change that by using another keyword, `AS`:

```go {linenos=inline}
SELECT *, ROUND(price * 0.5, 2) AS half_price FROM car;
```

### COALESCE and NULLIF

The `COALESCE` keyword takes unlimited arguments and will return the first argument that is not `NULL`.  

This is great for returning empty values as a string of your choice, say for example you want to return a list of all of the emails from the `person` table, yet some of them are missing, you can use `COALESCE` to convert them to a string such as `NA: Email not provided`. 

In the code below, `COALESCE` will return the email, as long as there is an email, and if there is not, it will return the second argument you give it, in this case this is the string `NA: Email not provided`.

```go {linenos=inline}
SELECT COALESCE(email, 'NA: Email not provided') FROM person;
```

This behaves similarly to `NULLIF` which will return a `NULL` as log as the arguments that you provide it match. For example, `NULLIF(20, 20)` will return a `NULL`, however `NULLIF(20, 3)` will return `20`.

```go {linenos=inline}
SELECT *, NULLIF(country_of_birth, 'China') AS null_if_china FROM person;
```

# Exporting results as a CSV

It's great to be able to see queries interactively but it's pretty not that useful unless you're looking for something specific, so how do we actually save queries as something useful like a `.csv` file?

The `COPY()` keyword allows us to do this, just wrap your query inside `COPY()` and specify a file type and a file location:

```go {linenos=inline}
COPY (
    SELECT person.first_name, person.last_name, person.country_of_birth, car.make, car.model, car.price
    FROM person
    JOIN car ON person.car_id=car.id
) TO '/home/jack/Documents/CodeStuff/SQL/Postgres/ExampleQuery.csv'
WITH CSV HEADER;
```

# Working with Timestmps and Dates

The `NOW()` kw will give us the current timestamp in date-time y-m-d h-m-s + TZ format with the resolution of `microseconds`. To get only the `date` from this timestamp, you can `cast` this as a date by using the `::` operator, so to get the current date yo ucan use:

```go {linenos=inline}
SELECT NOW():: DATE;
```

**Note**: Look up the timestamp and date/time datatypes.

### Adding and Sutracting with Dates

The `INERVAL` kw to calculate time intervals, for example:

```go {linenos=inline}
SELECT NOW() - INTERVAL '1 YEAR';

SELECT NOW() + INTERVAL '14 MONTHS';

SELECT NOW() - INTERVAL '1233 DAYS';
```

To be able to `cast` the timestamp to a date using `INTERVAL`, you'll need to wrap the call to `SELECT` in brakets:

```go {linenos=inline}
SELECT (NOW() + INTERVAL '164 DAYS'):: DATE;
```

### Extracting Fields

The `EXTRACT` kw to extract certain fields from the timestamp such as the day, month or year:

```go {linenos=inline}
SELECT EXTRACT (YEAR FROM NOW());
```
### The AGE function

The `AGE` kw can be used to convert a date to an age. This keyword takes two arguments which define the age based on a period of time: `AGE(To, From)`. For example in the person table, if we wanted to calculate the age of an employee, we would specify `AGE(NOW(), date_of_birth)` or `AGE(NOW()::DATE, date_of_birth)`:

```go {linenos=inline}
SELECT first_name, last_name, AGE(date_of_birth) AS age FROM person
ORDER BY age DESC
LIMIT 20;
```

# Primary Keys

### Adding a Primary Key

Use `ALTER TABLE person ADD PRIMARY KEY (id);` to add a primary key to a table.

### Filtering by Primary Key

Use `WHERE id` to filter results based on the primary key.

```go {linenos=inline}
SELECT * FROM person
WHERE id = 3;
```

### Unique constraints

Additional constraints can be added to the table by specifying them using `ADD CONSTRAINT`. To add a constraint that forces all email addresses to be unique, we could use the following:

```go {linenos=inline}
ALTER TABLE person ADD CONTRAINT unique_email_address UNIQUE(email);
```

Another example of a constraint could be if you want to limit the possible values that a variable can take, for example, if you're working with survey data where options for gender only include male or female, you might want to enforce a constraint where the value for gender can only take thes values. You could do this with the `person` table by enforcing the following using the `CHECK` kw:

```go {linenos=inline}  
ALTER TABLE person ADD CONSTRAINT gender_constraint CHECK (gender = 'FEMALE' OR gender = 'MALE');
```
To view all of the constraints applied to a table, you can use:

```go {linenos=inline}
\dt+ <TableName>
```

# Creating UUIDs

To create a universally unique identifier (`uuid`) to use a primary instead of a `bigserial`, we can use the extension built into Postgres. This can be enabled with:

```go {linenos=inline}
CREATE EXTENSION 'uuid-ossp';
```

To generate an example of one of these `uuid`s, use:

```go {linenos=inline}
SELECT uuid_generate_v4();
```

# On Conflict, Do Nothing

To avoid conflicts and errors, especialy when doing table inserts, you can make use of the `ON CONFLICT()` and `DO NOTHING` kws. For this to work, you will need to either specify a `primary key` or an otherwise `unique` column. If you try to make a table insert where the primary key is already taken, you can add an extra line to prevent throwing errors:

```go {linenos=inline}
INSERT INTO person (id, first_name, last_name, gender, email, date_of_birth, country_of_birth)
VALUES (235, 'Felicia', 'Stranio', 'FEMALE', DATE '1976-10-03', 'felicia1991@hotmail.com', 'Scotland')
ON CONFLICT (id) DO NOTHING;
```

# Upsert

This allows you to override existing data if there are any conflicts present. Take the example above, imagine if she just registered all of her information but a few seconds later she wanted to change her email address? Instead of `DO NOTHING` when there is a conflict on `id`, we can update the email instead using `UPDATE SET email = EXCLUDED.email;`:

```go {linenos=inline}
INSERT INTO person (id, first_name, last_name, gender, email, date_of_birth, country_of_birth)
VALUES (235, 'Felicia', 'Stranio', 'FEMALE', DATE '1976-10-03', 'felicia1991@gmail.com', 'Scotland')
ON CONFLICT (id) DO UPDATE SET email = EXCLUDED.email;
```

# Foreign Keys, Joins and Relationships

A `foreign key` is a key that references a `primary key` in another table. Let's create a new `.sql` file called `person-car` that contains the code to create two tables. The `CREATE TABLE` code in this file will be almost identical but we'll add foreign keys to our `person` table:

```sql {linenos=inline}
create table car (
	id BIGSERIAL NOT NULL PRIMARY KEY,
	make VARCHAR(100) NOT NULL,
	model VARCHAR(100) NOT NULL,
	price NUMERIC(19,2) NOT NULL
);

create table person (
	id BIGSERIAL NOT NULL PRIMARY KEY,
	first_name VARCHAR(50) NOT NULL,
	last_name VARCHAR(50) NOT NULL,
	gender VARCHAR(7) NOT NULL,
	email VARCHAR(100) NOT NULL,
	date_of_birth DATE NOT NULL,
	country_of_birth VARCHAR(50) NOT NULL,
    car_id BIGINT REFERENCES car (id),
    UNIQUE(car_id)
);
```

OK, so now that the tables exist and we've added data to them, let's assign a car to `Bradford`:

```go {linenos=inline}
UPDATE person SET car_id = 2 WHERE id = 2;
```

# Inner joins

An inner join aims to bind two tables `A` and `B`, the result will return all of the data in the intersection of the two like a venn diagram `A+B = C`. This means that with an inner join, the only results that will be returned, are those that have foreign keys. In the example of joining `car` on `person`, the only results that are returned, are those with a `car_id`.

Lets start by doing an inner join between our two tables `person` and `car`. In this case, our foreign key `car_id` in the `person` table links to the primary key `id` in the `car` table. This is done using the `JOIN` and `ON` kws. The correct syntax for this is :

```go {linenos=inline}
SELECT * FROM <TableA>
JOIN <TableB> ON <TableA>.ForeignKey = <TableB>.PrimaryKey;
```

So for our table join we would use:

```go {linenos=inline}
SELECT * FROM person
JOIN car ON person.car_id = car.id;
```

This isn't super useful as we don't get to choose the information that we want returned, so lets make it more flexible:

```go {linenos=inline}
SELECT first_name, last_name, country_of_birth, car.make, car.price
FROM person
JOIN car ON person.car_id = car.id;
```

### Left Joins

In the last query where we returned `*` from `person` and joined `car` on `person.car_id`, we were able to get `all` of the information about them while excluding everyone who didn't have a car. Well in this case, we can get all of this information `while including` people who `don't` have a car.

The syntax for this is identical to the last query except we add the kw `LEFT`:

```go {linenos=inline}
SELECT * FROM person
LEFT JOIN car ON person.car_id = car.id;
```

This can also be used to return only the results who do not have cars for example:

```go {linenos=inline}
SELECT * FROM person
LEFT JOIN car ON person.car_id = car.id
WHERE car.* IS NULL;
```

# Extensions

PostgreSQL is designed to be easily extensible and includes a load of additional functions that can give your databases more functionality such as the ability to generate `uuid`s. To view all of the available extensions use:

```go {linenos=inline}
SELECT * FROM pg_a
```

# UUIDs as primary keys

`uuid`s are never the same and are therefor perfect for primary keys. On top of security concerns, these make it so that you can combine data from different databases without having clashes based on having conflicting `BIGSERIAL` id types. To convert our previous database to use UUIDs instead of BIGSERIAL we have to make a few changes: 
  * Firstly we have to add a new column to our `VALUES` to specify our ids and change the column name of our ids to `person_uid` and `car_uid` whereas we did not have to specify this before.
  * We have to invoke the `uuid_generate_v4()` function to create the new UUID each time data is inserted.
  * We have to change the datatypes from `BIGSERIAL` to `UUID`
  * We also have to change the way that we refernce the `car_uid`



```sql {linenos=inline}
create table car (
	car_uid UUID NOT NULL PRIMARY KEY,
	make VARCHAR(100) NOT NULL,
	model VARCHAR(100) NOT NULL,
	price NUMERIC(19,2) NOT NULL
);

create table person (
	person_uid UUID NOT NULL PRIMARY KEY,
	first_name VARCHAR(50) NOT NULL,
	last_name VARCHAR(50) NOT NULL,
	gender VARCHAR(20) NOT NULL,
	email VARCHAR(100) NOT NULL,
	date_of_birth DATE NOT NULL,
	country_of_birth VARCHAR(50) NOT NULL,
    car_uid UUID REFERENCES car(car_uid),
    UNIQUE(car_uid),
	UNIQUE(email)
);

insert into person (person_uid, first_name, last_name, gender, email, date_of_birth, country_of_birth) values (uuid_generate_v4(), 'Chrisy', 'Heistermann', 'Genderfluid', 'cheistermann0@last.fm', '1938/08/21', 'Cuba');

insert into person (person_uid, first_name, last_name, gender, email, date_of_birth, country_of_birth) values (uuid_generate_v4(), 'Bradford', 'Dearnly', 'Genderqueer', 'bdearnly1@xinhuanet.com', '1969/07/19', 'Russia');

insert into person (person_uid, first_name, last_name, gender, email, date_of_birth, country_of_birth) values (uuid_generate_v4(), 'Kalina', 'Fransson', 'Genderfluid', 'kfransson2@who.int', '1961/06/28', 'Sweden');


insert into car (car_uid, make, model, price) values (uuid_generate_v4(), 'Mitsubishi', 'Starion', '98737.48');

insert into car (car_uid, make, model, price) values (uuid_generate_v4(), 'Ford', 'LTD', '29127.55');
```

# Window Functions

The best way to understand window functions is to review `aggregate functions`. The following example calculates the average price of all of the products in the `products` table:

```go {linenos=inline}
SELECT AVG(price) FROM products;`
```

To apply this function to subsets of rows, you use the `GROUP BY` clause. The following example will return the average price for each group of products.

```go {linenos=inline}
`SELECT group_name, AVG(price) FROM products
INNER JOIN product_groups USING (group_id)
GROUP BY group_name;
```

The `AVG()` function clearly reduces the number of rows returned by the query in both examples. The term `window` describes the set of rows on which the window function operates. A window function returns values from the rows inside the window. The following query returns the product name, the price, product group and average prices of each product group.

```go {linenos=inline}
SELECT product_name, price, group_name, AVG(price) OVER(PARTITON BY group_name)
FROM products
INNER JOIN product_groups USING(group_id);
```

In this case, the `AVG()` function works as a `window function` that operates on a set of rows defined by the `OVER` clause. Each set of rows is called a `window`.

The new syntax for this query is the `OVER` clause:

```go {linenos=inline}
AVG(price) OVER (PARTITION BY group_name)
```

In this syntax, the `PARTITION BY` clause distributes the rows of the result set into groups and the `AVG()` function is applied to each group to return the average price for each

### Window function syntax

PostgreSQL has a sophisticaed syntax for window function calls. The following illustrates the simplified version:

```go {linenos=inline}
window_function(arg1, arg2, arg3...) OVER (
	[PARTITION BY partition expression]
	[ORDER BY sort_expresison [ASC | DESC] [NULLS {FIRST | LAST}]]
)
```

The `PARTITION BY` clause is optional and divides rows into `multiple groups` or partitions to which the window function is applied. Like the example above, we used the product group to divide the products into groups (or parititons).

The `frame_clause` defines a subset of rows in the current partition to wihch the window function is applied. Tis subset of rows is called a `frame`.

If you use multiple window functions in a query:
```go {linenos=inline}
SELECT
wf1() OVER(PARTITION BY c1 ORDER BY c2),
wf1() OVER(PARTITION BY c1 ORDER BY c2),
FROM table_name;
```

You can use the `WINDOW` clause the shorten this query:

```go {linenos=inline}
SELECT
wf1() OVER w1
wf2() OVER w2
FROM table_name
WINDOW w AS (PARTITION BY c1 ORDER BY c2);
```

You can also use the `WINDOW` clause even if you only have one window function in your query:

```go {linenos=inline}
SELECT wf1() OVER w1
FROM table_name
WINDOW w AS (PARTITION BY c1 ORDER BY c2);
```

















