---

# Data Engineering Zoomcamp: Commands and Queries

This document contains commands, SQL queries, and results for the Data Engineering Zoomcamp.

---

## **Q1: Verify Python and Pip Version in Docker**
```bash
docker run -it --entrypoint bash python:3.12.8
pip --version
```
**Output:**  
`pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)`

---

## **Q2: Database Connection Details**
- **Hostname:** `db` (service name in Docker Compose).  
- **Port:** `5432` (internal container port).

---

## **Q3: Trip Distance Analysis (October 2019)**
```sql
-- Trips ≤ 1 mile
SELECT COUNT(*) AS trip_count
FROM green_tripdata_data
WHERE lpep_pickup_datetime >= '2019-10-01'
  AND lpep_dropoff_datetime < '2019-11-01'
  AND trip_distance <= 1;

-- Trips > 1 mile and ≤ 3 miles
SELECT COUNT(*) AS trip_count
FROM green_tripdata_data
WHERE lpep_pickup_datetime >= '2019-10-01'
  AND lpep_dropoff_datetime < '2019-11-01'
  AND trip_distance > 1 AND trip_distance <= 3;

-- Trips > 3 miles and ≤ 7 miles
SELECT COUNT(*) AS trip_count
FROM green_tripdata_data
WHERE lpep_pickup_datetime >= '2019-10-01'
  AND lpep_dropoff_datetime < '2019-11-01'
  AND trip_distance > 3 AND trip_distance <= 7;
```
**Results:**  
- ≤ 1 mile: **104,802**  
- > 1 mile and ≤ 3 miles: **198,924**  
- > 3 miles and ≤ 7 miles: **109,603**

---

## **Q4: Date with Maximum Trip Distance**
```sql
SELECT 
    DATE(lpep_pickup_datetime) AS pickup_date, 
    MAX(trip_distance) AS max_trip_distance
FROM green_tripdata_data
GROUP BY pickup_date
ORDER BY max_trip_distance DESC
LIMIT 1;
```
**Result:** **2019-10-31**

---

## **Q5: High-Revenue Pickup Locations (2019-10-18)**
```sql
SELECT 
    t."PULocationID", 
    z."Zone", 
    SUM(t.total_amount) AS total_amount
FROM green_tripdata_data t
JOIN zones z ON t."PULocationID" = z."LocationID"
WHERE DATE(t.lpep_pickup_datetime) = '2019-10-18'
GROUP BY t."PULocationID", z."Zone"
HAVING SUM(t.total_amount) > 13000
ORDER BY total_amount DESC;
```
**Top Zones:**  
1. East Harlem North  
2. East Harlem South  
3. Morningside Heights  

---

## **Q6: Maximum Tip from East Harlem North**
```sql
SELECT 
    t."DOLocationID", 
    z."Zone", 
    MAX(t.tip_amount) AS max_tip
FROM green_tripdata_data t
JOIN zones z ON t."DOLocationID" = z."LocationID"
WHERE DATE(t.lpep_pickup_datetime) BETWEEN '2019-10-01' AND '2019-10-31'
  AND t."PULocationID" = (SELECT "LocationID" FROM zones WHERE "Zone" = 'East Harlem North')
GROUP BY t."DOLocationID", z."Zone"
ORDER BY max_tip DESC
LIMIT 1;
```
**Result:** **JFK Airport**

---

