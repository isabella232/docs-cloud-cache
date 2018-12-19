---
title: Design Patterns
---
<a id="design-patterns"></a>

## <a id="inline-cache"></a> The Inline Cache

An inline cache places the caching layer between the app
and the backend data store.

![inline caching pattern](inline.png)

The app will want to accomplish CRUD
(CREATE, READ, UPDATE, DELETE) operations on its data.
The app’s implementation of the CRUD operations result in cache operations
that break down into cache lookups (reads) and/or cache writes.

The algorithm for a cache lookup quickly returns the cache entry
when the entry is in the cache.
This is a cache hit.
If the entry is not in the cache, it is a cache miss,
and code on the cache server retrieves the entry
from the backend data store.
In the typical implementation,
the entry returned from the backend data store on a cache miss
is written to the cache,
such that subsequent cache lookups of that same entry result in cache hits.

The implementation for a cache write typically creates or updates
the entry within the cache.
It also creates or updates the data store in one of the following ways:

- Synchronously, in a write-through manner
- Asynchronously, in a write-behind manner

Developers design the server code to implement this inline-caching pattern.
Developers implement the `CacheLoader` interface to handle cache misses.
Deploy its JAR file to the servers by following the directions in 
[Deploy an App JAR File to the Servers](./using-pcc.html#deploy-app-jars).

## <a id="lookaside-cache"></a> The Look-Aside Cache

The look-aside pattern of caching places the app in charge
of communication with both the cache and the backend data store.

![look-aside caching pattern](look-aside.png)

The app will want to accomplish CRUD
(CREATE, READ, UPDATE, DELETE) operations on its data.
That data may be 

- in both the data store and the cache
- in the data store, but not in the cache
- not in either the data store or the cache

The app’s implementation of the CRUD operations result in cache operations
that break down into cache lookups (reads) and/or cache writes.

The algorithm for a cache lookup returns the cache entry
when the entry is in the cache.
This is a cache hit.
If the entry is not in the cache, it is a cache miss,
and the app attempts to retrieve the entry from the data store.
In the typical implementation,
the entry returned from the backend data store is written to the cache,
such that subsequent cache lookups of that same entry result in cache hits.

The look-aside pattern of caching leaves the app free to implement
whatever it chooses if the data store does not have the entry.

The algorithm for a cache write implements one of these:

- The entry is either updated or created within the data store,
and the entry is updated within or written to the cache.
- The entry is either updated or created within the backend data store,
and the copy currently within the cache is invalidated.

<p class='note'><strong>Note:</strong> SDG (Spring Data GemFire) 
supports the look-aside pattern, as detailed at
<a href="https://docs.spring.io/spring-data/gemfire/docs/current/reference/html/#bootstrap-annotation-config-caching">Configuring Spring’s Cache Abstraction</a>.</p>

## <a id="active-active-WAN-pattern"></a> Bidirectional Replication Across a WAN

Two PCC service instances may be connected across a WAN to form
a single distributed system with asynchronous communication.
The cluster within each of the PCC service instances will host the same region.
Updates to either PCC service instance
are propagated across the WAN to the other PCC service instance.
The distributed system implements an eventual consistency
of the region
that also handles write conflicts which occur when a single
region entry is modified in both PCC service instances at the same
time.

In this active-active system, an external entity implements load-balancing 
by directing app connections to one of the two service instances. 
If one of the PCC service instances fails,
apps may be redirected to the remaining service instance.

This diagram shows multiple instances of an app interacting
with one of the two PCC service instances, cluster A and cluster B.
Any change made in cluster A is sent to cluster B,
and any change made in cluster B is sent to cluster A.

![Bidirectional WAN replication pattern](WAN-bidirectional.png)

## <a id="WAN-pattern"></a> Blue-Green Disaster Recovery

Two PCC service instances may be connected across a WAN to form
a single distributed system with asynchronous communication.
An expected use case propagates all changes to a region's data
from the cluster within one service instance (the primary) to the other.
The replicate increases the fault tolerance of the system by
acting as a hot spare.
In the scenario of the failure of an entire data center or an
availability zone,
apps connected to the failed site can be redirected 
by an external load-balancing entity to the replicate,
which takes over as the primary.

In this diagram, cluster A is primary, and it replicates all data
across a WAN to cluster B.

![WAN replication pattern](WAN1.png)

If cluster A fails,
cluster B takes over.

![WAN replicate becomes primary](WAN2.png)

## <a id="CQRS-WAN-pattern"></a> CQRS Pattern Across a WAN

Two PCC service instances may be connected across a WAN to form
a single distributed system that implements
a CQRS (Command Query Responsibility Segregation) pattern.
Within this pattern, commands are those that change the state,
where state is represented by region contents.
All region operations that change state are directed to the cluster within one
PCC service instance.
The changes are propagated asynchronously to the cluster within the other
PCC service instance via WAN replication,
and that other cluster provides only query access to the region data.

This diagram shows an app that may update the region
within the PCC service instance of cluster A.
Changes are propagated across the WAN to cluster B. 
The app bound to cluster B may only query the region data;
it will not create entries or update the region.

![CQRS WAN replication pattern](WAN-CQRS.png)

## <a id="hub-spoke-pattern"></a> Hub-and-Spoke Topology with WAN Replication

Multiple PCC service instances connected across a WAN
form a single hub and a set of spokes.
This diagram shows PCC service instance A is the hub,
and PCC service instances B, C, and D are spokes.

![Hub-and-Spoke pattern](hub-spoke.png)

A common implementation that uses this topology directs all app operations
that write or update region contents to the hub.
Writes and updates are then propagated asynchronously across the WAN
from the hub to the spokes.

## <a id="follow-the-sun-pattern"></a> Follow-the-Sun Pattern

Performance improves when operation requests originate
in close proximity to the service instance that handles
those requests.
Yet many data sets are relevant and used all over the world.
If the most active location for write and update operations moves over the
course of a day,
then a performant design pattern is a variation on the hub-and-spoke
implementation that changes which PCC service instance is the hub
to the most active location.

Form a ring that contains each PCC service instance that will
act as the hub.
Define a token to identify the hub.
Over time, pass the token from one PCC service instance to the 
next, around the ring.

This diagram shows PCC service instance A is the hub,
as it has the token, represented in this diagram as a star.
PCC service instances B, C, and D are spokes.
Write and update operations are directed to the hub.

![Follow-the-Sun pattern, A is the hub](follow-the-sun-1.png)

This diagram shows that the token has passed from A to B,
and B has become the hub.

![Follow-the-Sun pattern, B is the hub](follow-the-sun-2.png)
