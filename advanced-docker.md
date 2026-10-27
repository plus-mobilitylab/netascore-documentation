# Advanced Docker customisation

The page [running NetAScore in a Docker environment](How-to-run-the-project-in-a-Docker-environment) describes basic use. 
Here, we provide more detail on available options for customisation when using NetAScore with Docker:

```{contents}
:local:
```

# Build the Docker image from source

The easiest way to build and launch NetAScore is by using docker compose. The `docker-compose.yml` inside the main code directory is configured accordingly (it is intended for development - end users can use `examples/docker-compose.yml`, which will fetch a ready-made image). Therefore, the only command you need to execute should be:

`docker compose build`

If you want to build your own image, we recommend to use the build script provided in `scripts/build_docker_image.py`. This takes care of recording the code state of NetAScore in output metadata (see [How to contribute](How-to-contribute)). `docker compose run` then uses this image:

`python scripts/build_docker_image.py --image plusmobilitylab/netascore`

Alternatively, set `NETASCORE_COMMIT` when building with compose (Linux / macOS): `NETASCORE_COMMIT="$(git describe --tags --dirty --always)" docker compose build`

Then, once you are sure that all input datasets, settings and mode profile files are properly placed inside the `data` subdirectory, execute NetAScore:

`docker compose run netascore data/<your_settings_file>.yml`

The docker compose command makes sure that the database container will start as well as the main NetAScore workflow and that both can communicate as intended.

## The manual, stepwise approach

You can build the Docker image yourself from source using the following command from within the main code directory:

`docker build -t netascore .`

This builds a local docker image named `netascore`. The image is based on `debian:trixie` with Python 3.13 and includes osm2pgsql, osmium, the GDAL tools and the PostgreSQL / PostGIS client tools. To record the code state of NetAScore in the output metadata, pass it as build argument, or use `python scripts/build_docker_image.py`, which does this for you (see [How to contribute](How-to-contribute)):

```bash
docker build --build-arg NETASCORE_COMMIT="$(git rev-parse --abbrev-ref HEAD), $(git describe --tags --dirty --always)" -t netascore .
```

To manually create a network for communication between NetAScore and the PostgreSQL database running in Docker execute the following (required only once per computer):

`docker network create netascore-net`

Then, to run the workflow, first start the PostgreSQL database and attach it to the network:

```bash
docker run --name netascore-db --network=netascore-net \
        -e POSTGRES_PASSWORD=postgres -d postgis/postgis:18-3.6
```

```bash
# Map TCP port 5432 in the container to port 5433 on the Docker host:
docker run --name netascore-db --network=netascore-net -p 5433:5432 \
        -e POSTGRES_PASSWORD=postgres -d postgis/postgis:18-3.6
```

Make sure that the database connection in your `settings.yml` is set up to use the Docker network:

```yaml
database:
    host: netascore-db
    port: 5432
    dbname: postgres
    username: postgres
    password: postgres
```

Make sure that you have all necessary geofiles, settings and mode profile files in the `data` subdirectory, because this directory is mounted into the netascore container:

```bash
# linux and mac:
docker run -i -t --network=netascore-net \
        -v $(pwd)/data:/usr/src/netascore/data netascore data/settings.yml
```

```shell
# windows:
docker run -i -t --network=netascore-net \
        -v %cd%/data:/usr/src/netascore/data netascore data/settings.yml
```

# Advanced configuration

## Only the database runs in docker

If the database runs in docker and your NetAScore workflow directly on the local machine (outside of docker), then you have to configure your database to accept connections from the local machine:

```bash
docker run --name netascore-db --network=netascore-net -p 5432:5432 \
        -e POSTGRES_PASSWORD=postgres -d postgis/postgis:18-3.6
```

Your `database` section in the settings file should point to the local port which is mapped to the database on localhost:

```yaml
database:
  host: localhost
  port: 5432
  dbname: postgres
  username: postgres
  password: postgres
```

Now you can use the python script as described in ["Run NetAScore manually with Python"](Run-NetAScore-manually-with-Python).

## Only the script runs in docker

If the script runs inside the docker container, it needs access to the database outside of the docker ecosystem. If the external database runs on another host, provide the necessary connection information in the `database` section. If you have the database running on your local system, then the host needs the IP address or hostname of the local system. Please note that `127.0.0.1` or `localhost` will not work, because it would try to connect to the container's localhost. If you are unable to obtain the ip of your
machine, or you cannot establish a connection, use `gateway.docker.internal`* as the host, e.g.:

