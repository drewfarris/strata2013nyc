[sorry this one is pretty rough]

# BlinkDB

Last hours data insights – 
No notion of error – no error quantification mechanism

Each one of us creates a sample of our own. 

Samples – no lineage. Samples are created per query – ad hoc, useless for other sets of queries.

**What can it do?**

Support interactive sql queries over sets of data.

AVG, COUNT, SUM, PCE
FILTER / GROUP_BY
JOINS, nested queries, etc..

ML primitives. UDF’s

Interactivity.

100TB on 1000 machines:
1 disk per machine. Back of envelope

Disks: ½ - 1 hour (striped across all disks, network, processing) – to return an answer

Memory: 1-5 minutes, eliminate disk io, network and an processing .

How to get this down to 1 second.

**Query execution on samples.**

Geographical / Buffering Radio – indication of video quality. Higher buffering ratio mean bad performance.

What is the average buffering ration?
Process the entire table and develop and answer.
Uniform sample. Sampling rate ¼ - approximation of previous answer.
How to determine error. 
How approximate is the answer.
Bigger sample with smaller error. 

Smooth tradeoff between errors and time.

Interactive queries give you a level of error.

Noise floor.

Query Repsonse time (Seconds)
35T of data – how long to operate on complete date.

1/10 of the data with 0.02% of the answer.

Top N queries – worst or best performing attributes.

772 sec on 17TB input

Sample: Latency, 1.78s 1.7GB input (440x faster)

**BlinkDB**  

Creates a variety of random and stratified samples from underlying data
Stratified sample – bias to specific parts of data.

Fast and apprioximate answers. Supports Hive, Shark Presto. 


**Creating and Maintaining Samples**

*Uniform sample* – each row has equal probability of being in the final 
*Stratified sample* – on a given column – so there may be a uniform selection of each unique city.

```
CREATE TABLE A_sample AS SELECT * FROM A SAMPLEWIDTH 0.01 
```

Takes an argument returns a sample of the table.  1% of the table.

Something like the following in sql:

```
select a.* from a where rand() < 0.01 order by rand()
```

keep track of per block scaling information.
 - track lineage. RDD lineage.

Read a portion of the sample an still be statistically relevant.

```
CREATE TABLE A_sample AS SELECT * FROM A SAMPLEWIDTH 0.01 STRATIFY ON (col_A);
```

Make sure all the unique in col_a values are represented in the sample.

Equivalent to the following in sql:

```
Select a.*
From A join
select k, logic(count(*) as ration from A group by K
using k
where rand() < ration;
```

Keep track of per block scaling information.

--

```
select count(1) from A_sample where event = “foo”
```

Return confidence.

--

Integration with other frameworks – Hive, Shard, Facebook / PRESTO

--

Automatic sample management.

Sampling module. – automatically distributed 

Aggregardes / UDAFy

- sample – resamples continuously.
- Same query repeatedy . distribution of samples. A1 – AN gives you notion of error
- BlinkDB workload evaluator – asses percentage of your Hive/Sharq queries can be approximated correctlygives a per-query error/latency tradeoff
- Currently private beta / public release in 2 weeks.
 

Blinkdb.org

- 5-10 minutes to run local or on EC2
- Hands on exercise with AMP labs big data source

Summary 

- Approximate queries  means for interactive 

