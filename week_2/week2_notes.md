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

[What is Data Orchestration?](What is Data Orchestration?)

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

## Next
- Introduction to Prefect
- ETL with GCP and Prefect
- GCS to Big Query
- Parametrizing Flow and Deployments
- Schedules & Docker Storage with Infrastructure
- Prefect Cloud and Additional Resources