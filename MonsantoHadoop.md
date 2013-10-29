Monsanto and Hadoop
===============

erich.hochmuth@mostanto.com
eric.d.turcotte@monsanto.com

Goal is to Doubling Yield via :

- Breeding (Mendelian Genetics)
- Biotechnology (Gene Splicing outside normal breeding process)
- Agronomics (Systems Management applied to the firm, soil enrichment, water management)

Precision Agricultire

- Information
- Automation
- Weather Stations
- Field Sensors
- Collect as much information as we can 

Integration Farmings Systems - FieldScripts for 2014

Recommendation system for growers. For your field, what product to buy and how to plant it.

Variable rate hybrid planting prediction.
Planting density based on field conditions.
Fields are typically not uniform
Soil types have yield potential + also slop and land layout.

Today folks don't take advantage of field dynamics in order to identify planting prescripton or fertilization in order to maximize yield, but FieldScripts will do this.

- What to plant
- When to plant
- How to plant
- What the apply to crop
- When top harvest

To maximize yield potential.

Several years of historical yield data collected from 
Combined growers data with public and privately owned data

make assessment and provide instructions to farmer how to 

Platform Needs
 
- Load thousands of files containing spatial data 10s TB
- Spatial data in tabular, xml, vector, raster (1 inch resolution?) 
- Join and Link data spatialy)
- Make data available for data products such as field scripts
- Make data avialable for data scientists to develop

Spatial Data Types

- Raster / Image (Limited to the amount of data you can store)
- Vector (Points, polygons lines, standards for the association of metadata)
- Real World

Querying Vector data is difficult, so they standardize upon raster data for processing in hadoop.
(Lots of good bullets on this slide)

Data in Detail

- Enviornmental Observations (Elevation, Soil, Grower data
- 10m x 10m grids - recommendation for each grid unit.
- Taken the entire US and broken it into 10m x 10m grid.
- Extremely Dense Structured Data
- Several million raw data points, 1 data point every 18 minute, 5 Hz 5 samples per second and then needs to be interpolated into the 10m x 10m grid.
- Complex interactions - how is water retained in soil, determined by slope, look at how it is influenced by its neighbors. (Graph problem!)

Spatial UDFs.

##### Take 1: RDBMS Data Ingest and Data Processing

NFS => Load into RDBMD => Re-project => Generate Spatial Indexes => Generate Derived Attributes

- In RDBMS spatial
- PL/SQL
- 8% of the data took 35+ days to load
- TBs in indexes
- Multiple PAtches to DB needed
- Tradeoffs
  - Compressed vs. uncompressed
  - Performance vs. Storate
  - Read vs. Write

Huge indexes required to support performant queries, very expensive to execute these queries. Had to patch the database system. Large index sizes lead to compression to reduce storage requirements but killed performance.

- Options vs. Recommendations:
- Reduce functionality
- Move more to application layer
   
Needs:

- Scalable System
- Comples Geospatial Data Types
- Push analytics to data
- Commodity / cost effective storage
- Vendor support. 

##### Take 2: Ingestion Revisited

NFS => Hadoop Straming => HDFS

- 1000s of files into hdfs
- Standardize data
- Raster format is easily splittable
- Hadoop Streaming with GDAL
    - Counters used extensively
    - Due to libraries under the cover it's difficult to observe what's going on
    - Some of the tools don't work with HDFS data natively, so need to extract data from HDFS, write it locally and then put it back.
    
Creating Derived Attributes

- Process Raster data (Dense Matrix)
- Generic InputFormat & RecordReader for raster stat
- HFiles easilly transportable between clusters (Bulk Import)
- Challenges Tuninig Jobs
   - I/O Sort Factor
   - Split/Task Size
   
30 days of processing time reduced to less than a day.

## Geospatial in HBase

Needs
- Spatially enabled HBase Key (Target and Bulk random reads)
- Reduce/eliminate the name for index tables
- Scalable & Cost Efficient
- Support Targeted and Bulk Random Reader
- Optimize IO for reads.

Considerations
- Key overhead
- Use Gets vs. Scans
- Don't want to reat too much data
- REad data within a single field.

Possbie Solutions
- GeoHash most notable but best suited for sparse data. 
  - Split the search space and encode, e.g: 0 for northern, 1 for southern hemisphere, divide each in half and continue. Weave latitude and longitude.
  - MGRS with Alphanumeric keys? Edge cases around poles.
  - Quad trees & R-Trees.
 
Take 1: Global Coordinate System
 - Created a reference system, split globe into large areas. Implemented with a mathmetic formula instead of lookup. Simple and performant algorithm. Each area has a distinct index value. 
 - Nested set of indexed values associated. 
 
 Cell ID - based on large area on globe and next level down. 1 column family, 1 cq hold the attributes.
 
 Different data sets stored in a single table.
 
 One row for each grid, 120B rows. 1000's of gets per row. TBs of Key overhead.
 
Take 2: Super cell is 10,000 cells (1k square) -- at most 4 super cells must be read at a time. Best case is that the entire field is in a single suprtcel.

- 120 b unique polygons in total
- 1.5 trillion data points
- dense grid of the entire US.
- foundational architecture for other data sets
- fully unit tested implmeentation

Entire U.S in hadoop loaded in 18 hour load time, 3 months of development, 100% scalable.

#### Future Considerations

- HBase Filters (push functionality to servers, spatial intersection queries on the platform)
- Pre-computed aggregates at different resolutions (lower resolutons, larger areas)
- Distributed Vector Data Store with Solr/Lucene via CLoudera Search interesting spatial features here.
- Spatial UDFs (Pig, Hive and Impala)
- Metadata Repository.

#### Questions and Answers

How to determine how MongoDB vs. HBase? 

- push down analytics to data (they use Mahout, R)
- easier to reason about the way data is stored in HBase
- scalability 
- shard databases
- good vendor support

15 Node HBase Cluster -- 40 Nodes in production

[Didn't catch the rest -- the speakers won't repeating questions]









