# zoomcamp-hw-1

# Question 1
docker run -it --entrypoint bash python:3.12.8
pip --version

Answer: pip 24.3.1

# Question 2
postgres:5432

# Question 3
### run old code but change tpep to lpep

docker build -t taxi_ingest:v007 . 
export URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz"

docker run -it \
    --network=pg-network \
    taxi_ingest:v007 \
    --user=root \
    --password=root \
    --host=pgdatabase \
    --port=5432 \
    --db=ny_taxi \
    --table_name=green_taxi_trips \
    --url=${URL}

### create new script for lookup file
docker build -f Dockerfile.taxi_lookup -t lookup_ingest:v003 .

export URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv"

docker run -it \
    --network=pg-network \
    lookup_ingest:v003 \
    --user=root \
    --password=root \
    --host=pgdatabase \
    --port=5432 \
    --db=ny_taxi \
    --table_name=taxi_lookup \
    --url=${URL}

### run SQL scripts to get answers
SELECT COUNT(*) FROM public.green_taxi_trips
WHERE lpep_pickup_datetime >= '2019-10-01 00:00:00'
AND lpep_dropoff_datetime < '2019-11-01 00:00:00'
AND trip_distance <= 1;
--104802

SELECT COUNT(*) FROM public.green_taxi_trips
WHERE lpep_pickup_datetime >= '2019-10-01 00:00:00'
AND lpep_dropoff_datetime < '2019-11-01 00:00:00'
AND trip_distance > 1 AND trip_distance <= 3;
--198924

SELECT COUNT(*) FROM public.green_taxi_trips
WHERE lpep_pickup_datetime >= '2019-10-01 00:00:00'
AND lpep_dropoff_datetime < '2019-11-01 00:00:00'
AND trip_distance > 3 AND trip_distance <= 7;
--109603

SELECT COUNT(*) FROM public.green_taxi_trips
WHERE lpep_pickup_datetime >= '2019-10-01 00:00:00'
AND lpep_dropoff_datetime < '2019-11-01 00:00:00'
AND trip_distance > 7 AND trip_distance <= 10;
--27678

SELECT COUNT(*) FROM public.green_taxi_trips
WHERE lpep_pickup_datetime >= '2019-10-01 00:00:00'
AND lpep_dropoff_datetime < '2019-11-01 00:00:00'
AND trip_distance >= 10;
--35281


# Question 4
SELECT DATE(lpep_pickup_datetime), MAX(trip_distance)
FROM public.green_taxi_trips
GROUP BY DATE(lpep_pickup_datetime)
ORDER BY MAX(trip_distance) DESC;
2019-10-31, 515.89

# Question 5
SELECT l."Zone", SUM(g."total_amount")
FROM 
	public.green_taxi_trips g
LEFT JOIN 
	public.taxi_lookup l ON l."LocationID" = g."PULocationID"
WHERE
	DATE(g."lpep_pickup_datetime") = '2019-10-18'
GROUP BY l."Zone"

HAVING
	SUM(g."total_amount") > 13000
ORDER BY SUM(g."total_amount") DESC;

Answer: East Harlem North, East Harlem South, Morningside Heights

# Question 6

SELECT l."Zone", g."tip_amount", l2."Zone" as "Drop Off Zone"
FROM 
	public.green_taxi_trips g
LEFT JOIN 
	public.taxi_lookup l ON l."LocationID" = g."PULocationID"
LEFT JOIN 
	public.taxi_lookup l2 ON l2."LocationID" = g."DOLocationID"
WHERE
	DATE(g."lpep_pickup_datetime") >= '2019-10-01' 
	AND DATE(g."lpep_pickup_datetime") < '2019-11-01'
	AND l."Zone" = 'East Harlem North'
ORDER BY g."tip_amount" DESC;

Answer: JFK Airport
