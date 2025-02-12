# DATAWAREHOUSING-WITH-BIGQUERY

## TASK OVERVIEW DE ZOOMCAMP 2025
This project focuses on leveraging  Google Big Query as our datawarehouse for advanced analytics , In this module we learnt how to store data in Goggle Cloud Storage bucket, How to create tables (Extrenal, Regular/Materialized , etc) , How to partition and cluster tables for query improvement and cost reduction etc.

This README file  show cases the needed queries to solve the required questions for the datawarehouse module.

### PREREQUISTE
For this homework we will be using the Yellow Taxi Trip Records for January 2024 - June 2024 NOT the entire year of data Parquet Files from the New York City Taxi Data found here:
https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
If you are using orchestration such as Kestra, Mage, Airflow or Prefect etc. do not load the data into Big Query using the orchestrator.
Stop with loading the files into a bucket.

Load Script: You can manually download the parquet files and upload them to your GCS Bucket or you can use the linked script here:
You will simply need to generate a Service Account with GCS Admin Priveleges or be authenticated with the Google SDK and update the bucket name in the script to the name of your bucket. Nothing is fool proof so make sure that all 6 files show in your GCS Bucket before begining.

NOTE: You will need to use the PARQUET option files when creating an External Table

BIG QUERY SETUP:
Create an external table using the Yellow Taxi Trip Records.
Create a (regular/materialized) table in BQ using the Yellow Taxi Trip Records (do not partition or cluster this table).

## Answers to Google BIG QUERY Questions :
Question 1. What is count of records for the 2024 Yellow Taxi Data?
``` SQL
SELECT COUNT(*) FROM decisive-depth-449700-j5.nytaxi.hw3_external_yellow_tripdata;
```
ANSWER:  20,332,093

Question 2. Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.
What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?
``` SQL
-- Query for external table
SELECT COUNT(DISTINCT PULocationID ) FROM decisive-depth-449700-j5.nytaxi.hw3_external_yellow_tripdata;

-- Query for non-partitioned table
SELECT COUNT(DISTINCT PULocationID ) FROM decisive-depth-449700-j5.nytaxi.hw3_yellow_tripdata_non_partitioned;
```

ANSWER: 0 MB for the External Table and 155.12 MB for the Materialized Table

Question 3. Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table. Why are the estimated number of Bytes different?
``` SQL
-- Query for one field this resulted in 155.12 MB
SELECT  PULocationID FROM decisive-depth-449700-j5.nytaxi.hw3_yellow_tripdata_non_partitioned;

-- Query for two fields this resulted in 310.24 MB 
SELECT PULocationID ,DOLocationID  FROM decisive-depth-449700-j5.nytaxi.hw3_yellow_tripdata_non_partitioned;
```
ANSWER: BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.


Question 4. How many records have a fare_amount of 0?
``` SQL
SELECT COUNT(*) FROM decisive-depth-449700-j5.nytaxi.hw3_yellow_tripdata_non_partitioned
WHERE fare_amount =0 ;
```
ANSWER: 8,333

Question 5. What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)
``` SQL
CREATE OR REPLACE TABLE decisive-depth-449700-j5.nytaxi.hw3_yellow_tripdata_partitioned_clustered
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM decisive-depth-449700-j5.nytaxi.hw3_external_yellow_tripdata ;
```
ANSWER: Partition by tpep_dropoff_datetime and Cluster on VendorID

Question 6. Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime 2024-03-01 and 2024-03-15 (inclusive)

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values?

``` SQL
-- Query for non partitioned
SELECT DISTINCT(VendorID)
FROM decisive-depth-449700-j5.nytaxi.hw3_yellow_tripdata_non_partitioned
WHERE DATE(tpep_dropoff_datetime) BETWEEN '2024-03-01' AND '2024-03-15';

-- Query for partiotioned
SELECT DISTINCT(VendorID)
FROM decisive-depth-449700-j5.nytaxi.hw3_yellow_tripdata_partitioned_clustered
WHERE DATE(tpep_dropoff_datetime) BETWEEN '2024-03-01' AND '2024-03-15';
```
ANSWER: Partition by tpep_dropoff_datetime and Cluster on VendorID


Question 7. Where is the data stored in the External Table you created?

ANSWER: GCP Bucket - When we query an external table, BigQuery reads the data from the specified GCS URIs but does not ingest it into BigQuery storage.




Question 8. It is best practice in Big Query to always cluster your data:

ANSWER: False -  While clustering can improve query performance and reduce costs, it is not always necessary. The decision to cluster or not depends on your query patterns, table size, and how frequently specific filters are used
