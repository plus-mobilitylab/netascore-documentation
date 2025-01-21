# Requirements and Limitations
## Limitations

Please also consider the limitations of NetAScore:

- As with any data-driven method, quality of results depends on input data quality
- NetAScore uses modeling approaches that lead to some level of abstraction. Therefore, it can only be an approximation of reality and is designed to fit a certain purpose. However, we give you easy access to adjusting model parameters to your specific needs with the mode profile files.
- If you provide individual GeoPackages for optional data sets such as noise, buildings, etc., these need to be provided in the required format. Otherwise, processing will fail and give you an error message.

## Requirements for running Netascore with Python

Running NetAScore locally requires...

- Python 3.x
- recent PostrgeSQL with PostGIS extension
- psql
- ogr2ogr
- osm2pgsql
- raster 2 pgsql

...and the following Python-Libraries:

- gdal (Version 3.2.2)
- igraph
- pandas
- Flask (Version 2.0.3)
- psycopg2
- Jinja2 (earlier than Version 3.1.0)
- JinjaSql
- requests
- osm2geojson
- PyYaml

If you want more information about running NetAScore locally with python, more information can be found in the [guide for running NetAScore manually](Run-NetAScore-manually-with-Python)