# Land Graph 

Inspired by [https://github.com/acannistra/landwatch](https://github.com/acannistra/landwatch)

## Install

This project uses Poetry for managing Python dependencies and virtual environments.

To use the notebooks:

```
poetry install
poetry shell
jupyter notebook 
```

## The Data

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
