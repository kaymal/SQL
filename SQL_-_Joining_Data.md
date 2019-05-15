SQL: Joining Data
================
Kaymal

Outline:

  - Intro
  - INNER JOIN
      - USING
      - Self-Join
      - CASE, WHEN, THEN, ELSE, and END
      - INTO
  - Outer JOIN
      - LEFT JOIN
      - RIGHT JOIN
      - FULL JOIN
      - CROSS JOIN
  - Set Theory
      - UNION
      - UNION ALL
      - INTERSECT
      - EXCEPT
      - Semi-Join and Anti-Join
  - Subqueries (Nested Queries)
      - Inside WHERE
      - Inside SELECT
      - Inside FROM

## Introduction

``` r
# Install the package in R
#install.packages("RPostgreSQL")
library(RPostgreSQL)
```

    ## Loading required package: DBI

``` r
## Loading required package: DBI
pgdrv <- dbDriver(drvName = "PostgreSQL")

con <-DBI::dbConnect(pgdrv,
                    dbname="template1",
                    host="localhost", port=5432,
                    user = "username",
                    password = "password")
```

## INNER JOIN

``` sql
SELECT * 
FROM cities
```

| name        | country\_code | city\_proper\_pop | metroarea\_pop | urbanarea\_pop |
| :---------- | :------------ | ----------------: | -------------: | -------------: |
| Abidjan     | CIV           |           4765000 |             NA |        4765000 |
| Abu Dhabi   | ARE           |           1145000 |             NA |        1145000 |
| Abuja       | NGA           |           1235880 |        6000000 |        1235880 |
| Accra       | GHA           |           2070460 |        4010050 |        2070460 |
| Addis Ababa | ETH           |           3103670 |        4567860 |        3103670 |
| Ahmedabad   | IND           |           5570580 |             NA |        5570580 |
| Alexandria  | EGY           |           4616620 |             NA |        4616620 |
| Algiers     | DZA           |           3415810 |        5000000 |        3415810 |
| Almaty      | KAZ           |           1703480 |             NA |        1703480 |
| Ankara      | TUR           |           5271000 |        4585000 |        5271000 |

Displaying records 1 - 10

``` sql
SELECT COUNT(*) 
FROM cities
```

| count |
| :---- |
| 236   |

1 records

``` sql
SELECT * 
FROM countries
```

| code | name                 | continent     | region                    | surface\_area | indep\_year | local\_name                        | gov\_form                    | capital          | cap\_long | cap\_lat |
| :--- | :------------------- | :------------ | :------------------------ | ------------: | ----------: | :--------------------------------- | :--------------------------- | :--------------- | --------: | -------: |
| AFG  | Afghanistan          | Asia          | Southern and Central Asia |        652090 |        1919 | Afganistan/Afqanestan              | Islamic Emirate              | Kabul            |    691761 |   345228 |
| NLD  | Netherlands          | Europe        | Western Europe            |         41526 |        1581 | Nederland                          | Constitutional Monarchy      | Amsterdam        |    489095 |   523738 |
| ALB  | Albania              | Europe        | Southern Europe           |         28748 |        1912 | Shqiperia                          | Republic                     | Tirane           |    198172 |   413317 |
| DZA  | Algeria              | Africa        | Northern Africa           |       2381740 |        1962 | Al-Jazair/Algerie                 | Republic                     | Algiers          |    305097 |   367397 |
| ASM  | American Samoa       | Oceania       | Polynesia                 |           199 |          NA | Amerika Samoa                      | US Territory                 | Pago Pago        |  \-170691 | \-142846 |
| AND  | Andorra              | Europe        | Southern Europe           |           468 |        1278 | Andorra                            | Parliamentary Coprincipality | Andorra la Vella |     15218 |   425075 |
| AGO  | Angola               | Africa        | Central Africa            |       1246700 |        1975 | Angola                             | Republic                     | Luanda           |     13242 | \-881155 |
| ATG  | Antigua and Barbuda  | North America | Caribbean                 |           442 |        1981 | Antigua and Barbuda                | Constitutional Monarchy      | Saint John’s     |  \-618456 |   171175 |
| ARE  | United Arab Emirates | Asia          | Middle East               |         83600 |        1971 | Al-Imarat al-´Arabiya al-Muttahida | Emirate Federation           | Abu Dhabi        |    543705 |   244764 |
| ARG  | Argentina            | South America | South America             |       2780400 |        1816 | Argentina                          | Federal Republic             | Buenos Aires     |  \-584173 | \-346118 |

