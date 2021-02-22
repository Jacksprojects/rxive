---
title: "PostgreSQL at the terminal"
author: "Jack Quarm"
date: "1012-01-01"
tags:
- test
title: Test 1
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

# Getting started with PostgreSQL in Linux

## Logging into POSTGRES in a terminal

First start by switching to user 'postgres'
`su postgres`

Then launch postgres using the `psql` command in the terminal.

To start off, change the password of this user by running the SQL command `ALTER USER postgres WITH PASSWORD 'passwordGoesHere';`

# Listing Contents

## Viewing databases

Use `\l` to list all available databases.

## Viewing tables within a database

After connecting to a database, use `\dt` to see all available tables.

##  Viewing descriptions of tables

After connecting to a database, use `\d <TableName>` to see a description of a specific table

## Connecting to a database though the terminal

To connect to a database from the terminal, make sure that you're the user "postgres" then run the following:

`psql -h localhost -p 5432 -U postgres <DatabaseName>` Then you'll be promted for the password for the user 'postgres'.

`-h` Stands for "host"</br>
`-p` Stands for "port" - the default value is 5432</br>
`-U` Stands for "user"</br> 
The last argument is obviously the name of the database.

Alternatively, you can connect to psql from the terminal then use the `\c <DatabaseName>` command to connect directly to the database.

# Basics

## Creating databases

`CREATE DATABASE <DatabaseName>`

## Removing databases

`DROP DATABASE <DatabaseName>`

## Adding tables to a database

The basic syntax for adding new tables to a database follows this formula:

`CREATE TABLE table_name (`</br>
 &nbsp;  &nbsp;  &nbsp;  &nbsp; `Column name + data type + constraints if there are any`</br>
`)`</br>

Here is an example of adding a table to the "test" database:

`\c test`</br>
`CREATE TABLE person (`</br>
`id INT,`</br>
`first_name VARCHAR(50),`</br>
`last_name VARCHAR(50),`</br>
`date_of_birth DATE );`</br>

This is a good start, but this is not an example of best practices. The following code specifies constraints that enforce data validity.

`\c test`</br>
`CREATE TABLE person (`</br>
`id BIGSERIAL NOT NULL PRIMARY KEY,`</br>
`first_name VARCHAR(50) NOT NULL,`</br>
`last_name VARCHAR(50) NOT NULL,`</br>
`date_of_birth DATE ) NOT NULL;`</br>

Ah whoops, looks like I entered the table without the `gender` variable, lets see if we can add it after we've already created the table:

`ALTER TABLE person`</br>
`ADD COLUMN gender VARCHAR(6) NOT NULL;`</br>


## Descriptions of tables

To see the column names and their data types you can use the following query:

`SELECT column_name, data_type FROM information_schema.columns`</br>
`WHERE table_name = '<Table_name>';`</br>

## Inserting records into tables

The basic syntax for adding records into tables follows this formula:

`INSERT INTO person (`</br>
`	first_name,`</br>
`	last_name,`</br>
`	gender,`</br>
`	date_of_birth)`</br>
`VALUES ('Anne', 'Smith', 'FEMALE', DATE '1988-02-22');`</br>

Lets add 1000 new people using the site 'Mockaroo'. First specify the names of the columns you want and how many entries you want generated, then save the file as .sql

To run instructions from a file in the psql console, use the `\i` command followed by the location of the file which is this case was `\i /home/jack/Downloads/person.sql`. 

## Deleting records

* Normally you'll want to delete specific records, however you can also delete entries based on conditional logic using the `WHERE` kw. To delete a specific element you use the `DELETE` kw:

`DELETE FROM <TableName> WHERE id = <id>;`

## Updating records

* The  `UPDATE` kw allows us to update a record based on one or more columns. 

`UPDATE <TableName> SET <Column> = <Value>`</br>
`WHERE <PrimaryKey> = <Value>;`

* In our table there is somone called `Augusto` who has a `NULL` email. You can update his record by using `UPDATE`, specifying the columns we want to update using `SET`, then specifying that we're updting only his record by selecting his primary key with `WHERE`:

`UPDATE person SET email = 'augusto42@gmail.com'`</br>
`WHERE id = 234;`

* To do this with multiple columns, just specify more of them and separate them with a comma:

`UPDATE person SET email = 'felicia1992@hotmail.com', country_of_birth = 'Scotland'`</br>
`WHERE id = 235;`

