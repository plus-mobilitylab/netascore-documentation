# Quickstart-Guide
## How to get started?

To get a better impression of what this toolset and workflow provides, you can quickly start with processing a sample area, in this case Salzburg, Austria.
In this guide you will learn how to run NetAScore for a sample area, how to adjust the settings to [run it for your own area of interest](#run-netascore-for-your-own-area-of-interest) and how to [add more detail](#add-more-detail) by using additional datasets (e.g. a DEM).

_If you run into any bugs when working with NetAScore, please feel free to submit an issue, outlining your problem, as we are very happy to receive feedback and to further improve the toolbox._

### Easy quickstart: ready-made Docker image

The easiest way to get started is running the ready-made Docker image. All you need for this to succeed is a [Docker installation](https://docs.docker.com/engine/install/), running Docker Desktop and internet connection. In order to run NetAScore for the sample Area, follow these steps:

- Install [Docker](https://docs.docker.com/engine/install/) and follow the instructions presented after you selected your operating System and make sure docker is running, the pane on the bottom left of the docker-desktop window should be green, like this:

![example of docker running](resources/img/docker_running.png)

- download the [`docker-compose.yml`](https://github.com/plus-mobilitylab/netascore/blob/main/examples/docker-compose.yml) raw-file from the `examples` and save it to an empty directory, in this Tutorial the directory is called `NetAScore`

- in order to proceed and access the terminal, search for `cmd` in your windows search bar (or for terminal on your mac) and open the command prompt app (or Terminal). Proceed by navigating to your previously created empty directory. This can be done by either manually typing the directory`s file path or by dragging and dropping the file into the command line app, similar to the example below: 

![navigation to directory of choice](resources/img/nav_to_dir.png)


 You can tell that you are now in the correct directory if the filepath given for the directory matches what you previously entered.

- from within this directory, execute the following command from the terminal (you should now see a process starting, that opens a connection to the database). Docker will start downloading the NetAScore image and PostgreSQL database image, setup the environment and execute the workflow for the exemplary case of Salzburg, Austria:
  `docker compose run netascore`

![command 'docker compose run netascore' shown in terminal](resources/img/run_netascore_docker.png)


- running the docker image might take a while, as NetAScore first loads an area of interest by place name from Overpass API, then downloads the respective OpenStreetMap data and afterwards imports, processes and exports the final dataset. You can tell the run was completed successfully once the Database connection is closed, which looks like this:

![completed successful run](resources/img/cmd_complete.png)

-  A new subdirectory named `data` will be present after successful execution. Within this folder, the assessed network is stored in `netascore_salzburg.gpkg`. It includes *bikeability* in columns `index_bike_ft` and `index_bike_tf` and *walkability* in `index_walk_ft` and `index_walk_tf`. The extensions `ft` and `tf` refer to the direction along an edge: *from-to* or *to-from* node. These values represent the assessed suitability of a segment for cycling (*bikeability*) and walking (*walkability*).

![example of the created data directory, where the results of the computation can be found.](resources/img/find_data_folder.png)


#### What the results look like:

Currently, NetAScore does not come with a built-in visualization module. However, you can easily visualize the *bikeability* and *walkability* index by loading the resulting geopackage in [QGIS](https://qgis.org). Simply drag and drop the geopackage into a new QGIS project and select the `edge` layer. Then in layer preferences, open the _layer styling panel_ and define a symbology that visualizes one of the computed index values - e.g. `index_bike_ft` for *bikeability* (`_ft`: bikeability in forward-direction of each segment). Please note that from version 1.0 onwards, an index value of `0` refers to unsuitable infrastructure, whereas `1` represents well suited infrastructure.

This is an exemplary visualization of *bikeability* for Salzburg, Austria:

![Bikeability result for Salzburg, Austria](resources/img/NetAScore_Example.png)


## Run NetAScore for your own area of interest

The easiest way to run a network assessment for your own area of interest is by adapting the given example in `examples/settings_osm_query.yml`:

- create a new **subdirectory** named **`data`** (if you already ran the quickstart example, you can just use the `data` directory created)
- download the **settings template** from `examples/settings_osm_query.yml` [file link](https://github.com/plus-mobilitylab/netascore/blob/5099d4a7ff6a2c348fe39dd9f0dfb8b8ba973c32/examples/settings_osm_query.yml). 
- add the **mode profiles** for *bikeability* and *walkability* to the `data` directory: download both, `profile_bike.yml` and `profile_walk.yml` from the `examples` folder.
- **edit** your newly created **settings file** `settings_osm_query.yml` - e.g. to download data for the City of London:
  - provide a **`case_id`**  in `global` section (only alphanumeric characters are allowed; this will be added e.g. to the output file name) - e.g. `case_id: london`

![change area of interest](resources/img/change_area_of_interest.png)

  - specify a **`place_name`** that is used to query data from OSM in the section `import`: e.g. `place_name: City of London`

![change place name](resources/img/change_place_name.png)

- **run NetAScore** by executing the following line from a terminal inside the main directory (parent of `data`):

  `docker compose run netascore data/settings_osm_query.yml`

  (Here, the last argument represents the settings file to use. Note that this is the general way of specifying the settings file that should be used.)

_NetAScore allows you to directly download OpenStreetMap data via Overpass API. If you want to provide a local OSM-Export you can do so using the 'filename' property instead of the 'place_name' that is given in the 'settings_osm_query.yml'. In some cases you might want to specify your search, if the place name given is not unique (e.g. Frankfurt (Main) & Frankfurt (Oder)). To accomplish a specification you can either specify additional parameters:_

* `admin_level`: filters the given results for OSM `admin_level` property (see [OSM documentation](https://wiki.openstreetmap.org/wiki/Item:Q1074))
* `zip_code`: filters the given results for a ZIP code (if available in OSM data)

Another option is to set the `interactive`- Variable to `interactive: True`, to opt for an interactive prompt.

**_Please note: Network data is being queried based on the bounding box (rectangle) containing the polygon returned for the place name query. If you do not specify a reference system (global option target_srid), the UTM zone suitable for the centroid of the area of interest is used._**

For further information on additional options for OSM see the [settings documentation](#additional-options-for-osm).

## Add more detail

The example settings use OpenStreetMap data as the only input. While this gives a good first estimate of *bikeability* and *walkability*, utilizing additional input datasets can further improve the quality of results. NetAScore supports additional datasets such as *DEM* (digital elevation model) and *noise* (e.g. traffic noise corridors). If you want to add a DEM as an additional dataset, we recommend using a DEM with at least a 10 m resolution (or finer), to have the necessary level of detail. Please refer to the [settings documentation](Configuration-of-the-settings) for details on the contents of the settings file and possible modifications.

To add optional input data sets, see the exemplary workflow below:

### Example: adding a DEM as an additional dataset:

- acquire the file(s) for your area of interest - availability of DEM, noise map, etc. may largely depend on the area of interest - in our case a [DEM of Great Britain](https://environment.data.gov.uk/dataset/ce8fe7e7-bed0-4889-8825-19b042e128d2), in order to add elevation to our previous area of interest (London).

- add the file(s) to the `data` subdirectory (where the settings file and mode profiles are located)

- edit the settings file to add the new datasets and store it inside the `data` folder - in our case, because we want to add elevation and the projection differs from the default, we have to change the `dem` subsection of `optional` (see [`settings_osm_file.yml`](https://github.com/plus-mobilitylab/netascore/blob/main/examples/settings_osm_file.yml) for template):

  - `filename` has to be changed to the name of the respective DEM-file we want to use

  - `srid` has to be changed from the default to the EPSG-code of the DEM (in our examplary case it has to be 27700, as the DEM is projected in the British National Grid).

- execute NetAScore from the parent directory:
  `docker compose run netascore data/<your_settings_file>.yml` 
  (where `<your_settings_file>` refers to the file name you chose for the edited settings file)

Using the same workflow, other additional data could also be added, for more options see the [settings-documentation](Configuration-of-the-settings).

_For additional information on working with docker and NetAScore, please refer to the [docker-guide](How-to-run-the-project-in-a-Docker-environment)._

