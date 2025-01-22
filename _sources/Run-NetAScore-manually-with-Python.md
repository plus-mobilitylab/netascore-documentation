# Run NetAScore manually with Python
This sections describes how to run the project directly on your machine without Docker. For information on how to run it in a Docker environment see [docker-guide](How-to-run-the-project-in-a-Docker-environment). However, for a **quick start**, we recommend to use the **ready-made Docker image** as outlined in the section "How to get started" or the [Quickstart-Guide](Quickstart).

In order to run NetAScore, you need to prepare a **`data` directory** that contains at least the **settings file** and **mode profile files** for *bike* and *walk* profiles. You find **example files** in the `examples` subdirectory of the NetAScore repository. If available, also copy the input data sets and optional data sets to the `data` subdirectory in your repository root.

As a next step, make sure that you have a (local) **PostgreSQL** database running and that you specified the correct database connection details in the settings file. 

Please find more information on possible options available in the **settings file** and their meaning in the [configuration guide](Configuration-of-the-settings).

Make sure you have installed these **software dependencies**:

- Python 3.x
- recent PostgreSQL with PostGIS extension (make sure to enable all GDAL drivers if prompted; this should include psql, raster2pgsql and ogr2ogr)
- osm2pgsql ([osm2pgsql.org](https://osm2pgsql.org/doc/install.html)) (Please use a Version > 2.0, since the newer version has breaking changes that cause conflicts within NetAScore)

After installation of PostgreSQL, please make sure that its subfolder containing the binaries (e.g. `Program Files/PostgreSQL/<version>/bin`) is added to the PATH environment variable.

Then, install the **Python dependencies** for this project - with pip:

    RUN pip install -r requirements.txt

_If you run into issues with the installation via pip, you can also try using Anaconda for the installation, the necessary steps are outlined below._

Now you can **run NetAScore** by providing the filename of the settings file as commandline parameter to the Python script:

    python generate_index.py your-settings-file.yml

The processing can take several minutes up to several hours - depending on the size of the geodata and the hardware used. Therefore, we recommend to **start with a small example** first.

You can also use Anaconda to create a virtual environment with the following Python dependencies:

_We tested the installation using miniconda3 on Windows 10 (64-bit) and using the Anaconda powershell prompt on Windows 11 (64-Bit)._

- gdal (Version 3.6.2)
- igraph (Version 0.10.4)
- pandas (Version 1.5.1)
- Flask (Version 2.2.3)
- psycopg2 (Version 2.9.7)
- Jinja2 (Version 3.0.3)
- JinjaSql (Version 0.1.8)
- requests (Version 2.28.2)
- osm2geojson (Version 0.2.5)*
- PyYaml (Version 6.0)

*osm2geojson needs to be installed afterwards by using pip.

You can also import the environment file provided by running:

```
conda env create -f netascoreenvironment.yml
```
Afterwards activate the environment netascore3.9:
```
conda activate netascore3.9
```
You can now run NetAScore, using the prompt described above. 
_An important thing to note here, is to execute the command within miniconda or anaconda, as this ensures that the python version within the environment is used, rather than the system default._

If you want to manually install the requirements using conda, make sure to install the packages in the order given witihin the requirements.txt files, as some of the required versions may become overwritten otherwise, leading to conflicts.

## Additional commandline parameters

If you want to re-run the pipeline, but **skip certain steps**, use the **`--skip`** flag with a list of steps you want to skip:  `import`, `network`, `attributes`, `optional`, `index`, `export` (the steps are explained in detail in this readme file). In the following example, `import`, `network` and `attributes` steps will be skipped and only the `export` step is re-executed:

    python generate_index.py your-settings-file.yml --skip import network attributes

If you want to get more (or less) detailed **log outputs**, set the **`--loglevel`** flag to one of the following levels: 
`1` = MajorInfo, `2` = Info (default), `3` = Detailed, `4` = Debug

## Trouble shooting and resolving common issues

### Issues with the installation of packages and dependencies
If you have issues with the execution after installation or with installing any of the packages directly, make sure to check  versions of the packages installed in your environment and that you installed them in the right order. If the order of installation was changed, it may be that some of the versions specified were overwritten by the installation of another package.

If you are executing the program from the command line, make sure you have the virtual environment activated and the are using the right python version. Otherwise you may encounter some conflicts, as NetAScore may default to use the system-wide installation of Python, rather than the one specified within the virtual environment.

### NetAScore is stuck on the attribute step

If you encounter the calculations to be much slower than expected (for reference: Salzburg state rougly takes 10-15 minutes, the entirety of austria takes 1-2 hours), make sure that you put in a valid metric coordinate system for the `target_srid` in the `settings-file`. If an invalid or unfit coordinate system is provided, this error may occur.

### Error with the gradient calculations without the use of a DEM

If you ran NetAScore with a local database, make sure there is no other DEM for a different area of interest in the database, that may be used unknowingly. Either specify another DEM you want to use for your area of interest or specify to not include one in the [mode-profiles](Mode-profiles).

