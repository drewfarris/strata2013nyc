# Accumulo Strata NYC Community Meeting Notes

Monday, October 28, 2013 from 1:00 - 6:00 PM

## Outline

- HAWK/PXF demo from Pivotal  (Adam Shook)
- Jeremy Kepner (MIT) D4M Schema and Performance Testing
- Cloudera Talk on Accumulo Support via Cloudera Manager
- Demo from of Sqrrl Enterprise
- "Unconference Breakout Sessions"
   -	Compatibility with Hadoop 2
   -	Testing
   -	Performance improvements
   -	Features
   -	Fertilizing Accumulo

## D4M schema for Accumulo, Jeremy Kepner, MIT Lincoln Labs

  - D4M: [http://www.mit.edu/~kepner/D4M/]
  -	Jeremy presented the slides [http://www.mit.edu/~kepner/pubs/Kepner_2012_D4M_Slides.pdf]


## Breakout Discussion

### Security
   the problem of maintain usernames/passwords on the clients? How do people deal with this? The general consensus in Kerberos and if you don’t have a kerberized cluster you need to craft something by hand?

### Benchmarks

  -	Discussions of the Accumulo Benchmarks published as a part of the IEEE Big Data proceedings: http://bit.ly/1g7xJRD
-	Accumulo would benefit more from a head to head comparison – is YCSB++ good enough for this or does it omit tests that show where Accumulo shines?
-	From the Benchmark Discussions, how does it differ from HBASE other than “Security Labels” and “Iterators”? Ingest performance identified as one area.

### API / Types

  -	Is the low level API too level and do we need to provide something that’s higher level
  -	Sean and Ed discussed up the problems of  mapping high level data structures /schema to underlying Key / Value mechanism
-	Drew asked whether anyone from the Accumulo Community has spent looking at http://www.kiji.org
-	Naresh mentioned that an upcoming HadoopDC meetup will feature folks from Kiji
- https://issues.apache.org/jira/browse/ACCUMULO-1551

### Operationalization

  -	Sean mentioned the operations nightmare of managing iterator installations, also the problem of resource utilization of iterators – how does one prevent a single iterator from consuming all available resources.

## Jessica Seastrom, Solutions Architect, Cloudera

-	Cloudera Manager Accumulo Integration
-	Cloudera Parcels, what are they?
-	Alternative Binary Distribution Format in Cloudera Manager 4.5 – gzip’d tarfiles with metadata.
-	Installed side-by-side with other versions – activate the ones you want to be using. Facilitates updates / rollbacks.
-	Multiple install locations.
-	Demo of installing Accumulo via parcels in Cloudera Manager.

## Adam Fuchs, SQRRL.

-	SQRRL sits between the application level and Accumulo.
-	Easier to use Accumulo and expose features that are more digestable
-	Security / Policy centric features.
-	Common Design Patterns – encapsulated into a higher level API.
-	Documents and Graphs as core data models
-	Aggregation and Search of as operations on this data model.
-	Demos: Schema Exploration Tool for Wikipedia (Unstructured) and Twitter (Structured) data.
-	Lucene Query API + authorizations, downgrading presents the same results but with less metadata per result. JSON extended to add authorization annotations.
-	Edge based visualization of query results.
-	Link Search – first 1000 edges from Edgar dataset, visualized.  Each node represents an entity (owner, company,
-	Entity extraction and link extraction  to indicate co-occurrence. Neighborhood expansion with complete interaction.
-	Combination of Graph and Term search.
-	Analytics demo – SQL demonstrated to perform rollups, etc.
-	Summary: Denormalized Views, Document/Graph Search, Basic Aggregation and Big Picture analysis.


Enterprise Features, Disaster Recovery, need to back-end these with features  from the Open Source stack.

HBase cross-site replication feature ported into Accumulo. It would be useful to have additional support of this from the community.


Creating the right interfaces to map into graph processing frameworks and other
features.

## Billie Rinaldi – HOYA.

 - Steve Loughran –  most of the work on HBase but sufficiently pluggable to support Accumulo and other things in the future.
 - Use Cases: Small Clusters in a large YARN cluster, dynamic, self-healing, elastic, transient clusters, custom versions, more efficient utilization/sharing.
 - Hoya Example: Zookeeper, HDFS, YARN, Accumulo Tarball & Config, Commands: create, flex, freexe, thaw, list, destroy.
- Survivability across Resource Manager Restarts?
- Requirements: Hadoop 2.0 for YARN, Accumulo 1.6 (But Accumulo 1.5 might work).
- Masters don’t have to run on YARN Application Master.
- Documentation here: https://github.com/hortonworks/hoya/tree/develop/src/site/markdown
- https://issues.apache.org/jira/browse/YARN-896 - Avoid multiple tservers from starting on the same

## Breakout Discussion

### Hadoop 2.0
. Compatability with Hadoop 2: Make it work with YARN
. How do we test HDFS failover stuff with Hadoop 2 and Accumulo?
. 1.6 Has all of the warnings – tickets to say what you can do in 1.5
. HCatalog integration with Accumulo 1.5 + Hadoop 2 (See Ed’s GitHub) – change some of the poms to get it to work. Still a problem to map key/value structures
.  Need review of Sqoop integration - ACCUMULO-141
. HUE Integration -> Shell vs. Column Based GUI.
.  Thrift Abstraction how to deal with this in ACCUMULO? OSGI

### Testing
-	Testing is terrible (Hard to run the tests, Documentation around the Tests, Difficult to understand. A description of what the test is meant to test).
-	Testing is specific to systems it previously run on, having the ability to flex the number of time
-	Unit Testing
-	Continuous Tests
-	Random Walk Testing
-	More Robust Agitatior as per Jepson
-	Patches must include Unit Tests.
-	In 1.6 the Python tests have moved to Java
-	Intermittent Test Failures. “A test that fails doesn’t mean something is broken”
-	We just run unit tests on Apache infrastructure – Jenkins.
-	Pre-commit functional tests passing (Jenkins or Apache infrastructure)
-	Pre vs. Post commit testing?

### Barriers to Adoption
-	Lack of documentation as far as using Accumulo with other programming languages (pycummulo).
-	Simplify the interface. MongoDB gets this right.
-	How to make it easier to get started.
-	Thrift Proxy Disussion

### Performance Improvements:

What are the key performance indicators for Accumulo (E.g: Operations per second.)

Performance of Accumulo can be mystifying. Not clear how different
knobs are related to each other and what the tolerances are. Very
often it wasn’t clear how to tweak things appropriately without
leading performance off a cliff.

What to adjust?

  -	Tablet Size,
  - Number of Splits before Compaction
  - Etc.

  This can result in orders of magnitude performance.

What to measure? Time it took to run various use case:

  - Streaming Ingest
  - Bulk Input
  - High Volume Target Seeks
  - High Volume Scans.

Can we implement a tool to optimize or test various Accumulo
configurations for specific use cases in order to arrive at an ideal
configuration?

When scans of Accumulo data take 100x more than scans of the same
directory on HDFS how do we determine what the problem is?

  - Has anyone used the Starfish Hadoop profiler from Duke?
  - What can be done to improve data locality?
  - What is the current state of the art for Data locality?
  - What can be done to support short-circuit reads? If so, how to determine whether it’s on?
  -	Can we trace-enable measurement of data locality?

-	Do we use Grand challenges, e.g: can we achieve maximum throughput from disk? E.g: If I want to use an Iterator to filter out rows from a table, can I use the full bandwidth of drives or how far off am I?
-	Focus on RFile reading and writing speed
-	Alternate Filesystems? HDFS vs. QFS or GlusterFS in order to improve performance?
