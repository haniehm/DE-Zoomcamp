# Homework 3

## Bash Script and Google CLI for Uploading Parquet Files to Buckets

### Bash Script:
```bash
#!/bin/bash

# Set variables
BUCKET_NAME=hm-hw3
PROJECT_ID=dtc-de-course-445519
BASE_URL="https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2024-"
MONTHS=(01 02 03 04 05 06)
DOWNLOAD_DIR="."

# Authenticate & set project
gcloud auth login
gcloud config set project $PROJECT_ID

# Create the bucket if it doesn't exist
if ! gcloud storage buckets list --filter="name:$BUCKET_NAME" | grep $BUCKET_NAME; then
  echo "Creating bucket: $BUCKET_NAME..."
  gcloud storage buckets create gs://$BUCKET_NAME --location=us
fi

# Loop through the specified months
for month in "${MONTHS[@]}"; do
  FILE_NAME="yellow_tripdata_2024-${month}.parquet"
  URL="${BASE_URL}${month}.parquet"

  echo "Downloading $FILE_NAME..."
  wget -q $URL -O "${DOWNLOAD_DIR}/${FILE_NAME}" || { echo "Download failed: $FILE_NAME"; exit 1; }

  echo "Uploading $FILE_NAME to GCS..."
  gsutil cp "${DOWNLOAD_DIR}/${FILE_NAME}" gs://$BUCKET_NAME/ || { echo "Upload failed: $FILE_NAME"; exit 1; }

  echo "Cleaning up $FILE_NAME..."
  rm "${DOWNLOAD_DIR}/${FILE_NAME}"
done

echo "✅ All files uploaded successfully!"
```

### Terminal Commands:
```sh
chmod +x upload_tripdata.sh
# Triggering Authentication
./upload_tripdata.sh
```

## Initial Configurations

```sql
CREATE OR REPLACE EXTERNAL TABLE `dtc-de-course-445519.ny_taxi.yellow_tripdata_external`
OPTIONS (
  format = 'PARQUET',
  uris = ['gs://hm-hw3/*.parquet']
);

-- Load data from external source into a native BigQuery table
CREATE OR REPLACE TABLE `dtc-de-course-445519.ny_taxi.yellow_tripdata_native` AS
SELECT * FROM `dtc-de-course-445519.ny_taxi.yellow_tripdata_external`;

-- Create a materialized view from the native table
CREATE MATERIALIZED VIEW `dtc-de-course-445519.ny_taxi.yellow_tripdata_materialized_view` AS
SELECT * FROM `dtc-de-course-445519.ny_taxi.yellow_tripdata_native`;
```

## Queries

### Q1: Count Total Records
```sql
SELECT COUNT(*) FROM `dtc-de-course-445519.ny_taxi.yellow_tripdata_external`;
```
Output: **20,332,093**

### Q2: Count Unique `PULocationID`
```sql
SELECT DISTINCT COUNT(PULocationID) FROM dtc-de-course-445519.ny_taxi.yellow_tripdata_external;
SELECT DISTINCT COUNT(PULocationID) FROM dtc-de-course-445519.ny_taxi.yellow_tripdata_materialized_view;
```
**0 MB for the External Table and 155.12 MB for the Materialized Table**

### Q3: Columnar Querying Efficiency
```sql
SELECT PULocationID FROM `dtc-de-course-445519.ny_taxi.yellow_tripdata_native`;
SELECT PULocationID, DOLocationID FROM `dtc-de-course-445519.ny_taxi.yellow_tripdata_native`;
```
BigQuery is a columnar database, so querying two columns results in higher bytes processed.

### Q4: Count Zero Fare Records
```sql
SELECT COUNT(*) AS zero_fare_count
FROM `dtc-de-course-445519.ny_taxi.yellow_tripdata_native`
WHERE fare_amount = 0;
```

### Q5: Optimizing Table Performance
```sql
CREATE OR REPLACE TABLE `dtc-de-course-445519.ny_taxi.yellow_tripdata_optimized`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM `dtc-de-course-445519.ny_taxi.yellow_tripdata_native`;
```

### Q6: Performance Comparison Between Materialized and Optimized Tables
```sql
SELECT DISTINCT VendorID
FROM `dtc-de-course-445519.ny_taxi.yellow_tripdata_materialized_view`
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15';
-- (310.24 MB)

SELECT DISTINCT VendorID
FROM `dtc-de-course-445519.ny_taxi.yellow_tripdata_optimized`
WHERE tpep_dropoff_datetime BETWEEN '2024-03-01' AND '2024-03-15';
-- (26.84 MB)
```

## Q7: GCP Bucket
```sh
gcloud storage buckets list --filter="name:hm-hw3"
```

## Q8: Best Practices for Clustering in BigQuery

### Understanding Query Patterns
- Identify columns frequently used in `WHERE`, `GROUP BY`, `ORDER BY`, and `JOIN` clauses.

### Choosing Clustering Columns
- Use columns with high cardinality for effective clustering.

### Combining Partitioning and Clustering
- Partitioning reduces scanned data; clustering optimizes sorting and filtering.

### Monitoring and Optimizing
- Use query execution details to refine clustering strategies.

### Example: Partitioned and Clustered Table
```sql
CREATE OR REPLACE TABLE `dtc-de-course-445519.ny_taxi.yellow_tripdata_partitioned_clustered`
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM `dtc-de-course-445519.ny_taxi.yellow_tripdata_native`;
```