Displaying records 1 - 10

``` sql
SELECT cities.name AS city, countries.name AS country, countries.region 
FROM cities
  -- Inner join to countries
  INNER JOIN countries
    -- Match on the country codes
    ON cities.country_code = countries.code;
```

| city        | country              | region                    |
| :---------- | :------------------- | :------------------------ |
| Abidjan     | Cote d’Ivoire        | Western Africa            |
| Abu Dhabi   | United Arab Emirates | Middle East               |
| Abuja       | Nigeria              | Western Africa            |
| Accra       | Ghana                | Western Africa            |
| Addis Ababa | Ethiopia             | Eastern Africa            |
| Ahmedabad   | India                | Southern and Central Asia |
| Alexandria  | Egypt                | Northern Africa           |
| Algiers     | Algeria              | Northern Africa           |
| Almaty      | Kazakhstan           | Southern and Central Asia |
| Ankara      | Turkey               | Middle East               |

Displaying records 1 - 10

Get data from both the countries and economies tables to examine the
inflation rate for both 2010 and 2015.

``` sql
-- Select fields with aliases
SELECT c.code AS country_code, c.name, year, inflation_rate
FROM countries AS c
  -- Join to economies (alias e)
  INNER JOIN economies AS e
    -- Match on code
    ON c.code = e.code;
```

| country\_code | name        | year | inflation\_rate |
| :------------ | :---------- | ---: | --------------: |
| AFG           | Afghanistan | 2010 |            2179 |
| AFG           | Afghanistan | 2015 |          \-1549 |
| AGO           | Angola      | 2010 |            1448 |
| AGO           | Angola      | 2015 |           10287 |
| ALB           | Albania     | 2010 |            3605 |

Displaying records 1 - 5

Get the country name, its region, and the fertility rate for both 2010
and 2015 for each country.

``` sql
-- Select fields
SELECT c.code, c.name, c.region, p.year, fertility_rate
  -- 1. From countries (alias as c)
  FROM countries AS c
  -- 2. Join with populations (as p)
  INNER JOIN populations AS p
    -- 3. Match on country code
    ON c.code = p.country_code
```

| code | name        | region                    | year | fertility\_rate |
| :--- | :---------- | :------------------------ | ---: | --------------: |
| ABW  | Aruba       | Caribbean                 | 2010 |            1704 |
| ABW  | Aruba       | Caribbean                 | 2015 |            1647 |
| AFG  | Afghanistan | Southern and Central Asia | 2010 |            5746 |
| AFG  | Afghanistan | Southern and Central Asia | 2015 |            4653 |
| AGO  | Angola      | Central Africa            | 2010 |            6416 |

Displaying records 1 - 5

``` sql
-- Select fields
SELECT c.code, name, region, e.year, fertility_rate, unemployment_rate
  -- From countries (alias as c)
  FROM countries AS c
  -- Join to populations (as p)
  INNER JOIN populations AS p
    -- Match on country code
    ON c.code = p.country_code
  -- Join to economies (as e)
  INNER JOIN economies AS e
    -- Match on country code and year
    ON c.code = e.code AND e.year = p.year;
```

| code | name        | region                    | year | fertility\_rate | unemployment\_rate |
| :--- | :---------- | :------------------------ | ---: | --------------: | -----------------: |
| AFG  | Afghanistan | Southern and Central Asia | 2010 |            5746 |                 NA |
| AFG  | Afghanistan | Southern and Central Asia | 2015 |            4653 |                 NA |
| AGO  | Angola      | Central Africa            | 2010 |            6416 |                 NA |
| AGO  | Angola      | Central Africa            | 2015 |            5996 |                 NA |
| ALB  | Albania     | Southern Europe           | 2010 |            1663 |                 14 |

Displaying records 1 - 5

### USING

``` sql
SELECT c.name AS country, continent, l.name AS language, official
  -- From countries (alias as c)
  FROM countries AS c
  -- Join to languages (as l)
  INNER JOIN languages AS l
    -- Match using code
    USING(code)
```

| country     | continent | language | official |
| :---------- | :-------- | :------- | :------- |
| Afghanistan | Asia      | Dari     | TRUE     |
| Afghanistan | Asia      | Pashto   | TRUE     |
| Afghanistan | Asia      | Turkic   | FALSE    |
| Afghanistan | Asia      | Other    | FALSE    |
| Albania     | Europe    | Albanian | TRUE     |

Displaying records 1 - 5

### Self-Join

