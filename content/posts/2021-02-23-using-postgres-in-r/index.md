---
title: Using PostgreSQL in R
author: Jack Quarm
date: '2021-02-23'
slug: using-postgres-in-r
categories:
  - Databases
  - R
tags:
  - postgres
  - R
---

# Getting started

This post follows on from my last [blogpost](https://jacksprojects.netlify.app/posts/2021-02-22-postgresql-at-the-terminal/) about using PostgreSQL and goes without saying that you'll have to have PostgreSQL installed on your system before you can start using it, so make use it's installed and you can follow along. You can follow the instructions [here](https://www.postgresql.org/download/) to get started on whichever OS you're using.

### Connecting to postgres in R

To connect to Postgres in R, the first thing you'll need are the correct packages and a connection. First off, install `RPostgreSQL` and the `DBI` package if you haven't already.

As shown in the last blog post, to connect to a postgres database, you'll need a few pieces of information:

* username
* password
* host (if you're working locally, this will be 'localhost')
* port number (default is 5432)
* database name

You can just enter all of these as plain text if you're working locally, but its a bad habit, so we'll store some of these sensitive credentials as environment variables in case we're sharing our code with others.

To create these we can use `Sys.setenv()`. To create an environment variable for the username and password we can use: 

```r {linenos=inline}
print(Sys.setenv(user = 'YourUserName')) 
print(Sys.setenv(passwd = 'YourPassword'))
``` 

These should return `TRUE` once entered in the R console and we can double check that they work by entering `Sys.getenv("user")` and `Sys.getenv("passwd")` in the console.


Now that we have our environment variables, we can load the `RPostgreSQL` package and create a connection string. The database i'll be connecting to here is called 'test'. If you haven't set up any databases yet, check out my last [blogpost](https://jacksprojects.netlify.app/posts/2021-02-22-postgresql-at-the-terminal/) about using PostgreSQL.

```r {linenos=inline}
library(`RPostgreSQL`)

con <- dbConnect(PostgreSQL(), 
                host="localhost",
                user= Sys.getenv("user"), 
                password=Sys.getenv("passwd"), 
                dbname="test", 
                port="5432")
```

Now that we have a connection sting, we can  use this in conjunction with the `dbX()` functions to get information from our databases. Lets see what tables we have available in the `test` database:

```r {linenos=inline}
dbListTables(con)
```

```
## [1] "products"       "car"            "person"         "product_groups"
```

According to this, we have multiple tables to choose from. Lets query the `cars` table:

```r {linenos=inline}
dbGetQuery(con, 
'SELECT * FROM car LIMIT 10;')
```

```
##                                car_uid       make   model    price
## 1 7b41d89f-0489-432e-bad7-94a68462ccf0 Mitsubishi Starion 98737.48
## 2 b4b8b46d-2a7c-4f54-a065-772ad053a140       Ford     LTD 29127.55
```

Now, there are two main ways that we can make queries from r. We can either use `dbSendQuery()` followed by `dbFetchQuery()` which allows us to write queries in batches and requires us to afterwards use `dbClearResult()`, or we can just use `dbGetQuery()`. We'll stick to using this as we won't be doing complicated queries.

Now that we've made our first query lets close the connection to our database with

```r {linenos=inline}
dbDisconnect(con)
```

```
## [1] TRUE
```

Now, from looking at the data in this `test` database, it's looking a bit sparse, so lets add some more data to our database. Lets use the `gapminder` dataset that can be found [here](https://raw.githubusercontent.com/resbaz/r-novice-gapminder-files/master/data/gapminder-FiveYearData.csv). Download this and we can add it to our database.

Load in this new data and check that its in good shape.

```r {linenos=inline}
library(dplyr)
gapminder <- read.csv("~/Documents/CodeStuff/Datasets/gapminder.csv")
glimpse(gapminder)
```

```
## Rows: 1,704
## Columns: 6
## $ country   <fct> Afghanistan, Afghanistan, Afghanistan, Afghanistan, Afghani…
## $ year      <int> 1952, 1957, 1962, 1967, 1972, 1977, 1982, 1987, 1992, 1997,…
## $ pop       <dbl> 8425333, 9240934, 10267083, 11537966, 13079460, 14880372, 1…
## $ continent <fct> Asia, Asia, Asia, Asia, Asia, Asia, Asia, Asia, Asia, Asia,…
## $ lifeExp   <dbl> 28.801, 30.332, 31.997, 34.020, 36.088, 38.438, 39.854, 40.…
## $ gdpPercap <dbl> 779.4453, 820.8530, 853.1007, 836.1971, 739.9811, 786.1134,…
```

```r {linenos=inline}
str(gapminder)
```

```
## 'data.frame':    1704 obs. of  6 variables:
##  $ country  : Factor w/ 142 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
##  $ pop      : num  8425333 9240934 10267083 11537966 13079460 ...
##  $ continent: Factor w/ 5 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
##  $ gdpPercap: num  779 821 853 836 740 ...
```

Looks good. Lets connect to out database and create a new table using this data.

```r {linenos=inline}
con <- dbConnect(PostgreSQL(), 
                host="localhost",
                user= Sys.getenv("user"), 
                password=Sys.getenv("passwd"), 
                dbname="test", 
                port="5432")

dbWriteTable(con, "gapminder", gapminder)
```

```
## [1] TRUE
```
Now lets delete our local version of this dataset and start making queries.

```r {linenos=inline}
gapminder <- NULL

dbGetQuery(con,
           'SELECT * FROM gapminder LIMIT 10;'
           )
```

```
##    row.names     country year      pop continent lifeExp gdpPercap
## 1          1 Afghanistan 1952  8425333      Asia  28.801  779.4453
## 2          2 Afghanistan 1957  9240934      Asia  30.332  820.8530
## 3          3 Afghanistan 1962 10267083      Asia  31.997  853.1007
## 4          4 Afghanistan 1967 11537966      Asia  34.020  836.1971
## 5          5 Afghanistan 1972 13079460      Asia  36.088  739.9811
## 6          6 Afghanistan 1977 14880372      Asia  38.438  786.1134
## 7          7 Afghanistan 1982 12881816      Asia  39.854  978.0114
## 8          8 Afghanistan 1987 13867957      Asia  40.822  852.3959
## 9          9 Afghanistan 1992 16317921      Asia  41.674  649.3414
## 10        10 Afghanistan 1997 22227415      Asia  41.763  635.3414
```

Which where the 10 most populated countries in 1997?

```r {linenos=inline}
dbGetQuery(con,
           'SELECT gapminder.country, gapminder.pop, gapminder.year FROM gapminder 
           WHERE year = 1997
           ORDER BY gapminder.pop DESC
           LIMIT 10;'
           )
```

```
##          country        pop year
## 1          China 1230075000 1997
## 2          India  959000000 1997
## 3  United States  272911760 1997
## 4      Indonesia  199278000 1997
## 5         Brazil  168546719 1997
## 6       Pakistan  135564834 1997
## 7          Japan  125956499 1997
## 8     Bangladesh  123315288 1997
## 9        Nigeria  106207839 1997
## 10        Mexico   95895146 1997
```

Or we could just read the entire table into R as a dataframe and use dplyr to get the same results. This is a pretty small dataset so we can read it this way, but avoid doing this for larger files.

```r {linenos=inline}
gapminder <- dbReadTable(con, "gapminder")

gapminder %>%
  select(country, pop, year) %>%
  filter(year == 1997) %>%
  arrange(desc(pop)) %>%
  head(n=10)
```

```
##            country        pop year
## 298          China 1230075000 1997
## 706          India  959000000 1997
## 1618 United States  272911760 1997
## 718      Indonesia  199278000 1997
## 178         Brazil  168546719 1997
## 1174      Pakistan  135564834 1997
## 802          Japan  125956499 1997
## 106     Bangladesh  123315288 1997
## 1138       Nigeria  106207839 1997
## 994         Mexico   95895146 1997
```

And that's pretty much it, you're now up and running in R with postgres! Just don't forget to close your connection afterwards!

```r {linenos=inline}
dbDisconnect(con)
```
