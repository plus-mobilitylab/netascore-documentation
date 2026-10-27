# Requirements and Limitations

## Limitations

Please also consider the limitations of NetAScore:

- As with any data-driven method, quality of results depends on input data quality
- NetAScore uses modeling approaches that lead to some level of abstraction. Therefore, it can only be an approximation of reality and is designed to fit a certain purpose. However, we give you easy access to adjusting model parameters to your specific needs with the mode profile files.
- If you provide individual GeoPackages for optional data sets such as noise, buildings, etc., these need to be provided in the required format. Otherwise, processing will fail and give you an error message.
- OSM downloads via the Overpass API are limited to areas of up to 1500 km² (setting `max_download_area`). For larger areas, please use an OSM extract (e.g. from Geofabrik).
- Results of version 2.0 may differ from results of version 1.x (and from the publications referring to version 1.x), as indicator derivation was updated to reflect changes in OSM and GIP input data.

## Requirements for running NetAScore with Python

Running NetAScore locally requires...

- Python 3.9 or newer (tested with 3.13)
- recent PostgreSQL with PostGIS extension
- psql
- GDAL tools (`ogr2ogr`, `gdal_translate`) incl. the PostgreSQL driver
- osm2pgsql (version 1.9 or newer)
- osmium (osmium-tool) - for clipping OSM data to the area of interest
- raster2pgsql - only for DEM imports
- git (optional) - to record the code state of NetAScore in the output metadata

...and the following Python-Libraries:

- gdal
- geopandas (1.0 or newer)
- geopy
- jinjasql2
- psycopg2
- pyyaml

The conda environment `environment.yml` provided with NetAScore includes all of these except PostgreSQL and raster2pgsql.

You find more information about running NetAScore locally with python in the [guide for running NetAScore manually](Run-NetAScore-manually-with-Python)
