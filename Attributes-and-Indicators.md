# Attributes and Indicators

This page details on the indicators NetAScore currently allows to derive from input data. 

As data astructure and available attributes may vary between input data sets, indicator generation is implemented specifically per type of input network.
Currently, NetAScore implements this for OpenStreetMap and GIP (Austrian authoritative data). 
To learn more about the details how attributes are transformed into indicators, please have a look at [Attribute derivation from OSM](Attribute-derivation-from-OSM) and [GIP](Attribute-derivation-from-GIP).

The indicators are later combined to form a single numeric bikeability or walkability index per road segment.
In general, NetAScores combines indicator values by applying a weighted average. However, custom exceptions, so-called "value overrides" can be defined. 
In the default mode profile for bikeability an override is defined for emphasizing the negative impact of steep gradient (slope) with gravel surface.

Indicators marked with an asterisk `*` are differentiated by direction `[tf/ft]`.


## Access attributes

### access_car\_* / access_bicycle\_* / access_pedestrian\_*

Indicates the accessibility of a road segment for cars, bicycles or pedestrians. This refers to the legal acces. Values: `true`, `false`

## Additional attributes

### bridge

Indicates a road segment which belongs to a bridge: `true`, `false`

### tunnel

Indicates a road segment which belongs to a tunnel: `true`, `false`

## Indicators

### bicycle_infrastructure_*

Describes the presence of dedicated bicycle infrastructure along a road segment: `bicycle_way`, `mixed_way`, `bicycle_road`, `cyclestreet`, `bicycle_lane`, `bus_lane`, `no`.

Default data source: OpenStreetMap network geometry with attributes 

Default weight: 0.2 (bike) / - (walk)


### pedestrian_infrastructure_*

Describes the presence of dedicated pedestrian infrastructure: `pedestrian_area`, `pedestrian_way`, `mixed_way`, `stairs`, `sidewalk`, `no`.

Default data source: OpenStreetMap network geometry with attributes 

Default weight: - (bike) / 0.4 (walk)


### designated_route_*

Indicates, whether the segment is part of a designated cycling route. It is categorized by scope (highest has precedence): `local`, `regional`, `national`, `international`, `unknown`, `no`.

Default data source: OpenStreetMap cycle networks 

Default weight: 0.1 (bike) / - (walk)


### road_category

Describes the road category of a road segment: `primary`, `secondary`, `residential`, `service`, `calmed`, `no_mit`, `track`.

Default data source: OpenStreetMap network geometry with attributes 

Default weight: 0.3 (bike) / 0.3 (walk)


### pavement

Describes the condition of the road surface: `asphalt`, `gravel`, `cobble`, `soft`.

Default data source: OpenStreetMap network geometry with attributes 

Default weight: 0.1* (bike) / 0.3 (walk)

*Note: weight is increased in case of gravel combined with steep gradient


### max_speed\_* / max_speed_greatest

`max_speed_*` describes the speed limit (car) in the direction of travel: `0` to `130`.
`max_speed_greatest_*` uses the maximum value of speed limits for both directions along the segment.

Default data source: OpenStreetMap network geometry with attributes 

Default weight (directional): 0.1 (bike)


### number_lanes_*

Represents the number of lanes along the road segment.

Default data source: OpenStreetMap network geometry with attributes 

Default weight: - (bike) / 0.1 (walk)


### width

Represents the width of the road or path in meters.

Default data source: OpenStreetMap network geometry with attributes 

Default weight: -


### buildings

Represents the proportion of the area covered by buildings within a 30 meters buffer along the road segment: `0` to `100`.

Default data source: OpenStreetMap (building footprints)

Default weight: - (bike) / 0.1 (walk)


### crossings

Describes the number of pedestrian crossings per 100m segment length within a 10 meters buffer along the road segment.

Default data source: OpenStreetMap

Default weight: - (bike) / 0.2 (walk)


### facilities

Describes the number of facilities (POIs) per 100m segment length within a 30 meters buffer along the road segment.

Default data source: OpenStreetMap

Default weight: - (bike) / 0.3 (walk)


### greenness

Describes the proportion of the green area within a 30 meters buffer: `0` to `100`.
Default data source: OpenStreetMap

Default weight: - (bike) / 0.3 (walk)


### water

Describes the presence of water bodies within a 30 meters buffer: `true`, `false`.

Default data source: OpenStreetMap

Default weight: - (bike) / 0.4 (walk)

### parking_* (placeholder)

Describes designated streetside car parking: `yes`, `no`. Currently, this indicator is not computed due to data availability. However, if included in the mode profile, lack of indicator data will be documented accordingly in the `index_<mode>_*_robustness`-column.

Note: Implementation is planned for the future, given data availability.

Default weight: 0.1 (bike) / - (walk)
## Optional Indicators for _bikeability_ and _walkability_

Computation of these indicators may require provision of additional data sets (unless the network data set carries respective attributes - e.g. elevation data is included in the GIP network for Austria).

### gradient_*

Describes the gradient class of a road segment for downhill and uphill: `-4` to `4`.

| Class | Definition  |
|-------|-------------|
| 0     | 0 - 1,5 %   |
| 1     | > 1,5 - 3 % |
| 2     | > 3 - 6 %   |
| 3     | > 6 - 12 %  |
| 4     | > 12 %      |

The influence of gradient classes on the final index can be assigned per mode using the section `indicator_mapping` within mode profile files.

Default data source: OpenStreetMap network geometry and digital elevation model (DEM) 

Default weight:
- 0.1* (bike) / 0.3 (walk)

*Note: weight is increased in case of gravel combined with steep gradient


### noise

Describes the noise level of a road segment in decibel.

Default data source: Traffic noise polygons (open government data e.g. in Austria) 

Default weight: - (bike) / 0.3 (walk)



