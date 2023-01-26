# Week 1 Homework
REDO - Realized only half of the dataset had been ingested

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
SELECT lpep_pickup_datetime::date, MAX(trip_distance)
FROM green_taxi_data
WHERE lpep_pickup_datetime::date = '2019-01-10' OR
lpep_pickup_datetime::date = '2019-01-15' OR
lpep_pickup_datetime::date = '2019-01-18' OR
lpep_pickup_datetime::date = '2019-01-28'
GROUP BY 1;
```

OUTPUT
```
"2019-01-18"	80.96
"2019-01-15"	117.99
"2019-01-10"	64.2
"2019-01-28"	64.27
```

### Answer
    2019-01-15


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

OUTPUT
```
"count"
254
1282
```

### Answer
    1282, 254


## Question 6. Largest tip

### Code

```sql
SELECT MAX(tip_amount), zpu."Zone" AS "pickup_loc",
    zdo."Zone" AS "dropoff_loc"
FROM green_taxi_data t,
	zones zpu,
	zones zdo
WHERE t."PULocationID" = zpu."LocationID" AND
	t."DOLocationID" = zdo."LocationID" AND
	zpu."Zone" = 'Astoria'
GROUP BY zpu."Zone", zdo."Zone"
ORDER BY 1 DESC LIMIT 5;
```

OUTPUT
```
"max"	"pickup_loc"	"dropoff_loc"
88	"Astoria"	"Long Island City/Queens Plaza"
30	"Astoria"	"Central Park"
25	"Astoria"	"Jamaica"
25	"Astoria"	
18.16	"Astoria"	"Astoria"
```

### Answer
    Long Island City/Queens Plaza