Use the populations table to perform a self-join to calculate the
percentage increase in population from 2010 to 2015 for each country
code:

``` sql
SELECT p1.country_code,
       p1.size AS size2010, 
       p2.size AS size2015,
       -- calculate growth_perc
       ((p2.size - p1.size)/p1.size * 100.0) AS growth_perc
-- From populations (alias as p1)
FROM populations AS p1
  -- Join to itself (alias as p2)
  INNER JOIN populations AS p2
    -- Match on country code
    ON p1.country_code = p2.country_code
        -- and year (with calculation)
        AND p1.year = p2.year - 5
```

| country\_code      |   size2010 |  size2015 | growth\_perc |
| :----------------- | ---------: | --------: | -----------: |
| ABW                |     101597 |    103889 |     2.255972 |
| AFG                |   27962200 |  32526600 |    16.323297 |
| AGO                |   21220000 |  25022000 |    17.917192 |
| ALB                |    2913020 |   2889170 |   \-0.818875 |
| AND                |      84419 |     70473 |  \-16.519977 |
| \#\#\# CASE, WHEN, | THEN, ELSE | , and END |              |

Displaying records 1 - 5

Create a new field *geosize\_group* that groups the countries into three
groups: large, medium and small.

``` sql
SELECT name, continent, code, surface_area,
    -- First case
    CASE WHEN surface_area > 2000000 THEN 'large'
        -- Second case
        WHEN surface_area > 350000 THEN 'medium'
        -- Else clause + end
        ELSE 'small' END
        -- Alias name
        AS geosize_group
-- From table
FROM countries;
```

| name               | continent      | code    | surface\_area | geosize\_group |
| :----------------- | :------------- | :------ | ------------: | :------------- |
| Afghanistan        | Asia           | AFG     |        652090 | medium         |
| Netherlands        | Europe         | NLD     |         41526 | small          |
| Albania            | Europe         | ALB     |         28748 | small          |
| Algeria            | Africa         | DZA     |       2381740 | large          |
| American Samoa     | Oceania        | ASM     |           199 | small          |
| \#\#\# INTO        |                |         |               |                |
| Create \_countries | plus\_ table   | :       |               |                |
| \`\`\`             |                |         |               |                |
| /\*                |                |         |               |                |
| SELECT name, cont  | inent, code,   | surfac  |      e\_area, |                |
| CASE WHEN sur      | face\_area \>  | 2000000 |               |                |
| THEN               | ‘large’        |         |               |                |
| WHEN surfa         | ce\_area \> 35 | 0000    |               |                |
| THEN               | ‘medium’       |         |               |                |
| ELSE ’smal         | l’ END         |         |               |                |
| AS geosize         | \_group        |         |               |                |
| INTO countries\_pl | us             |         |               |                |
| FROM countries;    |                |         |               |                |
| \*/                |                |         |               |                |
| \`\`\`             |                |         |               |                |

Displaying records 1 - 5

Explore the relationship between the size of a country in terms of
surface area and in terms of population using grouping fields created
with CASE:

``` sql
/*
SELECT country_code, size,
  CASE WHEN size > 50000000
            THEN 'large'
       WHEN size > 1000000
            THEN 'medium'
       ELSE 'small' END
       AS popsize_group
INTO pop_plus       
FROM populations
WHERE year = 2015;
*/

-- Select fields
SELECT name, continent, geosize_group, popsize_group
-- From countries_plus (alias as c)
FROM countries_plus AS c
  -- Join to pop_plus (alias as p)
  INNER JOIN pop_plus AS p
    -- Match on country code
    ON c.code = p.country_code
-- Order the table    
ORDER BY geosize_group;
```

| name          | continent     | geosize\_group | popsize\_group |
| :------------ | :------------ | :------------- | :------------- |
| India         | Asia          | large          | large          |
| United States | North America | large          | large          |
| Saudi Arabia  | Asia          | large          | medium         |
| China         | Asia          | large          | large          |
| Kazakhstan    | Asia          | large          | medium         |

Displaying records 1 - 5

## Outer JOIN

### LEFT JOIN

Begin by performing an inner join with the `cities` table on the left
and the `countries` table on the right:

``` sql
-- Select fields
SELECT c1.name AS city, code, c2.name AS country,
       region, city_proper_pop
-- From left table (with alias)
FROM cities AS c1
  -- Join to right table (with alias)
  INNER JOIN countries AS c2
    -- Match on country code
    ON c1.country_code = c2.code
