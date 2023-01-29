---
toc: true
layout: post
description: A note on chapter 6 of the book Designing Data-Intensive Applications
categories: [markdown, Designing Data-Intensive Applications]
title: Chapter 6: Partitioning
---

# Chapter 6: Partitioning

> *partition* == *shard* in MongoDB, Elasticsearch, and SolrCloud == *region* in HBase == *tablet* in Bigtable == a *vnode* in Cassandra and Riak == *vBucket* in CouchBase.

##### Purpose

## Partitioning of Key-Value Data

> A Hot spot: A partition with a disproportionately high load.

### Partitioning by Key Range

Assign a range of **sorted keys** to a partition.

**Note:**

Sorting helps to improve the performance of range queries. Yet, the key should be chosen with caution to avoid hot spots in case the application often accesses the data with close keys in the sorted order.

### Partitioning by Hash of Key

Assign a range of hash values (hashing keys) to a partition.

**Note:**

This approach makes sure that keys are distributed evenly across partitions since it destroys the key order -  after hashing, two similar keys will no longer close in the sorted order. 

### Skewed Workloads and Relieving Hot Spots

In either case, there is still a chance to suffer the celebrity issue since it does not depend on the key ranges but the key itself. To solve this problem, we can add a random element to the key. For example, adding a 2-digit decimal to the key will make the data for that key split into 100 keys. Yet, this solution comes with a performance degradation when read since we have to request data from 100 keys instead of 1. Also, not all the key needs to be split, optimally, we should detect which key has high throughput and split it.

## Partitioning and Secondary Indexes

> The secondary index is used to search for occurrences of a particular value.

The problem with secondary indexes is that they don't neatly map to partitions.

### Partitioning Secondary Indexes by Document

![]({{ site.baseurl }}/images/eb58f064e477183561d38b8d1c0f721bf7617393.png "Partitioning secondary indexes by document")

In this approach, we create local indexes for each partition. In order words, each partition maintains its secondary indexes, covering only the documents available in that partition.

For example, after we create the secondary index on **color**. When a red car with ID **306** is added to the database, the database partition of partition 0 automatically adds the ID of 306 to the list of document IDs for the index entry `color:red`.

This approach adds up complexity on read. Whenever we need to query a list of cars using color, we need to send a request to all the partitions, retrieve all the data and then combine it to generate the result.

### Partitioning Secondary Indexes by Term

![]({{ site.baseurl }}/images/bc9c7d2a677ff25aa74382e6efcf6e4306558973.png "Partitioning secondary indexes by term")

In this approach, we apply the same mechanism as **partitioning by key range or partitioning by hash of key**. We create a global index covering all the data across the partitions. The global index will be partitioned by the range and stored across partitions. Now, instead of each partition maintaining a list of document IDs for the index entry `color:red`, only partition 0 maintains it. This helps to improve the performance when we need to scan a range of values using the color attribute. Yet, this approach adds up the complexity when writing, when we have multiple secondary indexes and each index partition, is placed on different partitions.

## Rebalancing Partitions

No matter what rebalancing schemes are used, they should meet the following requirements:

- After rebalancing, the load queries should be distributed evenly across nodes.

- While rebalancing is happening, the database should still accept reads and writes.

- No more data than necessary should be moved between nodes to reduce network and disk I/O loads.

### Rebalancing Strategies

Strategies to assign partitions to nodes.

#### Hash Mod N

Hash the partition key and mod the value by N to decide the target node.

**Caveat:**

When adding new a node or reducing a failed node, most of the key will need to move to another node, which is expensive.

#### Fixed number of partitions

Define a fixed number of partitions at the beginning and distributed the partitions evenly to nodes. Whenever, the number of nodes changes, the number of partitions on each node will be changed, either increasing when removing a node or decreasing when adding a node.

**Caveat:**

It is hard to choose the "just right" number of partitions at the beginning.

#### Dynamic number of partitions

We config the min and max sizes of the partitions and use these numbers as the boundaries. Whenever the size of a partition is out of this boundary, we either split the partitions or merge them. In case of splitting, when a large dataset is split into 2 halves, one half will be sent to another node in the cluster. The advantage of this approach is that the number of partitions adapts to the total data volume.

**Caveat:**

The database is started with a single partition. Yet, it can be overcome by pre-splitting.

#### Partitioning proportionally to nodes

The number of partitions stays unchanged when the number of nodes remains. Yet, it increases or decreases when we add a new node or remove a failed node respectively.

#### Request Routing

![]({{ site.baseurl }}/images/0f508f6930d682b8faa0a42fc4b67112ab413df8.png "Three different ways of routing a request to the right node")

In general, there are three approaches to routing a request to the correct node. No matter what approach we choose, the key challenge is how can we update the knowledge of partitions when the partitions move from one node to another node.
Many distributed database systems rely on third-party coordination services like ZooKeeper to keep track of cluster metadata.

![]({{ site.baseurl }}/images/0c28f90cd54f96892fbf0832a17ca34f07eeb0e9.png "Using ZooKeeper to keep track of assignment of partitions to nodes")

All the nodes and their partition mappings are registered in ZooKeeper. When there is a change in the mapping, ZooKeeper will notify the routing tier so that it can keep the routing information up to date.

## Glossary:

**Shared-nothing architecture == horizontal scaling == scaling out**

In this architecture, each machine of the virtual machine running the database software is called a node. Each node uses its CPUs, RAM, and disks independently. Any coordination between nodes is done at the software level, using a conventional network.
