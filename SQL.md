SQL Queries with R Notebook
================
Kaymal

SQL Queries with R Notebook
===========================

Outline: 
- Installing Necessary Packages 
- Connection - Basic Queries
  *
  *

Installing Necessary Packages
-----------------------------

``` r
#install.packages("RMariaDB")
library(DBI)
library(RMariaDB)

# Install config for credentials
# install.packages("config")
```

Connection
----------

``` r
db <- config::get("DB")
```

``` r
con <- dbConnect(RMariaDB::MariaDB(),
      user = db$user, password =db$password, dbname = "sakila", port=3306)
```

``` sql
SHOW TABLES
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
