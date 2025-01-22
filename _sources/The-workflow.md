# The Workflow
NetAScore uses six steps for generating *bikeability* and *walkability* indicators from network data and optional auxillary data sets. For each step, parameters can be defined for applying the generic workflow to a specific use case. These parameters are defined in a [settings file](https://github.com/plus-mobilitylab/netascore/blob/main/examples/settings_osm_query.yml) in YAML (`.yml`) format. 

The settings file contains information about:

- how to connect to the database
- which parameters to use in the process
- which mode profiles (e.g. bike or walk) to use for generating index values
- what datasets to import
- how and where to export the results
- global settings such as SRID (spatial reference system)

The settings file itself does not contain geodata. It specifies all parameters such as which input files to be used. You find details on which options are available and what they mean in the wiki section [settings](Configuration-of-the-settings). In general, each step of the workflow has its respective section in the settings file. NetAScore also comes with a set of examplary settings files in the `examples` subdirectory. These are a perfect starting point for adapting NetAScore to your specific demands.

The system relies on a PostgreSQL database and a Python script that is executed.

The database can be

- stand-alone on the local machine
- provided in a Docker container (see [docker-guide](How-to-run-the-project-in-a-Docker-environment) for details)
- on a remote machine

The python script can either be executed directly, or run in a Docker container. More information can be found in the [user guide](How-to-run-the-project).

**The NetAScore workflow consists of the following steps:**

```{contents}
:local:
```


In the following subsections, we explain these individual steps.

## 1. import_step

In this step a network dataset is imported. Currently `osm` (OpenStreetMap) and `gip` (Austrian authoritative dataset) are supported, resulting in several database tables with the prefix `osm_` or `gip_`.

`osm`

- The necessary database tables are imported from a PBF or XML file, as defined in the settings file. The geometry is not transformed - this will be done in the network_step. Another option is to directly query OpenStreetMap data by providing a place name or bounding box.
- Additionally, the optional datasets `building`, `crossing`, `facility`, `greenness`, `water` are derived as database tables. The geometry is transformed.

`gip`

- The necessary sub-datasets are extracted from the ZIP files of the OGD dataset, as defined in the settings file.
- The extracted TXT files are parsed to CSV and SQL files, containing the raw data and `create table` statements.
- The database tables are imported from the SQL, CSV, GPKG files, primary keys are added, attributes are cleaned. The geometry is not transformed.

## 2. optional_step

In this step optional datasets are imported. Currently `dem`, `noise`, `building`, `crossing`, `facility`, `greenness` and `water` are supported, resulting in identically named database tables.

`dem`
- A database raster table is created from a GeoTIFF file representing surface elevation, as defined in the settings file. Raster reprojection is not necessary. Projections are handled in the `attributes` step.
- The SRID of the input file needs to be defined in the settings file.
- **Please note:** DEM input is not considered for **GIP**-based workflow, as elevation data is already provided with the network dataset.

`noise`, `building`, `crossing`, `facility`, `greenness`, `water`
- A respective database table is generated from each geopackage file. The geometry is transformed.

## 3. network_step

In this step the final network dataset is created, resulting in the database tables: `network_edge`, `network_node`.

`osm`
- The network dataset is reduced to necessary edges.
- The network dataset is reduced to necessary attributes.
- The geometry is transformed.
- The intersections are corrected, considering tunnels and bridges.
- The indoor edges are corrected.
- A new, unique edge ID is created in addition to original `osm_id`.

`gip`

- Different sub-datasets are merged.
- The network dataset is reduced to necessary edges.
- The network dataset is reduced to necessary attributes.
- The geometry is transformed.

## 4. attributes_step

In this step, necessary [attributes / indicators](Attributes-and-Indicators) for routing applications and the index calculation are derived from the final network dataset and optional datasets, resulting in the database tables: `network_edge_attributes`, `network_node_attributes`, `network_edge_export`.
- Attributes are derived: `access_*`, `bridge`, `tunnel`
- Indicators for the index calculation are derived into predefined value domains so that the resulting values are comparable regardless of the source dataset.
- Indicators that can not be derived, because of missing datasets or attributes, are set to NULL.
- The table `network_edge_export` is created with a standardized table scheme independent of the source dataset.

## 5. index_step

In this step indices will be calculated based on the mode profiles defined in the settings file, resulting in the tables: `network_edge_index`, `export_edge`, `export_node`.
- The index is calculated as a weighted average over all available indicators for every edge.
  - Original values of the indicators are rated with numeric values between 1 (best) and 0 (worst) regarding suitability for a given mode (e.g. bike or walk).
  - According to the weights defined in the mode profile, the indicators are then combined into a single index value per mode and edge (per direction).
- The tables `export_edge` and `export_node` are created by joining the resulting datasets from the previous steps, including all attributes, indicators and indices.

## 6. export_step

In this step the tables `export_edge` and `export_node` are exported, as defined in the settings file.

`geopackage`
- The tables `export_edge` and `export_node` are exported to one geopackage with the layers `edge` and `node`.