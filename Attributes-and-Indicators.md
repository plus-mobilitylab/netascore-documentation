# Attributes and Indicators

This page details on the indicators NetAScore currently allows to derive from input data.

As data structure and available attributes may vary between input data sets, indicator generation is implemented specifically per type of input network.
Currently, NetAScore implements this step for OpenStreetMap (OSM) and GIP (Austrian authoritative data).
To learn more about the details how attributes are transformed into indicators, please have a look at [Attribute derivation from OSM](Attribute-derivation-from-OSM) and [GIP](Attribute-derivation-from-GIP).

The indicators are later combined to form a single numeric bikeability or walkability index per road segment.
In general, NetAScores combines indicator values into the final index by applying a weighted average. However, custom exceptions, so-called "value overrides" can be defined.
In the default mode profile for bikeability an override is defined for emphasizing the negative impact of steep gradient (slope) with gravel surface.

Indicators marked with an asterisk `*` are differentiated by direction `[tf/ft]`.

Indicators derived from additional datasets (e.g. buildings, noise) are only computed for edges within the area covered by the dataset. Edges outside the coverage get no value (NULL), which lowers the robustness of their index.

## Access attributes

### access_car\_* / access_bicycle\_* / access_pedestrian\_*

Indicates the accessibility of a road segment for cars, bicycles or pedestrians. This refers to the legal access. Values: `true`, `false`

Stairs (`highway=steps` in OSM) are interpreted as never being accessible by car or bicycle, even with explicit permissions set (in many cases such constellations result from tagging mistakes).

## Additional attributes

### bridge

Indicates a road segment which belongs to a bridge: `true`, `false`

### tunnel

Indicates a road segment which belongs to a tunnel: `true`, `false`

## Indicators

### bicycle_infrastructure_*

Describes the presence of dedicated bicycle infrastructure along a road segment: `bicycle_way`, `mixed_way`, `mixed_way_non_designated`, `bicycle_lane`, `bus_lane`, `bicycle_road`, `shared_lane`, `no`, `no_separate`.

`bicycle_road` refers to bicycle roads (e.g. *Fahrradstraße*) that are open to motor traffic. Bicycle roads without car access are classified as `bicycle_way`, `mixed_way` or `mixed_way_non_designated`, depending on pedestrian use.

Default data source: OpenStreetMap network geometry with attributes

Default weight: 0.2 (bike) / - (walk)

### pedestrian_infrastructure_*

Describes the presence of dedicated pedestrian infrastructure: `pedestrian_area`, `pedestrian_way`, `mixed_way`, `mixed_way_non_designated`, `stairs`, `sidewalk`, `no`, `no_separate`.

Default data source: OpenStreetMap network geometry with attributes

Default weight: - (bike) / 0.4 (walk)

### designated_route_*

Indicates, whether the segment is part of a designated cycling route. It is categorized by scope (highest has precedence): `local`, `regional`, `national`, `international`, `unknown`, `no`. Routes that only apply to one direction of a way are only regarded for this direction.

Default data source: OpenStreetMap bicycle route relations

Default weight: 0.1 (bike) / - (walk)

### road_category

Describes the road category of a road segment: `primary`, `secondary`, `residential`, `service`, `calmed`, `no_mit`, `track`.

Bicycle roads open to motor traffic are categorized as `calmed`, bicycle roads without car access as `no_mit` (no motorized traffic).

Default data source: OpenStreetMap network geometry with attributes

Default weight: 0.3 (bike) / 0.3 (walk)

### pavement

Describes the condition of the road surface: `asphalt`, `gravel`, `cobble`, `soft`.

Default data source: OpenStreetMap network geometry with attributes

Default weight: 0.1* (bike) / - (walk)

*Note: weight is increased in case of gravel, soft or cobble combined with steep gradient

### max_speed\_* / max_speed_greatest

`max_speed_*` describes the speed limit (car) in the direction of travel: `0` to `130`.
`max_speed_greatest_*` uses the maximum value of speed limits for both directions along the segment.

Default data source: OpenStreetMap network geometry with attributes

Default weight: 0.1 (bike, `max_speed`) / 0.3 (walk, `max_speed_greatest`)