# Basic Queries

## Basic verbs

* **SELECT**
* **WHERE**
* **ORDER BY**
* **DISTINCT**
* **LIMIT**

## Getting started

To return **all columns** from a table use the command:
`SELECT * FROM person;`</br>

To select **specific columns**, just specify them followed by a comma:

`SELECT first_name, email FROM person;`</br>

To **order** the results (the default is ascending) specify the column that you want to order by:

`SELECT first_name, email FROM person`</br>
`ORDER BY email;`</br>

If you want to view **only unique values** ordered alphabetically, use the DISTINCT keyword then order by:

`SELECT DISTINCT country FROM person`</br>
`ORDER BY country;`

If you want to view only the records that fit **specific levels**, you can specify these with the WHERE keyword:

`SELECT * FROM person`</br>
`WHERE GENDER = 'MALE' AND country = 'Iceland';`

To return only a **specific number of levels** specify the number of records you want returned using the LIMIT keyword:

`SELECT * FROM person`</br>
`WHERE GENDER = 'Male' LIMIT 10;`

If you want to do this starting from row 5 for example, you can specify this using the OFFSET keyword:

`SELECT * FROM person`</br>
`WHERE GENDER = 'Male'`</br>
`OFFSET 5 LIMIT 10;`

# Queries with multiple conditions

* Multiple conditions can be specified after **AND** statements, just gotta wrap 'em with `()`

`SELECT * FROM person`</br>
`WHERE gender = 'FEMALE' AND (date_of_birth > '1980-01-01' OR date_of_birth < '1955-01-01')`</br>
`ORDER BY date_of_birth;`</br>

* To speed up writing **OR** statements if you have multiple conditons, you want to avoid repetition like this:

`SELECT * FROM person`</br>
`WHERE country_of_birth = 'China' OR country_of_birth = 'Brazil' OR country_of_birth = 'France'`</br>
`ORDER BY country_of_birth, date_of_birth;`</br>

* Instead, use the **IN** keyword, which takes an array of values:

`SELECT * FROM person`</br>
`WHERE country_of_birth IN ('China', 'Brazil', 'France')`</br>
`ORDER BY country_of_birth, date_of_birth;`</br>

# More Keywords

## Limit, Offset and Fetch

`SELECT * FROM person`</br>
`FETCH FIRST 10 ROWS ONLY;`</br>

## Between

* The **BETWEEN** keyword allows you to filter out rows by specifying a start and an end:

`SELECT * FROM person`</br>
`WHERE date_of_birth BETWEEN '1950-01-01' AND '1955-01-01'`</br>
`ORDER BY date_of_birth;`</br>

## LIKE and ILIKE

* `LIKE` and `ILIKE` can be used for pattern matching for returning values that match certain contions, pretty much like regular expressions:
* The `LIKE` keyword is also **case sensitive**, whereas **ILIKE** is not.

`SELECT * FROM person`</br>
`WHERE email LIKE '%mozilla%';`</br>

* The query above will return any results that contain the characters **mozilla**. The wildcard `%` means that there can be anything infront or behind the string.
* The `_` underscore character means that the query will have to match single characters, for example to get all of the three letter names from the database:

`SELECT first_name FROM person`</br>
`WHERE first_name LIKE '___'`</br>
`ORDER BY first_name;`</br>


## GROUP BY

* The `GROUP BY` condition is used in conjunction with an **Aggregate function** to return sets of values. The following query will return a list of countries and the number of people from the database who come from those countries. This works by calling the **COUNT(*)** function: 

`SELECT country_of_birth, COUNT(*) FROM person`</br>
`GROUP BY country_of_birth`</br>
`ORDER BY COUNT(*) DESC;`</br>

* Combining `GROUP BY` with `DISTINCT` can be a useful query. Here's an example:

`SELECT make, COUNT(DISTINCT model) AS n_models FROM car` </br>
`GROUP BY make` </br>
`ORDER BY n_models DESC;`


## HAVING

* This is used to add additional filters after making an aggregation. For example, with the query above, we aggregate by counting the number of results for each country in the table. Using the `HAVING` keyword, we can then filter the aggregated results using more conditions:

`SELECT country_of_birth, COUNT(*) FROM person` </br>
`GROUP BY country_of_birth` </br>
`HAVING COUNT(*) > 20` </br>
`ORDER BY COUNT(*) DESC;`</br>


