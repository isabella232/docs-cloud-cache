---
title:  Region Design
---
<a id="region-design"></a>

Cached data are held in GemFire regions.
Each entry within a region is a key/value pair.
The choice of key and region type affect
the performance of the design.
There are two basic types of regions: partitioned and replicated.
The distinction between the two types is based on how 
entries are distributed among servers
that host the region.

## <a id="keys"></a> Keys

Each region entry must have a unique key.
Use a wrapped primitive type of `String`, `Integer`, or `Long`.
Experienced designers have
a slight preference of `String` over `Integer` or `Long`. 
Using a `String` key enhances the development and debugging
environment by permitting the use of a REST API (Swagger UI),
as it only works with `String` types.

## <a id="partitioned-regions"></a> Partitioned Regions

A partitioned region distributes region entries across servers
by using hashing.
The hash of a key maps an entry to a bucket.
A fixed number of buckets are distributed across the servers
that host the region.

Here is a diagram that shows a single partitioned region (highlighted)
with very few entries to illustrate partitioning.

![Partitioned Region](partitioned-region.png)

A partitioned region is the proper type of region to use when
one or both of these situations exist:

- The region holds vast quantities of data.
There may be so much data that you need to add more servers to scale the system up.
PCC can be scaled up without downtime; to learn more, see [Updating a Pivotal Cloud Cache Service Instance](./update-instance.html).

- Operations on the region are write-heavy, meaning that there are a lot of entry updates.

Redundancy adds fault tolerance to a partitioned region.
Here is that same region,
but with the addition of a single redundant copy of each each entry.
The buckets drawn with dashed lines are redundant copies.
Within the diagram, the partitioned region is highlighted.

![Partitioned Region with Redundancy](partitioned-redundant-region.png)

With one redundant copy,
the GemFire cluster can tolerate a single server failure
or a service upgrade 
without losing any region data.
With one less server,
GemFire revises which server holds the primary copy of an entry.

A partitioned region without redundancy permanently loses data during a service upgrade or if a server goes down.
All entries hosted in the buckets on the failed server are lost.

### <a id="partitioned-types"></a> Partitioned Region Types for Creating Regions on the Server

Region types associate a name with a particular region configuration.
The type is used when creating a region.
Although more region types than these exist,
use one of these types to ensure that no region data is lost
during service upgrades or if a server fails.
These partitioned region types are supported:

- `PARTITION_REDUNDANT`
Region entries are placed into the buckets that are
distributed across all servers hosting the region.
In addition, GemFire keeps and maintains a declared number of
redundant copies of all entries.
The default number of redundant copies is 1.
- `PARTITION_REDUNDANT_HEAP_LRU`
Region entries are placed into the buckets that are
distributed across all servers hosting the region.
GemFire keeps and maintains a declared number of redundant copies.
The default number of redundant copies is 1.
As a server (JVM) reaches a heap usage of 65% of available heap,
the server destroys entries as space is needed for updates.
The oldest entry in the bucket where a new entry lives
is the one chosen for destruction.
- `PARTITION_PERSISTENT`
Region entries are placed into the buckets that are
distributed across all servers hosting the region,
and all servers persist all entries to disk.
- `PARTITION_REDUNDANT_PERSISTENT`
Region entries are placed into the buckets that are
distributed across all servers hosting the region,
and all servers persist all entries to disk.
In addition, GemFire keeps and maintains a declared number of
redundant copies of all entries.
The default number of redundant copies is 1.
- `PARTITION_REDUNDANT_PERSISTENT_OVERFLOW`
Region entries are placed into the buckets that are
distributed across all servers hosting the region,
and all servers persist all entries to disk.
In addition, GemFire keeps and maintains a declared number of
redundant copies of all entries.
The default number of redundant copies is 1.
As a server (JVM) reaches a heap usage of 65% of available heap,
the server overflows entries to disk when it needs to make space
for updates.
- `PARTITION_PERSISTENT_OVERFLOW`
Region entries are placed into the buckets that are
distributed across all servers hosting the region,
and all servers persist all entries to disk.
As a server (JVM) reaches a heap usage of 65% of available heap,
the server overflows entries to disk when it needs to make space
for updates.

## <a id="replicated-regions"></a> Replicated Regions

Here is a replicated region with very few entries (four)
to illustrate the distribution of entries across servers.
For a replicated region,
all servers that host the region have a copy of every entry.

