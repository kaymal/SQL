SQL Intro: Querying Databases
================
Kaymal

Structured Query Language (SQL) is a language for interacting with data stored in a *relational database*. The focus of this notebook is **querying databases**, yet one can also use SQL to modify a database or to create one. [Sakila](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=2ahUKEwipzNLVifHgAhURqaQKHXPWAfoQFjAAegQIBhAB&url=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Fsakila%2Fen%2F&usg=AOvVaw2NuLo1XbYVnyU8rBTLsipX) Database is used in this study.

Outline:

-   Installing Necessary Packages
-   Connection and Credentials
-   Basic SQL Commands
    -   Viewing the Data
    -   Selecting Data
        -   SELECT
        -   DISTINCT
        -   LIMIT
        -   COUNT
    -   Filtering Results
        -   WHERE
        -   WHERE AND/OR
        -   BETWEEN
        -   WHERE IN
        -   IS NULL/IS NOT NULL
        -   LIKE/NOT LIKE
    -   Aggregate Functions
        -   COUNT
        -   AVG, MAX, MIN, SUM
        -   WHERE + Aggregate Function
    -   Sorting, Grouping, Merging Data
        -   ORDER BY
        -   GROUP BY
        -   HAVING

Installing Necessary Packages
-----------------------------

``` r
#install.packages("RMariaDB")
library(DBI)
library(RMariaDB)
library(RMySQL)

library(knitr)
```

``` r
# Install config for credentials
# install.packages("config")
library(config)
```

Connection and Credentials
--------------------------

For security reasons, it is always better not to write username or passwords in a Notebook. Creating a `config.yml` file and reading in the credentials from that is one option for security.

``` r
db <- config::get("DB")
```

``` r
# Initiate a connection with RMySQL

con = dbConnect(RMySQL::MySQL(),
      user = db$user, password = db$password, dbname = "sakila", port = 3306)

# Alternatively, initiate a connection with RMariaDB
#con = dbConnect(RMariaDB::MariaDB(),
#      user = db$user, password =db$password, dbname = "sakila", port=3306)

# Remove credentials
rm(db)
```

Basic SQL Commands
------------------

### Viewing the Data

``` sql
-- Print names of the tables in the 'sakila' Database
SHOW TABLES;
```

| Tables\_in\_sakila |
|:-------------------|
| actor              |
| actor\_info        |
| address            |
| category           |
| city               |
| country            |
| customer           |
| customer\_list     |
| film               |
| film\_actor        |

### Selecting Data

#### SELECT

We can select data **FROM** a table using the **SELECT** keyword.

``` sql
SELECT 'First query'
AS first_result;
```

| first\_result |
|:--------------|
| First query   |

``` sql
-- Select all columns from the 'film' table
SELECT *
FROM film;
```

| film\_id | title            | description                                                                                                           | release\_year |  language\_id|  original\_language\_id|  rental\_duration|  rental\_rate|  length|  replacement\_cost| rating | special\_features                | last\_update        |
|:---------|:-----------------|:----------------------------------------------------------------------------------------------------------------------|:--------------|-------------:|-----------------------:|-----------------:|-------------:|-------:|------------------:|:-------|:---------------------------------|:--------------------|
| 1        | ACADEMY DINOSAUR | A Epic Drama of a Feminist And a Mad Scientist who must Battle a Teacher in The Canadian Rockies                      | 2006          |             1|                      NA|                 6|          0.99|      86|              20.99| PG     | Deleted Scenes,Behind the Scenes | 2006-02-15 05:03:42 |
| 2        | ACE GOLDFINGER   | A Astounding Epistle of a Database Administrator And a Explorer who must Find a Car in Ancient China                  | 2006          |             1|                      NA|                 3|          4.99|      48|              12.99| G      | Trailers,Deleted Scenes          | 2006-02-15 05:03:42 |
| 3        | ADAPTATION HOLES | A Astounding Reflection of a Lumberjack And a Car who must Sink a Lumberjack in A Baloon Factory                      | 2006          |             1|                      NA|                 7|          2.99|      50|              18.99| NC-17  | Trailers,Deleted Scenes          | 2006-02-15 05:03:42 |
| 4        | AFFAIR PREJUDICE | A Fanciful Documentary of a Frisbee And a Lumberjack who must Chase a Monkey in A Shark Tank                          | 2006          |             1|                      NA|                 5|          2.99|     117|              26.99| G      | Commentaries,Behind the Scenes   | 2006-02-15 05:03:42 |
| 5        | AFRICAN EGG      | A Fast-Paced Documentary of a Pastry Chef And a Dentist who must Pursue a Forensic Psychologist in The Gulf of Mexico | 2006          |             1|                      NA|                 6|          2.99|     130|              22.99| G      | Deleted Scenes                   | 2006-02-15 05:03:42 |