## Calculating MAX, MIN and SUM

To use the `max` and `min` functions, they take the column names as arguments. The following query will return the max price. Simple.

`SELECT MAX(price) FROM car;` <br>

To get the average of all car prices, you can use the `AVG()` keyword in the exact same way. You'll want to use this with the `ROUND()` keyword otherwise you'll get a result with a stupid number of sigfigs.

`SELECT ROUND(AVG(price)) FROM car;` <br>

To find the average price of each make, we can execute the following with `GROUP BY`:

`SELECT make, ROUND(AVG(price)) FROM car`<br>
`GROUP BY make`<br>
`ORDER BY AVG(price);`

To find the sum of all of the cars, we can use the `SUM()` keyword:

`SELECT SUM(price) FROM car;`

To find the sum the prices of all of the cars, we do the same thing but use the `GROUP BY` keyword again:

`SELECT make, SUM(price) FROM car`<br>
`GROUP BY make`
`ORDER BY SUM(price);`

## Returning altered values

Say we wanted to know what half of the price would be for eveyr car in our table? We can return custom variables by creating them using `SELECT`. If we want to return everything but we also wanted to know half of the price of each car with 2 decimal places:

`SELECT *, ROUND(price * 0.5, 2) FROM car;`

This works, but it returns all of our results under the column name `round` which sucks, lets see if we can change that by using another keyword, `AS`:

`SELECT *, ROUND(price * 0.5, 2) AS half_price FROM car;`

# How to export results as CSV

* It's great to be able to see queries interactively but it's pretty pointless unless you're looking for something specific, so how do we actually save queries as something useful like a `.csv` file?
* The `COPY()` keyword allows us to do this, just wrap your query inside `COPY()` and specify a file type and a file location:





`COPY (`</br>
    `SELECT person.first_name, person.last_name, person.country_of_birth, car.make, car.model, car.price`</br>
    `FROM person`</br>
    `JOIN car ON person.car_id=car.id`</br>
`) TO '/home/jack/Documents/CodeStuff/SQL/Postgres/ExampleQuery.csv'`</br>
`WITH CSV HEADER;`

# COALESCE and NULLIF

The `COALESCE` keyword takes unlimited arguments and will return the first argument that is not `NULL`.  

This is great for returning empty values as a string of your choice, say for example you want to return a list of all of the emails from the `person` table, yet some of them are missing, you can use `COALESCE` to convert them to a string such as `NA: Email not provided`. 

In the code below, `COALESCE` will return the email, as long as there is an email, and if there is not, it will return the second argument you give it, in this case this is the string `NA: Email not provided`.

`SELECT COALESCE(email, 'NA: Email not provided') FROM person;`

This behaves similarly to `NULLIF` which will return a `NULL` as log as the arguments that you provide it match. For example, `NULLIF(20, 20)` will return a `NULL`, however `NULLIF(20, 3)` will return `20`.

SELECT *, NULLIF(country_of_birth, 'China') AS null_if_china FROM person;


# Working with Timestmps and Dates

* The `NOW()` kw will give us the current timestamp in date-time y-m-d h-m-s + TZ format with the resolution of `microseconds`.
* To get only the `date` from this timestamp, you can `cast` this as a date by using the `::` operator, so to get the current date yo ucan use:

`SELECT NOW():: DATE;`

**Note**: Look up the timestamp and date/time datatypes.

## Adding and Sutracting with Dates

* The `INERVAL` kw to calculate time intervals, for example:

`SELECT NOW() - INTERVAL '1 YEAR';` </br>

`SELECT NOW() + INTERVAL '14 MONTHS';` </br>

`SELECT NOW() - INTERVAL '1233 DAYS';`

* To be able to `cast` the timestamp to a date using `INTERVAL`, you'll need to wrap the call to `SELECT` in brakets:

`SELECT (NOW() + INTERVAL '164 DAYS'):: DATE;`

## Extracting Fields

* The `EXTRACT` kw to extract certain fields from the timestamp such as the day, month or year:

`SELECT EXTRACT (YEAR FROM NOW());` </br>

## The AGE function

* The `AGE` kw can be used to convert a date to an age.
* This keyword takes two arguments which define the age based on a period of time: `AGE(To, From)`
* For example in the person table, if we wanted to calculate the age of an employee, we would specify `AGE(NOW(), date_of_birth)` or `AGE(NOW()::DATE, date_of_birth)`:

