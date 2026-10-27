# Adapting Mode profiles

## Mode profiles

The mode profile files are written in YAML and contain definitions of model parameters. The first part defines indicator weights, and the second part provides numeric value mappings for all indicator values. These files are contained in the `data` folder.

To set an indicator to be ignored for index computation, its weight can be set to `NULL`. Indicators without a weight in the profile are ignored as well.

Mode profiles are validated when the settings are loaded, before any processing step runs. Unknown keys (e.g. typos in indicator names) are reported as warnings.

The default mode profiles for bikeability and walkability can be found at `/examples/profile_bike.yml` and `/examples/profile_walk.yml`. An additional example profile for recreational cycling is `/examples/profile_bike_tourism.yml`. Profile files require `version: 2.0`. Older profile files are rejected. Profile definition looks like this (first the indicator weights):

```yaml
version: 2.0

weights:
  bicycle_infrastructure: 0.2
  pedestrian_infrastructure: NULL
  designated_route: 0.1
  road_category: 0.3
  max_speed: 0.1
  max_speed_greatest: NULL
  parking: 0.1
  pavement: 0.1
  width: NULL
  gradient: 0.1
  number_lanes: NULL
  facilities: NULL
  crossings: NULL
  buildings: NULL
  greenness: NULL
  water: NULL
  sights: NULL
  noise: NULL
```

In addition to indicator weights that are used to compute the index as weighted average, value overrides can be specified. Here, either the index can be set to a fixed value or indicator weights can be replaced. This allows to handle specific situations such as steep gradient in combination with gravel surface (as shown in the following example):

```yaml
overrides:
  - description: combination of gradient and pavement (steep and loose/rough)
    indicator: pavement
    output:
      type: weight
      for: [pavement, gradient]
    mapping:
      "{gravel, soft, cobble}":
        indicator: gradient
        classes:  # steep: at least 6 % uphill or downhill
          ge6: 1.6
          le-6: 1.6

```

The last section of a mode profile contains the value mappings per indicator. These represent individual contributions of specific indicator values - such as a specific type of cycling infrastructure - to infrastructure suitability.

Exemplary indicator value mapping:

```yaml
indicator_mapping:
  - indicator: bicycle_infrastructure
    mapping:
      "bicycle_way": 1
      "mixed_way": 0.9
      "mixed_way_non_designated": 0.9
      "bicycle_lane": 0.75
      "bus_lane": 0.75
      "bicycle_road": 0.6
      "shared_lane": 0.5
      "no": 0
      "no_separate": 0
```

Value assignments are given either as `mapping` or as `classes`:

- `mapping` assigns values to exact indicator values, e.g. `"bicycle_way": 1`. A list of values can be given in curly brackets, e.g. `"{gravel, soft, cobble}": 0.5`.
- `classes` assigns values to value ranges of numeric indicators. Each key consists of an operator and a number: `g` (greater than), `ge` (greater than or equal), `l` (less than), `le` (less than or equal), `e` (equal) and `ne` (not equal). Classes are evaluated in the given order and the first match applies.

Additionally, `_default_` assigns a value to all values not listed, and `NULL` assigns a value to missing indicator values. Instead of a value, a nested assignment for another indicator can be given (as in the override example above).

Since version 2.0, the `gradient` is given in percent (positive: uphill). Its value assignments need to use `classes` - profiles using `mapping` for the gradient are rejected. The default profiles reproduce the classes of version 1.x:

```yaml
  - indicator: gradient  # gradient in % (positive: uphill); classes are evaluated in the given order
    classes:
      ge12:  0     # >= 12 %
      ge6:   0.25  # 6 to 12 %
      ge3:   0.4   # 3 to 6 %
      ge1.5: 0.5   # 1.5 to 3 %
      g-1.5: 0.9   # -1.5 to 1.5 % (flat)
      g-3:   1     # -3 to -1.5 %
      g-6:   0.95  # -6 to -3 %
      g-12:  0.35  # -12 to -6 %
      le-12: 0     # <= -12 %
```

```{note}
**Updating mode profiles from version 1.x:** set `version: 2.0`, replace the gradient `mapping` by `classes` (see above), add value assignments for `parking` (`allowed`, `implicit`, `forbidden`) and for the bicycle infrastructure class `bicycle_road`. Class limits for `facilities`, `crossings` and `sights` refer to counts per hectare since version 2.0 (previously per 100 m of edge length). `water` is a value from 0 to 100 (previously `true` / `false`). The default walk profile no longer overrides the index of sidewalks along primary and secondary roads.
```