``` sql
-- Select the 'title' column from the 'film' table
SELECT title
FROM film;
```

| title            |
|:-----------------|
| ACADEMY DINOSAUR |
| ACE GOLDFINGER   |
| ADAPTATION HOLES |
| AFFAIR PREJUDICE |
| AFRICAN EGG      |

``` sql
-- Select the 'title' and 'release_year' columns from the 'film' table
SELECT title, release_year
FROM film;
```

| title            | release\_year |
|:-----------------|:--------------|
| ACADEMY DINOSAUR | 2006          |
| ACE GOLDFINGER   | 2006          |
| ADAPTATION HOLES | 2006          |
| AFFAIR PREJUDICE | 2006          |
| AFRICAN EGG      | 2006          |

#### DISTINCT

When we want to return the unique values in a column, we use **DISTINCT** keyword.

``` sql
-- Get different types of 'rating' values. 
SELECT DISTINCT rating
FROM film;
```

| rating |
|:-------|
| PG     |
| G      |
| NC-17  |
| PG-13  |
| R      |

#### LIMIT

We can use `LIMIT` to return a certain number of lines.

``` sql
-- Select all columns from the 'film' table. Limit the output with 2 results.
SELECT *
FROM film
LIMIT 2;
```

| film\_id | title            | description                                                                                          | release\_year |  language\_id|  original\_language\_id|  rental\_duration|  rental\_rate|  length|  replacement\_cost| rating | special\_features                | last\_update        |
|:---------|:-----------------|:-----------------------------------------------------------------------------------------------------|:--------------|-------------:|-----------------------:|-----------------:|-------------:|-------:|------------------:|:-------|:---------------------------------|:--------------------|
| 1        | ACADEMY DINOSAUR | A Epic Drama of a Feminist And a Mad Scientist who must Battle a Teacher in The Canadian Rockies     | 2006          |             1|                      NA|                 6|          0.99|      86|              20.99| PG     | Deleted Scenes,Behind the Scenes | 2006-02-15 05:03:42 |
| 2        | ACE GOLDFINGER   | A Astounding Epistle of a Database Administrator And a Explorer who must Find a Car in Ancient China | 2006          |             1|                      NA|                 3|          4.99|      48|              12.99| G      | Trailers,Deleted Scenes          | 2006-02-15 05:03:42 |

#### COUNT

``` sql
-- Get the number of rows in the 'film' table
SELECT COUNT(*)
FROM film;
```

| COUNT(\*) |
|:----------|
| 1000      |

``` sql
-- Print the number of distinct 'rating's in the 'film' table
SELECT COUNT(DISTINCT rating)
FROM film;
```

| COUNT(DISTINCT rating) |
|:-----------------------|
| 5                      |

### Filtering Results

#### WHERE

**WHERE** is used to filter the results. We use comparison operators (`=`, `<=`, `<`, `<>`, etc.) along with WHERE. Note that we use `=` instead of `==`, and `<>` instead of `!=`.

``` sql
-- Select all details for films which are released in 2006
SELECT *
FROM film
WHERE release_year = 2006;
```