`SELECT first_name, last_name, AGE(date_of_birth) AS age FROM person` </br>
`ORDER BY age DESC` </br>
`LIMIT 20;` </br>

# Primary Keys

## Adding a Primary Key

* Use `ALTER TABLE person ADD PRIMARY KEY (id);` to add a primary key to a table.

## Filtering by Primary Key

* Use `WHERE id` to filter results based on the primary key.

`SELECT * FROM person`</br>
`WHERE id = 3;`

## Unique constraints

* Additional constraints can be added to the table by specifying them using `ADD CONSTRAINT`.
* To add a constraint that forces all email addresses to be unique, we could use the following:

`ALTER TABLE person ADD CONTRAINT unique_email_address UNIQUE(email);`

* Another example of a constraint could be if you want to limit the possible values that a variable can take, for example if you're working with binary gender information, you might want to enforce a constraint where the value fo gender could ould be male or female.
* You could do this with the `person` table by enforcing the following using the `CHECK` kw:
  
`ALTER TABLE person ADD CONSTRAINT gender_constraint CHECK (gender = 'FEMALE' OR gender = 'MALE');`

* To view all of the constraints applied to a table, you can use `\dt+ <TableName>`.

# Creating UUIDs

* To create a universally unique identifier (`uuid`) to use a primary instead of a `bigserial`, we can use the extension built into Postgres. To enable the extension use:

`CREATE EXTENSION 'uuid-ossp';`</br>

* To see an example of one of these `uuid`s, use:

`SELECT uuid_generate_v4();`

# On Conflict, Do Nothing

* To avoid conflicts and errors, especialy when doing table inserts, you can make use of the `ON CONFLICT()` and `DO NOTHING` kws.
* For this to work, you will need to either specify a `primary key` or an otherwise `unique` column.
* If you try to make a table insert where the primary key is already taken, you can add an extra line to prevent throwing errors:

`INSERT INTO person (id, first_name, last_name, gender, email, date_of_birth, country_of_birth)` </br>
`VALUES (235, 'Felicia', 'Stranio', 'FEMALE', DATE '1976-10-03', 'felicia1991@hotmail.com', 'Scotland')` </br>
`ON CONFLICT (id) DO NOTHING;`

# Upsert

* This allows you to override existing data if there are any conflicts present. **Not too sure why you would use this and not just `UPDATE` but ok...**
* Take the exmaple above, imagine if she just registered all of her information but a few seconds later she wanted to changer her email address? Instead of `DO NOTHING` when there is a conflict on `id`, we can update the email instead using `UPDATE SET email = EXCLUDED.email;`:


`INSERT INTO person (id, first_name, last_name, gender, email, date_of_birth, country_of_birth)` </br>
`VALUES (235, 'Felicia', 'Stranio', 'FEMALE', DATE '1976-10-03', 'felicia1991@gmail.com', 'Scotland')` </br>
`ON CONFLICT (id) DO UPDATE SET email = EXCLUDED.email;`

# Foreign Keys, Joins and Relationships

* First of all, a `foreign key` is a key that references a `primary key` in another table.
* Let's create a new sql file called `person-car` that contains the code to create two tables. The `CREATE TABLE` code in this file will be almost identical but we'll add foreign keys to our `person` table:

```
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

* OK, so now that the tables exist and we've added data to them, let's assign a car to `Bradford`:

`UPDATE person SET car_id = 2 WHERE id = 2;`

## Inner joins

* An inner join aims to bind two tables `A` and `B`, the result will return all of the data in the intersection of the two like a venn diagram `A+B = C`. This means that with an inner join, the only results that will be returned, are those that have foreign keys. In the example of joining `car` on `person`, the only results that are returned, are those with a `car_id`.
* Lets start by doing an inner join between our two tables `person` and `car`. In this case, our foreign key `car_id` in the `person` table links to the primary key `id` in the `car` table.
* This is done using the `JOIN` and `ON` kws. The correct syntax is :

`SELECT * FROM <TableA>`</br>
`JOIN <TableB> ON <TableA>.ForeignKey = <TableB>.PrimaryKey;`

* So for our table join we would do:

`SELECT * FROM person`</br>
`JOIN car ON person.car_id = car.id;`

* This isn't super useful as we don't get to choose the information that we want returned, so lets make it more flexible:

`SELECT first_name, last_name, country_of_birth, car.make, car.price`</br>
`FROM person`</br>
`JOIN car ON person.car_id = car.id;`

## Left Joins

*  So, in the last query where we returned `*` from `person` and joined `car` on `person.car_id`, we were able to get `all` of the information about them while excluding everyone who didn't have a car. Well in this case, we can get all of this information `while including` people who `don't` have a car.
* The syntax for this is identical as the last query except we add the kw `LEFT`:

