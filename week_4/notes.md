# Week 4 Notes

## Prerequisites

### These datasets should be in GCS
- Yellow taxi data - Years 2019 and 2020
- Green taxi data - Years 2019 and 2020
- fhv data - Year 2019

Links to data: https://github.com/DataTalksClub/nyc-tlc-data/

### Must have a running warehouse
I'm using BigQuery (other option is local Postgres)

### Set up DBT for BigQuery

1. Create an account [here](https://www.getdbt.com/signup/)
2. Connect BigQuery warehouse to DBT using [these instructions](https://docs.getdbt.com/docs/collaborate/manage-access/set-up-bigquery-oauth) or these [more detailed instructions](https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/week_4_analytics_engineering/dbt_cloud_setup.md)

## Introduction to analytics engineering

### 00:00 1 - What is Analytics Engineering?
### 00:17 Data domain developments
### 01:02 Roles in a data team
### 02:20 Tooling
### 03:06 2 - Data modeling concepts
### 03:11 ETL vs ELT
### 04:12 Kimball's Dimensional Modeling
### 05:05 Elements of Dimensional Modeling
### 05:50 Architecture of Dimensional Modeling

## What is dbt?

### 00:00 What is dbt?
### 01:03 How does dbt work?
### 02:04 How to use dbt?
### 03:30 How are we going to use dbt?

## Starting a dbt project

### 00:00 Create a new dbt project presentation (cloud or terminal)
### 02:25 BigQuery dataset and schemas
### 03:10 Reporsitory structure
### 03:24 Satrt the project in dbt cloud 
### 04:05 dbt_proyect.yml in dbt cloud 

## Development of dbt models

### 00:00 Anatomy of a dbt model 
### 03:15 The FROM clause: Sources and seeds
### 06:40 The FROM clause: ref macro
### 08:38 Define a source and develop the first model (stg_green_tripdata)
### 17:24 Definition and usage of macros 
### 24:10 Importing and using dbt packages
### 29:18 Definition of variables and setting a variable from the cli
### 33:33 Add second model (stg_yellow_tripdata)
### 35:16 Creating and using dbt seed (taxi_zones_lookup and dim_zone)
### 41:20 Unioning our models in fact_trips and understanding dependencies

## Testing and documenting dbt models

### 0:00 Testing
### 4:28 Deployment

## Deploying a dbt project

### 0:00 Theory
### 5:12 Demo
### -- creates Production environment
### -- creates Job to run periodically
### -- in settings changes Artifacts

## Visualising the transformed data (Google Data Studio)

### 0:00 Google Data Studio
### 1:50 changes default aggregations
### 3:18 creates report
### 6:20 Adds control, title, and other graphs