| film\_id | title            | description                                                                                                           | release\_year |  language\_id|  original\_language\_id|  rental\_duration|  rental\_rate|  length|  replacement\_cost| rating | special\_features                | last\_update        |
|:---------|:-----------------|:----------------------------------------------------------------------------------------------------------------------|:--------------|-------------:|-----------------------:|-----------------:|-------------:|-------:|------------------:|:-------|:---------------------------------|:--------------------|
| 1        | ACADEMY DINOSAUR | A Epic Drama of a Feminist And a Mad Scientist who must Battle a Teacher in The Canadian Rockies                      | 2006          |             1|                      NA|                 6|          0.99|      86|              20.99| PG     | Deleted Scenes,Behind the Scenes | 2006-02-15 05:03:42 |
| 2        | ACE GOLDFINGER   | A Astounding Epistle of a Database Administrator And a Explorer who must Find a Car in Ancient China                  | 2006          |             1|                      NA|                 3|          4.99|      48|              12.99| G      | Trailers,Deleted Scenes          | 2006-02-15 05:03:42 |
| 3        | ADAPTATION HOLES | A Astounding Reflection of a Lumberjack And a Car who must Sink a Lumberjack in A Baloon Factory                      | 2006          |             1|                      NA|                 7|          2.99|      50|              18.99| NC-17  | Trailers,Deleted Scenes          | 2006-02-15 05:03:42 |
| 4        | AFFAIR PREJUDICE | A Fanciful Documentary of a Frisbee And a Lumberjack who must Chase a Monkey in A Shark Tank                          | 2006          |             1|                      NA|                 5|          2.99|     117|              26.99| G      | Commentaries,Behind the Scenes   | 2006-02-15 05:03:42 |
| 5        | AFRICAN EGG      | A Fast-Paced Documentary of a Pastry Chef And a Dentist who must Pursue a Forensic Psychologist in The Gulf of Mexico | 2006          |             1|                      NA|                 6|          2.99|     130|              22.99| G      | Deleted Scenes                   | 2006-02-15 05:03:42 |

#### WHERE AND/OR

``` sql
-- Select all details for films which are released in 2006 and have rating 'G'
SELECT *
FROM film
WHERE release_year = 2006
AND rating = 'G';
```

|  film\_id| title             | description                                                                                                           | release\_year |  language\_id|  original\_language\_id|  rental\_duration|  rental\_rate|  length|  replacement\_cost| rating | special\_features              | last\_update        |
|---------:|:------------------|:----------------------------------------------------------------------------------------------------------------------|:--------------|-------------:|-----------------------:|-----------------:|-------------:|-------:|------------------:|:-------|:-------------------------------|:--------------------|
|         2| ACE GOLDFINGER    | A Astounding Epistle of a Database Administrator And a Explorer who must Find a Car in Ancient China                  | 2006          |             1|                      NA|                 3|          4.99|      48|              12.99| G      | Trailers,Deleted Scenes        | 2006-02-15 05:03:42 |
|         4| AFFAIR PREJUDICE  | A Fanciful Documentary of a Frisbee And a Lumberjack who must Chase a Monkey in A Shark Tank                          | 2006          |             1|                      NA|                 5|          2.99|     117|              26.99| G      | Commentaries,Behind the Scenes | 2006-02-15 05:03:42 |
|         5| AFRICAN EGG       | A Fast-Paced Documentary of a Pastry Chef And a Dentist who must Pursue a Forensic Psychologist in The Gulf of Mexico | 2006          |             1|                      NA|                 6|          2.99|     130|              22.99| G      | Deleted Scenes                 | 2006-02-15 05:03:42 |
|        11| ALAMO VIDEOTAPE   | A Boring Epistle of a Butler And a Cat who must Fight a Pastry Chef in A MySQL Convention                             | 2006          |             1|                      NA|                 6|          0.99|     126|              16.99| G      | Commentaries,Behind the Scenes | 2006-02-15 05:03:42 |
|        22| AMISTAD MIDSUMMER | A Emotional Character Study of a Dentist And a Crocodile who must Meet a Sumo Wrestler in California                  | 2006          |             1|                      NA|                 6|          2.99|      85|              10.99| G      | Commentaries,Behind the Scenes | 2006-02-15 05:03:42 |

If we want to combine `AND` and `OR`, we need to use paranthesis.

``` sql
-- Select all details for films which are released in 2006 and have rating 'G' or 'R'
SELECT *
FROM film
WHERE release_year = 2006
AND (rating = 'G' OR rating = 'R');
```

