Riak @ Gilt
====

Testing Riak for Multiple Data-Center Support: A Case Study

- Jim Englert
- Gilt Groupe

What

- One Week Hackation
- Gilt's Offices in Dublin
- Evaluate Riak as a replacement for MongoDB
- From Basho: Seth Thomas and Steve Vinoski
- 3-4 Engineers from Gilt

## What is Gilt?

Flash Sales - show duration, limited inventory (Products sell out within minutes)

  - Lots of personalization
  - International Company (Summer time in Australia vs. NYC) 
  - e.g: Understand location of customer and place correct items for sale.
  - e.g: Product category recommendation
  
Lots of traffic at noon - this is when the flash sales start.
Micro-service architecture, with hundreds of service
Service growth over time is a hockey stick with the invlection point at Jan-12 - APIs around services, but hard to combine data. 

Why Microservices?

- Makes it easy to develop new features
- Iterate quickly
- Separates Development Groups

Currently based entirely on Java VM technologies

Hard to scale RoR beyond 10 people (concurrent users?)

Realtime Personalization
 - look alike models
 - collaborative filtering
 - correlation between size and 
 
Complex Event Processing

Needs to be realtime because they don't have the same inventory every day. Gaining information overnight is not really that useful. Most users arrive at noon and stay 15 minutes, can't play with different techniques throughout the day.

CeP - which sales are doing well, which countries work really well - localization

e.g: Tumi in China spike? Present Tumi to Chinese customers.
e.g: Shopping cart analysis.

## K/V Store Needs

- Multi  datacenter
- Active / Active
- But some functionality only needs to live in one datacenter.
- e.g: Personalized E-mail (keep this off of website )
- Move this function to either EC2 or different servers.
- Requests should not cross datacenters (the links go down and they're never as fast as you'd like them to be).


## Why RIAK

- No Active-Active on MongoDB (Active/Slave)
- High Performance (rumored, wanted to test this out in Hackathon)
- People! (The people that represent the company)
  - Difference open source projects have different communities. 
  
### What is Riak?

  KV Store
  Based on Dynamo
  Supports Secondary Indexes

Amazon Dynamo consistent hashing to write data to same node every time but also write to some subset of all nodes. Eventually consistent. 

Key Value stores don't typically have secondary indexes. Having this capability is pretty useful. Use this feature in Mongo a great deal.

## How to Test It?

Port a single Service and see how it goes.
Ported user service:

- Scala Service
- Http Requests -> JSON response
- Stores information for the user
- At least one request per page load (ton of requests)
-Performance and scale is essential.

Some requirements:

- 150K per minute peak
- ~1ms response time at peak (Magic number is ~5ms)
- With Mongo they had problems with performance under write load (needed to throttle writes)
- Active-Active deploy difficult.

## Installation Nodes

Used two production data-centers
- Different sides of the country
- Two Rings
- 5 nodes per ring

## Bitcask vs Leveldb

- Two different storage options
- Bitcask does not support 2ndary indexes, built side collections

## Rewrite SVC-User

- Scala services
- [Uses cake pattern][1]
- Much easier than anticipated
- Cake pattern make it easy to swap out components
- (Hates the cake pattern, but it worked out here)

[1]: http://jonasboner.com/2008/10/06/real-world-scala-dependency-injection-di/

## Testing Strategy, BASH the THING!

Real traffic!

- [Splitter][2] (used to send traffic to both services) - discarded response from Riak Backend

[2]: https://www.github.com/ebowman/splitter

```                             
                                    /--- Process - Riak
Load Balancer (Zeus) -> Splitter --<
                                    \--- Process - Old
```

- Zeus Bench to generate HTTP Load
- Simple scala script simulates 1000's of inputs per second

## Failure Cases

- Kill Random nodes (10 nodes, 2 rings of 5)
- Performance degrades temporarilly with dead nodes.

Gossip Protocol to replicate data aross multiple nodes when nodes die.

## What went wrong?

Each datacenter had 5 modes, process keeps them in sync. 3 Replicas per data centers. Network partitions can be challenging which is why 3 is a magic number.

## Q & A

Imagine that you've sored item in cart in Riak and link goes down, half of the cart is lost. Write code to merge two carts? Write whatever logic 

Used Zookeper in each data center. "In-sync properties file" - points to the correct node authoritatize -- don't talk outside of the datacenter that much.

[Bits of conversation I missed here]

If the link between two data centers is broken, you are out of luck.

Riak vs. Cassandra

(Cassandra not really explored)
Also uses Mongo and Voldemort.
Had negative experience with Cassandra a couple years ago.






