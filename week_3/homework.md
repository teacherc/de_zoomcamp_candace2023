## Week 3 Homework

## Question 1:

### Answer
What is the count for fhv vehicle records for year 2019?
- 43,244,696

### Code
Web to GCS for fhv data
```python
from pathlib import Path
import pandas as pd
from prefect import flow, task
from prefect_gcp.cloud_storage import GcsBucket
from prefect.tasks import task_input_hash
from datetime import timedelta

@task(retries=3, cache_key_fn=task_input_hash, cache_expiration=timedelta(days=1))
def fetch(dataset_url: str) -> pd.DataFrame:
    """Read taxi data from web into pandas DataFrame"""
    df = pd.read_csv(dataset_url)
    return df

@task()
def write_local(df: pd.DataFrame, dataset_file: str) -> Path:
    """Write DataFrame out locally as gz file"""
    path = Path(f"data/fhv/{dataset_file}.csv.gz")
    df.to_csv(path, compression="infer")
    return path

@task()
def write_gcs(path: Path) -> None:
    """Upload local .gz file to GCS"""
    gcs_block = GcsBucket.load("zoom-gcs")
    gcs_block.upload_from_path(from_path=path, to_path=path)
    return

@flow()
def etl_web_to_gcs(year: int, month: int) -> None:
    """The main ETL function"""
    dataset_file = f"fhv_tripdata_{year}-{month:02}.csv.gz"
    dataset_url = f"https://github.com/DataTalksClub/nyc-tlc-data/releases/download/fhv/fhv_tripdata_{year}-{month:02}.csv.gz"

    df = fetch(dataset_url)
    path = write_local(df, dataset_file)
    write_gcs(path)

@flow()
def etl_parent_flow(
    months: list[int], years: list[int]
):
    for year in years:
        for month in months:
            etl_web_to_gcs(year, month)

if __name__ == '__main__':
    months = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
    years = [2019]
    etl_parent_flow(months, years)
```

Example URI
```
gs://first_prefect_bucket/data/fhv/fhv_tripdata_2019-01.csv.gz 
```

uri to find relevant fhv files (when creating native table via BQ UI)
```
first_prefect_bucket/data/fhv/fhv_tripdata*
```

Creating an external table
```sql 
CREATE EXTERNAL TABLE neural-caldron-375619.fhv
OPTIONS (
  uris = ['gs://first_prefect_bucket/data/fhv/fhv_tripdata*.csv.gz'],
  format = 'CSV';
```

Troubleshooting:
- Had to download ncdu and use it to delete old Prefect cache files because my VM ran out of memory (Prefect was holding 16 GB)
- After deleting old cache files, had to take out the cache code


## Question 2:
Write a query to count the distinct number of affiliated_base_number for the entire dataset on both the tables.</br> 
What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?

### Answer
- 0 MB for the External Table and 317.94MB for the BQ Table 

### Code
```SQL
SELECT COUNT (DISTINCT(Affiliated_base_number)) FROM `neural-caldron-375619.fhv.native` LIMIT 1000;

SELECT COUNT(DISTINCT(Affiliated_base_number)) FROM `neural-caldron-375619.fhv.external` LIMIT 1000;
```

## Question 3:
How many records have both a blank (null) PUlocationID and DOlocationID in the entire dataset?

### Answer
- 717,748

### Code

SQL query (external)
```SQL
SELECT COUNT(*)  
FROM `neural-caldron-375619.fhv.external` 
WHERE PUlocationID IS NULL
AND DOlocationID IS NULL;
```

## Question 4:
What is the best strategy to optimize the table if query always filter by pickup_datetime and order by affiliated_base_number?
- Partition by pickup_datetime Cluster on affiliated_base_number


## Question 5:
Implement the optimized solution you chose for question 4. Write a query to retrieve the distinct affiliated_base_number between pickup_datetime 2019/03/01 and 2019/03/31 (inclusive).</br> 
Use the BQ table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 4 and note the estimated bytes processed. What are these values? Choose the answer which most closely matches.

### Answer 
- 647.87 MB for non-partitioned table and 23.06 MB for the partitioned table

### Code
Created table (2.25 GB)
```SQL
CREATE OR REPLACE TABLE neural-caldron-375619.fhv.fhv_partitoned_clustered
PARTITION BY
  DATE(pickup_datetime) 
  CLUSTER BY affiliated_base_number AS
SELECT * FROM neural-caldron-375619.fhv.native;
```

Query (Partitioned - 23.05 MB, non-partitioned - 647.87)
```SQL
SELECT COUNT(DISTINCT(affiliated_base_number))
FROM neural-caldron-375619.fhv.fhv_partitoned_clustered
WHERE DATE(pickup_datetime) BETWEEN '2019-03-01' AND '2019-03-31';
```

## Question 6: 
Where is the data stored in the External Table you created?
- GCP Bucket


## Question 7:
It is best practice in Big Query to always cluster your data:
- False