|  film\_id| title            | description                                                                                                           | release\_year |  language\_id|  original\_language\_id|  rental\_duration|  rental\_rate|  length|  replacement\_cost| rating | special\_features              | last\_update        |
|---------:|:-----------------|:----------------------------------------------------------------------------------------------------------------------|:--------------|-------------:|-----------------------:|-----------------:|-------------:|-------:|------------------:|:-------|:-------------------------------|:--------------------|
|         2| ACE GOLDFINGER   | A Astounding Epistle of a Database Administrator And a Explorer who must Find a Car in Ancient China                  | 2006          |             1|                      NA|                 3|          4.99|      48|              12.99| G      | Trailers,Deleted Scenes        | 2006-02-15 05:03:42 |
|         4| AFFAIR PREJUDICE | A Fanciful Documentary of a Frisbee And a Lumberjack who must Chase a Monkey in A Shark Tank                          | 2006          |             1|                      NA|                 5|          2.99|     117|              26.99| G      | Commentaries,Behind the Scenes | 2006-02-15 05:03:42 |
|         5| AFRICAN EGG      | A Fast-Paced Documentary of a Pastry Chef And a Dentist who must Pursue a Forensic Psychologist in The Gulf of Mexico | 2006          |             1|                      NA|                 6|          2.99|     130|              22.99| G      | Deleted Scenes                 | 2006-02-15 05:03:42 |
|         8| AIRPORT POLLOCK  | A Epic Tale of a Moose And a Girl who must Confront a Monkey in Ancient India                                         | 2006          |             1|                      NA|                 6|          4.99|      54|              15.99| R      | Trailers                       | 2006-02-15 05:03:42 |
|        11| ALAMO VIDEOTAPE  | A Boring Epistle of a Butler And a Cat who must Fight a Pastry Chef in A MySQL Convention                             | 2006          |             1|                      NA|                 6|          0.99|     126|              16.99| G      | Commentaries,Behind the Scenes | 2006-02-15 05:03:42 |

#### BETWEEN

``` sql
-- Select all details from the 'table'sales_by_film_category' table where total sales are between 4500 and 5000.
SELECT *
FROM sales_by_film_category
WHERE total_sales
BETWEEN 4500 AND 5000;
```

| category  |  total\_sales|
|:----------|-------------:|
| Sci-Fi    |       4756.98|
| Animation |       4656.30|
| Drama     |       4587.39|

#### WHERE IN

When we need to filter based on multiple values, we can use `WHERE IN` instead of multiple `OR`s.

``` sql
-- Select all details from the 'table'sales_by_film_category' for the specified categories.
SELECT *
FROM sales_by_film_category
WHERE category IN ('Animation', 'Action', 'Drama')
```

| category  |  total\_sales|
|:----------|-------------:|
| Animation |       4656.30|
| Drama     |       4587.39|
| Action    |       4375.85|

#### IS NULL/IS NOT NULL

`NULL` is particularly usefull when we want to learn whether there is a missing/unknown value in data.

``` sql
-- Select all details from the 'film_list' where 'category' is missing.
SELECT *
FROM film_list
WHERE category IS NULL;
```

Table: 0 records

FID title description category price length rating actors ---- ------ ------------ --------- ------ ------- ------- -------

``` sql
-- Count the number of missing values of the  'original_language_id' coulmn from the 'film' table.
SELECT COUNT(*)
FROM film
WHERE original_language_id IS NULL;
```

| COUNT(\*) |
|:----------|
| 1000      |

#### LIKE/ NOT LIKE

We can use `LIKE` and `NOT LIKE` for filtering by a *pattern*. Wildcard `%` is used to match zero or more characters, while `_` is used to match a single character.

``` sql
-- Select title from film where title starts with MAT.
SELECT title
FROM film
WHERE title LIKE 'MAT%';
```

| title          |
|:---------------|
| MATRIX SNOWMAN |

``` sql
-- Select title from film where title's second and third characters are A and T respectively.
SELECT title
FROM film
WHERE title LIKE '_AT%';
```

| title              |
|:-------------------|
| CAT CONEHEADS      |
| CATCH AMISTAD      |
| DATE SPEED         |
| FATAL HAUNTED      |
| GATHERING CALENDAR |
| HATE HANDICAP      |
| MATRIX SNOWMAN     |
| NATIONAL STORY     |
| NATURAL STOCK      |
| PATHS CONTROL      |

### Agregate Functions

#### COUNT

#### AVG, MAX, MIN, SUM

#### WHERE + Aggregate Function

### Sorting, Grouping, Merging Data

#### ORDER BY

#### GROUP BY

#### HAVING

------------------------------------------------------------------------

``` r
# Disconnect
dbDisconnect(con)
```
