# Run NetAScore manually with Python

This sections describes how to run the project directly on your machine without Docker. For information on how to run it in a Docker environment see [docker-guide](How-to-run-the-project-in-a-Docker-environment). However, for a **quick start**, we recommend to use the **ready-made Docker image** as outlined in the section "How to get started" or the [Quickstart-Guide](Quickstart).

In order to run NetAScore, you need to prepare a **`data` directory** that contains at least the **settings file** and **mode profile files** for *bike* and *walk* profiles. You find **example files** in the `examples` subdirectory of the NetAScore repository. If available, also copy the input data sets and optional data sets to the `data` subdirectory in your repository root.

As a next step, make sure that you have a (local) **PostgreSQL** database running and that you specified the correct database connection details in the settings file.

Please find more information on possible options available in the **settings file** and their meaning in the [configuration guide](Configuration-of-the-settings).

## Software dependencies

NetAScore uses the following software:

- Python 3.9 or newer (tested with 3.13) and several Python libraries (see `requirements.txt`)
- PostgreSQL with PostGIS extension (database)
- osm2pgsql (version 1.9 or newer, for the flex output) and osmium - for OSM imports
- GDAL tools (`ogr2ogr`, `gdal_translate`) incl. the PostgreSQL driver
- psql (PostgreSQL client)
- raster2pgsql (part of PostGIS) - only for importing a DEM (optional dataset `dem`)
- git (optional) - to record the code state of NetAScore in the output metadata

### Recommended installation: conda environment

*If you don't have conda installed yet, we recommend the `miniconda` installation - see: https://www.anaconda.com/download/success?reg=skipped-miniconda*

The conda environment file `environment.yml` in the NetAScore repository includes all of the requirements stated above except the database and `raster2pgsql` (included: Python 3.13, GDAL incl. its PostgreSQL driver, geopandas, geopy, osm2pgsql, osmium-tool, psycopg2, pyyaml and jinjasql2):

```bash
conda env create -f environment.yml
conda activate netascore
```

`raster2pgsql` is only needed for DEM imports. On Linux and macOS you can add it to the environment with:

```bash
conda install -n netascore -c conda-forge postgis
```

On Windows (where PostGIS is not available via conda), use `raster2pgsql` of your PostGIS installation (e.g. installed with the Stack Builder of PostgreSQL). Make sure that its subfolder containing the binaries (e.g. `Program Files/PostgreSQL/<version>/bin`) is added to the PATH environment variable. NetAScore runs `raster2pgsql` from outside the conda environment without loading the GDAL plugins of the environment, which prevents version conflicts.

*Important: always execute NetAScore within the activated conda environment, as this ensures that the Python version and tools of the environment are used, rather than the system default.*

### Alternative: system packages and pip

Install osm2pgsql, osmium, GDAL (incl. Python bindings), and the PostgreSQL / PostGIS client tools with the package manager of your system or using graphical installers. Then install the **Python dependencies** with pip:

```bash
pip install -r requirements.txt
```

This is how the Docker image is built. Note: the Python package `gdal` needs to match the GDAL version installed on your system (e.g. `pip install gdal==$(gdal-config --version)`).

## Run NetAScore

Now you can **run NetAScore** by providing the filename of the settings file as commandline parameter to the Python script:

```bash
python generate_index.py data/your-settings-file.yml
```

Mode profile files are looked up in the directory of the settings file. The settings file may also be located outside the `data` directory. Input files referenced in the settings are always read from the `data` directory, and results are written there.

The processing can take several minutes up to several hours - depending on the size of the geodata and the hardware used. Therefore, we recommend to **start with a small example** first.

At the start, NetAScore prints its version and code state (Git branch and commit). At the end of a run, a summary is printed.

## Additional commandline parameters

If you want to re-run the pipeline, but **skip certain steps**, use the **`--skip`** flag with a list of steps you want to skip. The steps are executed in this order: `import`, `network`, `optional`, `attributes`, `index`, `export` (the steps are explained in detail in [The Workflow](The-workflow)). In the following example, `import`, `network` and `optional` are skipped. `attributes`, `index` and `export` still run:

```bash
python generate_index.py data/your-settings-file.yml --skip import network optional
```

If you change `buffer` or `clip` and skip the import step, NetAScore reports an error if the imported OSM data does not cover the new clip area. In that case, run the import step again.

If you want to get more (or less) detailed **log outputs**, set the **`--loglevel`** flag to one of the following levels:
`1` = MajorInfo, `2` = Info (default), `3` = Detailed, `4` = Debug. Warnings and errors are always shown. Internal messages and the Python traceback of errors are only shown at log level `3` or `4`.

## Trouble shooting and resolving common issues

### Issues with the installation of packages and dependencies

If you have issues with the execution after installation or with installing any of the packages directly, make sure to check versions of the packages installed in your environment. We recommend the conda environment (`environment.yml`), as it contains matching versions of GDAL, osm2pgsql and osmium.

If you are executing the program from the command line, make sure you have the virtual environment activated and the are using the right python version. Otherwise you may encounter some conflicts, as NetAScore may default to use the system-wide installation of Python, rather than the one specified within the virtual environment.

### NetAScore is stuck on the attribute step

If you encounter the calculations to be much slower than expected (for reference: Salzburg state rougly takes 10-15 minutes, the entirety of austria takes 1-2 hours), make sure that you put in a valid metric coordinate system for the `target_srid` in the `settings-file`. If an invalid or unfit coordinate system is provided, this error may occur.

### Unexpected gradient values without the use of a DEM

Datasets imported into the shared data schema of the database (e.g. a DEM imported for another case) are used by all cases, if they cover the network. If you ran NetAScore with a local database, check the layer `meta_datasets` of the output GeoPackage: it lists the input datasets used and their origin. Either specify another DEM you want to use for your area of interest or specify to not include the gradient in the [mode-profiles](Mode-profiles).

### OSM download fails

If the Overpass API servers are overloaded, OSM downloads may fail. NetAScore tries several servers and reports the errors. Areas larger than 1500 km² should not be downloaded via the Overpass API (setting `max_download_area`). In both cases, download an OSM extract (e.g. from [Geofabrik](https://download.geofabrik.de)), save it to the `data` directory and provide it as `filename` in the `import` section. Keep `place_name`, `bbox` or `aoi_filename` to clip the extract to your area of interest.

### Results changed after updating to version 2.0

NetAScore 2.0 imports OSM data with the flex output of osm2pgsql. After updating, run the import step again for existing OSM data. Also, mode profiles and settings files from version 1.x need to be updated (`version: 2.0`, gradient classes, new parking values). See the [changelog](https://github.com/plus-mobilitylab/netascore/blob/main/CHANGELOG.md) for all changes.
