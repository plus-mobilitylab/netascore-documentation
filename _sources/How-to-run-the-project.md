# How to run NetAScore
If you want to run NetAScore you can choose between two modes, you can either use our [ready-to-use setup](How-to-run-the-project-in-a-Docker-environment) that utilizes a docker image or you can run it [manually using Python](Run-NetAScore-manually-with-Python). 
If you run into any issues, please feel free to submit an issue, outlining your problem, as we are very happy to receive feedback and to further improve the toolbox.

<!---expand the intro--->
```{warning}
When running NetAScore, please keep in mind that it works with OSM data, which can change over time, this is important especially if you want to compare datasets generated over a larger timespan. 
```

## Running NetAScore in Docker or better directly on your machine in Python?

The great advantage of running NetAScore in Docker is that you do not have to install any software or python dependencies on your machine. Only docker is required. Also the configuration is very simple, as you get a ready-to-use setup including a PostgreSQL database with PostGIS extension.

However, running NetAScore in Docker can potentially be slower on macOS and Windows than running the scripts directly on the machine in Python. We provide a possible solution to this issue in [docker-guide](How-to-run-the-project-in-a-Docker-environment) - the performance can be optimized by using a different way to mount the Docker volume. If you prefer to execute NetAScore directly on your machine, virtual environments or Conda may help with managing all Python requirements.

