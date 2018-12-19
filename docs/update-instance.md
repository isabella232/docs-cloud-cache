---
title: Updating a Pivotal Cloud Cache Service Instance
---

You can apply all optional parameters to an existing service instance using the `cf update-service` command. You can, for example, scale up a cluster by increasing the number of servers.

Previously specified optional parameters are persisted through subsequent updates. To return the service instance to default values, you must explicitly specify the defaults as optional parameters.

For example, if you create a service instance with five servers using a plan that has a default value of four servers:

<pre class='terminal'>
$ cf create-service p-cloudcache small my-cloudcache -c '{"num_servers": 5}'
</pre>

And you set the `new_size_percentage` to 50%:

<pre class='terminal'>
$ cf update-service my-cloudcache -c '{"new_size_percentage": 50}'
</pre>

Then the resulting service instance has `5` servers and `new_size_percentage` of 50% of heap.

## <a id="cluster-rebalancing"></a>Rebalancing a Cluster

When updating a cluster to increase the number of servers, the available heap size is increased. When this happens, PCC automatically rebalances data in the cache to distribute data across the cluster.

This automatic rebalancing does not occur when a server leaves the cluster and later rejoins, for example when a VM is re-created, or network connectivity lost and restored.
 In this case, you must manually rebalance the cluster using the gfsh [`rebalance` command](http://gemfire.docs.pivotal.io/geode/tools_modules/gfsh/command-pages/rebalance.html) while authenticated as a cluster operator. 
<p class="note"><strong>Note</strong>: You must first [connect with gfsh](#gfsh-connect) before you can use the `rebalance` command.</p>

## <a id="cluster-restart"></a>Restarting a Cluster

Restarting a cluster stops and restarts each cluster member in turn,
issuing a rebalance as each restarted server joins the cluster.

<p class="note warning"><strong> WARNING:</strong> Restart of a cluster may cause data loss.</p>

There is a potential for data loss when restarting a cluster;
the region type and number of servers in the cluster
determine whether or not data is lost.

- **All data is lost when restarting a cluster with these region types and
number of servers:**

    - Partitioned regions without redundancy or persistence.
As each server is stopped, the region entries hosted in buckets on
that stopped server are permenently lost.
    - Replicated regions without persistence on a cluster that has
a single server.
    - A Dev Plan cluster loses all data, as there is a single server
and no region persistence.

- **No data is lost when restarting the cluster with these region types
and number of servers:**

    - Replicated regions for clusters with more than one server.
    - Persistent regions will not lose data,
as all data is on the disk and available upon restart of a server.
    - Partitioned regions with redundancy.
When the server with the primary copy of an entry is stopped,
the redundant copy still exists on a running server.

To restart a cluster, use the cluster operator credentials to
run the command:

<code>cf update-service SERVICE-INSTANCE-NAME -c '{"restart": true}'</code>

For example:

<pre class='terminal'>
$ cf update-service my-cluster -c '{"restart": true}'
</pre>

## <a id="plan-updates"></a> About Changes to the Service Plan

Your PCF operator can change details of the service plan available on the Marketplace. If your operator changes the default value of one of the optional parameters, this does not affect existing service instances.

However, if your operator changes the allowed values of one of the optional parameters, existing instances that exceed the new limits are not affected, but any subsequent service updates that change the optional parameter must adhere to the new limits.

For example, if the PCF operator changes the plan by decreasing the maximum value for `num_servers`, any future service updates must adhere to the new `num_servers` value limit.

You might see the following error message when attempting to update a service instance:

<pre class='terminal'>
$ cf update-service  my-cloudcache -c '{"num_servers": 5}'
Updating service instance my-cloudcache as admin...
FAILED
Server error, status code: 502, error code: 10001, message: Service broker error: Service cannot be updated at this time, please try again later or contact your operator for more information
</pre>

This error message indicates that the operator has made an update to the plan used by this service instance. You must wait for the operator to apply plan changes to all service instances before you can make further service instance updates.

