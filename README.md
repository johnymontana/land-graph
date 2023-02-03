# Land Graph 

Analyzing public lands and US Congress using graphs. Inspired by [https://github.com/acannistra/landwatch](https://github.com/acannistra/landwatch)

![](img/data_model.png)

One of the most powerful features of graph databases like Neo4j is the ability to combine datasets and query across them. This project combines data about US public lands into a single graph database for analysis using graph data science and graph visualization.

Currently supports the following data sources:

* Protected areas - [USGS Protected Areas Database](https://www.usgs.gov/programs/gap-analysis-project/science/pad-us-data-download?qt-science_center_objects=0#qt-science_center_objects)

* Bills, Legislators, Committees from US Congress - [Propublica Congress API](https://projects.propublica.org/api-docs/congress-api/)

* ??? 


## Install

This project uses Poetry for managing Python dependencies and virtual environments.

To use the notebooks:

```
poetry install
poetry shell
jupyter notebook 
```

## The Data Import Process

![](img/data_import.png)

First, download the [USGS Protected Areas Database](https://www.usgs.gov/programs/gap-analysis-project/science/pad-us-data-download?qt-science_center_objects=0#qt-science_center_objects). NOTE: You may need to refresh download link [here](https://www.sciencebase.gov/catalog/item/622256afd34ee0c6b38b6bb7).


```
wget -O data/region9.zip https://prod-is-usgs-sb-prod-content.s3.amazonaws.com/622256afd34ee0c6b38b6bb7/PADUS3_0_Region_9_GeoJSON.zip\?AWSAccessKeyId\=AKIAI7K4IX6D4QLARINA\&Expires\=1672066494\&Signature\=955c4BEVO7XGRLQlSGyLaWIwqqM%3D
```

Extract zip file:

```
unzip data/region9.zip -d data/region9
```

Next, reproject to WGS84 using `ogr2ogr`:

```
ogr2ogr -s_srs ESRI:102039 -t_srs EPSG:4326 -f geojson data/region9/output.json data/region9/PADUS3_0Fee_Region9.json
```

Set root of `LOAD CSV` to `data` directory in neo4j.conf

```
server.directories.import=/Users/lyonwj/github/johnymontana/land-graph/data
```

Also, increase Neo4j maximum memory heap size in `neo4j.conf` if needed:

```
server.memory.heap.max_size=1G
```

Enable file import with APOC in `conf/apoc.conf`:

```
apoc.import.file.enabled=true
```

// TODO: prepare database for import by creating constraints

```
CREATE CONSTRAINT FOR (p:Parcel) REQUIRE p.name IS UNIQUE;
CREATE CONSTRAINT FOR (g:Geometry) REQUIRE g.FID IS UNIQUE;
```

Import in Neo4j using `apoc.load.csv`:

```Cypher
CALL apoc.load.json('file:///region9/output.json') 
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
SET g:Polygon
```

Next, to load US Congressional data using the ProPublica API see [`notebooks/import.ipynb`](notebooks/import.ipynb)

## Visualizing With Neo4j Bloom

![](img/bloom1.png)
