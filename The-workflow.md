# The Workflow

NetAScore uses six steps for generating *bikeability* and *walkability* indicators from network data and optional auxillary data sets. For each step, parameters can be defined for applying the generic workflow to a specific use case. These parameters are defined in a [settings file](https://github.com/plus-mobilitylab/netascore/blob/main/examples/settings_osm_query.yml) in YAML (`.yml`) format.

The settings file contains information about:

- how to connect to the database
- which parameters to use in the process
- which mode profiles (e.g. bike or walk) to use for generating index values
- what datasets to import
- how and where to export the results
- global settings such as SRID (spatial reference system)

The settings file itself does not contain geodata. It specifies all parameters such as which input files to use. You find details on which options are available and what they mean in the [settings](Configuration-of-the-settings) documentation. In general, each step of the workflow has its respective section in the settings file. NetAScore also comes with a set of examplary settings files in the `examples` subdirectory. These are a perfect starting point for adapting NetAScore to your specific demands.

The system relies on a PostgreSQL database, several geo-processing utilities, and a Python script that manages the whole workflow.

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

In this step a network dataset is imported. Currently `osm` (OpenStreetMap), `gip` (previous Austrian GIP OGD export) and `gip2` (GIP 2.0 OGD export) are supported, resulting in several database tables with the prefix `osm_`, `gip_` or `gip2_`.

`osm`

- The area of interest is determined from a place name, a bounding box or an AOI file (if given). If no reference system is specified, the UTM zone of the area of interest (or of the imported OSM data) is used.
- OSM data is either downloaded via the Overpass API for the area of interest, or read from a PBF or XML file, as defined in the settings file. If an area of interest is given, only the relevant part of the OSM file is imported (clipped with osmium).
- The OSM data is imported with osm2pgsql (flex output, `resources/osm2pgsql_flex.lua`). Besides points, lines and polygons, the member ways of bicycle route relations are imported (table `osm_bicycle_route`). The geometry is not transformed - this is done in the network_step.
- Additionally, the optional datasets `building`, `crossing`, `facility`, `greenness`, `water`, `parking` and `sights` are derived as database tables (except that these datasets are given explicitly in the `optional` section). The geometry is transformed.

`gip`

- The necessary sub-datasets are extracted from the ZIP files of the OGD dataset, as defined in the settings file.
- The extracted TXT files are parsed to CSV and SQL files, containing the raw data and `create table` statements.
- The database tables are imported from the SQL, CSV, GPKG files, primary keys are added, attributes are cleaned. The geometry is not transformed.

`gip2`

- The GIP 2.0 OGD export is read from a single IDF file, as defined in the settings file.
- The necessary sub-datasets are extracted from this file.
- The database tables are imported, primary keys are added, attributes are cleaned. The geometry is not transformed.

For `gip` and `gip2`, existing GIP tables are reused if they were imported from the same file (depending on `on_existing`). Unpacked and converted files are only reused if they were created from the current file.

Input data (area of interest, OSM download, datasets derived from OSM, GIP tables) is shared between cases in the database schema `netascore_data`. A dataset registry records the source and settings each dataset was created from. Depending on the setting `on_existing`, existing data is reused if it matches the current settings (see [settings](Configuration-of-the-settings)).

## 2. network_step

In this step the final network dataset is created, resulting in the database tables: `network_edge`, `network_node`.

`osm`

- The network dataset is reduced to necessary edges.
- The network dataset is reduced to necessary attributes.
- The geometry is transformed.
- The network is clipped to the area of interest (setting `clip`). Edges crossing the boundary of the clipping area are kept as a whole.
- The intersections are corrected, considering tunnels and bridges.
- The indoor edges are corrected.
- A new, unique `edge_id` is created in addition to the original `osm_id`. It is derived from the OSM way id and the position of the segment along the way (`osm_id * 10000 + segment number`), which allows to compare segments between separate runs of NetAScore.
- The bicycle route memberships of the network's ways are stored with the case (table `network_bicycle_route`), so that the following steps do not depend on OSM tables in the shared database schema.

`gip`, `gip2`

- Different sub-datasets are merged.
- The network dataset is reduced to necessary edges (optionally filtered by `regional_code` or `bbox`).
- The network dataset is reduced to necessary attributes.
- The geometry is transformed.

## 3. optional_step

In this step optional datasets are imported. Currently `dem`, `noise`, `building`, `crossing`, `facility`, `greenness`, `water`, `parking` and `sights` are supported, resulting in identically named database tables. For GIP, an OSM file can be provided to derive `building`, `crossing`, `facility`, `greenness`, `water`, `parking` and `sights`.

This step runs after the network step: by default, optional datasets are only imported for the extent of the network (plus a margin of 50 m), unless `clip: none` is set for the respective dataset.

`dem`

- A database raster table is created from a GeoTIFF file representing surface elevation, as defined in the settings file. Raster reprojection is not necessary. Projections are handled in the `attributes` step.
- The SRID of the input file needs to be defined in the settings file.
- **Please note:** DEM input is not considered for **GIP**-based workflow, as elevation data is already provided with the network dataset.

`noise`, `building`, `crossing`, `facility`, `greenness`, `water`, `parking`, `sights`

- A respective database table is generated from each input file. The geometry is transformed.
- For `noise`, the noise values are read from the column given by `column_noise` and can be mapped to numeric values with `value_mapping`.

## 4. attributes_step

In this step, necessary [attributes / indicators](Attributes-and-Indicators) for index calculation and potential routing applications are derived from the imported network dataset and optional datasets, resulting in the database tables: `network_edge_attributes`, `network_node_attributes`, `network_edge_export`.

- Input datasets are checked against the network: datasets covering only parts of the network result in no indicator value (NULL) for edges outside their coverage, datasets not covering the network are not used, and datasets in a different coordinate system are transformed.
- Attributes are derived: `access_*`, `bridge`, `tunnel`
- Indicators for the index calculation are transformed into predefined value domains so that the resulting values are comparable regardless of the source dataset and spatial extent.
- Indicators that can not be derived, because of missing datasets or attributes, are set to NULL.
- The table `network_edge_export` is created with a standardized table scheme independent of the source dataset. If requested (`include_source_attributes`), attributes of the input data are added.

## 5. index_step

In this step indices are calculated based on the mode profiles defined in the settings file, resulting in the tables: `network_edge_index`, `export_edge`, `export_node`.

- The index is calculated as a weighted average of all available indicators for every edge.
  - Original values of the indicators are rated with numeric values between 1 (best) and 0 (worst) regarding suitability for a given mode (e.g. bike or walk).
  - According to the weights defined in the mode profile, the indicators are then combined into a single index value per mode and edge (per direction).
  - The robustness of the index (`index_<profile>_ft_robustness` / `_tf_robustness`) represents the share of the weights of the available indicators relative to the total of weights. This means that missing input data for an indicator reduces the index robustness accordingly.
- The tables `export_edge` and `export_node` are created by joining the resulting datasets from the previous steps, including all attributes, indicators and indices.

## 6. export_step

In this step the tables `export_edge` and `export_node` are exported, as defined in the settings file.

`geopackage`

- The tables `export_edge` and `export_node` are exported to one geopackage with the layers `edge` and `node`.
- Metadata of the run is added as additional layers:
  - `meta_run`: NetAScore version and code state, settings (as YAML, without passwords), input file incl. OSM data timestamp, processing steps of the last run incl. completion time, run duration
  - `meta_steps`: per processing step: time, duration, NetAScore version and settings of the run in which it was executed last
  - `meta_datasets`: input datasets with their usage status, share of the network covered and origin (file)
  - `meta_profiles`: mode profiles with the original content of the profile files, SHA-256 checksum and weights
  - `meta_areas`: area of interest, clip area, network extent and coverage of the input datasets
- Default layer styles for QGIS are added: they are applied when a layer is opened in QGIS. For the `edge` layer, there is one style per mode profile visualizing its index (higher value of both directions); the style of the first profile is activated by default (use right-click on the layer > `Styles` to choose another one).

The processing steps network, attributes, index and export always replace their previous outputs in the database. Use the command line option `--skip` to skip steps (see [Run NetAScore manually with Python](Run-NetAScore-manually-with-Python)). At the end of a run, a summary is printed to the console showing you which input data sets were used.