-- Order by descending country code
ORDER BY code DESC;
```

| city            | code   | country        | region             | city\_proper\_pop |
| :-------------- | :----- | :------------- | :----------------- | ----------------: |
| Harare          | ZWE    | Zimbabwe       | Eastern Africa     |           1606000 |
| Lusaka          | ZMB    | Zambia         | Eastern Africa     |           1742980 |
| Cape Town       | ZAF    | South Africa   | Southern Africa    |           3740030 |
| Johannesburg    | ZAF    | South Africa   | Southern Africa    |           4434830 |
| Durban          | ZAF    | South Africa   | Southern Africa    |           3442360 |
| Change the quer | y to a | left join, and | compare the number |       of records: |

Displaying records 1 - 5

``` sql
SELECT c1.name AS city, code, c2.name AS country,
       region, city_proper_pop
FROM cities AS c1
  -- 1. Join right table (with alias)
  LEFT JOIN countries AS c2
    -- 2. Match on country code
    ON c1.country_code = c2.code
-- 3. Order by descending country code
ORDER BY code DESC;
```

| city               | code    | country        | region             | city\_proper\_pop |
| :----------------- | :------ | :------------- | :----------------- | ----------------: |
| Taichung           | NA      | NA             | NA                 |           2752410 |
| Tainan             | NA      | NA             | NA                 |           1885250 |
| Kaohsiung          | NA      | NA             | NA                 |           2778920 |
| Bucharest          | NA      | NA             | NA                 |           1883420 |
| Taipei             | NA      | NA             | NA                 |           2704970 |
| New Taipei City    | NA      | NA             | NA                 |           3954930 |
| Harare             | ZWE     | Zimbabwe       | Eastern Africa     |           1606000 |
| Lusaka             | ZMB     | Zambia         | Eastern Africa     |           1742980 |
| Cape Town          | ZAF     | South Africa   | Southern Africa    |           3740030 |
| Ekurhuleni         | ZAF     | South Africa   | Southern Africa    |           3178470 |
| First 6 records ar | e not i | ncluded in the | first query result |                \! |

Displaying records 1 - 10

Determine the average gross domestic product (GDP) per capita by region
in 2010

``` sql
-- Select fields
SELECT region, AVG(gdp_percapita) AS avg_gdp
-- From countries (alias as c)
FROM countries AS c
  -- Left join with economies (alias as e)
  LEFT JOIN economies AS e
    -- Match on code fields
    ON c.code = e.code
-- Focus on 2010
WHERE year = 2010
-- Group by region
GROUP BY region
-- Order by descending avg_gdp
ORDER BY avg_gdp DESC;
```

| region                    | avg\_gdp |
| :------------------------ | -------: |
| Western Europe            |  5813096 |
| North America             |  4791151 |
| Australia and New Zealand |  4479238 |
| Nordic Countries          |  4135832 |
| Eastern Asia              |  2620585 |
| Southern Europe           |  2292641 |
| British Islands           |  2179074 |
| Middle East               |  1820464 |
| Baltic Countries          |  1263103 |
| Caribbean                 |  1187166 |

Displaying records 1 - 10

### RIGHT JOIN

Right joins aren’t as common as left joins. One reason why is that you
can always write a right join as a left join.

### FULL JOIN

Choose records in which region corresponds to North America or is NULL.

``` sql
SELECT name AS country, code, region, basic_unit
-- From countries
FROM countries
  -- Join to currencies
  FULL JOIN currencies
    -- Match on code
    USING (code)
-- Where region is North America or null
WHERE region = 'North America' OR region IS NULL
-- Order by region
ORDER BY region;
```

| country       | code | region        | basic\_unit          |
| :------------ | :--- | :------------ | :------------------- |
| Canada        | CAN  | North America | Canadian dollar      |
| United States | USA  | North America | United States dollar |
| Bermuda       | BMU  | North America | Bermudian dollar     |
| Greenland     | GRL  | North America | NA                   |
| NA            | TMP  | NA            | United States dollar |

Displaying records 1 - 5

Choose records in which countries.name starts with the capital letter
‘V’ or is NULL. Arrange by `countries.name` in ascending order to
more clearly see the results.

``` sql
SELECT countries.name, code, languages.name AS language
-- From languages
FROM languages
  -- Join to countries
  FULL JOIN countries
    -- Match on code
    USING (code)
