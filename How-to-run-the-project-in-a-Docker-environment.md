# Run NetAScore in Docker
In this file, we describe how to run all components or only parts in Docker.
There are two components involved:

1. NetAScore (Python source or docker image)
2. a PostGIS-enabled database (which is also provided as docker image)

```{contents}
:local:
```

## Quickstart

For a more in-depth guide, please refer to the more detailed [Quickstart](Quickstart).

NetAScore comes with a `docker compose` configuration in `docker-compose.yml` and a demo configuration, so you can simply run an example workflow by following these two steps (if you don't have Docker installed yet, please [install the Docker Engine](https://docs.docker.com/engine/install/) first):

- download the [`docker-compose.yml`](https://github.com/plus-mobilitylab/netascore/blob/main/examples/docker-compose.yml) file from the `examples` directory to an empty directory
- from within this directory, execute the following command from a terminal:
  `docker compose run netascore`

NetAScore first loads an area of interest from Overpass Turbo API, then downloads the respective OpenStreetMap data and afterwards imports, processes and exports the final dataset. A new subdirectory named `data` will be present after successful execution. Within this folder, the assessed network is stored in `netascore_salzburg.gpkg`. It includes *bikeability* in columns `index_bike_ft` and `index_bike_tf` and *walkability* in `index_walk_ft` and `index_walk_tf`. The extensions `ft` and `tf` refer to the direction along an edge: *from-to* or *to-from* node.

## Run NetAScore for your own area of interest

The easiest way to run a network assessment for your own area of interest is by adapting the given example in `examples/settings_osm_query.yml`:

- create a new **subdirectory** named **`data`** (if you already ran the quickstart example, you can just use the `data` directory created)
- download the [**settings template**](https://github.com/plus-mobilitylab/netascore/blob/main/examples/settings_osm_query.yml) or copy it from `examples/settings_osm_query.yml`
- add the **mode profiles** for *bikeability* and *walkability* to the `data` direcotry: copy both, `profile_bike.yml` and `profile_walk.yml` from the `examples` folder.
- **edit** your newly created **settings file** `settings_osm_query.yml` - e.g. to download data for the City of London:
  - provide a **`case_id`**  in `global` section (only alphanumeric characters are allowed - please avoid special characters such as German "Umlaute" etc.; this will be added e.g. to the output file name) - e.g. `case_id: london`
  - specify a **`place_name`** that is used to query data from OSM in the section `import`: e.g. `place_name: City of London` (please note: currently, this must equal the OSM "name" tag of your target area - you may check this using e.g. www.openstreetmap.org)
  - for editing this file we recommend using a code editor such as Visual Studio Code, Notepad++ or comparable which handles text encodings properly
- **run NetAScore** by executing the following line from a terminal inside the main directory (parent of `data`):
  `docker compose run netascore data/settings_osm_query.yml`
  (here, the last argument represents the settings file to use)

## Add more detail
The example settings use OpenStreetMap data as the only input. While this gives a good first estimate of *bikeability* and *walkability*, utilizing additional input datasets can further improve the quality of results. NetAScore supports additional datasets such as *DEM* (digital elevation model) and *noise* (e.g. traffic noise corridors). Please refer to the [settings documentation](Configuration-of-the-settings) for details.

To add optional input data sets, follow these steps:

- acquire the file(s) for your area of interest - availability of DEM, noise map, etc. may largely depend on the area of interest
- add the file(s) to the `data` subdirectory (where the settings file and mode profiles are located)
- edit the settings file to add the new datasets and store it inside the `data` folder
- execute NetAScore from the parent directory:
  `docker compose run netascore data/<your_settings_file>.yml` (where `<your_settings_file>` refers to the file name you chose for the edited settings file)

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