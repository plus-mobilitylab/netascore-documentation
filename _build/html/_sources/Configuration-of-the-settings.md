# Settings file

NetAScore uses a settings file that provides the necessary information to perform all steps for computing *bikeability* and *walkability*. It is written in the YAML markup language (general information about YAML can be found at https://yaml.org/).

We provide a set of **example settings files** with NetAScore. You find them inside the subdirectory `examples`. The most common to start from is `settings_osm_query.yml`. This is the file used by default in the Docker image. It is configured for easy use with all components running in Docker and no local input files required. See the [Quickstart](Quickstart) for instructions to execute NetAScore in this example setup. All available settings are listed (partly commented out) in the example files.

Settings files for NetAScore consist of several **sections** which correspond to the respective **processing step** of NetAScore. You may omit settings sections of processing steps that you will not execute. E.g. if you skip the `export` step, then you don't need to provide an `export` section. Exception: always provide the `import` section - it is used throughout the workflow.

In this document, we provide an overview of the structure of settings files and details on available settings for each processing step.

Besides the settings file, you also need to provide mode profiles. For each mode you want to compute a suitability index for you need to provide a corresponding **mode profile file**. NetAScore ships with default mode profiles provided in the `examples/` subdirectory. You may simply copy them to the `data` directory for usage in your own queries. Mode profile files are looked up in the directory of the settings file.

```{note}
Unknown keys in the settings file (e.g. typos) are reported as warnings and ignored. Please check the console output for such warnings if a setting does not seem to have an effect.
```

## Structure

The settings file can consist of the following **sections**:

- **version**:
  mandatory, single value. NetAScore 2.0 requires `2.0`. Older settings files are rejected.
- [**global**](#section-global):
  general settings such as case name (identifier) and target reference system (SRID) to use
- [**database**](#section-database):
  connection parameters for the PostgreSQL database
- [**import**](#section-import):
  mandatory: essential information for importing core (network) datasets
- [**optional**](#section-optional):
  information on optional datasets to import
- [**index**](#section-index):
  options for the index computation
- [**profiles**](#section-profiles):
  specification of indicator weights and indicator value mappings per mode profile - e.g. for *bikeability* and *walkability*
- [**export**](#section-export):
  information for exporting results

## Section `global`

### Property `target_srid`

**This parameter is optional.** If omitted, NetAScore determines a suitable reference system automatically:

- **OSM with an area of interest** (`place_name`, `bbox` or `aoi_filename`): the UTM zone of the centroid of the area of interest
- **OSM file import without area of interest**: the UTM zone of the mean position of the imported ways
- **GIP**: EPSG:32633 (WGS 84 / UTM zone 33N)

The reference system determined is stored with the area of interest of the case and used by all processing steps - also if you re-run single steps later (e.g. with `--skip import`).

Specify the SRID (spatial reference system as EPSG-code) to use for processing and data output.
Datasets will be imported and if necessary transformed to this reference system.
**Please note: currently only metric unit systems are supported such as UTM**.

**Example**:

- for WGS 84 UTM zone 33N use: 32633

```{warning}
Make sure to *only* use a metric unit system, otherwise the toolbox cannot run successfully.
```

### Property `case_id`

This parameter allows you to specify a unique identifier for the current task defined in the settings file. It may be useful if working with different areas of interest, different mode profiles, or when re-executing parts of a workflow. This identifier is also used in the default export file names (using `<case_id>` as a placeholder).

The `case_id` also names the area of interest and the database schema of the case (`netascore_case_<case_id>`). Different cases can therefore share one database.

**Note**: Only alphanumeric characters [A-Z, a-z, 0-9] and '_' are allowed. Other characters will be removed. The `case_id` may have at most 40 characters.

## Section `database`

Connection details for a PostgreSQL database are required in order to import, transform,  assess and export data.

Example settings are provided in the `examples` directory. These are a good starting point for further customization.

The **default settings** for using the **Docker** setup (NetAScore and PostgreSQL running in containers) are:

```yaml
database:
  host: netascore-db
  port: 5432
  dbname: postgres
  username: postgres
  password: postgres
```

If the `database` section or single properties are missing, these defaults are used for `host`, `port` and `dbname`.

If you want to connect to a **local PostgreSQL instance** outside of Docker, but running NetAScore in Docker you can use the following settings:

```yaml
database:
  host: gateway.docker.internal
  port: 5432              # port of existing PostgreSQL server on localhost
  dbname: postgres        # name of target database
  username: postgres      # local postgres user
  password: postgres
```

For using the whole toolset without Docker, just provide connection details for your PostgreSQL database as you do with any other software.

**Note**: For security reasons, you should **provide `username` and `password` as environment variables** if you do not work within a test environment. In this case just set `DB_USERNAME` and `DB_PASSWORD` accordingly and remove `username` and `password` keys from the settings file. Passwords are never written to the output metadata, also if prvided in the settings file.

```{note}
Before version 2.0, the `database` section accepted the property `on_existing`. It is no longer used: the processing steps (network, attributes, index, export) always replace their previous outputs. Use the command line option `--skip` to skip steps. For input data sets, `on_existing` is available in the sections `import` and `optional`.
```

## Section `import`

### Property `type`

Specify the type of geodata you want to import. At the moment, the following file formats
are supported:

- value `osm`: OpenStreetMap
- value `gip`: Austrian authoritative dataset *"Graphenintegrations-Plattform GIP"* (previous OGD routing export) - see also: [http://www.gip.gv.at](http://www.gip.gv.at/en/index.html)
- value `gip2`: Austrian authoritative dataset *"Graphenintegrations-Plattform GIP"* (GIP 2.0 OGD export) - see also: [http://www.gip.gv.at](http://www.gip.gv.at/en/index.html)

### Property `filename`

Refers to the file containing the geodata.

- OSM data can be imported from `PBF` or `XML` format, which can be downloaded e.g. from https://download.geofabrik.de. For directly downloading a small AOI with NetAScore, use one of the options outlined in the following section "Additional options for OSM".
- GIP data for Austria can be imported from the `IDF` export files which can be downloaded from [https://www.data.gv.at](https://www.data.gv.at/katalog/dataset/3fefc838-791d-4dde-975b-a4131a54e7c5)
  - Previous export (`type: gip`): split `IDF` package (typically `A_routingexport_ogd_split.zip`)
  - GIP 2.0 (`type: gip2`): single `IDF` file (typically `A_routingexport_ogd.zip` or `A_routingexport_ogd.txt`)

```{note}
For GIP, this property was called `filename_A` before version 2.0. Please rename it to `filename`.
```

### Property `on_existing`

This setting defines how to handle existing data from a previous run when importing input data, e.g. when re-running a case with the same `case_id`. For OSM, it applies to the area of interest (polygon), the OSM download, and the datasets derived from OSM (`building`, `crossing`, `facility`, `greenness`, `water`, `parking`, `sights`). For GIP, it applies to the imported GIP tables, which are shared by all GIP cases.

The possible values are:

- **`reuse`** (default): reuse existing data if it matches the current settings. If it does not match (e.g. you changed `place_name`, `bbox`, `aoi_filename`, `buffer`, `clip` or the input file - compared by name, size and modification time), NetAScore stops with an error.
- **`reuse_or_overwrite`**: reuse existing data if it matches the current settings, otherwise import it again, following the new settings.
- **`overwrite`**: always import again (replaces existing data).
- **`abort`**: terminate program execution if data exists (will report an error) - in this case, you are responsible to manually resolve conflicts.

NetAScore records the settings each area of interest (polygon) and each dataset was created from in a dataset registry in the database. This is how it recognizes whether existing data matches the current settings. Datasets derived from OSM are only reused if they cover the current import area. An existing OSM download is only reused if it is valid and covers the area of interest. If no valid download exists (e.g. because a previous run failed or was stopped during the download), the data is downloaded again (with `reuse`, an invalid download is reported as error).

For GIP, the network and attributes steps check that the GIP tables in the database were imported from the file given in the settings (they may have been replaced by another case in the meantime).

```{note}
The values `skip` and `delete` of versions before 2.0 are still accepted: `skip` corresponds to `reuse` and `delete` to `overwrite`.
```

### Additional options for OSM

NetAScore allows you to **directly download OpenStreetMap data** via Overpass API. You can simply provide one of the properties `place_name`, `bbox` or `aoi_filename` instead of the `filename` property. If you provide a `filename` **and** one of these properties, the OSM file provided is clipped to the area of interest: only the relevant part of the file is imported and processed.

- property **`place_name`**: Name of a place, referring to a polygon. Typically you could use the name of a city or city district. E.g. 'Salzburg' or 'City of London'. The place is looked up using OSM Nominatim. Only boundary results (polygons) are used, with the highest importance first. If the place name given is not unique, you can opt for an interactive prompt by adding the setting `interactive: True`. Otherwise the first result is used. Use a more precise place name if the first result is not the place you want, e.g. "Oberndorf, Salzburg, Österreich". You may check results via the search on [openstreetmap.org](https://www.openstreetmap.org) (which uses Nominatim).

  **Please note**: If the polygon returned for the place name is a multipolygon, only its largest part is used. If you do not specify a reference system (global option `target_srid`), the UTM zone suitable for the centroid of the area of interest is used.

- property **`bbox`**: Bounding box of your area of interest in WGS 84 (min. latitude, min. longitude, max. latitude, max. longitude) - given as list or comma-separated string, e.g. for Salzburg (Austria) use: `bbox: [47.7957, 13.0117, 47.8410, 13.0748]`
  **Please note**: If you do not specify a reference system (global option `target_srid`), the UTM zone suitable for the centroid of the area of interest is used.

- property **`aoi_filename`**: Name of a geofile in the data directory (e.g. a GeoPackage) that contains the area of interest as polygon geometry. If the file contains several features and values in the column `name` match `case_id`, those features are used. Otherwise all features are combined. Files in other coordinate reference systems than WGS 84 are transformed. Files without coordinate reference system are rejected.

  **Please note**: If you do not specify a reference system (global option `target_srid`), the UTM zone suitable for the centroid of the area of interest is used.

- property **`buffer`**: This option allows you to specify a spatial buffer (in meters) to enlarge the area of interest given by place name, bounding box or AOI file. If omitted or empty, no buffer is applied. Invalid values (e.g. text or negative numbers) are reported as error.

- property **`clip`**: Defines how the network is clipped to the area of interest:
  - `bbox` (default): bounding box of the area of interest, extended by `buffer`
  - `aoi`: geometry of the area of interest, extended by `buffer`
  - `none`: no clipping - for OSM file imports, the whole file is imported

  Edges crossing the boundary of the clipping area are kept as a whole. Clipping requires [osmium](https://osmcode.org/osmium-tool/) (included in the Docker image and the conda environment).

- property **`max_download_area`**: Maximum area (in km²) of OSM downloads from the Overpass API. Default: `1500`. For larger areas, please download an OSM extract (e.g. from [Geofabrik](https://download.geofabrik.de)), save it to the data directory and provide it as `filename`. Keep `place_name`, `bbox` or `aoi_filename` to clip the extract to your area of interest. Large downloads may fail or overload the Overpass API servers.

- properties **`include_rail`** and **`include_aerialway`**: These optional `boolean` tags allow you to include railway and aerialway features into the network dataset. This may be useful for visualization and specific types of analysis. For example, provide `include_rail: True` if you want railway geometry to be included in the output data set.

- For importing OpenStreetMap data into the database, NetAScore uses [osm2pgsql](https://osm2pgsql.org/) (version 1.9 or newer) with the flex output configuration `resources/osm2pgsql_flex.lua`.

```{note}
If the Overpass API servers are overloaded, downloads may fail. NetAScore tries several Overpass API endpoints and reports errors (e.g. timeouts or incomplete data). If all endpoints fail, retry later or download an OSM extract and provide it via the `filename` setting instead.
```

```{note}
The place name filters `admin_level` and `zip_code` of versions before 2.0 are no longer available. Please use a more precise `place_name` instead.
```

### Additional options for GIP

For GIP (`type: gip` and `type: gip2`), you can restrict the network to a part of Austria:

- property **`regional_code`**: municipality code(s) (*Gemeindekennzahl*), prefixed with `AT`. A single value or a list. `%` is a wildcard - e.g. `AT50101` (City of Salzburg), `[AT50101, AT50338]`, or `AT5%` (State of Salzburg).
- property **`bbox`**: bounding box in WGS 84 (min. latitude, min. longitude, max. latitude, max. longitude) - given as list or comma-separated string, e.g. `bbox: [47.75, 12.98, 47.86, 13.12]`

### Example import section

This is an example for OpenStreetMap import using a place name query:

```yaml
import:
  type: osm
  on_existing: reuse_or_overwrite
  place_name: Salzburg
  interactive: True
  buffer: 1000
  clip: bbox
```

The following example uses a local AOI polygon instead of a place name:

```yaml
import:
  type: osm
  on_existing: reuse_or_overwrite
  aoi_filename: aoi.gpkg
  buffer: 1000
```

The following example uses an existing OSM extract for Austria downloaded from [geofabrik.de](https://download.geofabrik.de) and clips it to a bounding box in Salzburg:

```yaml
import:
  type: osm
  filename: austria-latest.osm.pbf
  bbox: [47.7957, 13.0117, 47.8410, 13.0748]
  buffer: 1000
```

Without `place_name`, `bbox` or `aoi_filename` (or when `clip: none` is set), the whole OSM file is imported and processed.

The following example imports the GIP 2.0 export for the State of Salzburg:

```yaml
import:
  type: gip2
  filename: A_routingexport_ogd.zip
  regional_code: AT5%
```

## Section `optional`

This section specifies data that can optionally be imported for increased quality of results.

The following optional data is currently supported:

- **dem** (GeoTIFF): digital elevation model (preferably a terrain model)
- **noise** (GeoPackage: Polygon, MultiPolygon): polygons with noise attribute (e.g. traffic noise)
- **osm** (PBF or XML): only relevant for GIP import, if OpenStreetMap data should be used to infer the layers `building`, `crossing`, `facility`, `greenness`, `water`, `parking` and `sights`

The following layers can be supplied if OpenStreetMap data is not used or if you wish to use custom datasets:

- **building** (GeoPackage: Polygon)
- **crossing** (GeoPackage: Point, LineString)
- **facility** (GeoPackage: Point, Polygon)
- **greenness** (GeoPackage: Polygon)
- **water** (GeoPackage: LineString, Polygon)
- **parking** (GeoPackage: LineString, Polygon)
- **sights** (GeoPackage: Point, Polygon)

Instead of a GeoPackage, vector data can also be provided in other formats readable by GDAL (e.g. Shapefile, GeoJSON).

Optional datasets are imported after the network step. By default, only the part of each dataset within the extent of the network (plus a margin of 50 m) is imported.

```{note}
For **OSM** imports, the datasets `building`, `crossing`, `facility`, `greenness`, `water`, `parking` and `sights` are derived from the OSM data of the import step - unless they are given in the `optional` section (e.g. `building` with a GeoPackage): datasets listed explicitly take precedence and are not derived from OSM. An `osm` subsection is not needed and is ignored.
For **GIP** imports, a `dem` subsection is ignored: gradients are computed from the elevation data contained in the GIP dataset.
```

### Common properties

Each optional dataset accepts the following properties in addition to `filename`:

- `on_existing`: how to handle a dataset that already exists in the database - `reuse` (default), `reuse_or_overwrite`, `overwrite` or `abort` (see [`on_existing` in section import](#property-on_existing)). Datasets are shared between cases and only reused if they were imported from the same file with the same settings.
- `clip`: `network` (default) imports only data within the network extent (plus 50 m); `none` imports the whole file.
- `coverage` (not for `osm`): name of a GeoPackage with polygon(s) of the area covered by the dataset. Default: the extent of the data. In the attributes step, edges outside the coverage of a dataset get no indicator value (NULL) instead of e.g. `0`. Datasets not covering the network at all are not used (indicator fully removed from index computation).

Default values for all optional datasets can be set in the subsection `defaults`:

```yaml
optional:
  defaults:
    on_existing: reuse_or_overwrite
    clip: network
  dem:
    filename: austria.tif
    srid: 31287
```

Datasets in a different coordinate reference system than the network are transformed in the attributes step.

### Subsection `dem`

The DEM (digital elevation model) is used to add elevation values to the network nodes and to calculate the gradient of a road segment. Node elevations are interpolated bilinearly between neighbouring raster cells.

- Property `filename`: name of the file (GeoTIFF) to be imported
- Property `srid`:  spatial reference system identifier (SRID) of the dataset as EPSG code, e.g. `31287` or `EPSG:31287`

For Austria a 10 m x 10 m DEM can be downloaded here: [https://www.data.gv.at](https://www.data.gv.at/katalog/dataset/b5de6975-417b-4320-afdb-eb2a9e2a1dbf)

```{note}
Depending on your area of application, you can either use a Digital Surface Model or a Digital Elevation Model (terrain model), as the suitability of the models may differ depending on your specific use case. Note that Digital Surface Models contain vegetation and buildings - this may bias derived elevation especially for narrow roads and at coarse spatial resolutions.

```

### Subsection `noise`

The noise dataset contains a mapping of noise levels in decibels, represented as polygons with associated noise attribute.

- `filename`: name of the file to be imported
- `layer`: name of the layer to import - required if the file contains more than one layer with geometry
- `column_noise`: name of the column containing the noise values. Default: `noise`
- `value_mapping`: maps (non-numeric) values of the noise column to numeric noise values in dB. Values without entry are treated as areas without noise data.

For Austrian states the noise datasets can be downloaded here: [https://www.inspire.gv.at](https://geometadatensuche.inspire.gv.at/metadatensuche/srv/ger/catalog.search#/metadata/125ec87c-7120-48a7-bd2c-2718cbf878c6)

Example for a noise dataset with noise classes given as text:

```yaml
optional:
  noise:
    filename: noise.gpkg
    layer: noise_lden
    column_noise: dB_class
    value_mapping:
      "55-59": 57
      "60-64": 62
```

### Subsection `osm`

This is only relevant when working with GIP data as main input. An OSM dataset can be used to derive the following optional datasets: `building`, `crossing`, `facility`, `greenness`, `water`, `parking`, `sights`. Datasets of this list that are given as separate subsection (e.g. `building` with a GeoPackage) are imported from that file instead.

- `filename`: name of the OSM file to be imported
- `on_existing`, `clip`: see [common properties](#common-properties). With `clip: network` (default), only OSM data within the network extent is imported (requires osmium).

### Subsections `building`, `crossing`, `facility`, `greenness`, `water`, `parking`, `sights`

If these datasets are not derived from an OSM dataset, they can be imported from individual data sets. This might be useful e.g. when working with local, authoritative data sets. Datasets given here take precedence over the derivation from OSM (both for the OSM import and for the optional `osm` subsection).

- `filename`: name of the file to be imported
- `on_existing`, `clip`, `coverage`: see [common properties](#common-properties)

## Section `index`

- property **`compute_explanation`**: if `True`, the columns `index_<profile>_ft_explanation` and `index_<profile>_tf_explanation` are added to the output. They list the indicators used for index computation with their individual contributions to the index (JSON). Set it to `False` for a smaller output, e.g. in case of large networks. Default: `False`.

```yaml
index:
  compute_explanation: True
```

## Section `profiles`

NetAScore uses weights to represent the importance of individual indicators for a specific profile such as for cycling or walking. Different use cases may have different weights. Additionally, numeric indicator values are assigned to original attribute values in the mode profiles.

We include well-tested default mode profiles for cycling as well as walking with NetAScore. For general purpose assessments we recommend to utilize these profiles by copying the respective mode profile files `profile_bike.yml` and `profile_walk.yml` from `examples` to the `data` directory and referencing them from the settings file as follows:

```yaml
profiles:
  -
    profile_name: bike
    filename: profile_bike.yml
    filter_access_bike: True
  -
    profile_name: walk
    filename: profile_walk.yml
    filter_access_walk: True
```

You may edit these profiles or create your own custom profiles and add them to this section of the settings file. The `profile_name` value is included in the column name of the resulting index: e.g. `index_bike_ft`. Mode profiles are validated when the settings are loaded, before any processing step runs.

Since NetAScore version 1.1.0, index values are only computed for edges with legal access per mode. This filter is indicated by `filter_access_<mode>: True`. You may include segments accessible to other modes by adding multiple lines of `filter_access_<mode>`.  Possible modes are: `bike`, `walk` and `car`. For example, if you want to compute bikeability for all segments that are accessible by bike but also for those only open to pedestrians (e.g. for routing, where the bike can be pushed) you may use:

``````yaml
profiles:
  -
    profile_name: bike_incwalk
    filename: profile_bike.yml
    filter_access_bike: True
    filter_access_walk: True
``````

For details on weight file definition and computation steps involved, please refer to the [mode profiles](Mode-profiles) and [attributes](Attributes-and-Indicators) documentation.

## Section `export`

Currently, NetAScore supports exporting the assessed network into a geopackage file. You can define this as follows:

```yaml
export:
  type: geopackage
  filename: osm_network.gpkg
```

If you provide a `case_id` in the `global` settings section, you can include it in the export filename by using the placeholder `<case_id>`:

```yaml
export:
  type: geopackage
  filename: netascore_<case_id>.gpkg
```

The GeoPackage contains the layers `edge` and `node`, metadata layers describing the run (see [The Workflow](The-workflow)), and default layer styles for QGIS.

You can optionally include attributes of the input data in the edge export layer with `include_source_attributes`:

- `True`: a predefined set of columns
- a custom list of column names (GIP) or OSM tag keys (OSM), e.g. `[name, highway, lit, sidewalk:left]`

Source attributes are exported with the prefix `osm_` (OSM) or `gip_` (GIP and GIP 2.0).

```yaml
export:
  type: geopackage
  filename: netascore_<case_id>.gpkg
  include_source_attributes: [name, highway, lit, sidewalk:left]
```

The predefined set for OSM contains the tags `name`, `highway`, `ref`, `maxspeed`, `surface`, `smoothness`, `tracktype`, `width`, `lit`, `oneway`, `junction`, `access`, `motor_vehicle`, `foot`, `bicycle`, `sidewalk`, `cycleway`, `segregated` and `service`. For GIP, the predefined sets are listed in `core/attributes_step.py`; GIP columns from the linear use (`LinkUse` / `LINEAR_USE_PART`) and bike/hike tables carry the additional prefixes `lu_` / `lup_` and `bh_`.
