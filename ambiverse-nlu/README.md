# Supported tags and respective `Dockerfile` links

* latest [(latest/Dockerfile)](https://github.com/ambiverse-nlu/dockerfiles/blob/master/ambiverse-nlu/Dockerfile)
* latest-lowmem [(latest-lowmem/Dockerfile)](https://github.com/ambiverse-nlu/dockerfiles/blob/master/ambiverse-nlu/latest-lowmem/Dockerfile)
* 1.1.0 [(1.1.0/Dockerfile)](https://github.com/ambiverse-nlu/dockerfiles/blob/master/ambiverse-nlu/1.1.0/Dockerfile)
* 1.0.0 [(1.0.0/Dockerfile)](https://github.com/ambiverse-nlu/dockerfiles/blob/master/ambiverse-nlu/1.0.0/Dockerfile)

      
# AmbiverseNLU Dockerfile
This docker images is the official image of the [AmbiverseNLU](https://github.com/ambiverse-nlu/ambiverse-nlu) project.
The image contains the full repository of the code, and it starts a jetty webservice on `8080` port.

## Environment Variables
This image has one environment variable that need to be setup. 

### AIDA_CONF
This environmental variable is used to define the configuration used by the running webservice. 
There are two types of configuration depending on the database (postgres or cassandra), and they start with the database name and have an ending `_db` or `_cass` correspondingly. The configuration used it tightly connected with the database dump. 

These are the available configuration for the webservice:

* aida_20180120_cs_de_en_es_ru_zh_v18_db
* aida_20180120_cs_de_en_es_ru_zh_v18_cass
* aida_20180120_cs_de_en_es_ru_zh_v18_cass_2
* aida_20180120_b3_de_en_v18_db
* aida_20180120_b3_de_en_v18_cas
* default 

## Starting with `docker run`
If you want to start the container and link it to the `nlu-db-postgres` docker container that has the PostgreSQL database dump you need to the the following.
First start the PostreSQL docker container that contains the database dump:

~~~~~~~~
docker run -d --restart=always --name nlu-db-postgres \
  -p 5432:5432 \
  -e POSTGRES_DB=aida_20180120_cs_de_en_es_ru_zh_v18 \
  -e POSTGRES_USER=ambiversenlu \
  -e POSTGRES_PASSWORD=ambiversenlu \
  ambiverse/nlu-db-postgres
~~~~~~~~

&nbsp;

Then start the AmbiverseNLU container by linking the running PostgreSQL container.
~~~~~~~~
docker run -d --restart=always --name ambiverse-nlu \
 -p 8080:8080 \
 --link nlu-db-postgres:db \
 -e AIDA_CONF=aida_20180120_cs_de_en_es_ru_zh_v18_db \
 ambiverse/ambiverse-nlu
~~~~~~~~

&nbsp;

Similarly for Cassandra, first start the Cassandra container that contains the database dump:

~~~~~~~~
docker run -d --restart=always --name nlu-db-cassandra \
 -p 7000:7000 \
 -p 7001:7001 \
 -p 9042:9042 \
 -p 7199:7199 \
 -p 9160:9160 \
 -e DATABASE_NAME=aida_20180120_cs_de_en_es_ru_zh_v18 \
 ambiverse/nlu-db-cassandra
~~~~~~~~

&nbsp;
Then start the AmbiverseNLU container by linking the running Cassandra container.
~~~~~~~~
docker run -d --restart=always --name ambiverse-nlu \
 -p 8080:8080 \
 --link nlu-db-cassandra:db \
 -e AIDA_CONF=aida_20180120_cs_de_en_es_ru_zh_v18_cass \
 ambiverse/ambiverse-nlu
~~~~~~~~

&nbsp;

### ... or via `docker-stack deploy` or `docker-compose`
Example service-postgres.yml for [AmbiverseNLU](https://github.com/ambiverse-nlu/ambiverse-nlu):
~~~~~~~~
version: '3.1'

services:

  db:
    image: ambiverse/nlu-db-postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: ambiversenlu
      POSTGRES_DB: aida_20180120_cs_de_en_es_ru_zh_v18
      POSTGRES_USER: ambiversenlu
      
  nlu:
    image: ambiverse/ambiverse-nlu
    restart: always
    depends_on:
      - db
    ports:
      - 8080:8080
    environment:
      AIDA_CONF: aida_20180120_cs_de_en_es_ru_zh_v18_db
~~~~~~~~

&nbsp;

Run `docker stack deploy -c service-postgres.yml cassandra` (or `docker-compose -f service-postgres.yml up`), wait for it to initialize completely.

Example service-cassandra.yml for [AmbiverseNLU](https://github.com/ambiverse-nlu/ambiverse-nlu):
~~~~~~~~
version: '3.1'

services:

  db:
    image: ambiverse/nlu-db-cassandra
    restart: always
    environment:
      DATABASE_NAME: aida_20180120_cs_de_en_es_ru_zh_v18

  nlu:
    image: ambiverse/ambiverse-nlu
    restart: always
    depends_on:
      - db
    ports:
      - 8080:8080
    environment:
      AIDA_CONF: aida_20180120_cs_de_en_es_ru_zh_v18_cass
~~~~~~~~

&nbsp;

Run `docker stack deploy -c service-cassandra.yml cassandra` (or `docker-compose -f service-cassandra.yml up`), wait for it to initialize completely.

If you want to create a cluster of nodes, you can add another db service, and add the `CASSANDRA_SEEDS` env variable with values of both services, like this:

~~~~~~~~
version: '3.1'

services:

  db:
    image: ambiverse/nlu-db-cassandra
    restart: always
    environment:
      DATABASE_NAME: aida_20180120_cs_de_en_es_ru_zh_v18
      CASSANDRA_SEEDS: db,db1
  db1:
      image: ambiverse/nlu-db-cassandra
      restart: always
      environment:
        DATABASE_NAME: aida_20180120_cs_de_en_es_ru_zh_v18
        CASSANDRA_SEEDS: db,db1

  nlu:
    image: ambiverse/ambiverse-nlu
    restart: always
    depends_on:
      - db
      - db1
    ports:
      - 8080:8080
    environment:
      AIDA_CONF: aida_20180120_cs_de_en_es_ru_zh_v18_cass
~~~~~~~~

## Running a custom configuration
If you want to run a custom configuration, i.e. you have a running database server which is not a docker container, you can create a `.properties` files yourself, and link the file. 
For example, you have cassandra running one real servers, and you want the service to use this cassandra cluster, you create the file `cassandra.properties` and mount the file as a docker volume with the following command:

~~~~~~~~
docker run -d --restart=always --name ambiverse-nlu \
 -p 8080:8080 \
 -e AIDA_CONF=aida_20180120_cs_de_en_es_ru_zh_v18_cass \
 -v $(pwd)/cassandra.properties:/ambiverse-nlu/src/main/config/aida_20180120_cs_de_en_es_ru_zh_v18_cass/cassandra.properties \
 ambiverse/ambiverse-nlu
~~~~~~~~

Similarly for PostgreSQL:

~~~~~~~~
docker run -d --restart=always --name ambiverse-nlu \
 -p 8080:8080 \
 -e AIDA_CONF=aida_20180120_cs_de_en_es_ru_zh_v18_db \
 -v $(pwd)/database_aida.properties:/ambiverse-nlu/src/main/config/aida_20180120_cs_de_en_es_ru_zh_v18_db/database_aida.properties \
 ambiverse/ambiverse-nlu
~~~~~~~~ 