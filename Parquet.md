Parquet
==============

Compare this with ORCFiles from Hortonworks[3]

[3]:

State of the asrt columnar storage for Hadoop

- Limits IO to data actually needed, loading only the columns that need to be accessed
- Saves space (columnar compresses better, type specific encodings).
- Vectorized execution engine.

Homogenous values in type and content, efficient encoding of integers for example (zig zag, diff) - 
Encoding of categorical types, etc.

Cheap to vectorize operations.

Open source collaboration between Twitter an Cloudera

Common file format definition: Language independent, Formally specified.

**Benchmark Results in Impala Here**

TPC-H[1] Benchmark lineitem table @ 1TB scale factor

Size comparisons

Space savings of heavyweight compression but not the CPU overhead.

[1]: 


TPC-DS[2] Benchmark.

Comparion of:

- Text
- Seq/Snappy
- RC w/ Snappy
- Parquet w/ Snappy.

[2]:

### Criteo: The Context

- Billiona of new events
- ~ Columns per log
- Heavy workload
- BI Analysts using Hive and RCFile


- MapRed compat w/ Hive
- Evolving Schemas
- Pushdown
- Interoperability with other engines: Pig, Impala, etc.

Hive 0.11: Parquet vs ORC

TPC-DS

orc-snappy 35%
parquet-snappy 33% 
~ 1/3 of original size.

~50 Mappers
2 x 6 cores, 96 GB ram 14 x 3TB disk

## Twitter Production Results

Access logs, 30 columns
Thrift binary in block compressed files (LZO)
New Format: Parquet (LZO)

Little overhead when using all rows in order to assemble columns.

## Production Savings @ Twitter

Petabytes of storage saved
Example jobs taking advantege of projection pushdown

- Job 1 (Pig): reading 32% less data, 20% of task time spent 
- Job 2 (Scalding) 14/35 columns read, reading 80% less data, 66% task time saving.
- Terabytes of scanning saved everyday, re-allocation of nodes.


## Format

- Row Group: A group of rows in columnar format, contains column chunks, each chunk is made of pages

Buffer in memory the entire row-group.

Max size buggered in memory while writing
One or more per split while reading
roughly 50M < rowgroup < 1G

- Column chunk

Page is a unit of access and unit of compression -- 8K < page < 1M

### Nested Record Sharing

algorithm borrowed from google dremel's column IO
repitition level, definition level, value
level values are bound by the depth of the chema: stored in a compact form.

Bit Packing & Run Length Encoding [4]

[4]: https://blog.twitter.com/2013/dremel-made-simple-with-parquet

Differences in Parquet vs. ORC

Parquet: repition definition levels capture the structure - one column per leaf

ORC: Extra column for each Map or List to record size
one column per node in the schema
Array<int> is two columns size & content.

### Reading Records

Record level API to integrate with exsiting row based engines (Hive, Pig, MR).
Aware of dictionary encoding: enables optimizations
Assembles projection for any subset of the columns.

### PRojection Push Down

- Automated in Pig and Hive - load only the fields needed 
- API Scalding / MR - they added the ability to specify the fileds
- Lives on top of the Column level API. 

### Integration APIS

Schema definition and record materialization
- Impala, Pig, Hive, Thrift, Avro, ProtocolBuffers?
- Translate from one schem
a model to another in order to move to Parqet
 - Event based SAX style materialization layer, no double conversion.


### Encoding

- Bit Packing: Small integers encoded in the min bits reuired
- Custom RLE to incode long runs of the same characters
- Dictionaey Encododing
- Extensible // Encoding can vary across pages baaed on data.
 
Delta Encoding / Specified Dictionaries / Boolean Values


Goal: use across the entire hadoop stack.


