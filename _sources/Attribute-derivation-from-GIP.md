
# Attribute derivation from GIP
## Description of the input data
For Austria, indicators for bikeability and walkability index computation cen be derived from the GIP network data (the Austrian authoritative reference system for transport infrastructure data). NetAScore uses the GIP OGD export as basis (`A_routingexport_ogd_split.zip`), which is available from [data.gv.at](https://www.data.gv.at/katalog/en/dataset/3fefc838-791d-4dde-975b-a4131a54e7c5#resources).

As the GIP data set already includes elevation data, there is no need to provide a DEM in order to derive the *gradient* indicator. However, if you wish to consider *(traffic) noise*, you need to supply a respective noise polygon file. For Austria, such data is freely available from [inspire.gv.at](https://geometadatensuche.inspire.gv.at/metadatensuche/srv/eng/catalog.search#/metadata/125ec87c-7120-48a7-bd2c-2718cbf878c6). If needed, multiple files (covering different AOIs) have to be joined into a single input file before running NetAScore.


# Indicators

_More information about the process applied to derive all indicators will follow!_

For now, please refer to the general definition of indicators in [Attributes and Indicators](Attributes-and-Indicators) and check the source code at [`sql/templates/gip_attributes.sql.j2`](https://github.com/plus-mobilitylab/netascore/blob/main/sql/templates/gip_attributes.sql.j2).
