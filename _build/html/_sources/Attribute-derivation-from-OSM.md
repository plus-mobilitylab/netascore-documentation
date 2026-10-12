# Attribute derivation from OSM

NetAScore allows to derive indicators for bikeability and walkability index computation from OpenstreetMap network data.
By default, OSM data is automatically queried using the **Overpass API**. See also [Configuration of the settings](Configuration-of-the-settings) for additional options.

Since `OSM` is maintained by volunteers via open collaboration, attributes might be mapped in various ways. For interpretation of OSM attributes to derive indicator values NetAScore follows the recommended tagging schemes as shown in the OSM wiki, but tries to also cover frequently used alternative specifications.

While some of the NetAScore categories may overlap with OSM categories, there may be some differences. Therefore, please make sure to read the NetAScore indicator descriptions for further information.

# Attributes

Rules in a table are applied from top to bottom and the first match is used. A value written as `""` means the tag is missing. A `!=` comparison also matches a missing tag, except `access!=no` for `calmed`: a missing `access` does not match when `motor_vehicle` is missing.

Please refer to the general definition of indicators in [Attributes and Indicators](Attributes-and-Indicators) and to the implementation in [`sql/templates/osm_attributes.sql.j2`](https://github.com/plus-mobilitylab/netascore/blob/main/sql/templates/osm_attributes.sql.j2). Parking, water, sights, facilities and greenness are selected in [`core/import_step.py`](https://github.com/plus-mobilitylab/netascore/blob/main/core/import_step.py).

*The queries can be tested in [Overpass turbo](https://overpass-turbo.eu/). Longer queries are linked as `.txt` files. Zoom to the area of interest before running: those files query only the current map view.*

## bicycle_infrastructure_[tf/ft]

**Description:** Defines the bicycle infrastructure along a segment. Both directions (`ft` and `tf`) receive the same value. `highway=steps` is `no` before the rules below are tested.

| Value<br/>__________________ | Description<br/>_______________________________ | Example <br/>____________________________ | OSM<br/>_________________________________ | Overpass-QL Query<br/>_________________________________ |
| :--- | :--- | :--- | :--- | :--- |
| bicycle_way | Separate way, that is exclusively used by cyclists | ![Example of a designated bicycle path](https://wiki.openstreetmap.org/w/images/thumb/7/7d/Path-bicycledesignated.jpg/450px-Path-bicycledesignated.jpg?20130812161048) | highway=cycleway + foot!=yes/designated/permissive <br/>cycleway/cycleway:left/right/both=track/opposite_track/segregated/yes + foot!=yes/designated <br/>bicycle=yes/designated/official + segregated=yes + foot!=yes/designated + motor_vehicle=no/"" <br/>bicycle_road/cyclestreet=yes + maxspeed="" + motor_vehicle=no/"" + (foot!=yes/designated or segregated=yes or sidewalk=yes/separate/opposite/left/right or sidewalk:both=yes/separate or sidewalk:left/right=yes/separate/opposite) | [bicycle way query](resources/attributes/bicycle_way.txt) |
| mixed_way | Way with mixed use, dedicated to cyclists and pedestrians, but not used by MIV | ![Example of a mixed way](https://upload.wikimedia.org/wikipedia/commons/b/ba/Radweg_Schee_Silschede_cut.jpg) | cycleway=track/opposite_track/segregated/yes/opposite + foot=designated <br/>cycleway:left/right/both=track/opposite_track/segregated/yes + foot=designated <br/>highway=cycleway + foot=designated <br/>highway=cycleway + segregated=no + foot!=yes <br/>highway=footway + bicycle=designated + footway!=sidewalk <br/>highway=pedestrian + bicycle=designated <br/>bicycle_road/cyclestreet=yes + maxspeed="" + motor_vehicle=no/"" + (foot=designated or segregated=no) <br/>bicycle=designated + foot=designated + (motor_vehicle=no/destination/"" or access=no or highway=path/bridleway/cycleway/footway/track + tracktype!=grade4/grade5) | [mixed way query](resources/attributes/mixed_way.txt) |
| bicycle_lane | Bicycle lane, dedicated to the use by cyclists, e.g. on road-markings or cycle track | ![Example of a bicycle lane](https://thumb.wikimedia.org/wikipedia/commons/thumb/4/49/387soft_separation_Graf-Moltke-Str_Bremen.jpg/500px-387soft_separation_Graf-Moltke-Str_Bremen.jpg) | cycleway/cycleway:left/right/both=lane/opposite_lane<br/>cycleway:both:lane=advisory<br/>cycleway:lane=exclusive/advisory/pictogram | [Bicycle lane query](resources/attributes/bicycle_lane.txt) |
| shared_lane | Lane shared by cyclists and motor vehicles | - | cycleway/cycleway:left/right/both=shared_lane | [shared lane query](resources/attributes/shared_lane.txt) |
| bus_lane | Bus lane with designated bicycle access | ![example of a lane shared between bus and cyclists](https://wiki.openstreetmap.org/w/images/thumb/e/ea/2010-01-02_15.19.16.jpg/800px-2010-01-02_15.19.16.jpg?20100102164456) | cycleway/cycleway:left/right/both=share_busway/opposite_share_busway | [bus lane query](resources/attributes/bus_lane.txt) |
| mixed_way_non_designated | Way with mixed use, dedicated to cyclists and pedestrians, tagged with yes instead of designated | - | highway=cycleway + foot=yes<br/>highway=footway + bicycle=yes<br/>highway=pedestrian + bicycle=yes<br/>cycleway=track/opposite_track/segregated/yes/opposite + foot=yes<br/>cycleway:left/right/both=track/opposite_track/segregated/yes + foot=yes<br/>bicycle=yes/designated + foot=yes/designated + (motor_vehicle=no/destination/"" or access=no or highway=path/bridleway/cycleway/footway/track + tracktype!=grade4/grade5) | [mixed way query](resources/attributes/bicycle_mixed_way_non_designated.txt) |
| no_separate | Bicycle infrastructure mapped as its own way, not on the road centerline | - | cycleway/cycleway:left/right/both=separate | [separate cycleway query](resources/attributes/bicycle_no_separate.txt) |
| no | No rule above matched. | - | | |

## pedestrian_infrastructure_[tf/ft]

**Description:** Defines the pedestrian infrastructure along a segment. Both directions (`ft` and `tf`) receive the same value.

| Value<br/>__________________ | Description<br/>_______________________________ | Example <br/>_________________________________ | OSM<br/>_________________________________ | Overpass-QL Query<br/>_________________________________ |
| :--- | :--- | :--- | :--- | :--- |
| stairs | A set of stairs. | ![Example of a set of stairs](https://upload.wikimedia.org/wikipedia/commons/c/c6/%22Wheeling_ramp%22%2C_Skeldergate_Bridge_-_geograph.org.uk_-_2424314.jpg) | highway=steps | nwr["highway"="steps"]\(area.searchArea\) |
| pedestrian_area | Road or an area mainly or exclusively for pedestrians. | ![Example of a pedestrian area](https://upload.wikimedia.org/wikipedia/commons/1/1c/Cambridge_Rd_-_geograph.org.uk_-_1189572.jpg) | highway=pedestrian + bicycle!=designated | nwr["highway"="pedestrian"]["bicycle"!="designated"]\(area.searchArea\) |
| pedestrian_way | Way accessible to pedestrians exclusively and segregated from other traffic. | ![Example of a designated footway](https://wiki.openstreetmap.org/w/images/4/4d/Path-footdesignated.jpg?20080521104831) | highway=footway + foot=yes/designated/permissive/"" + bicycle!=yes/designated/permissive + footway!=sidewalk | nwr["highway"="footway"]["foot"\~"^\(yes\|designated\|permissive\)$"]["bicycle"!\~"^\(yes\|designated\|permissive\)$"]["footway"!="sidewalk"]\(area.searchArea\); nwr["highway"="footway"][!"foot"]["bicycle"!\~"^\(yes\|designated\|permissive\)$"]["footway"!="sidewalk"]\(area.searchArea\); |
| mixed_way | Road designated to pedestrians that is also accessible to cyclists. | ![Example of a mixed way](https://thumb.wikimedia.org/wikipedia/commons/thumb/5/58/Trakt_pieszorowerowy_wzd%C5%82u%C5%BC_Warty_w_Poznaniu_przy_ulicy_Ewangelickiej_-_kwiecie%C5%84_2018.jpg/500px-Trakt_pieszorowerowy_wzd%C5%82u%C5%BC_Warty_w_Poznaniu_przy_ulicy_Ewangelickiej_-_kwiecie%C5%84_2018.jpg) | highway=cycleway + foot=designated <br/>highway=footway + bicycle=designated <br/>highway=pedestrian + bicycle=designated <br/>cycleway=track/opposite_track/segregated/yes/opposite + foot=designated <br/>cycleway:left/right/both=track/opposite_track/segregated/yes + foot=designated <br/>bicycle_road/cyclestreet=yes + maxspeed="" + motor_vehicle=no/"" + (foot=designated or segregated=no) <br/>bicycle=designated + foot=designated + (motor_vehicle=no/destination/"" or access=no or highway=path/bridleway/cycleway/footway/track + tracktype!=grade4/grade5) | [mixed way query](resources/attributes/pedestrian_mixed_way.txt) |
| sidewalk | Indication that a sidewalk for pedestrian access is present on a road. | ![Example of a road with a sidewalk](https://thumb.wikimedia.org/wikipedia/commons/thumb/8/8c/PompeiiStreet.jpg/500px-PompeiiStreet.jpg) | footway=sidewalk<br/>sidewalk=yes/left/right/both/opposite<br/>sidewalk:both=yes/both<br/>sidewalk:left/right=yes/both/opposite | [sidewalk query](resources/attributes/sidewalk.txt) |
| mixed_way_non_designated | Way with mixed use, tagged with yes instead of designated | - | highway=cycleway + foot=yes<br/>highway=footway + bicycle=yes<br/>highway=pedestrian + bicycle=yes<br/>cycleway=track/opposite_track/segregated/yes/opposite + foot=yes<br/>cycleway:left/right/both=track/opposite_track/segregated/yes + foot=yes<br/>bicycle_road/cyclestreet=yes + maxspeed="" + motor_vehicle=no/"" + (foot=yes or segregated=no)<br/>bicycle=yes/designated + foot=yes/designated + (motor_vehicle=no/destination/"" or access=no or highway=path/bridleway/cycleway/footway/track + tracktype!=grade4/grade5) | [mixed way query](resources/attributes/pedestrian_mixed_way_non_designated.txt) |
| no_separate | sidewalk or cycleway mapped as its own way | - | cycleway/cycleway:left/right/both=separate + foot=yes/designated<br/>sidewalk/sidewalk:left/right/both=separate | [no separate sidewalk query](resources/attributes/pedestrian_no_separate.txt) |
| no | No rule above matched. | | | |

## designated_route[tf/ft]

**Description:** Designated bicycle routes (in Austria, e.g. Donauradweg). Both directions receive the same value. Only lines tagged `route=bicycle` are used, and only when their geometry contains the segment. The highest matching scope wins.

| Value<br/>__________________ | Description<br/>_______________________________ | OSM <br/>_________________________________ | Overpass-QL Query<br/>_________________________________ |
| :--- | :--- | :--- | :--- |
| international | Designates that a road or path is part of an International Cycling Network route. | route=bicycle + network=icn <br/>route=bicycle + icn=yes | nwr["route"="bicycle"]["network"="icn"]\(area.searchArea\); nwr["route"="bicycle"]["icn"="yes"]\(area.searchArea\); |
| national | Designates that a road or path is part of a National Cycling Network route. | route=bicycle + network=ncn <br/>route=bicycle + ncn=yes | nwr["route"="bicycle"]["network"="ncn"]\(area.searchArea\); nwr["route"="bicycle"]["ncn"="yes"]\(area.searchArea\); |
| regional | Designates that a road or path is part of a Regional Cycling Network route. | route=bicycle + network=rcn <br/>route=bicycle + network=regional <br/>route=bicycle + rcn=yes | nwr["route"="bicycle"]["network"="rcn"]\(area.searchArea\); nwr["route"="bicycle"]["network"="regional"]\(area.searchArea\); nwr["route"="bicycle"]["rcn"="yes"]\(area.searchArea\); |
| local | Designates that a road or path is part of a Local Cycling Network route. | route=bicycle + network=lcn <br/>route=bicycle + lcn=yes | nwr["route"="bicycle"]["network"="lcn"]\(area.searchArea\); nwr["route"="bicycle"]["lcn"="yes"]\(area.searchArea\); |
| unknown | Bicycle route without one of the network scopes above. | route=bicycle | nwr["route"="bicycle"]\(area.searchArea\); |
| no | Segment is not contained in a bicycle route. | | |

## road_category

**Description:** Defines one road category per segment.

| Value<br/>__________________ | Description<br/>_______________________________ | Example <br/>_________________________________ | OSM<br/>_________________________________ | Overpass-QL Query<br/>_________________________________ |
| :--- | :--- | :--- | :--- | :--- |
| primary | highway linking large towns | ![primary road in urban area](https://wiki.openstreetmap.org/w/images/thumb/c/c5/Dscf0172_600.jpg/360px-Dscf0172_600.jpg) ![primary road in rural area](https://wiki.openstreetmap.org/w/images/thumb/c/c3/Primary-photo.jpg/200px-Primary-photo.jpg) | highway=primary/primary_link/motorway/motorway_link | nwr[highway~"^\(primary\|primary_link\|motorway\|motorway_link\)$"]\(area.searchArea\); |
| secondary | highways which are not part of major routes, but nevertheless form a link in the national route network;  cross-regional connecting roads between bigger locations | ![secondary road in an urban area](https://wiki.openstreetmap.org/w/images/thumb/3/3d/Highway_Tram01.jpg/360px-Highway_Tram01.jpg)![secondary road in a rural area](https://wiki.openstreetmap.org/w/images/thumb/7/7b/Highway_secondary-photo.jpg/360px-Highway_secondary-photo.jpg) | highway=secondary/secondary_link <br/>maxspeed=100/90/80/70 + motor_vehicle!=no | nwr[highway~"^\(secondary\|secondary_link\)$"]\(area.searchArea\); nwr["maxspeed"~"^\(100\|90\|80\|70\)$"]["motor_vehicle"!="no"]\(area.searchArea\); |
| residential | road in residential area, with negligible transit traffic and Minor roads, connecting local centers of a large town or city or smaller settlements | ![residential road in urban area](https://wiki.openstreetmap.org/w/images/thumb/1/12/Dscf0181_600.jpg/360px-Dscf0181_600.jpg) ![residential road in rural area](https://wiki.openstreetmap.org/w/images/thumb/f/f8/Brake-K200-Schmalenfleth1.JPG/360px-Brake-K200-Schmalenfleth1.JPG) | highway=residential/tertiary/tertiary_link + motor_vehicle!=no <br/>highway=unclassified + motor_vehicle!=no <br/>maxspeed=50/40 + motor_vehicle=yes/designated/"" | nwr[highway~"^\(residential\|tertiary\|tertiary_link\)$"]["motor_vehicle"!="no"]\(area.searchArea\); nwr["highway"="unclassified"]["motor_vehicle"!="no"]\(area.searchArea\); nwr["maxspeed"~"^\(50\|40\)$"]["motor_vehicle"~"^\(yes\|designated\)$"]\(area.searchArea\); nwr["maxspeed"~"^\(50\|40\)$"][!"motor_vehicle"]\(area.searchArea\); |
| service | road, whose main purpose is access to a building, service station, beach, business park etc. | ![service road](https://upload.wikimedia.org/wikipedia/commons/e/e6/Alley_leading_from_the_High_Street_to_Green_Lane_Shopping_Centre_-_geograph.org.uk_-_942796.jpg) | highway=service + motor_vehicle!=no + (bicycle or foot = yes/designated/official, or bicycle/foot missing) | [service road query](resources/attributes/service.txt) |
| calmed | Road with very low speed limits and other pedestrian friendly traffic rules. | | highway=living_street <br/>maxspeed=30/20/15/10/7/5/5.4/4 + bicycle=yes + foot=yes + (motor_vehicle=yes/designated or motor_vehicle="" + access present and not no) | [calmed road query](resources/attributes/calmed.txt) |
| no_mit | generic or multi-purpose path, not used by motorised vehicles | ![example of a way classified no_mit](https://thumb.wikimedia.org/wikipedia/commons/thumb/b/b5/Wien_-_Graben_%281%29.JPG/500px-Wien_-_Graben_%281%29.JPG) | highway=footway/cycleway/pedestrian<br/>highway=service + motor_vehicle=no<br/>highway=bridleway + bicycle/foot=designated<br/>highway=path + surface=paved/asphalt/chipseal/concrete/paving_stones/brick/bricks/compacted<br/>highway=path + bicycle=designated/official<br/>highway=track + tracktype=grade1/grade2 + (width>=1.5 or width missing) | [no_mit query](resources/attributes/no_mit.txt) |
| track | generic or multi-purpose path, not used by motorised vehicles and with worse conditions than no_mit | ![example of a path](https://wiki.openstreetmap.org/w/images/thumb/4/49/GuideFootPathCycleYes.jpg/200px-GuideFootPathCycleYes.jpg) | highway=path + surface!=paved/asphalt/chipseal/concrete/paving_stones/brick/bricks/compacted<br/>highway=track + motor_vehicle=no/"" + tracktype set, not grade1/grade2<br/>highway=steps/platform<br/>highway=bridleway/path/track/service/residential/unclassified, except when bicycle=no and foot=no | [track query](resources/attributes/track.txt) |

## pavement

**Description:** Defines the pavement of segments.

| Value<br/>__________________ | Description<br/>_______________________________ | Example <br/>_________________________________ | OSM<br/>_________________________________ | Overpass-QL Query<br/>_________________________________ |
| :--- | :--- | :--- | :--- | :--- |
| asphalt | A feature that is predominantly paved; i.e., it is covered with paving stones, concrete or bitumen and well suited to be accessed by bike | ![example of asphalt road](https://thumb.wikimedia.org/wikipedia/commons/thumb/4/40/Asphaltbelag.jpg/500px-Asphaltbelag.jpg) | (surface=paved) <br/>surface=asphalt<br/> surface=chipseal <br/>surface=concrete <br/>surface=metal | nwr[surface~"^\(paved\|asphalt\|chipseal\|concrete\|metal\)$"] |
| gravel | A feature that is predominantly unsealed (unpaved); i.e., it has a loose covering , but still has some kind of covering like loose stones or gravel, that eases access by bike | ![example or gravel road](https://thumb.wikimedia.org/wikipedia/commons/thumb/b/b5/%C5%A0t%C4%9Brkov%C3%A1_cesta_u_Pl%C3%A1%C5%A1t%C3%ADku.jpg/500px-%C5%A0t%C4%9Brkov%C3%A1_cesta_u_Pl%C3%A1%C5%A1t%C3%ADku.jpg) | surface=paving_stones <br/>surface=bricks <br/>surface=brick <br/>surface=compacted <br/>surface=fine_gravel <br/>surface=gravel <br/>surface=pebblestone <br/>surface=unpaved <br/>surface=grass_paver | nwr[surface~"^\(paving_stones\|bricks\|brick\|compacted\|fine_gravel\|gravel\|pebblestone\|unpaved\|grass_paver\)$"] |
| soft | A feature that is predominantly unsealed (unpaved); i.e., it has a loose covering that considerably hinder access by bike, like earth or grass | ![example of road with soft pavement](https://thumb.wikimedia.org/wikipedia/commons/thumb/c/c6/Gfp-florida-big-shaols-state-park-forest-trail.jpg/500px-Gfp-florida-big-shaols-state-park-forest-trail.jpg) | surface=dirt <br/>surface=earth <br/>surface=grass <br/>surface=sand <br/>surface=wood <br/>surface=woodchips <br/>surface=ground <br/>surface=ground;grass <br/>surface=soil <br/>surface=dirt/sand | nwr[surface~"^\(dirt\|earth\|grass\|sand\|wood\|woodchips\|ground\|ground;grass\|soil\|dirt/sand\)$"] |
| cobble | cobblestone road, unevenness may be unpleasant while biking. | ![example of cobblestone road](https://thumb.wikimedia.org/wikipedia/commons/thumb/c/c2/Kohlenbrennerbr%C3%BCcke%2C_sett_paved_white_lines_01.jpg/500px-Kohlenbrennerbr%C3%BCcke%2C_sett_paved_white_lines_01.jpg) | surface=sett<br/> surface=unhewn_cobblestone <br/>surface=cobblestone | nwr[surface~"^\(sett\|unhewn_cobblestone\|cobblestone\)$"] |

## number_lanes_[tf/ft]

**Description:** Number of lanes in each direction. Only integer tags with one or two digits are used.

| Direction | OSM |
| :--- | :--- |
| ft | `lanes:forward`; otherwise `lanes` when `oneway=yes`; otherwise half of `lanes` |
| tf | `lanes:backward`; otherwise `0` when `oneway=yes`; otherwise half of `lanes` |

## max_speed_[tf/ft]

**Description:** Speed limit for motorized vehicles, in km/h. `max_speed_ft`, `max_speed_tf` and `max_speed_greatest` all receive this value, capped at 130. Rules are applied from top to bottom.

| Value | OSM |
| :--- | :--- |
| tag value | `maxspeed` is an integer with 1–3 digits |
| 50 | highway=residential/living_street/tertiary + access=designated/destination/yes/"" |
| 0 | highway=cycleway/footway/tertiary, or bicycle=yes/designated, or foot=yes/designated/official/permissive, or highway=track/path/unclassified + bicycle!=no + foot!=no + motorcar!=yes + tracktype=grade3/grade4/grade5 |
| 130 | highway=motorway |
| 100 | highway=motorway_link |
| 80 | highway=primary_link |
| 0 | access=no, or motorcar=no, or highway=steps |
| 10 | motorcar=agricultural, or highway=path, or highway=track + access!=no |
| 20 | highway=living_street, or highway=service/residential + access=private, or highway=construction + tracktype set |
| 30 | highway=service |
| 5 | highway=raceway |
| 50 | highway=unclassified, or highway=construction + ref="" |
| 70 | highway=construction + ref=B304 |
| 100 | highway=primary/secondary |

## facility_ratio

**Description:** Number of facility POIs per 100 m of segment length within 30 meters. Counted are points and polygons whose `amenity` or `tourism` value is in the facility list in `core/import_step.py`.

## greenness_ratio

**Description:** Share of green area in a 30 meter buffer, from `0` to `100`. Green polygons are `landuse` of `forest`, `grass`, `meadow`, `village_green`, `recreation_ground`, `vineyard`, `flowerbed`, `farmland`, `heath`, `nature_reserve`, `park` or `greenfield`, `leisure` of `garden`, `golf_course` or `park`, or `natural` of `tree`, `wood`, `grassland`, `heath` or `scrub`.

## water_ratio

**Description:** Water score from `0` to `100` in a 30 meter buffer. Of the polygon area share and the boundary length inside the buffer (boundary length / segment length × 100), the larger value is used. Streams (`waterway` or `natural=water`, excluding tunnels) then add their overlapping length / segment length × 100, and only when that overlap is between 80% and 160% of the segment length. The sum is capped at 100.

[water query](resources/attributes/water.txt)

## parking

**Description:** Car parking along a segment: `allowed`, `implicit`, `forbidden`. The first match is used. `allowed` is a parking key (`parking`, `parking:left`, `parking:right`, `parking:both`) of `surface`, `street_side`, `lane`, `layby`, `on_kerb`, `half_on_kerb`, `shoulder` or `yes`, or a `parking:lane` key (`parking:lane`, `parking:lane:left`, `parking:lane:right`, `parking:lane:both`) of `both`, `left`, `right`, `parallel`, `perpendicular`, `marked`, `diagonal` or `yes`. A parking area (`amenity=parking`) is `allowed` unless it is underground, multi-storey, or `level=-1`. `implicit` is `motor_vehicle` of `yes`, `designated`, `private`, `destination`, `customers` or `delivery`. Otherwise it is `highway` of `residential`, `unclassified`, `tertiary`, `secondary`, `primary`, `primary_link`, `secondary_link` or `motorway_junction`, with `access` of `designated`, `destination`, `yes`, `private`, `customers`, `delivery` or missing, and at least one parking key missing. `forbidden` requires `access` of `designated`, `destination`, `yes` or missing, and either `highway` of `cycleway`, `pedestrian`, `bridleway` or `footway`, or at least one parking key missing or `no` while `motor_vehicle` is `no`, `discouraged` or missing. A parking line is taken from a 1.75 meter buffer, keeping the line with the greatest overlap. A parking area overrides that line when the overlapping area divided by the segment length is greater than 0.33.

[parking query](resources/attributes/parking.txt)

## sights

**Description:** Number of sights per 100 m of segment length within a 30 meter buffer. Sights are points with `historic` other than `no`, or `tourism` of `attraction`, `museum`, `artwork`, `viewpoint`, `picnic_site`, `zoo`, `theme_park` or `gallery`.

[sights query](resources/attributes/sights.txt)

## crossing_ratio

**Description:** Number of `highway=crossing` points, lines or areas per 100 m of segment length within 10 meters.

## building_ratio

**Description:** Share of building area in a 20 meter buffer along the segment, from `0` to `100`. Overlapping buildings are unioned, and the result is capped at 100.
