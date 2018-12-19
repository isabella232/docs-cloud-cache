---
title: Using Pivotal Cloud Cache
---

## <a id="create-regions"></a> Create Regions with gfsh

After connecting with gfsh as a `cluster_operator_XXX`, you can define a new cache region.

The following command creates a partitioned region with a single redundant copy:

<pre class='terminal'>
gfsh>create region --name=my-cache-region --type=PARTITION_REDUNDANT_HEAP_LRU
     Member      | Status
---------------- | -------------------------------------------------------
cacheserver-z2-1 | Region "/my-cache-region" created on "cacheserver-z2-1"
cacheserver-z3-2 | Region "/my-cache-region" created on "cacheserver-z3-2"
cacheserver-z1-0 | Region "/my-cache-region" created on "cacheserver-z1-0"
cacheserver-z1-3 | Region "/my-cache-region" created on "cacheserver-z1-3"
</pre>

See [Region Design](../app-development.html#region-design)
for guidelines on choosing a region type.

You can test the newly created region by writing and reading values with gfsh:

<pre class='terminal'>
gfsh>put --region=/my-cache-region --key=test --value=thevalue
Result      : true
Key Class   : java.lang.String
Key         : test
Value Class : java.lang.String
Old Value   : NULL


gfsh>get --region=/my-cache-region --key=test
Result      : true
Key Class   : java.lang.String
Key         : test
Value Class : java.lang.String
Value       : thevalue
</pre>

In practice, you should perform these get/put operations from a deployed PCF app. To do this, you must bind the service instance to these apps.

## <a id="java-build-pack-requirement"></a> Java Build Pack Requirements

To ensure that your app can use all the features from PCC,
use the latest buildpack.
The buildpack is available on GitHub at [cloudfoundry/java-buildpack](https://github.com/cloudfoundry/java-buildpack).


## <a id="bind-service"></a> Bind an App to a Service Instance

Binding your apps to a service instance enables the apps to connect to the service instance and read or write data to the region.
Run `cf bind-service APP-NAME SERVICE-INSTANCE-NAME` to bind an app to your service instance.
Replace `APP-NAME` with the name of the app.
Replace `SERVICE-INSTANCE-NAME` with the name you chose for your service instance.

<pre class='terminal'>
$ cf bind-service my-app my-cloudcache
</pre>

Binding an app to the service instance provides connection information through the `VCAP_SERVICES` environment variable.
Your app can use this information to configure components, such as the GemFire client cache, to use the service instance.

The following is a sample `VCAP_SERVICES` environment variable:

```
{
  "p-cloudcache": [
    {
      "credentials": {
	"locators": [
	  "10.244.0.4[55221]",
	  "10.244.1.2[55221]",
	  "10.244.0.130[55221]"
	],
	"urls": {
	  "gfsh": "https://cloudcache-1.example.com/gemfire/v1",
	  "pulse": "https://cloudcache-1.example.com/pulse"
	},
	"users": [
	  {
	    "password": "some_developer_password",
	    "username": "developer_XXX"
	  },
	  {
	    "password": "some_password",
	    "username": "cluster_operator_XXX"
	  }
	]
      },
      "label": "p-cloudcache",
      "name": "test-service",
      "plan": "caching-small",
      "provider": null,
      "syslog_drain_url": null,
      "tags": [],
      "volume_mounts": []
    }
  ]
}
```

## <a id="pulse"></a> Use the Pulse Dashboard

You can access the Pulse dashboard for a service instance by accessing the pulse-url you [obtained from a service key](#create-service-key) in a web browser.

Use either the `cluster_operator_XXX` or `developer_XX` credentials to authenticate.

## <a id="nozzle"></a> Access Service Instance Metrics

To access service metrics, you must have **Enable Plan** selected under **Service Plan Access** on the page where you configure your tile properties. (For details, see the [Configure Service Plans](./operator.html#plan-config) page.)

PCC service instances output metrics to the Loggregator Firehose. You can use the
[Firehose plugin](https://docs.cloudfoundry.org/loggregator/cli-plugin.html) to
view metrics output on the CF CLI directly or connect the output to any other
[Firehose nozzle](https://docs.cloudfoundry.org/loggregator/architecture.html#nozzles);
for example, the nozzle for [Datadog](https://github.com/cloudfoundry-attic/datadog-firehose-nozzle).

PCC v1.3.x supports metrics for the whole cluster and metrics for each member.
Each server and locator in the cluster outputs metrics.

### Service Instance (Cluster-wide) Metrics
* serviceinstance.MemberCount: the number of VMs in the cluster
* serviceinstance.TotalHeapSize: the total MBs of heap available in the cluster
* serviceinstance.UsedHeapSize: the total MBs of heap in use in the cluster

### Member (per-VM) Metrics
* member.GarbageCollectionCount: the number of JVM garbage collections that have occured on this member since startup
* member.CpuUsage: the percentage of CPU time used by the Gemfire process
* member.GetsAvgLatency: the avg latency of GET requests to this Gemfire member
* member.PutsAvgLatency: the avg latency of PUT requests to this Gemfire member
* member.JVMPauses: the number of JVM pauses that have occured on this member since startup
* member.FileDescriptorLimit: the number of files this member allows to be open at once
* member.TotalFileDescriptorOpen: the number of files this member has open now
* member.FileDescriptorRemaining: the number of files that this member could open before hitting its limit
* member.TotalHeapSize: the number of megabytes allocated for the heap
* member.UsedHeapSize: the number of megabytes currently in use for the heap
* member.UnusedHeapSizePercentage: the percentage of the total heap size that is not currently being used

## <a id="nozzle-service-metrics"></a> Access Service Broker Metrics

Service broker metrics are on by default and can be accessed through the [Firehose nozzle plugin](https://docs.cloudfoundry.org/loggregator/cli-plugin.html). For more information on broker metrics, see [On Demand Broker Metrics](https://docs.pivotal.io/svc-sdk/odb/0-16/operating.html#metrics).

## <a id="exporting_logs"></a> Export gfsh Logs

You can get logs and `.gfs` stats files from your PCC service instances using the `export logs` command in gfsh.

1. Use the [Connect with gfsh over HTTPS](#gfsh-connect-https) procedure to connect to the service instance for which you want to see logs.
1. Run `export logs`.
1. Find the ZIP file in the directory where you started gfsh. 
   This file contains a folder for each member of the cluster.
   The member folder contains the associated log files and stats files for that member.

For more information about the gfsh export command,
see the [gfsh export documentation](https://gemfire.docs.pivotal.io/geode/tools_modules/gfsh/command-pages/export.html).

## <a id="deploy-app-jars"></a> Deploy an App JAR File to the Servers

You can deploy or redeploy an app JAR file to the servers in the cluster.

To do an initial deploy of an app JAR file after connecting within gfsh
using the cluster operator credentials,
run this gfsh command:

<code>deploy --jar=PATH-TO-JAR/FILENAME.jar</code>

For example,

<pre class='terminal'> gfsh>deploy --jar=working-directory/myJar.jar </pre>

To redeploy an app JAR file after connecting within gfsh
using the cluster operator role,
do the following: 		

1. Run this gfsh command to remove the existing JAR file:

    <code>undeploy --jar=PATH-TO-JAR/FILENAME.jar</code>

    For example,

    <pre class='terminal'> gfsh>undeploy --jar=current-jars/myJar.jar </pre>

1. Run this gfsh command to deploy the updated JAR file:

    <code>gfsh>deploy --jar=PATH-TO-UPDATED-JAR/FILENAME.jar</code>

    For example,

    <pre class='terminal'>gfsh>deploy --jar=newer-jars/myJar.jar</pre>

1. Run this command to restart the cluster and load the updated JAR file:

     <code> cf update-service SERVICE-INSTANCE-NAME -c '{"restart": true}' </code>

     For example,

     <pre class='terminal'>$ cf update-service my-service-instance -c '{"restart": true}'</code>