```yaml
database:
  host: gateway.docker.internal
  port: 5432
  dbname: postgres
  username: postgres
  password: postgres
```

\* If this does not work for you, you can also try using `host.docker.internal` as the host.

## Access the docker database through a local PostgreSQL-Instance:

If you want to access the data stored in the Docker database instance you can do so e.g. using pgAdmin.

First run NetAScore through docker once, to start up all necessary processes. Afterwards check if the run was successful and if the Docker container is active.

Continue in PGAdmin and register a new server (Servers>Register>Server...).
Name your instance e.g. `netascore-db` and implement the following settings in the `Connection-tab`:

- set the host to: ***localhost***
- set the port to `5433`
- the both password and user to `postgres`
- save

![Postgres DB configuration](resources/img/postgres_config_docker.png)


The database should now be visible in pgAdmin and contain the data of the last case you ran.

<!-- TODO: explain datasets visible in DB-->

If you want to use a different password, username or another port, this can be configured in your `docker_compose.yml` file.

![Database customisation of the docker compose file](resources/img/docker-compose_custom.png)

The `port`-Section specifies the internal mapping between the port in the Docker container and the one of the local DB-Instance and the `environment`-settings specify the credentials for the Docker-DB.

## Troubleshooting and performance improvement

### Performance when running NetAScore in Docker

When using NetAScore in a docker image on mac or windows, overall performance of the pipeline can be 3-5 times slower compared to executing NetAScore in local Python or in Docker on Linux. This is caused by slow docker volume mounts and might be an issue for computations on large input files.
To resolve this issue, you can either execute the python script on your machine (outside Docker) or copy the files into a volume using the following steps:

```bash
docker volume create netascore-storage

docker create -t --network=netascore-net --name netascore-pipe \
        -v netascore-storage:/usr/src/netascore/data netascore data/settings.yml

docker cp data/. netascore-pipe:/usr/src/netascore/data

docker start netascore-pipe
```

To monitor the progress (logs), run:

```bash
docker logs -f netascore-pipe
```

This command will show the logs of the container and will follow the logs. You can stop the command with `ctrl+c`.

To copy the resulting files back to your local system, you can use the following command:

```bash
docker cp netascore-pipe:/usr/src/netascore/data/YOUR_RESULT_FILE1.gpkg .
docker cp netascore-pipe:/usr/src/netascore/data/YOUR_RESULT_FILE2.gpkg .
```

### Memory issues with large datasets

In case you experience errors when processing large datasets, please make sure that you have enough memory and disk space available.
Furthermore, it might be necessary to dedicate more memory to the database container. This can be done by adding the following line to `docker-compose.yml` within the section `netascore-db` (adjust the amount of memory to your needs):

```yaml
shm_size: 2gb
```

Then, the `netascore-db`-section of `docker-compose.yml` should look like this:

```yaml
netascore-db:
    image: postgis/postgis:18-3.6
    platform: linux/amd64
    shm_size: 2gb
    ports:
    - "5433:5432"
    environment:
    - POSTGRES_USER=postgres
    - POSTGRES_PASSWORD=postgres
    - POSTGRES_DB=postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 20s
      retries: 120
```

## Overwrite the osm2pgsql configuration for OSM import to database

For importing OpenStreetMap data into the database, NetAScore uses [osm2pgsql](https://osm2pgsql.org/) with its flex output. The import is configured in the Lua file `resources/osm2pgsql_flex.lua`, which NetAScore provides within its Docker container. It imports all tagged objects (not only those NetAScore currently uses), so further datasets can be derived from the OSM tables with SQL only. In order to customize the import you may perform the following steps:

- copy the file from the repository (`resources/osm2pgsql_flex.lua`)
- adapt the settings according to your needs (NetAScore relies on the tables `osm_point`, `osm_line`, `osm_polygon` and `osm_bicycle_route` and their columns)
- mount the file into the docker container when running it

```bash
# linux and mac:
docker run -i -t --network=netascore-net \
        -v $(pwd)/osm2pgsql_flex.lua:/usr/src/netascore/resources/osm2pgsql_flex.lua \
        -v $(pwd)/data:/usr/src/netascore/data netascore data/settings.yml
```

```shell
# windows:
docker run -i -t --network=netascore-net \
        -v %cd%/osm2pgsql_flex.lua:/usr/src/netascore/resources/osm2pgsql_flex.lua \
        -v %cd%/data:/usr/src/netascore/data netascore data/settings.yml
```

```{note}
The style file `resources/default.style` of versions before 2.0 (pgsql output of osm2pgsql) is no longer used. Custom styles need to be adapted to the flex configuration.
```
