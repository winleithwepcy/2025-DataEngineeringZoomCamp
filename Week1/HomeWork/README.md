# 2025 Data-Engineering-Zoomcamp
# Module 1 Homework: Docker & SQL
In this homework, the environment for Docker and SQL is prepared and practiced.

# Question 1: Understanding docker first run
Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash.  
What's the version of pip in the image?  
### Answer:  
Running ```docker run -it --entrypoint=bash python:3.12.8``` provides the version of pip in the image which is ```24.3.1```

# Question 2: Understanding Docker networking and docker-compose
Given the following ```docker-compose.yaml```, what is the ```hostname``` and ```port``` that **pgadmin** should use to connect to the postgres database? 
### Answer:
```db:5432```


# Question 3: Trip Segmentation Count
During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, **respectively**, happened:  
### Answer:
```104,802; 198,924; 109,603; 27,678; 35,189```
  
```sql
SELECT   
	COUNT(*) FILTER (WHERE trip_distance <= 1.00) AS up_to_1_mile,
    COUNT(*) FILTER (WHERE trip_distance > 1.00 AND trip_distance <= 3.00) AS between_1_and_3_miles,
    COUNT(*) FILTER (WHERE trip_distance > 3.00 AND trip_distance <= 7.00) AS between_3_and_7_miles,
    COUNT(*) FILTER (WHERE trip_distance > 7.00 AND trip_distance <= 10.00) AS between_7_and_10_miles,
    COUNT(*) FILTER (WHERE trip_distance > 10.00) AS over_10_miles
FROM green_taxi_data
	 WHERE lpep_pickup_datetime >= '2019-10-01 00:00:00'
	 AND lpep_pickup_datetime < '2019-11-01 00:00:00'
	 AND lpep_dropoff_datetime >= '2019-10-01 00:00:00'
	 AND lpep_dropoff_datetime < '2019-11-01 00:00:00';
```
# Question 4: Longest trip for each day

Which was the pick up day with the longest trip distance? Use the pick up time for your calculations.  
Tip: For every day, we only care about one single trip with the longest distance.
### Answer:
```2019-10-31```
### One Solution
```sql
SELECT lpep_pickup_datetime AS pick_up_day
FROM green_taxi_data
WHERE trip_distance = (SELECT MAX(trip_distance) FROM green_taxi_data);
```
### Alternative Solution
```sql
SELECT lpep_pickup_datetime AS pick_up_day
FROM green_taxi_data
ORDER BY trip_distance DESC
LIMIT (1);
```
# Question 5: Three biggest pickup zones

Which were the top pickup locations with over 13,000 in ```total_amount (across all trips)``` for 2019-10-18?  
Consider only ```lpep_pickup_datetime``` when filtering by date.
### Answer:
```East Harlem North, East Harlem South, Morningside Heights```
```sql
SELECT 
    zpu."Zone" AS pickup_zone,
    SUM(t.total_amount) AS total_amount
FROM zones zpu
LEFT JOIN green_taxi_data t
    ON zpu."LocationID" = t."PULocationID"
WHERE t.lpep_pickup_datetime >= '2019-10-18 00:00:00'
  AND t.lpep_pickup_datetime < '2019-10-19 00:00:00'
GROUP BY zpu."Zone"
HAVING SUM(t.total_amount) > 13000
ORDER BY total_amount DESC;
```

# Question 6: Largest tip
For the passengers picked up in October 2019 in the zone named "East Harlem North" which was the drop off zone that had the largest tip?  
Note: it's ```tip``` , not ```trip```
### Answer:
```JFK Airport```
```sql
SELECT
    zpu."Zone" AS drop_off_zone,
    MAX(t.tip_amount) AS max_tip
FROM zones zpu
JOIN green_taxi_data t ON zpu."LocationID" = t."DOLocationID"
WHERE t.lpep_pickup_datetime BETWEEN '2019-10-01 00:00:00' AND '2019-10-31 23:59:59'
    AND t."PULocationID" IN (
        SELECT "LocationID"
        FROM zones
        WHERE "Zone" = 'East Harlem North'
    )
GROUP BY zpu."Zone"
ORDER BY max_tip DESC
LIMIT 1;
```
# Question 7. Terraform Workflow

Which of the following sequences, **respectively**, describes the workflow for:  
Downloading the provider plugins and setting up backend,  
Generating proposed changes and auto-executing the plan  
Remove all resources managed by terraform
### Answer:
```terraform init, terraform apply -auto-approve, terraform destroy```