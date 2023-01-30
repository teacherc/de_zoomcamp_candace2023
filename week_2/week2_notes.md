# Week 2 Notes

## What is a Data Lake?

- A data lake allows us to gather vast amounts of unstructured data for later use
- A data warehouse is where structured data can be accessed
- An ETL (export, transform, load) typically applies to small amounts of data
    - This is a schema on write framework - data is structured as it is written to the data warehouse
- An ELT (export, load, and transform) is better for large amounts of data
    - This allows for use of a data lake
    - Schema on read is when data is structured as it is pulled out of a data lake (or read)
    - This is innovative because of the fact that data can become valuable right away (it can be loaded as-is and used when needed)
- Data lake "gotchas" include the lack of metadata and versioning, the propensity for the lake to become an unusuable swamp, and the inability to join data in the lake
- Cloud providers for data lakes:
    - Google Cloud Platform: Google Cloud Storage
    - AWS: S3
    - Microsoft Azure: Azure Blob

## What is Data Orchestration?

### Notes from sources I've found

[What is Data Orchestration?](https://www.youtube.com/watch?v=iyw9puEmTrA)

- The goal of data engineering is to turn data into useful information
- Just like a conductor helps a group of musicians play their instruments at the right pace and intensity, a data orchestrator helps us capture data from a variety of sources by coordinating multiple systems and manipulating settings (example: dialing compute resources up or down depending on where we are in the pipeline, turning microservices on and off, etc)
- Before the cloud, most data pipelines were run at night and all equipment would live in one data center
- Now, our systems are dispersed in the cloud, and we do not want to only have to ingest data at night

### Course video notes

- A data flow combines disparate applications together - these applications are often from many different vendors
- Examples of disparate applications and services:
    - A cleansing script in Pandas
    - Data transformation with DBT
    - ML use case
- A workflow orchestration tool allows us to turn code into a workflow that can be scheduled, run, and observed
- Workflow configurations include:
    - The order of execution (of tasks)
    - The packaging (Docker, Kubernetes, sub-process)
    - The type of delivery (concurrency, async, Ray, DAGs, etc)
- A workflow orchestration tool executes the workflow run
    - Scheduling
    - Remote execution
    - Privacy
    - Knows when to restart or retry an operation
    - Visibility / Alerts
    - Parametarization
    - Integrates with many systems
- Prefect is a modern, open source orchestration tool 

## Introduction to Prefect

### Introduction to Prefect concepts

#### Prepare virtual environment (on a virtual machine)

- Start your GCP VM instance and note the external IP
- Navigate to your ```~/.ssh``` folder and find the config file
- Edit the config file - replace the old ```HostName``` address with the new external IP and save the file
- Head to VS Code and start the ssh session (mine is called ```dezoomcamp```)
- Navigate to the [Prefect Code](https://github.com/discdiver/prefect-zoomcamp.git) for the DE Zoomcamp
- Click on the green ```Code``` button and copy the HTTPS URL
- In your VSCode ssh terminal, type ```git clone https://github.com/discdiver/prefect-zoomcamp.git``` (or the URL you found)
- Use ```open``` in VSCode to open the folder you've cloned
- Open this file in VSCode: ```/home/{user}]/prefect-zoomcamp/flows/01_start/ingest_data.py```
- Use Conda to create an virtual environment ```conda create -n zoomcamp python=3.9```
- Activate the environment you've just created. My example: ```conda activate zoomcamp```
- Navigate to the folder with the Prefect code in it and use pip to install the requirements ```pip install -r requirements.txt ```
- Double check the version of Prefect installed in your virtual environment with ```prefect version```
- Navigate to the data engineering zoomcamp folder you created in week 1 (in your VM)
- Open a new terminal in your OS - Find the folder with the Dockerfile and yaml - run ```docker-compose up``` from that folder in the ssh terminal
- In VSCode, forward the ports ```8080```, ```5432```, and ```8888```
- Using pgcli or pgadmin in the browser, drop your existing tables


#### Running the initial ingestion script (without orchestration)
- Modify ```ingest_data.py``` in the Prefect code (I had to change the username, password, and port)
```python
if __name__ == '__main__':
    user = "root"
    password = "root"
    host = "localhost"
    port = "5432"
    db = "ny_taxi"
    table_name = "yellow_taxi_trips"
    csv_url = "https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

    ingest_data(user, password, host, port, db, table_name, csv_url)
```

- From the ```~/prefect-zoomcamp/flows/01_start``` folder in your VSCode terminal, run ```python ingest_data.py```
- Use pgcli or pgadmin to check the first 100 rows and count the number of rows (I had ```1_369_765``` rows)

#### How will orchestration be better?
- We won't have to manually handle dependencies and run the script when we need data ingestion to happen
- We'll have visibility about what does and does not happen

#### Integrating Prefect

Pre-reads: 
- To learn more about what decorators do in Python, watch this [Tech with Tim](https://www.youtube.com/watch?v=tfCz563ebsU) video. 
- If you are new to programming and need more information about objects in Python, watch this [CS Dojo](https://www.youtube.com/watch?v=wfcWRAxRVBA) clip.

- Add the Prefect imports in ```ingest_data.py```

```python
from prefect import flow, task
```
A ```flow``` is the most basic Python object in Prefect. It is a container of workflow logic that allow us to interact with flows as if they are functions (with inputs, outputs, etc).

We add ```flow``` logic to the ingestion script by using decorators.

First, we'll create a ```main()``` function with a Prefect flow decorator. Then, we'll tell Python to run this function when the program is initialized. All of the parameters that were established in ```if __name__ == '__main__':``` now exist in the ```main()``` function.

```python
@flow(name="Ingest Flow")
def main():
    user = "root"
    password = "root"
    host = "localhost"
    port = "5432"
    db = "ny_taxi"
    table_name = "yellow_taxi_trips"
    csv_url = "https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

    ingest_data(user, password, host, port, db, table_name, csv_url)

if __name__ == '__main__':
    main()
```

Flows contain tasks. Now, we can turn the ```ingest_data()``` function into a task by adding the ```@task``` decorator.

```python
@task()
def ingest_data(user, password, host, port, db, table_name, url):
    
    # the backup files are gzipped, and it's important to keep the correct extension
    # for pandas to be able to open the file
    if url.endswith('.csv.gz'):
        csv_name = 'yellow_tripdata_2021-01.csv.gz'
    else:
        csv_name = 'output.csv'

    os.system(f"wget {url} -O {csv_name}")
    postgres_url = f'postgresql://{user}:{password}@{host}:{port}/{db}'
    engine = create_engine(postgres_url)

    df_iter = pd.read_csv(csv_name, iterator=True, chunksize=100000)

    df = next(df_iter)

    df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
    df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)

    df.head(n=0).to_sql(name=table_name, con=engine, if_exists='replace')

    df.to_sql(name=table_name, con=engine, if_exists='append')


    while True: 

        try:
            t_start = time()
            
            df = next(df_iter)

            df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
            df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)

            df.to_sql(name=table_name, con=engine, if_exists='append')

            t_end = time()

            print('inserted another chunk, took %.3f second' % (t_end - t_start))

        except StopIteration:
            print("Finished ingesting data into the postgres database")
            break
```

Tasks are not required for flows, but they are special because they can receive upstream metadata about the state of dependencies before the function is run. This means that the task can wait on the completion of another task.

We'll add arguments to our task decorator so logging and retries happen:

```python
@task(log_prints=True, retries=3)
```

We can clean up our code by taking out the print statements from the ```while True``` statement because of the logging we've added to the task. NOTE: I kept the ```while True``` statement because I want this flow to iterate over the entire csv (not just the first 100_000 rows). In the video, she eliminates ```while True``` entirely.

Our resulting ```ingest_data.py``` file looks like this:
```python
import os
import argparse
from time import time
import pandas as pd
from sqlalchemy import create_engine
from prefect import flow, task

@task(log_prints=True, retries=3)
def ingest_data(user, password, host, port, db, table_name, url):
    
    # the backup files are gzipped, and it's important to keep the correct extension
    # for pandas to be able to open the file
    if url.endswith('.csv.gz'):
        csv_name = 'yellow_tripdata_2021-01.csv.gz'
    else:
        csv_name = 'output.csv'

    os.system(f"wget {url} -O {csv_name}")
    postgres_url = f'postgresql://{user}:{password}@{host}:{port}/{db}'
    engine = create_engine(postgres_url)

    df_iter = pd.read_csv(csv_name, iterator=True, chunksize=100000)

    df = next(df_iter)

    df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
    df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)

    df.head(n=0).to_sql(name=table_name, con=engine, if_exists='replace')

    df.to_sql(name=table_name, con=engine, if_exists='append')
    
    while True: 

        try:            
            df = next(df_iter)

            df.tpep_pickup_datetime = pd.to_datetime(df.tpep_pickup_datetime)
            df.tpep_dropoff_datetime = pd.to_datetime(df.tpep_dropoff_datetime)

            df.to_sql(name=table_name, con=engine, if_exists='append')

        except StopIteration:
            break


@flow(name="Ingest Flow")
def main():
    user = "root"
    password = "root"
    host = "localhost"
    port = "5432"
    db = "ny_taxi"
    table_name = "yellow_taxi_trips"
    csv_url = "https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

    ingest_data(user, password, host, port, db, table_name, csv_url)

if __name__ == '__main__':
    main()
```

#### Testing our flow
- Use pgcli or pgadmin to drop any tables you've ingested ```DROP TABLE yellow_taxi_trips```
- Run ```python ingest_data.py```


## Next
- ETL with GCP and Prefect
- GCS to Big Query
- Parametrizing Flow and Deployments
- Schedules & Docker Storage with Infrastructure
- Prefect Cloud and Additional Resources