`SELECT * FROM person` </br>
`LEFT JOIN car ON person.car_id = car.id;`

* This can also be used to return only the results who do not have cars for example:

`SELECT * FROM person` </br>
`LEFT JOIN car ON person.car_id = car.id;` </br>
`WHERE car.* IS NULL;`

# Extensions

* PostgreSQL is designed to be easily extensible and includes a load of additional functions that can give your databases more functionality such as the ability to generate `uuid`s.
* To view all of the available extensions use the `SELECT * FROM pg_a`

# UUIDs as primary keys

* `uuid`s are never the same and are therefor perfect for primary keys. On top of security concerns, these make it so that you can combine data from different databases without having clashes based on having conflicting `BIGSERIAL` id types.
* To convert our previous database to use UUIDs instead of BIGSERIAL we have to make a few changes: 
  * Firstly we have to add a new column to our `VALUES` to specify our ids and change the column name of our ids to `person_uid` and `car_uid` whereas we did not have to specify this before.
  * We have to invoke the `uuid_generate_v4()` function to create the new UUID each time data is inserted.
  * We have to change the datatypes from `BIGSERIAL` to `UUID`
  * We also have to change the way that we refernce the `car_uid`



```
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

* The best way to understand window functions is to review `aggregate functions`. The following example calculates the average price of all of the products in the `products` table:

`SELECT AVG(price) FROM products;`</br>

* To apply this function to subsets of rows, you use the `GROUP BY` clause. The following example will return the average price for each group of products.

`SELECT group_name, AVG(price) FROM products`</br>
`INNER JOIN product_groups USING (group_id)`</br>
`GROUP BY group_name;`

* The `AVG()` function clearly reduces the number of rows returned by the query in both examples. The term `window` describes the set of rows on which the window function operates. A window function returns values from the rows inside the window.
* The following query returns the product name, the price, product group and avaerage prices of each product group.

`SELECT product_name, price, group_name, AVG(price) OVER(PARTITON BY group_name)`</br>
`FROM products`</br>
`INNER JOIN product_groups USING(group_id);`

* In this case, the `AVG()` function works as a `window function` that operates on a set of rows defined by the `OVER` clause. Each set of rows is called a `window`.
* The new syntax for this query is the `OVER` clause:
  
`AVG(price) OVER (PARTITION BY group_name)`

* In this syntax, the `PARTITION BY` clause distributes the rows of the result set into groups and the `AVG()` function is applied to each group to return the average price for each

## Window function syntax

* PostgreSQL has a sophisticaed syntax for window function calls. The following illustrates the simplified version:

```
window_function(arg1, arg2, arg3...) OVER (
	[PARTITION BY partition expression]
	[ORDER BY sort_expresison [ASC | DESC] [NULLS {FIRST | LAST}]]
)
```

* The `PARTITION BY` clause is optional and divides rows into `multiple groups` or partitions to which the window function is applied. Like the example above, we used the product group to divide the products into groups (or parititons).
* The `frame_clause` defines a subset of rows in the current partition to wihch the window function is applied. Tis subset of rows is called a `frame`.
* If you use multiple window functions in a query:

`SELECT`</br>
`wf1() OVER(PARTITION BY c1 ORDER BY c2),`</br>
`wf1() OVER(PARTITION BY c1 ORDER BY c2),`</br>
`FORM table_name;`

* You can use the `WINDOW` clause the shorten this query:

`SELECT` </br>
`wf1() OVER w1` </br>
`wf2() OVER w2` </br>`
`FROM table_name`
`WINDOW w AS (PARTITION BY c1 ORDER BY c2);`

* You can also use the `WINDOW` clause even if you only have one window function in your query:

`SELECT wf1() OVER w1` </br>
`FROM table_name` </br>
`WINDOW w AS (PARTITION BY c1 ORDER BY c2);`

# The ROW_NUMBER(), RANK() and DENSE_RANK functions

* These three functions assign an integer to each row based on its order in its result set.

















