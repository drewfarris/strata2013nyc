Hadoop Adventures with Adam Kawa of Spotify
=========================

#### Adventure 1

resource intensive hive queries
immediately after expaning cluster

valide, correct, fail on large datasets

other users can run the same query with no problem
user has the correct hdfs permissions / directories.

when the user runs  the query - cluster experiences stability problems. 
less responsive namenode
losing connection with data nodes 

get exception trying to get groups for user <USERNAME>

ShellBasedUnixGroupsMatching

14592 times during 8 minutes (4768/min peak)

username come from cluent
groupname is resolved on namenode (id -Gn username)
if the user does not hav an account on the namenode `ExitCodeException` is thrown
large queries - thousands of tasks, many exceptions
Those exceptions will freeze namenode, freeze for longer than 10 minutes, dadanodes marked dead

Quick and dirty: account for users on NameNode
User AD/LDAP for user-group rsolution

`hadoop.security.group.mapping.ldap.*` settings
Or Kerberos

Out fix. posixGroup sucks, so we used a workaround using nsswitch.conf to point LDAP - solved at the operating system level and not the hadoop issue.

Not a public park, but like a private zoo where you need a ticket to be let in -- know who will used the cluster, create permissions etc.

- Identify Who is abusing the cluster by access HDFS too frequently.
- Parse the NameNode logs regularly for Exception message.

#### Adventure 2

Many datanodes marked dead by NameNode because they could not sent heartbeat
when DN is marked dead an expensive replication starts, and map tasks can only process only non-local blocks.
Also, `BlockMissingException`

ps shows DataNode daemonds exists, `jstack` doesn't show anything interesting

Some of the DN's were able to join the cluster after a few minutes.
Why are they blocked?

`dfs.namenode.heartbeat.recheck-interval`

What if we up this?

to 600

Crazy idea: deploy this change and don't measure it's impact.

#### Adventure 3

Cluster becomes unstable after around 1 hour of uptime.
After 1 hour, all of the datanodes are marked dead.
Why does it happen around 1 hour?

Data node sends a block report every hour, could this be related? Are we having a storm of block reports after the first hour? OVerload the namenode and cause garbage collection?

`blockreport.initialDelay` --- wait random seconds before first blockreport and then send each hour.

Increase the heap of the namnode and tweak sizes of generations.

Let's avoid restarts and *do it all at the same time*!

Couple hours later…
Couple days later…

there are 10779564 missing blocks!
Large number of dead datanodes.

First finding:

heartbeat check ingerval was milliseconds! not seconds! 31 seconds instead of 20 minutes. By mistake made the interval much shorter.

Initial delay was bad. 
