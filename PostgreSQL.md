PostgreSQL uses a client/server model
The PostgreSQL server can handle multiple concurrent connections from clients. To achieve this it starts ("forks") a new process for each connection.

```bash
# install and start postgreSQL
brew install postgresql
brew services start postgresql

# create db
createdb mydb

# drop db
dropdb mydb

# activated for the mydb database
# can have option, for example -s means single step mode
psql mydb
# command line will become
# if you are superuser
mydb=#
# if not superuser
mydb=>
```

```bash
# help (SQL commands)
mydb=> \h
# quit db
mydb=> \q
# reads in command from file
mydb=> \i basics.sql

# some simple queries
mydb=> SELECT version();
mydb=> SELECT 2 + 2;
mydb=> SELECT current_date;
```

### SQL

```sql
# create table and drop table
CREATE TABLE weather (
    city            varchar(80),
    temp_lo         int,           -- low temperature
    temp_hi         int,           -- high temperature
    prcp            real,          -- precipitation
    date            date
);

CREATE TABLE cities (
    name            varchar(80),
    location        point          -- point is a PostgreSQL-specific data type.
);

DROP TABLE tablename;

## INSERT
# insert row into table: remember the order of the columns
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');
INSERT INTO cities VALUES ('San Francisco', '(-194.0, 53.0)');

# insert row into table: list the columns explicitly
INSERT INTO weather (city, temp_lo, temp_hi, prcp, date)
    VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');
INSERT INTO weather (date, city, temp_hi, temp_lo)
    VALUES ('1994-11-29', 'Hayward', 54, 37);

# load large amounts of data from flat-text file

COPY weather FROM '/home/user/weather.txt';

## QUERY
# query table
SELECT * FROM weather -- * shorthand for "all columns"
# same as 
SELECT city, temp_lo, temp_hi, prcp, date FROM weather;

# write expressions
SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;

SELECT * FROM weather
    WHERE city = 'San Francisco' AND prcp > 0.0;

SELECT * FROM weather
    ORDER BY city;

SELECT * FROM weather
    ORDER BY city, temp_lo;

SELECT DISTINCT city
    FROM weather;

SELECT DISTINCT city
    FROM weather
    ORDER BY city;

## Joins Query: A query that accesses multiple rows of the same or different tables at one time is called a join query.

SELECT *
    FROM weather, cities
    WHERE city = name;

# select 
SELECT city, temp_lo, temp_hi, prcp, date, location
    FROM weather, cities
    WHERE city = name;

# qualify the column names in case there are duplicate column names in two tables 
SELECT weather.city, weather.temp_lo, weather.temp_hi,
       weather.prcp, weather.date, cities.location
    FROM weather, cities
    WHERE cities.name = weather.city;

# equivalent to 上上一个 Query
SELECT *
    FROM weather INNER JOIN cities ON (weather.city = cities.name);

# outer join (If no matching row is found we want some "empty values" to be substituted for the cities table's columns.)
SELECT *
    FROM weather LEFT OUTER JOIN cities ON (weather.city = cities.name);

# join tabel against it self
SELECT W1.city, W1.temp_lo AS low, W1.temp_hi AS high,
    W2.city, W2.temp_lo AS low, W2.temp_hi AS high
    FROM weather W1, weather W2   -- W1, W2 are aliases
    WHERE W1.temp_lo < W2.temp_lo
    AND W1.temp_hi > W2.temp_hi;

# anoter example of aliases
SELECT *
    FROM weather w, cities c
    WHERE w.city = c.name;

## Aggregate function: computes a single result from multiple input rows
SELECT max(temp_lo) FROM weather;

# subquery using aggregate function 
SELECT city FROM weather
    WHERE temp_lo = (SELECT max(temp_lo) FROM weather);

# combine with GROUP BY clauses
# get the maximum low temperature observed in each city with:
SELECT city, max(temp_lo)
    FROM weather
    GROUP BY city;

# filter using HAVING
SELECT city, max(temp_lo)
    FROM weather
    GROUP BY city
    HAVING max(temp_lo) < 40;

# filter city using LIKE
SELECT city, max(temp_lo)
    FROM weather
    WHERE city LIKE 'S%' -- LIKE operator does pattern matching
    GROUP BY city
    HAVING max(temp_lo) < 40;

# difference betwoeen WHERE and HAVING
# WHERE selects input rows before aggregate(normally)
# HAVING after(normally)

## Update existing rows
UPDATE weather
    SET temp_hi = temp_hi - 2,  temp_lo = temp_lo - 2
    WHERE date > '1994-11-28';

## Delete
DELETE FROM weather WHERE city = 'Hayward';

# delete all rows of a table
DELETE FROM tablename;
```

### Advanced faature

```sql
## Views
# use VIEW to reuse query
CREATE VIEW myview AS
    SELECT city, temp_lo, temp_hi, prcp, date, location
        FROM weather, cities
        WHERE city = name;

SELECT * FROM myview;

## Foreign Keys
# referential integrity （make sure that no one can insert rows in the weather table that do not have a matching entry in the cities table.）
CREATE TABLE cities (
        city     varchar(80) primary key,
        location point
);

CREATE TABLE weather (
        city      varchar(80) references cities(city),
        temp_lo   int,
        temp_hi   int,
        prcp      real,
        date      date
);

# now one can no longer insert a row into weather with city name not defined in tabel cities. 
INSERT INTO weather VALUES ('Berkeley', 45, 53, 0.0, '1994-11-28'); -- should fail

## Transactions

```




- SQL is case insensitive about key words and identifiers, except when identifiers are double-quoted to preserve the case (not done above).

- PostgreSQL supports the standard SQL types int, smallint, real, double precision, char(N), varchar(N, date, time, timestamp, and interval, as well as other types of general utility and a rich set of geometric types. PostgreSQL can be customized with an arbitrary number of user-defined data types. Consequently, type names are not key words in the syntax, except where required to support special cases in the SQL standard.

## Refs
- [insert millions records faster](https://stackoverflow.com/questions/19682414/how-can-mysql-insert-millions-records-faster)
- [How to update 10 million+ rows in MySQL single table as Fast as possible](https://dba.stackexchange.com/questions/119621/how-to-update-10-million-rows-in-mysql-single-table-as-fast-as-possible)
- [Facebook database design](https://stackoverflow.com/questions/1009025/facebook-database-design)