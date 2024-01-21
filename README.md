# Data-Engineeering-Zoom-Camp
This course aims at giving out the basic knowledge of Data Engineering, the authors are intending to help student make their first step into data engineering by teaching basic concepts about data engineering. The tools and technonlogies that are being covered here are just a subset of the tools and concepts that are being used in the Data engineering industry.

## What does Data Engineers do ??
In a very high level view data engineers create and maintain data infrastructures. They create databases and create a way for data to flow into such databases and a way for other tools to access such data such as dashboards and analysts.

In a simple form you could install database in your local machine and design ETL tools to fetch data transform and load it into your database and give your dashboard and analysts access to such database.

However, doing so will mean that if I am running MacOS and my colleagues are running say Windows and Linux will have a lot of issues that are stemming from the infrastructure difference especially as the projects become more complex.

The solution is to install Database and management systems using Docker, in this way it will be easy for us to work and collaborate 

## Docker(Conternerization)
As data engineer your work will involve collaborating with other engineers and colleagues such as Data Analyst and Data Scientists, and the important part of that is the ability for you to be able to consistently collabolate with out difficulties stemming from difference of infrastructure used during development.

In another perspective as a data engineer you will need manage your infrastucture in a more sofsicated ways similar to the way you manage the application.

To be able to meet the above needs we need a conternerization tool such as Docker. It should be noted that there are a lot of such tools but docker is relatively more common.

Docker here will be used to launch database, its management system and a simple data pipeline.

### Postgres only on Docker.
Once we have docker installed in our machine then we can run posgres as application as a container from the distributed postgres image using the following commands 

For linux and MacOS
```
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
```

For Windos will be 
```
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v c:/Users/alexe/git/data-engineering-zoomcamp/week_1_basics_n_setup/2_docker_sql/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
```
This database can be accessed and explored by installing pgcli tool which can be installed by the following command
```
pip install pgcli
```
and we can connect to the database using the following command. 
```
pgcli -h localhost -p 5432 -u root -d ny_taxi
```
While pqcli tool can be used to interact with the database that we are running, it is not the most efficient especially when it comes to complex querries.

PgAdmin is a more efficient application that is used to connect to the database and allow us to interact with the database and it is more efficient for querries. We can install this application as another container but we will have to make sure that both containers are able to communicate.

To be able to make two containers communicate then we need to make sure that they are were launched on the same network, this can be achieved by describing network as the parameter.

### Postgres and PgAdmin on Docker
As described above, if we need to have more power over the database we will need to use PgAdmin which give us more efficient way to interact witht he database. 

Because PgAdmin will need to be able to talk to the database directly then we need to make sure that the Postgres and PgAdmin containers are all installed in the same network. This makes sure that when PgAdmin is looking for the Postgres database will be able to find it and connect. 

Command to create a network which should be created before we run the containers
```
docker network create pg-network
```

Command to run a postgres container within a network; note that you will have to also give it a name
```
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v c:/Users/alexe/git/data-engineering-zoomcamp/week_1_basics_n_setup/2_docker_sql/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  --network=pg-network \
  --name pg-database \
  postgres:13
```

Command to run PgAdmin container within a network
```
docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  --network=pg-network \
  --name pgadmin-2 \
  dpage/pgadmin4
```

To this point we are able to run applications in docker, and we have spin up our database and its management application. We can see how to get data into the database.

## Data Ingestion 
As long as our postgres is running in docker inside our machine, we can write scripts that connect to such database and ingest data into the database. The difference for when the container is in network and when it is not lies on the fact that when we have network it will need to be added. 

The ability to build images that are packaged with the depencies make it easier to run some applications with complex dependencies. Let say that our script to load data into the database has some complex dependancy on some packages, then we can define what packages are needed and then package that into image using the docker build command. Then whenever docker run this image it will make sure that all the dependancies have been packaged.

The definition of the needed packages are being stored inside the dockerfile

The command to build docker image 
```
docker build -t taxi_ingest:v001 .
```
In this case we are packaging a script data_ingest.py and the dependencies needed for this script to run.

Everytime we would want to run this script we can just run the command
```
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

docker run -it \
  --network=pg-network \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pg-database \
    --port=5432 \
    --db=ny_taxi \
    --table_name=yellow_taxi_trips \
    --url=${URL}
```
Mind the other inputs are the arguments for the script.

This is very handy when you share this project to someone who is running different infrastructure because they wont have to spend time setting up the infrastructure.

## Docker Compose
All that we have seen above can be easy to handle as long as there is few containers to be launched, but in a real world scenario its is usually not the case and projects can be very complicated and contains a lot of containers.

Docker compose is a way that docker takes instruction of what containers to be spin up and how those containers are related  and instead of launching each container one by one you can just do the docker-compose up command to get the described containers spin up and spin down.
