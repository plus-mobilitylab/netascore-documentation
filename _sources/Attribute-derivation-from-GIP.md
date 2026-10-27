# Attribute derivation from GIP

## Description of the input data

For Austria, indicators for bikeability and walkability index computation can be derived from the GIP network data (the Austrian authoritative reference system for transport infrastructure data). NetAScore uses the GIP OGD export as basis, which is available from [data.gv.at](https://www.data.gv.at/katalog/dataset/3fefc838-791d-4dde-975b-a4131a54e7c5). The legacy export (`type: gip`) is the split package `A_routingexport_ogd_split.zip`. Recent GIP 2.0 (`type: gip2`) is a single IDF file, typically `A_routingexport_ogd.zip` or `A_routingexport_ogd.txt`. The network can be restricted to parts of Austria with the import settings `regional_code` (municipality codes) or `bbox` - see [settings](Configuration-of-the-settings).

As the GIP data set already includes elevation data, there is no need to provide a DEM in order to derive the *gradient* indicator. A DEM given in the `optional` section is not imported for GIP (a note is issued). However, if you wish to consider *(traffic) noise*, you need to supply a respective noise polygon file. For Austria, such data is freely available from [inspire.gv.at](https://geometadatensuche.inspire.gv.at/metadatensuche/srv/eng/catalog.search#/metadata/125ec87c-7120-48a7-bd2c-2718cbf878c6). If needed, multiple files (covering different AOIs) have to be joined into a single input file before running NetAScore.

Indicators for walkability such as buildings, crossings, facilities, greenness, water, parking and sights are derived from an OSM file given in the `optional` section (subsection `osm`), or from individual GeoPackages. By default, the OSM file is clipped to the extent of the GIP network.

# Indicators

*More information about the process applied to derive all indicators will follow!*

For now, please refer to the general definition of indicators in [Attributes and Indicators](Attributes-and-Indicators) and check the source code at [`sql/templates/gip_attributes.sql.j2`](https://github.com/plus-mobilitylab/netascore/blob/main/sql/templates/gip_attributes.sql.j2) (`type: gip`) and [`sql/templates/gip2_attributes.sql.j2`](https://github.com/plus-mobilitylab/netascore/blob/main/sql/templates/gip2_attributes.sql.j2) (`type: gip2`), as well as the functions `sql/functions/gip_*.sql` and `sql/functions/gip2_*.sql`.

Bicycle roads (*Fahrradstraße*, bike feature `FRS`) are classified as `bicycle_road` (bicycle infrastructure) and `calmed` (road category) if they are accessible by car. Without car access, they are classified as `bicycle_way`.

Designated bicycle routes apply to both directions of a link (as included in the GIP input data).

# Source attributes

With the export setting `include_source_attributes`, GIP attributes can be added to the edge export layer (prefix `gip_`). Attributes of the linear use (`LinkUse` / `LINEAR_USE_PART`) carry the additional prefix `lu_` (`gip`) or `lup_` (`gip2`), attributes of the bike/hike data (`BikeHike` / `BIKE_HIKE`) the prefix `bh_`. The predefined sets of attributes are listed in [`core/attributes_step.py`](https://github.com/plus-mobilitylab/netascore/blob/main/core/attributes_step.py).
