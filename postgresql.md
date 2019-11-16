# Notes about PostgreSQL

### Create table

```
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

### Drop table

```
DROP TABLE tablename;
```

### Create rows

```
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27'); -- use the implicit order of columns

INSERT INTO cities VALUES ('San Francisco', '(-194.0, 53.0)');

INSERT INTO weather (city, temp_lo, temp_hi, prcp, date) -- specify the columns you want, better
    VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');
```

### Column types