### number_lanes_*

Represents the number of lanes along the road segment.

Default data source: OpenStreetMap network geometry with attributes

Default weight: - (bike) / 0.1 (walk)

### width

Represents the width of the road or path in meters.

Default data source: OpenStreetMap network geometry with attributes

Default weight: -

### buildings

Represents the proportion of the area covered by buildings within a 20 meters buffer along the road segment: `0` to `100`.

Default data source: OpenStreetMap (building footprints)

Default weight: - (bike) / 0.1 (walk)

### crossings

Describes the number of pedestrian crossings per hectare of the area within a 10 meters buffer around the road segment.

Default data source: OpenStreetMap

Default weight: - (bike) / 0.2 (walk)

### facilities

Describes the number of facilities (POIs) per hectare of the area within a 30 meters buffer around the road segment.

Default data source: OpenStreetMap

Default weight: - (bike) / 0.3 (walk)

### greenness

Describes the proportion of the green area within a 30 meters buffer: `0` to `100`.
Default data source: OpenStreetMap

Default weight: - (bike) / 0.3 (walk)

### water

Describes the proportion of water within a 30 meters buffer, including streams running along a segment: `0` to `100`.

Default data source: OpenStreetMap

Default weight: - (bike) / 0.4 (walk)

### parking

Describes car parking along a road segment: `allowed`, `implicit`, `forbidden`. The value `allowed` is set for street parking mapped on at least one side, or a parking area that is not underground, multi-storey, or `level=-1`. `forbidden` is used for parking mapped as prohibited on both sides, and all ways without (unrestricted) motor traffic or where parking is not possible along the road (e.g. motorways). `implicit` is a road open to motor traffic without information on parking. Tag lists are provided at [Attribute derivation from OSM](Attribute-derivation-from-OSM).

Default data source: OpenStreetMap

Default weight: 0.1 (bike) / - (walk)

```{note}
In version 1.x, parking was a placeholder with impact on index robustness. Custom mode profiles from 1.x need to map the values `allowed`, `implicit` and `forbidden`. See the current example files of V 2.0 for reference.
```

### sights

Describes the number of sights per hectare of the area within a 30 meters buffer around the road segment.

Default data source: OpenStreetMap

Default weight: - (bike) / - (walk). The exemplary mode profile for bicycle tourism (`profile_bike_tourism.yml`) uses a weight of 0.4.

## Optional Indicators for *bikeability* and *walkability*

Computation of these indicators may require provision of additional data sets (unless the network data set carries respective attributes - e.g. elevation data is included in the GIP network for Austria).

### gradient_*

Describes the gradient of a road segment in percent, in the direction of travel (positive: uphill, negative: downhill). Bridges and tunnels are considered to have no gradient.

The influence of the gradient on the final index is assigned per mode with value ranges (`classes`) in the section `indicator_mapping` of the mode profile files. The default mode profiles use the following classes (same result as in version 1.x, where the gradient was given as classes from `-4` to `4`):

| Class (of V 1.x) | Gradient |
| --- | --- |
| 0 | > -1.5 % and < 1.5 % |
| 1 / -1 | >= 1.5 % and < 3 % / <= -1.5 % and > -3 % |
| 2 / -2 | >= 3 % and < 6 % / <= -3 % and > -6 % |
| 3 / -3 | >= 6 % and < 12 % / <= -6 % and > -12 % |
| 4 / -4 | >= 12 % / <= -12 % |

For OSM, node elevations are interpolated bilinearly from the DEM. This reduces unrealistic gradient values for short edges.

Default data source: OpenStreetMap network geometry and digital elevation model (DEM)

Default weight:

- 0.1* (bike) / 0.3 (walk)

*Note: weight is increased in case of gravel, soft or cobble combined with steep gradient

### noise

Describes the noise level of a road segment in decibel. Edges within the coverage of the noise data that do not intersect any noise polygon get the value `0`. Non-numeric noise values (e.g. classes) can be mapped to numeric values using the settings key (`value_mapping`).

Default data source: Traffic noise polygons (open government data e.g. in Austria)

Default weight: - (bike) / 0.3 (walk)
