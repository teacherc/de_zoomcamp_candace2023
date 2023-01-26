# Week 1 Homework

## Question 1. Knowing docker tags

### Code
```console
docker build --help
```

### Answer: 

    --iidfile string


## Question 2. Understanding docker first run

### Code

```console
docker run -it python:3.9 bash
pip list
```

### Answer
    3


## Question 3

### Code

```sql
SELECT COUNT(*)
FROM green_taxi_data
WHERE lpep_pickup_datetime::date = '2019-01-15'
AND lpep_dropoff_datetime::date = '2019-01-15';
```

# Answer
    20530

## Question 4. Largest trip for each day

### Code

```sql

SELECT SUM(trip_distance)
FROM green_taxi_data
WHERE lpep_pickup_datetime::date = '2019-01-10'

UNION

SELECT SUM(trip_distance)
FROM green_taxi_data
WHERE lpep_pickup_datetime::date = '2019-01-15'

UNION

SELECT SUM(trip_distance)
FROM green_taxi_data
WHERE lpep_pickup_datetime::date = '2019-01-18'

UNION

SELECT SUM(trip_distance)
FROM green_taxi_data
WHERE lpep_pickup_datetime::date = '2019-01-28';

```

### Answer
    2019-01-18


## Question 5. The number of passengers

### Code
```sql
SELECT COUNT(*)
FROM green_taxi_data
WHERE lpep_pickup_datetime::date = '2019-01-01'
AND passenger_count::integer = '2'

UNION

SELECT COUNT(*)
FROM green_taxi_data
WHERE lpep_pickup_datetime::date = '2019-01-01'
AND passenger_count::integer = '3';

```

### Answer
    None of the above


## Question 6. Largest tip

### Code

```sql
SELECT *
FROM
    green_taxi_data t,
    zones zpu,
	zones zdo
WHERE
	t."PULocationID" = zpu."LocationID" AND
    t."DOLocationID" = zdo."LocationID" AND
	zpu."Zone" = 'Astoria'
ORDER BY tip_amount DESC
LIMIT 1;
```

### Answer
    Long Island City/Queens Plaza
