# zoomcamp-hw-1

# Question 1
docker run -it --entrypoint bash python:3.12.8
pip --version

Answer: pip 24.3.1

# Question 2
postgres:5432

# Question 3
docker build -t taxi_ingest:v001 . 
export URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz"

docker run -it \
    --network=pg-network \
    taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pgdatabase \
    --port=5432 \
    --db=ny_taxi \
    --table_name=green_taxi_trips \
    --url=${URL}

# Pickup here need to load second link but have to make new python script