# Attribute derivation from OSM
NetAScore allows to derive indicators for bikeability and walkability index computation from OpenstreetMap network data. 
By default, OSM data is automatically queried using the *overpass turbo* API. See also [Configuration of the settings](Configuration-of-the-settings) for additional options.

Since `OSM` is maintained by volunteers via open collaboration, attributes might be mapped in various ways. For interpretation of OSM attributes to derive indicator values NetAScore follows the recommended tagging schemes as shown in the OSM wiki, but tries to also cover frequently used alternative specifications. 

While some of the NetAScore categories may overlap with OSM categories, there may be some differences. Therefore, please make sure to read the NetAScore indicator descriptions for further information.


# Attributes
_More Information about the exact derivation of individual attributes will follow soon!_ 

For now, please refer to the general definition of indicators in [Attributes and Indicators](Attributes-and-Indicators) and check the source code at [`sql/templates/osm_attributes.sql.j2`](https://github.com/plus-mobilitylab/netascore/blob/main/sql/templates/osm_attributes.sql.j2).

_The Queries given, that were used to request data and form the categories can be tested out using [Overpass turbo API](https://overpass-turbo.eu/#), (longer queries are provided as a separately linked `.txt` file for better readability)._


# Attributes
## bicycle_infrastructure_[tf/ft]

**Description:** Defines the bicycle infrastructure along a segment in both directions.


| Value<br/>__________________| Description<br/>_______________________________| Example <br/>____________________________|OSM<br/>_________________________________ |Overpass-QL Query<br/>_________________________________|
| :-------------------- |:--------------------------| :-----|:------|:--------|
| bicycle way          | Separate way, that is exclusively used by cyclists | ![Example of a designated bicycle path](https://wiki.openstreetmap.org/w/images/thumb/7/7d/Path-bicycledesignated.jpg/450px-Path-bicycledesignated.jpg?20130812161048)|bicycle=yes/designated + segregated=yes <br/><br/>cycleway=track/opposite_track/segregated/yes + foot!=yes/designated <br/><br/>cycleway=track/opposite_track/segregated/yes + segregated=yes <br/><br/>highway=cycleway + foot!=yes/designated <br/><br/>highway=cycleway + segregated=yes <br/><br/>bicycle_road=yes + foot!=yes/designated + maxspeed="" + motor_vehicle=no <br/><br/>bicycle_road=yes + sidewalk:\*=separate/yes + motor_vehicle=no <br/><br/>bicycle_road=yes + segregated=yes + motor_vehicle=no <br/><br/>cyclestreet=yes + foot!=yes/designated + motor_vehicle=no <br/><br/>cyclestreet=yes + sidewalk:\*=separate/yes + motor_vehicle=no <br/><br/>cyclestreet=yes + segregated=yes + motor_vehicle=no|[bicycle way query](https://github.com/christian-werner/testing-sandbox/blob/3d08904610572b243f8cc0876f2aca2ebbaecb49/attribute-ressources/bicycle_way.txt)|       
| mixed way            | Way with mixed use, dedicated to cyclists and pedestrians, but not used by MIV     |   ![Example of a mixed way](https://upload.wikimedia.org/wikipedia/commons/b/ba/Radweg_Schee_Silschede_cut.jpg) |cycleway=track/opposite_track/segregated/yes + foot=yes/designated <br/><br/>cycleway=track/opposite_track/segregated/yes + segregated=no <br/><br/>highway=cycleway + foot=yes/designated <br/><br/>highway=cycleway + segregated=no <br/><br/>bicycle_road=yes + foot=yes/designated + maxspeed="" + motor_vehicle=no <br/><br/>bicycle_road=yes + segregated=no + motor_vehicle=no <br/><br/>cyclestreet=yes + foot=yes/designated + motor_vehicle=no <br/><br/>cyclestreet=yes + segregated=yes + motor_vehicle=no <br/><br/>highway=footway + bicycle=yes/designated + access!=no/private + footway!=sidewalk <br/><br/>highway=pedestrian + bicycle=yes/designated| [mixed way query](https://github.com/christian-werner/testing-sandbox/blob/3d08904610572b243f8cc0876f2aca2ebbaecb49/attribute-ressources/mixed_way.txt)       |       
| bicycle lane         | bicycle lane, dedicated to the use by cyclists, e.g. on road-markings or cycle track      |   ![Example of a bicycle lane](https://upload.wikimedia.org/wikipedia/commons/thumb/4/49/387soft_separation_Graf-Moltke-Str_Bremen.jpg/800px-387soft_separation_Graf-Moltke-Str_Bremen.jpg)  |cycleway:*=lane <br/>cycleway:*=lane + oneway:bicycle=no<br/>cycleway:*=opposite_lane + oneway:bicycle=no <br/>cycleway:lane=exclusive/advisory/pictogram|[Bicycle lane query](https://github.com/christian-werner/testing-sandbox/blob/3d08904610572b243f8cc0876f2aca2ebbaecb49/attribute-ressources/bicycle_lane.txt)       |      
| bus lane             | bus lane with designated bicycle access      |   ![example of a lane shared between bus and cyclists](https://wiki.openstreetmap.org/w/images/thumb/e/ea/2010-01-02_15.19.16.jpg/800px-2010-01-02_15.19.16.jpg?20100102164456)  |cycleway=share_busway<br/>cycleway=opposite_share_busway<br/>cycleway=opposite_share_busway+oneway:bicycle=no<br/>_-> Use oneway:bicycle=no together with cycleway:left=share_busway and/or cycleway:right=share_busway, instead._|nwr[cycleway~"^\(share_busway\|opposite_share_busway\)$"].\(area.searchArea\) |
| no (or missing value) | absence of any bicycle infrastructure    |    - |       |       |

## pedestrian_infrastructure_[tf/ft]

**Description:** Defines the pedestrian  infrastructure along a segment in both directions. 

| Value<br/>__________________| Description<br/>_______________________________| Example <br/>_________________________________|OSM<br/>_________________________________ |Overpass-QL Query<br/>_________________________________|
|:--------------	        |:--------------	|:---------------|:-----------------------|:-----------------------|
| Pedestrian area  	|Road or an area mainly or exclusively for pedestrians.|![Example of a pedestrian area](https://upload.wikimedia.org/wikipedia/commons/1/1c/Cambridge_Rd_-_geograph.org.uk_-_1189572.jpg)   	|highway=pedestrian    	|nwr["highway"="pedestrian"][bicycle!~"^\(yes\|designated\|permissive\|destination\)$"]\(area.searchArea\)|  	
| Pedestrian way  	|Way accessible to pedestrians exclusively and segregated from other traffic.|![Example of a designated footway](https://wiki.openstreetmap.org/w/images/4/4d/Path-footdesignated.jpg?20080521104831)   	|highway=footway + footway!=sidewalk + bicycle!=yes/designated|nwr["highway"="footway"][bicycle!\~"^\(yes\|designated\)$"][access!\~"^\(no\|private\)$"]["footway"!="sidewalk"]\(area.searchArea\);|
| Mixed way  	   	|Road designated to pedestrians that is also accessible to cyclists.   	|![Example of a mixed way](https://upload.wikimedia.org/wikipedia/commons/thumb/5/58/Trakt_pieszorowerowy_wzd%C5%82u%C5%BC_Warty_w_Poznaniu_przy_ulicy_Ewangelickiej_-_kwiecie%C5%84_2018.jpg/800px-Trakt_pieszorowerowy_wzd%C5%82u%C5%BC_Warty_w_Poznaniu_przy_ulicy_Ewangelickiej_-_kwiecie%C5%84_2018.jpg)   	|_See bicycle_infrastructure!_ |[mixed way query](https://github.com/christian-werner/testing-sandbox/blob/3d08904610572b243f8cc0876f2aca2ebbaecb49/attribute-ressources/mixed_way.txt) |
| Stairs  	   	|A set of stairs.|![Example of a set of stairs](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c6/%22Wheeling_ramp%22%2C_Skeldergate_Bridge_-_geograph.org.uk_-_2424314.jpg/450px-%22Wheeling_ramp%22%2C_Skeldergate_Bridge_-_geograph.org.uk_-_2424314.jpg)   	|highway=steps    	|nwr["highway"="steps"]\(area.searchArea\)    |
| Sidewalk |indication that a sidewalk for pedestrian access is present on a road.|![Example of a road with a sidewalk](https://upload.wikimedia.org/wikipedia/commons/thumb/8/8c/PompeiiStreet.jpg/448px-PompeiiStreet.jpg)|highway=* + sidewalk=yes/left/right/both/separate|[Sidewalk Query](https://github.com/christian-werner/testing-sandbox/blob/3d08904610572b243f8cc0876f2aca2ebbaecb49/attribute-ressources/sidewalk.txt) |
|no     |No suitable pedestrian infrastructure or missing value. |

## designated_route[tf/ft]
**Description:** Designated bicycle routes (in Austria, e.g. Donauradweg), in both directions. 
| Value<br/>__________________| Description<br/>_______________________________| OSM <br/>_________________________________|Overpass-QL Query<br/>_________________________________ |
|:---	        |:---	        |:---	        |:---	|
|local   	|Designates that a road or path is part of a Local Cycling Network route.|network=lcn <br/>lcn=yes/proposed|(<br/>nwr["network"="lcn"]\(area.searchArea\);<br/>nwr["lcn"="yes"]\(area.searchArea\);<br/>);|   	        
|regional 	|Designates that a road or path is part of a Regional Cycling Network route.|network=rcn <br/>rcn=yes/proposed   	        |(<br/>nwr["network"="rcn"]\(area.searchArea\);<br/>nwr["rcn"="yes"]\(area.searchArea\);<br/>);   	|   	        
|national       |Designates that a road or path is part of a National Cycling Network route.|network=ncn <br/>ncn=yes/proposed   	        |(<br/>nwr["network"="ncn"]\(area.searchArea\);<br/>nwr["ncn"="yes"]\(area.searchArea\);<br/>);   	|   	        
|international  |Designates that a road or path is part of an International Cycling Network route.|network=icn <br/>icn=yes/proposed   	        |(<br/>nwr["network"="icn"]\(area.searchArea\);<br/>nwr["icn"="yes"]\(area.searchArea\);<br/>);   	|   	        

## road_category[tf/ft]
**Description:** Defines the road category along a segment in both directions. 
| Value<br/>__________________| Description<br/>_______________________________| Example <br/>_________________________________|OSM<br/>_________________________________ |Overpass-QL Query<br/>_________________________________|
|:---	        |:---	        |:---	        |:---	|:---	        |
|primary   	|highway linking large towns|![primary road in urban area](https://wiki.openstreetmap.org/w/images/thumb/c/c5/Dscf0172_600.jpg/360px-Dscf0172_600.jpg) ![primary road in rural area](https://wiki.openstreetmap.org/w/images/thumb/c/c3/Primary-photo.jpg/200px-Primary-photo.jpg)  	        |highway=primary/primary_link  	|nwr[highway~"^\(primary\|primary_link\)$"]\(area.searchArea\);   	        |
|secondary      |highways which are not part of major routes, but nevertheless form a link in the national route network;  cross-regional connecting roads between bigger locations|![secondary road in an urban area](https://wiki.openstreetmap.org/w/images/thumb/3/3d/Highway_Tram01.jpg/360px-Highway_Tram01.jpg)![secondary road in a rural area](https://wiki.openstreetmap.org/w/images/thumb/7/7b/Highway_secondary-photo.jpg/360px-Highway_secondary-photo.jpg)   	        |highway=secondary/secondary_link  	|nwr[highway~"^\(secondary\|secondary_link\)$"]\(area.searchArea\);         |
|residential    |road in residential area, with negligible transit traffic and Minor roads, connecting local centers of a large town or city or smaller settlements|![residential road in urban area](https://wiki.openstreetmap.org/w/images/thumb/1/12/Dscf0181_600.jpg/360px-Dscf0181_600.jpg) ![residential road in rural area](https://wiki.openstreetmap.org/w/images/thumb/f/f8/Brake-K200-Schmalenfleth1.JPG/360px-Brake-K200-Schmalenfleth1.JPG)  	        |highway=residential/tertiary/unclassified  |nwr[highway~"^\(residential\|tertiary\|tertiary_link\|unclassified\)$"]\(area.searchArea\);|
|service        |road, whose main purpose is access to a building, service station, beach, business park etc.|![service road](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/Alley_leading_from_the_High_Street_to_Green_Lane_Shopping_Centre_-_geograph.org.uk_-_942796.jpg/200px-Alley_leading_from_the_High_Street_to_Green_Lane_Shopping_Centre_-_geograph.org.uk_-_942796.jpg)   	        |highway=service  	|[service road query](https://github.com/christian-werner/testing-sandbox/blob/3d08904610572b243f8cc0876f2aca2ebbaecb49/attribute-ressources/service.txt)|
|calmed         |Road with very low speed limits and other pedestrian friendly traffic rules.|   	        | highway=living_street  	|(<br/>nwr["highway"="living_street"][access!\~"^\(no\|private\)$"]\(area.searchArea\);<br/>nwr[maxspeed\~"^\(20\|10\|15\|4\)$"][access!\~"^\(no\|private\)$"][motor_vehicle!\~"^\(yes\|designated\)$"]\(area.searchArea\);<br/>);|
|no_mit         |generic or multi-purpose path, not used by motorised vehicles|![example of a way classified no_mit](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b5/Wien_-_Graben_%281%29.JPG/220px-Wien_-_Graben_%281%29.JPG)|highway=bridleway<br/>highway=track + tracktype=grade1/grade2<br/>highway=path + surface=pave/chipseal/asphalt/concrete/paving_stones/bricks/brickcompacted _→ see pavement for more info!_|(<br/>nwr[highway\~"^\(bridleway\)$"]\(area.searchArea\);<br/>nwr["highway"="path"][access!\~"^\(no\|private\)$"][vehicle!\~"^\(no\|private\)$"][surface\~"^\(paved\|asphalt\|chipseal\|concrete\|paving_stones\|bricks\|brick\|compacted\)$"]\(area.searchArea\);<br/>nwr["highway"="track"]["motor_vehicle"="no"][access!\~"^\(no\|private\)$"][vehicle!\~"^\(no\|private\)$"][tracktype\~"^\(grade1\|grade2\)$"]\(area.searchArea\);<br/>);  	        |
|track           |generic or multi-purpose path, not used by motorised vehicles and with worse conditions than no_mit|![example of a path](https://wiki.openstreetmap.org/w/images/thumb/4/49/GuideFootPathCycleYes.jpg/200px-GuideFootPathCycleYes.jpg)|highway=path + surface!=paved/asphalt/chispeal/concrete/paving_stones/bricks/brick/compacted<br/>highway=track + motor_vehicle=no + tracktype=grade3/4/5| (<br/>nwr["highway"="path"][access!\~"^\(no\|private\)$"][vehicle!\~"^\(no\|private\)$"][surface!\~"^\(paved\|asphalt\|chipseal\|concrete\|paving_stones\|bricks\|brick\|compacted\)$"]\(area.searchArea\);<br/>nwr["highway"="track"]["motor_vehicle"="no"][access!\~"^\(no\|private\)$"][vehicle!\~"^\(no\|private\)$"][tracktype\~"^\(grade3\|grade4\|grade5\)$"]\(area.searchArea\);<br/>);|

## pavement
**Description:** Defines the pavement of segments. 
| Value<br/>__________________| Description<br/>_______________________________| Example <br/>_________________________________|OSM<br/>_________________________________ |Overpass-QL Query<br/>_________________________________|
|:---	        |:---	        |:---	        |:---	|:---	        |
|asphalt   	        |A feature that is predominantly paved; i.e., it is covered with paving stones, concrete or bitumen and well suited to be accessed by bike |![example of asphalt road](https://upload.wikimedia.org/wikipedia/commons/thumb/4/40/Asphaltbelag.jpg/220px-Asphaltbelag.jpg)   	        |(surface=paved) <br/>surface=asphalt<br/> surface=chipseal <br/>surface=concrete <br/>surface=metal  	|nwr[surface~"^\(paved\|asphalt\|chipseal\|concrete\|metal\)$"]|
|gravel   	        |A feature that is predominantly unsealed (unpaved); i.e., it has a loose covering , but still has some kind of covering like loose stones or gravel, that eases access by bike|![example or gravel road](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b5/%C5%A0t%C4%9Brkov%C3%A1_cesta_u_Pl%C3%A1%C5%A1t%C3%ADku.jpg/200px-%C5%A0t%C4%9Brkov%C3%A1_cesta_u_Pl%C3%A1%C5%A1t%C3%ADku.jpg)|surface=paving_stones <br/>surface=bricks <br/>surface=brick <br/>surface=compacted <br/>surface=fine_gravel <br/>surface=gravel <br/>surface=pebblestone <br/>(surface=ground) <br/>(surface=unpaved)| nwr[surface~"^\(paving_stones\|bricks\|brick\|compacted\|fine_gravel\|gravel\|pebblestone\|ground\|unpaved\)$"]|
|soft   	        |A feature that is predominantly unsealed (unpaved); i.e., it has a loose covering that considerably hinder access by bike, like earth or grass|![example of road with soft pavement](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c6/Gfp-florida-big-shaols-state-park-forest-trail.jpg/200px-Gfp-florida-big-shaols-state-park-forest-trail.jpg)|surface=dirt <br/>surface=earth <br/>surface=grass <br/>surface=sand <br/>surface=wood <br/>surface=woodchips <br/>(surface=ground)|nwr[surface~"^\(dirt\|earth\|grass\|sand\|wood\|woodchips\|ground\|soil\|dirt/sand\)$"]|
|cobble   	        |cobblestone road, unevenness may be unpleasant while biking.|![example of cobblestone road](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c2/Kohlenbrennerbr%C3%BCcke%2C_sett_paved_white_lines_01.jpg/200px-Kohlenbrennerbr%C3%BCcke%2C_sett_paved_white_lines_01.jpg)|surface=sett<br/> surface=unhewn_cobblestone <br/>(surface=cobblestone) <br/>surface=grass_paver| nwr[surface~"^\(sett\|unhewn_cobblestone\|cobblestone\|grass_paver\)$"]|


## number_lanes[tf/ft]
**Description:** Defines the number of lanes along one segment in both directions. 

## max_speed_[tf/ft]
**Description:** Defines maximum speed for motorized vehicles along segments in both directions. 

## max_speed
**Description:** Defines maximum speed for motorized vehicles.


## facility_ratio
**Description:** Counts the facilities (POIs) within a distance of 30 meters along a segment and ratios it to the segment length.

## greenness_ratio
**Description:** Describes the proportion of the green area within a 30 meters buffer: `0` to `100`.

## water_ratio
**Description:** Detects whether a water area is within a 30 meter distance to a segment or if a stream runs along a segment.

## crossing_ratio
**Description:** Counts the number of crossings within a 10 meter distance to the segment and ratios it to the segment length. 

## building_ratio
**Description:** Ratios the area of buildings within a 30 meter Buffer along a segment to the total area of the buffer.

