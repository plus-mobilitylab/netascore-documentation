# Adapting Mode profiles
## Mode profiles

The mode profile files are written in YAML and contain definitions of model parameters. The first part defines indicator weights, and the second part provides numeric value mappings for all indicator values. These files are contained in the `data` folder. 

To set an indicator to be ignored for index computation, its weight can be set to `NULL`.

The default mode profiles for bikeability and walkability can be found at `/examples/profile_bike.yml` and `/examples/profile_walk.yml`. Their definition looks like this:

```yaml
version: 1.1

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
        mapping:
          "{-4, -3, 3, 4}": 1.6

```

The last section of a mode profile contains the value mappings per indicator. These represent individual contributions of specific indicator values - such as a specific type of cycling infrastructure - to infrastructure suitability.

Exemplary indicator value mapping:
```yaml
indicator_mapping:
  - indicator: bicycle_infrastructure
    mapping:
      "bicycle_way": 1
      "mixed_way": 0.9
      "bicycle_lane": 0.75
      "bus_lane": 0.75
      "shared_lane": 0.5
      "undefined": 0.2
      "no": 0
```