-- Where countries.name starts with V or is null
WHERE countries.name LIKE 'V%' OR countries.name IS NULL
-- Order by ascending countries.name
ORDER BY countries.name;
```

| name    | code | language         |
| :------ | :--- | :--------------- |
| Vanuatu | VUT  | Tribal Languages |
| Vanuatu | VUT  | English          |
| Vanuatu | VUT  | French           |
| Vanuatu | VUT  | Other            |
| Vanuatu | VUT  | Bislama          |

Displaying records 1 - 5

### CROSS JOIN

CROSS JOINs create all possible combinations of two tables.

Explore languages potentially and most frequently spoken in the cities
of Hyderabad, India and Hyderabad, Pakistan.

``` sql
-- Select fields
SELECT c.name AS city, l.name AS language
-- From cities (alias as c)
FROM cities AS c        
  -- Join to languages (alias as l)
  CROSS JOIN languages AS l
-- Where c.name like Hyderabad
WHERE c.name LIKE 'Hyder%';
```

| city              | language |
| :---------------- | :------- |
| Hyderabad (India) | Dari     |
| Hyderabad         | Dari     |
| Hyderabad (India) | Pashto   |
| Hyderabad         | Pashto   |
| Hyderabad (India) | Turkic   |
| \#\# Set Theory   |          |

Displaying records 1 - 5

### UNION

Duplicates are removed by using UNION.

Combine the `economies2010` and `economies2015` tables into one table.

``` sql
-- Select fields from 2010 table
SELECT *
  -- From 2010 table
  FROM economies2010
    -- Set theory clause
    UNION
-- Select fields from 2015 table
SELECT *
  -- From 2015 table
  FROM economies2015
-- Order by code and year
ORDER BY code, year;
```

| code | year | income\_group       | gross\_savings |
| :--- | ---: | :------------------ | -------------: |
| AFG  | 2010 | Low income          |          37133 |
| AFG  | 2015 | Low income          |          21466 |
| AGO  | 2010 | Upper middle income |          23534 |
| AGO  | 2015 | Upper middle income |          \-425 |
| ALB  | 2010 | Upper middle income |          20011 |

Displaying records 1 - 5

### UNION ALL

We can use `UNION ALL` to include duplicates.

Determine all combinations (include duplicates) of country code and year
that exist in either the economies or the populations tables. Order by
code then year.

``` sql
-- Select fields
SELECT code, year
  -- From economies
  FROM economies
    -- Set theory clause
    UNION ALL
-- Select fields
SELECT country_code, year
  -- From populations
  FROM populations
-- Order by code, year
ORDER BY code, year;
```

| code | year |
| :--- | ---: |
| ABW  | 2010 |
| ABW  | 2015 |
| AFG  | 2010 |
| AFG  | 2010 |
| AFG  | 2015 |

Displaying records 1 - 5

### INTERSECT

Look at the records in common for country code and year for the
`economies` and `populations` tables.

``` sql
-- Select fields
SELECT code, year
  -- From economies
  FROM economies
    -- Set theory clause
    INTERSECT
-- Select fields
SELECT country_code, year
  -- From populations
  FROM populations
-- Order by code and year
ORDER BY code, year;
```

| code | year |
| :--- | ---: |
| AFG  | 2010 |
| AFG  | 2015 |
| AGO  | 2010 |
| AGO  | 2015 |
| ALB  | 2010 |

Displaying records 1 - 5

Which countries also have a city with the same name as their country
name?

``` sql
-- Select fields
SELECT name
  -- From countries
  FROM countries
    -- Set theory clause
    INTERSECT
-- Select fields
SELECT name
  -- From cities
  FROM cities;
```

| name      |
| :-------- |
| Singapore |
| Hong Kong |

2 records

### EXCEPT

We use `EXCEPT` to include only the records that are in one ttable , but
not the other.

Get the names of cities in cities which are not noted as capital cities
in countries as a single field result.

``` sql
-- Select field
SELECT name
  -- From cities
  FROM cities
    -- Set theory clause
    EXCEPT
-- Select field
SELECT capital
  -- From countries
  FROM countries
-- Order by result
ORDER BY name;
```

| name       |
| :--------- |
| Abidjan    |
| Ahmedabad  |
| Alexandria |
| Almaty     |
| Auckland   |

Displaying records 1 - 5

### Semi-Joins and Anti-Joins

We use these joins in a way similar to a WHERE clause.

#### Semi-Joins

``` sql
-- Select code
SELECT code
  -- From countries
  FROM countries
