# Run NetAScore in Docker
In this file, we describe how to run all components or only parts in Docker.
There are two components involved:

1. NetAScore (Python source or docker image)
2. a PostGIS-enabled database (which is also provided as docker image)

```{contents}
:local:
```

## Quickstart

For a more in-depth guide, please refer to the more detailed [Quickstart](Quickstart). It covers the example workflow, running NetAScore for your own area of interest and adding datasets such as a DEM or noise.

## Manual use of the Docker image

If you want to use the NetAScore Docker image without docker compose or in a custom setting, you may simply get the latest version of the NetAScore image using:

```bash
docker pull plusmobilitylab/netascore:latest
```

To run the workflow with an existing postgres database, simply follow these steps:

- create a directory named `data` and place all geofiles inside
- add mode profile files and settings file to this directory (see example files provided in the code repository)
- adjust settings to your needs in the `settings.yml` file - see the [settings documentation](Configuration-of-the-settings) for reference
- finally, execute the workflow using:

```bash
docker run -i -t -v <dir_to_data_directory>:/usr/src/netascore/data plusmobilitylab/netascore data/settings.yml
```


## Further customisation

for further customisation please have a look at the [advanced docker customisation-page](advanced-docker), where you can find:
- How to access the Docker-Database from an external source
- How to manually use the Docker image
- Only running the Database in Docker
- Only running the Script in Docker