![Replicated Region](replicated-region.png)

GemFire maintains copies of all region entries on all servers.
GemFire takes care of distribution and keeps the entries
consistent across the servers.

A replicated region is the proper type of region to use when
one or more of these situations exist:

- The region entries do not change often.
Each write of an entry must be propagated to all servers
that host the region.
As a consequence,
performance suffers when many concurrent write accesses
cause subsequent writes to all other servers hosting the region.
- The overall quantity of entries is not so large as to push the limits of memory space for a single server.
The PCF service plan sets the server memory size.
- The entries of a region are frequently accessed
together with entries from other regions.
The entries in the replicated region are always available
on the server that receives the access request,
leading to better performance.

### <a id="replicated-types"></a> Replicated Region Types for Creating Regions on the Server

Region types associate a name a particular region configuration.
These replicated region types are supported:

- `REPLICATE`
All servers hosting the region have a copy of all entries.
- `REPLICATE_HEAP_LRU`
All servers hosting the region have a copy of all entries.
As a server (JVM) reaches a heap usage of 65% of available heap,
the server destroys entries as it needs to make space for updates.
- `REPLICATE_PERSISTENT`
All servers hosting the region have a copy of all entries,
and all servers persist all entries to disk.
- `REPLICATE_PERSISTENT_OVERFLOW`
All servers hosting the region have a copy of all entries.
As a server (JVM) reaches a heap usage of 65% of available heap,
the server overflows entries to disk as it need to make space
for updates.

## <a id="persistent-regions"></a> Persistence

Persistence adds a level of fault tolerance to a PCC service
instance by writing all region updates to local disk.
Disk data, hence region data, is not lost upon cluster failures
that exceed redundancy failure tolerances.
Upon cluster restart, regions are reloaded from the disk,
avoiding the slower method of restart that reacquires data using
a database of record.

Creating a region with one of the region types that includes
`PERSISTENT` in its name
causes the instantiation of local disk resources
with these default properties:

- Synchronous writes. All updates to the region generate
operating system disk writes to update the disk store.
- The disk size is part of the instance configuration.
See [Configure Service Plans](operator.html#plan-config)
for details on setting the persistent disk types for the server.
Choose a size that is at least twice as large as
the expected maximum quantity of region data, with an absolute
minimum size of 2 GB.
Region data includes both the keys and their values.
- Warning messages are issued when a 90% disk usage threshold
is crossed.

## <a id="app-regions"></a> Regions as Used by the App

The client accesses regions hosted on the servers by creating a cache
and the regions.
The type of the client region determines if
data is only on the servers or 
if it is also cached locally by the client in addition to being on
the servers.
Locally cached data can introduce consistency issues,
because region entries updated on a server are
not automatically propagated to the client's local cache.

Client region types associate a name with a particular
client region configuration.

- `PROXY` forwards all region operations to the servers.
No entries are locally cached.
Use this client region type unless there is a compelling reason to use 
one of the other types.
Use this type for all Twelve-Factor apps
in order to assure stateless processes are implemented.
Not caching any entries locally
prevents the app from accidentally caching state.
- `CACHING_PROXY` forwards all region operations to the servers,
and entries are locally cached.
- `CACHING_PROXY_HEAP_LRU` forwards all region operations to the servers,
and entries are locally cached.
Locally cached entries are destroyed when the app’s
usage of cache space causes its JVM to hit
the threshold of being low on memory.

## <a id="region-design-example"></a> An Example to Demonstrate Region Design

Assume that on servers, a region holds entries representing customer data.
Each entry represents a single customer.
With an ever-increasing number of customers,
this region data is a good candidate for a partitioned region.

Perhaps another region holds customer orders.
This data also naturally maps to a partitioned region.
The same could apply to a region that holds order shipment data
or customer payments.
In each case, the number of region entries continues to grow over time,
and updates are often made to those entries,
making the data somewhat write heavy.

A good candidate for a replicated region would be the data
associated with the company’s products.
Each region entry represents a single product.
There are a limited number of products,
and those products do not often change.

Consider that as the client app goes beyond the most simplistic
of cases for data representation,
the PCC instance hosts all of these regions
such that the app can access all of these regions.
Operations on customer orders, shipments,
and payments all require product information.
The product region benefits from access to all its entries 
available on all the cluster's servers,
again pointing to a region type choice of a replicated region.