-- Where region is Middle East
WHERE region = 'Middle East';
```

| code |
| :--- |
| ARE  |
| ARM  |
| AZE  |
| BHR  |
| GEO  |

Displaying records 1 - 5

``` sql
-- Select distinct fields
SELECT DISTINCT name
  -- From languages
  FROM languages
-- Where in statement
WHERE code IN
  -- Subquery
  (SELECT code
   FROM countries
   WHERE region = 'Middle East')
-- Order by name
ORDER BY name;
```

| name        |
| :---------- |
| Arabic      |
| Aramaic     |
| Armenian    |
| Azerbaijani |
| Azeri       |

Displaying records 1 - 5

#### Anti-Joins

Anti-join is particularly useful in identifying which records are
causing an incorrect number of records to appear in join queries.

Identify the currencies used in Oceanian countries\! Use an anti-join to
determine which countries were not included\!

``` sql
-- Select fields
SELECT code, name
  -- From Countries
  FROM countries
  -- Where continent is Oceania
  WHERE continent = 'Oceania'
    -- And code not in
    AND code NOT IN
    -- Subquery
    (SELECT code
     FROM currencies);
```

| code      | name                            |
| :-------- | :------------------------------ |
| ASM       | American Samoa                  |
| FJI       | Fiji Islands                    |
| GUM       | Guam                            |
| FSM       | Micronesia, Federated States of |
| MNP       | Northern Mariana Islands        |
| \#\# Subq | ueries                          |

Displaying records 1 - 5

### Inside WHERE

Figure out which countries had high average life expectancies (at the
country level) in 2015. (We want only records that were above 1.15 \*
100 in terms of life expectancy for 2015)

``` sql
-- Select fields
SELECT *
  -- From populations
  FROM populations
-- Where life_expectancy is greater than
WHERE life_expectancy >
  -- 1.15 * subquery
  1.15 * (
   SELECT AVG(life_expectancy)
   FROM populations
   WHERE year = 2015)
  AND year = 2015;
```

| pop\_id | country\_code | year | fertility\_rate | life\_expectancy |     size |
| ------: | :------------ | ---: | --------------: | ---------------: | -------: |
|      19 | ABW           | 2015 |            1647 |      7.55736e+14 |   103889 |
|       3 | ALB           | 2015 |            1793 |      7.80145e+14 |  2889170 |
|      15 | ARG           | 2015 |            2308 |      7.63342e+14 | 43416800 |
|      17 | ARM           | 2015 |            1517 |      7.47971e+14 |  3017710 |
|      13 | ATG           | 2015 |            2063 |      7.61002e+14 |    91818 |

Displaying records 1 - 5

Get the urban area population for only capital cities.

``` sql
-- 2. Select fields
SELECT name, country_code, urbanarea_pop
  -- 3. From cities
  FROM cities
-- 4. Where city name in the field of capital cities
WHERE name IN
  -- 1. Subquery
  (SELECT capital
   FROM countries)
ORDER BY urbanarea_pop DESC;
```

| name    | country\_code | urbanarea\_pop |
| :------ | :------------ | -------------: |
| Beijing | CHN           |       21516000 |
| Dhaka   | BGD           |       14543100 |
| Tokyo   | JPN           |       13513700 |
| Moscow  | RUS           |       12197600 |
| Cairo   | EGY           |       10230400 |

Displaying records 1 - 5

### Inside SELECT

Select the top five countries in terms of number of cities appearing in
the cities table.

``` sql
SELECT name AS country,
  (SELECT COUNT(*)
   FROM cities
   WHERE countries.code = cities.country_code) AS cities_num
FROM countries
ORDER BY cities_num DESC, country
LIMIT 5;
```

| country  | cities\_num |
| :------- | ----------: |
| China    |          36 |
| India    |          18 |
| Japan    |          11 |
| Brazil   |          10 |
| Pakistan |           9 |

5 records

### Inside FROM

Determine the number of languages spoken for each country, identified by
the country’s local name.

``` sql
-- Select fields
SELECT local_name, subquery.lang_num
  -- From countries
  FROM countries,
    -- Subquery (alias as subquery)
    (SELECT code, COUNT(*) AS lang_num
     FROM languages
     GROUP BY code) AS subquery
  -- Where codes match
  WHERE countries.code = subquery.code
-- Order by descending number of languages
ORDER BY lang_num DESC;
```

| local\_name  | lang\_num |
| :----------- | --------: |
| Zambia       |        19 |
| YeItyop´iya  |        16 |
| Zimbabwe     |        16 |
| Bharat/India |        14 |
| Nepal        |        14 |

Displaying records 1 - 5
