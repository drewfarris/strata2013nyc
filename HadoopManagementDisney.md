Integrated Hadoop Management
====

### Zach Snyder
Systems Engineer

Strata NYC 2013


Disney, ESPN, Disney's Global Ad Systems.

Old ads platform vs. new platform based on Hadoop. Used for Counting Ads. 

Cluster didn't stay up all of the time. 

Environment didn't run well. 

How they brought Hadoop into the environment,
Deployment from manual to automated process


- Our Environment
- Deployment 


## Environment

Multiple Generation of hardware over 4 years.

3 Hadoop Clusters
- Development Stating and Production
- Use cases drive by end user

Production Cluster Contains 144 Nodes
- 1.12 PB
- 3388 M/R Slots
- 4 Generations of DataNode Hardware
- Blades VM, SAN, NAS - goal is to use the best tools where they can be used


#### Deployment Before

- Datacenter Team Racks and Stacks
- Servers Configued By Hand / Also used Cobbler
- Configuration Driven from Checklists; Memory
- Drive Layouts
- Package INstallation
- Security 

Monitoring Distributed Across Tools
- Old way was Ganglia but didn't tie into anything


Hadoop Configurations were distributed by hand. Different configurations on different servers was problematic

Linear, folks at consoles, difficult to parallelize. More than 20 steps and datanodes took 4 hours.

Result: Low consistency

#### Deployment Configuration

More Cobbler 
Moved build process into kickstart scripts
Later introduced Chef
Shifted Monitoring to Cloudera Manager - consolidation of monitoring

#### Deployment Today

Datacenter Team Racks
System is provisioned by Cobbler and Chef
-- OS Core config driven by cobbler kickstart

Hosts are boostrapped in Chef
- Chef install
- security
- configuration and tuning
- cloudera manager installed

Servers appear in Cloudera Manager, ready for role assignment.

- Data nodes take ~1 hour  unattended
- every node is thie same
- Chef runs every 15 minutes asserting intended state

Considered switching fomr ext4 to xfs due to formatting time.

Result

- from 20 steps to 4 steps.
- saved 3 hours per node
- data nodes requirem minutes to provision
- nodes can be provisioned in parallel.


#### Not done yet

- Hand Crafted to Automated Deployments took 3+ Years
- Not done yet, opportunities to autmoate
- Little tolerance for downtime, even less for automation failure.

Process tends to be slow moving.

**Provisioning Demo**

Enables other use cases - testing, problem verification.

Internal shim for chef that works with the internal openstack environment.

Server comes up, downloads chef, bootstraps and downloads cookbooks. Yum repos, java, ruby. Fixes permissions and installs cloudera manager processes. Uses yum to do this. Installs 125 packages in the background.

[It's taking a long time do to this]

Last thing it does is create the cloudera manager application and creates service. Chef process takes 120s or so.

Cloudera manager is likely the most important tool they have as far as hadoop is concerned. New node shows up without roles, no hadoop anything. New host wizard used to add hosts and installs data with parcels or packages.

Disney is big fans of Parcels - parcels got very far to keeping hosts consistent.

With parcels, all of the code is included up front.

Cloudera manager 4.5 from CHDu4

Parcels are installed from Cloudera Manager server

Cluster stored in a secure enclave.

Enough cloudera marketing stuff.

#### Final wrapup.

Host inspector complains about THP, shows how knife ssh is used to adjust this.

Host templates are used to determine what servers should run where. 


#### Other Software

- Orion
- Nimsoft
- ServiceDesk

Host management and monitoring is done through CM. 

**Orion**

Software monitoring tool from SolarRules used to be the main monitor for this environment.

- Host Monitoring (Ping, CPU, Memory, Disk, Hardware) 
- Application Monitoring (Hadoop HTTP interfaces, jobtracker, tasktracker, webhdfs)


**Nimsoft**

CA Nimsoft
SNMP Gateway - cloudera SNMP alerts
Future Application monitoring (Strong API)

**ServiceDesk**

CA Servicedesk
Service Management Tool
- Incident, REquest, Change and PRoblem Tracking
- COnfiguration management knowkledge base

API Layer between Cloudera Manger and Orion, Nimsoft

###### Cloudera Manager API for monitoring used 2 years ago

Examples to get cluster status:

```
GET /api/v5/clusters/demo/services/hdfs1
```

## Cloudera Manager 

Use multiple instances via:

```
GET /api/v5/cm/deployment 

Migrate settings between Cloudera MAnagers.

## Summary

Build Process Saves Tim
- Save 4 man hours per build
- Multiplied by 150+ nodes
- Build Consistency saves time troubleshooting

Next seps
- continue to eliminate manual steps
- more chef, less cobbler kickstart
- further integrate cloudera manager

Went from completely manual process to something that as not manual

### Q & A

*Why Chef overr Puppet* -- relationship with opscode team.
*How big are test clusters* -- Hadoop as a service for other teams … [missed this one]

Test in dev, environments driven through chef, promote changes through environments

Using the GUI, having heated discussion as to right way. 

What place is authority on how nodes should be configured. For now, roles are assigned so rarely, it makes sense to assign roles in the GUI vs. 

What is installed via chef vs. CLoudera manager

Security packages, Operation System LEven Stuff. 

Difficulty Deploying Custom Code mixed mode via both Chef and Cloudera Manager. Custom code via parcels are not there yet.

