# Notes about PostgreSQL

### Create table

```sql
CREATE TABLE weather (
    city            varchar(80),
    temp_lo         int,           -- low temperature
    temp_hi         int,           -- high temperature
    prcp            real,          -- precipitation
    date            date
);

CREATE TABLE cities (
    name            varchar(80),
    location        point
);
```

### Auto-increment primary key

Use `SERIAL` or `BIGSERIAL`.

```
CREATE TABLE books (
  id              SERIAL PRIMARY KEY,
  title           VARCHAR(100) NOT NULL,
  primary_author  VARCHAR(100) NULL
);
```

### Drop table

```sql
DROP TABLE tablename;
```

### Create rows

```sql
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27'); -- use the implicit order of columns

INSERT INTO cities VALUES ('San Francisco', '(-194.0, 53.0)');

INSERT INTO weather (city, temp_lo, temp_hi, prcp, date) -- specify the columns you want, better
    VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');
```

### Column types

```sql
TODO
```

### Querying

```sql
SELECT * FROM weather; -- all columns
```

### Use an expression as a column

```sql
SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;
-- AS relabels the column
```

### Use WHERE to qualify the query

You can use `AND`, `OR`, `NOT`

```sql
SELECT * FROM weather
    WHERE city = 'San Francisco' AND prcp > 0.0;
```

### Order results

```sql
SELECT * FROM weather
    ORDER BY city;
    
-- order by two columns
SELECT * FROM weather
ORDER BY city, temp_lo;
```

### Remove duplicates

```sql
SELECT DISTINCT city
    FROM weather
    ORDER BY city;
 ```
 
 ### Joins
 
 ```sql
 SELECT *
    FROM weather, cities
    WHERE city = name;
```




