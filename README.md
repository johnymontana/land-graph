# Land Graph 

Inspired by [https://github.com/acannistra/landwatch](https://github.com/acannistra/landwatch)

![](img/bloom1.png)

Combining datasets with graphs!

Protected areas - [USGS Protected Areas Database](https://www.usgs.gov/programs/gap-analysis-project/science/pad-us-data-download?qt-science_center_objects=0#qt-science_center_objects)

Bills, Legislators, Committees from US Congress - [Propublica Congress API](https://projects.propublica.org/api-docs/congress-api/)


## Install

This project uses Poetry for managing Python dependencies and virtual environments.

To use the notebooks:

```
poetry install
poetry shell
jupyter notebook 
```

## The Data

First, download the [USGS Protected Areas Database](https://www.usgs.gov/programs/gap-analysis-project/science/pad-us-data-download?qt-science_center_objects=0#qt-science_center_objects), reproject and convert to GeoJSON with `ogr2ogr`:

```
ogr2ogr -s_srs ESRI:102039 -t_srs EPSG:4326 -f geojson output.json PADUS3_0Fee_Region9.json
```

```
CALL apoc.load.json('file:///output.json') 
YIELD value
UNWIND value.features AS feature
WITH feature WHERE feature.geometry.type = "Polygon"
MERGE (p:Parcel {name: feature.properties.Unit_Nm})
ON CREATE SET 
  p += feature.properties
MERGE (g:Geometry {FID: p.FID})
ON CREATE SET 
  g.coordinates = [coord IN feature.geometry.coordinates[0] | point({latitude: coord[1], longitude: coord[0]})]
MERGE (p)-[:HAS_GEOMETRY]->(g)
```
