# Module 1 Homework: Docker & SQL

## Question 1. Understanding docker first run 

Run docker with the `python:3.12.8` image in an interactive mode, use the entrypoint `bash`.

What's the version of `pip` in the image?

- 24.3.1
- 24.2.1
- 23.3.1
- 23.2.1
### Answer: 24.3.1

``` bash
docker run -it --rm python:3.12.8 pip --version
```


## Question 2. Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that **pgadmin** should use to connect to the postgres database?

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

- postgres:5433
- localhost:5432
- db:5433
- postgres:5432
- db:5432

If there are more than one answers, select only one of them

### Answer: postgres:5432 and db:5432

##  Prepare Postgres

Run Postgres and load data as shown in the videos
We'll use the green taxi trips from October 2019:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz
```

You will also need the dataset with zones:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

Download this data and put it into Postgres.

You can use the code from the course. It's up to you whether
you want to use Jupyter or a python script.

## Question 3. Trip Segmentation Count

During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, **respectively**, happened:
1. Up to 1 mile
2. In between 1 (exclusive) and 3 miles (inclusive),
3. In between 3 (exclusive) and 7 miles (inclusive),
4. In between 7 (exclusive) and 10 miles (inclusive),
5. Over 10 miles 

Answers:

- 104,802;  197,670;  110,612;  27,831;  35,281
- 104,802;  198,924;  109,603;  27,678;  35,189
- 104,793;  201,407;  110,612;  27,831;  35,281
- 104,793;  202,661;  109,603;  27,678;  35,189
- 104,838;  199,013;  109,645;  27,688;  35,202

 ### Answer: 104,838;  199,013;  109,645;  27,688;  35,202
  ```sql
  SELECT case when trip_distance <= 1 then '1. Up to 1 mile'
  when trip_distance > 1 and trip_distance <= 3 then '2. In between 1 (exclusive) and 3 miles (inclusive)'
  when trip_distance > 3 and trip_distance <= 7 then '3. In between 3 (exclusive) and 7 miles (inclusive)'
  when trip_distance > 7 and trip_distance <= 10 then '4. In between 7 (exclusive) and 10 miles (inclusive)'
  else '5. Over 10 miles'
  end as trip_segment,
  count(*) as trip_count
  from green_taxi_trips
  group by 1;
  ```


## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance?
Use the pick up time for your calculations.

Tip: For every day, we only care about one single trip with the longest distance. 

- 2019-10-11
- 2019-10-24
- 2019-10-26
- 2019-10-31

### Answer: 2019-10-31

```sql
SELECT pickup_date as max_trip_pickup_date from 
(SELECT date(lpep_pickup_datetime) as pickup_date,
max(trip_distance) as max_trip_distance
from green_taxi_trips
group by 1 order by 2 desc) limit 1;
```


## Question 5. Three biggest pickup zones

Which were the top pickup locations with over 13,000 in
`total_amount` (across all trips) for 2019-10-18?

Consider only `lpep_pickup_datetime` when filtering by date.
 
- East Harlem North, East Harlem South, Morningside Heights
- East Harlem North, Morningside Heights
- Morningside Heights, Astoria Park, East Harlem South
- Bedford, East Harlem North, Astoria Park

### Answer: East Harlem North, East Harlem South, Morningside Heights
```sql
SELECT b."Zone",
sum(a.total_amount) as total_amount
from green_taxi_trips a
left join public.taxi_zones b
on a."PULocationID" = b."LocationID"
where date(a.lpep_pickup_datetime) = '2019-10-18'
group by 1
having sum(a.total_amount) > 13000;
```

## Question 6. Largest tip

For the passengers picked up in October 2019 in the zone
named "East Harlem North" which was the drop off zone that had
the largest tip?

Note: it's `tip` , not `trip`

We need the name of the zone, not the ID.

- Yorkville West
- JFK Airport
- East Harlem North
- East Harlem South

### Answer: JFK Airport
```sql
with merged_table as
(SELECT a.*,
b."Zone" as "Pickup_Zone",
c."Zone" as "Dropoff_Zone"
from green_taxi_trips a
left join taxi_zones b
on a."PULocationID" = b."LocationID"
left join taxi_zones c
on a."DOLocationID" = c."LocationID")

SELECT "Dropoff_Zone", max(tip_amount) as max_tip_amount
from merged_table where "Pickup_Zone" = 'East Harlem North'
group by 1 order by 2 desc limit 1;
```


## Terraform



## Question 7. Terraform Workflow

Which of the following sequences, **respectively**, describes the workflow for: 
1. Downloading the provider plugins and setting up backend,
2. Generating proposed changes and auto-executing the plan
3. Remove all resources managed by terraform`

Answers:
- terraform import, terraform apply -y, terraform destroy
- teraform init, terraform plan -auto-apply, terraform rm
- terraform init, terraform run -auto-approve, terraform destroy
- terraform init, terraform apply -auto-approve, terraform destroy
- terraform import, terraform apply -y, terraform rm

### Answer: terraform init, terraform apply -auto-approve, terraform destroy

