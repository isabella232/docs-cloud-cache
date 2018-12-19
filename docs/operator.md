---
title: Pivotal Cloud Cache Operator Guide
owner: Cloud Cache Engineers
---


This document describes how a Pivotal Cloud Foundry (PCF) operator can install, configure, and maintain Pivotal Cloud Cache (PCC).

## <a id="requirements"></a> Requirements for Pivotal Cloud Cache

#### Service Network

You must have access to a Service Network in order to install PCC. 

## <a id="install"></a> Installing and Configuring Pivotal Cloud Cache

With an Ops Manager role (detailed in [Understand Roles in Ops Manager](https://docs.pivotal.io/pivotalcf/opsguide/config-rbac.html#about))
that has the proper permissions to install and configure,
follow these steps to install PCC on PCF:

1. Download the tile from the [Pivotal Network](https://network.pivotal.io/products/cloud-cache).
1. Click **Import a Product** to import the tile into Ops Manager.
1. Click the **+** symbol next to the uploaded product description.
1. Click on the Cloud Cache tile.
1. Complete all the configuration steps in the [Configure Tile Properties](#settings-config) section below.
1. Return to the Ops Manager Installation Dashboard and click **Apply Changes** to complete the installation of the PCC tile.  

### <a id="settings-config"></a> Configure Tile Properties

Configure the sections listed on the left side of the page. 

![Tile Configuration Sections](tile-configuration-sections.png)

After you complete a section, a green check mark appears next to the section name.
Each section name must show this green check mark before you can complete your installation.

* [Assign AZs and Networks](#azs)
* [Settings](#settings)
* [Service Plans](#plan-config), including the Dev Plan
* [Syslog](#syslog)
* [Errands](#errands)
* [Resource Config](#resource-config)
* [Stemcell](#stemcell)

####<a name='azs'></a> Assign Availability Zones and Networks

To select AZs and networks for VMs used by PCC, do the following:

1. Click **Assign AZs and Networks**.

1. Configure the fields on the **Assign AZs and Networks** pane as follows:
   <table>
    <tr><th>Field</th><th>Instructions</th></tr>
    <tr><td><strong>Place singleton jobs in</strong></td>
        <td>Select the region that you want for singleton VMs.</td></tr>
    <tr><td><strong>Balance other jobs in</strong></td>
        <td>Select the AZ(s) you want to use for distributing other GemFire VMs.
            Pivotal recommends selecting all of them.</td></tr>
    <tr><td><strong>Network</strong></td>
        <td>Select your PAS (or Elastic Runtime) network.</td>
    <tr><td><strong>Service Network</strong></td>
        <td>Select the network to be used for GemFire VMs.</td> 
  </table>

1. Click **Save**.

####<a name='settings'></a> Settings: Smoke Tests

The smoke-tests errand that runs after tile installation.
The errand verifies that your installation was successful.
By default, the `smoke-test` errand runs on the `system` org and the `p-cloudcache-smoke-test` space.

<p class="note"><strong>Note</strong>: Smoke tests will fail unless you enable global default application security groups (ASGs). You can enable global default ASGs by binding the ASG to the <code>system</code> org without specifying a space. To enable global default ASGs, use <code>cf bind-running-security-group</code>.</p>

To select which plan you want to use for smoke tests, do the following:

1. Click **Settings**.

2. Select a plan to use when the `smoke-tests` errand runs.

     Ensure the selected plan is enabled and configured.
     For information about configuring plans, see [Configure Service Plans](#plan-config) below.
     If the selected plan is not enabled, the `smoke-tests` errand fails.

     Pivotal recommends that you use the smallest four-server plan for smoke tests.
     Because smoke tests create and later destroy this plan, using a very small plan reduces installation time.
     ![Configuring Smoke Tests Plan](select-smoke-test-plan.png)

3. Click **Save**.

#### Settings: Allow Outbound Internet Access

By default, outbound internet access is not allowed from service instances. 

If BOSH is configured to use an external blob store, you need allow outbound internet access from service instances.
Log forwarding and backups, which require external endpoints, might also require internet access.

To allow outbound internet access from service instance, do the following:

1. Click **Settings**.

2. Select **Allow outbound internet access from service instances (IaaS-dependent)**.

    ![Outbound Internet Access](allow-outbound-access.png)

    <p class="note"><strong>Note</strong>: Outbound network traffic rules also depend on your IaaS settings. 
      Consult your network or IaaS administrator to ensure that your IaaS allows outbound traffic to the external networks you need.</p>


3. Click **Save**.


####<a name='syslog'></a> Syslog

By default, syslog forwarding is not enabled in PCC.
However, PCC supports forwarding syslog to an external log management service (for example, Papertrail, Splunk, or your custom enterprise log sink).
The broker logs are useful for debugging problems creating, updating, and binding service instances.

To enable remote syslog for the service broker, do the following:

1. Click **Syslog**.
   ![Configuring Log Management Service](syslog-config.png)
2. Configure the fields on the **Syslog** pane as follows:
   <table>
    <tr><th>Field</th><th>Instructions</th></tr>
    <tr><td><strong>Enable Remote Syslog</strong></td>
        <td>Select to enable.</td></tr>
    <tr><td><strong></strong>External Syslog Address</td>
        <td>Enter the address or host of the syslog server for sending logs, for example, `logs.example.com`.</td></tr>
    <tr><td><strong></strong>External Syslog Port</td>
        <td>Enter the port of the syslog server for sending logs, for example,` 29279`.</td></tr>
    <tr><td><strong>Enable TLS for Syslog</strong></td>
    <td>Select to enable secure log transmission through TLS. Without this, remote syslog sends unencrypted logs. We recommend enabling TLS, as most syslog endpoints such as Papertrail and Logsearch require TLS.</td></tr>
    <tr><td><strong>Permitted Peer for TLS Communication. This is required if TLS is enabled.</strong></td>
        <td>If there are several peer servers that can respond to remote syslog connections, then
            provide a regex, such as `*.example.com`.</td></tr>
    <tr><td><strong>CA Certificate for TLS Communication</td>
        <td>If the server certificate is not signed by a known authority, for example, an internal syslog server,
            provide the CA certificate of the log management service endpoint.</td></tr>
    <tr><td><strong>Send service instance logs to external</td>
        <td>By default, only the broker logs are forwarded to your configured log management service.
            If you want to forward server and locator logs from all service instances, select this.
            This lets you monitor the health of the clusters, although it generates a large volume of logs.<br><br>
            If you don't enable this, you get only the broker logs which include information about service instance creation, 
            but not about on-going cluster health.</td></tr>
  </table>

3. Click **Save**.

<a name="plan-config"></a>
#### Configure Service Plans

You can configure five individual plans for your developers. Select the **Plan 1** through **Plan 5** tabs to configure each of them.

![Configuring a Plan](configure-plan.png)

The **Plan Enabled** option is selected by default. If you do not want to add this plan to the CF service catalog, select **Plan Disabled**. You must enable at least one plan.

The **Plan Name** text field allows you to customize the name of the plan. This plan name is displayed to developers when they view the service in the Marketplace.

The **Plan Description** text field allows you to supply a plan description. The description is displayed to developers when they view the service in the Marketplace.

The **Enable metrics for service instances** checkbox enables metrics for service instances created using the plan.
Once enabled, the metrics are sent to the Loggregator Firehose.

The **CF Service Access** drop-down menu gives you the option to display or not display the service plan in the Marketplace.
**Enable Service Access** displays the service plan the Marketplace.
**Disable Service Access** makes the plan unavailable in the Marketplace. If you choose this option, you cannot make the plan available at a later time. 
**Leave Service Access Unchanged** makes the plan unavailable in the Marketplace by default, but allows you to make it available at a later time.

The **Service Instance Quota** sets the maximum number of PCC clusters that can exist simultaneously.

When developers create or update a service instance, they can specify the number of servers in the cluster. The **Maximum servers per cluster** field allows operators to set an upper bound on the number of servers developers can request. If developers do not explicitly specify the number of servers in a service instance, a new cluster has the number of servers specified in the **Default Number of Servers** field.

The **Availability zones for service instances** setting determines which AZs are used for a particular cluster. The members of a cluster are distributed evenly across AZs.

<p class="note warning"><strong> WARNING!</strong> After you've selected AZs for your service network, you cannot add additional AZs; doing so causes existing service instances to lose data on update. </p>

The remaining fields control the VM type and persistent disk type for servers and locators. The total size of the cache is directly related to the number of servers and the amount of memory of the selected server VM type. We recommend the following configuration:

* For the **VM type for the Locator VMs** field, select a VM that has at least 2 CPUs, 1&nbsp;GB of RAM and 4&nbsp;GB of disk space.
* For the **Persistent disk type for the Locator VMs** field, select 10&nbsp;GB or higher.
* For the **VM type for the Server VMs** field, select a VM that has at least 2 CPUs, 4&nbsp;GB of RAM and 8&nbsp;GB of disk space.
* For the **Persistent disk type for the server VMs** field, select 10 GB or higher.

When you finish configuring the plan, click **Save** to save your configuration options.

####<a id='dev-plan'></a> Configure a Dev Plan

A Dev Plan is a type of service plan.
Use a Dev Plan for development and testing.
The plan provides a single locator and server,
which are colocated within a single VM. 

The page for configuring a Dev Plan is similar to the page for configuring 
other service plans.
To configure the Dev Plan, input information in the fields and make selections from the options on the **Plan for test development** page. 

![Dev Plan Configuration Sections](dev-plan.png)

If you have enabled post-deploy scripts in your BOSH Director,
a region is automatically created.
To confirm that post-deploy scripts are enabled, navigate to the **Director Config** pane of Ops Manger Director and
verify that **Enable Post Deploy Scripts** is selected.

![Enable post deploy scripts](ops-man-post-deploy.png)

####<a name='errands'></a> Errands

By default, post-deploy and pre-delete errands always run. 
Pivotal recommends keeping these defaults.
However, if necessary, you can change these defaults as follows.

For general information about errands in PCF, see [Managing Errands in Ops Manager](https://docs.pivotal.io/pivotalcf/customizing/managing_errands.html)

1. Click **Errands**.

2. Change the setting for the errands.

3. Click **Save**.

####<a name='stemcell'></a> Stemcell

Ensure you import the correct type of stemcell indicated on this tab.

You can download the latest available stemcells from the [Pivotal Network](https://network.pivotal.io/products/stemcells/).

PCC 1.4 can use either v2.1.x or v2.0.x of 
Pivotal Application Service (PAS)
and PCF Operations Manager (Ops Manager).
As of v2.1.0, manage stemcells in the Ops Manager dashboard.
Stemcells do not appear in this tile configuration.

## <a id="set_quotas"></a> Setting Service Instance Quotas

<%= partial '../../p-cloud-cache/odb/set_quotas' %>

## <a id="monitoring"></a> Monitoring Pivotal Cloud Cache Service Instances

PCC clusters and brokers emit service metrics. 
You can use any tool that has a corresponding Cloud Foundry nozzle to read and monitor these metrics in real time.

In the descriptions of the metrics,
KPI stands for Key Performance Indicator.

### <a id="service-instance-metrics"></a> Service Instance Metrics

#### <a id="service-instance-MemberCount"></a> Member Count

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>serviceinstance.MemberCount</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>Returns the number of members in the distributed system.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Every second</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>count</td>
   </tr>
   <tr>
      <th>Warning Threshold</th>
      <td>less than the manifest member count</td>
   </tr>
   <tr>
      <th>Suggested Actions</th>
      <td>This depends on the expected member count, which is available in the BOSH manifest. If the number expected is different from the number emitted, this is a critical situation that may lead to data loss, and the reasons for node failure should be investigated by examining the service logs.</td>
   </tr>
   <tr>
      <th>Why a KPI?</th>
      <td>Member loss due to any reason can potentially cause data loss.</td>
   </tr>
</table>

#### <a id="service-instance-TotalHeapSize"></a> Total Available Heap Size

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>serviceinstance.TotalHeapSize</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>Returns the total available heap, in megabytes, across all instance members.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Every second</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>pulse</td>
   </tr>
   <tr>
      <th>Why a KPI?</th>
      <td>If the total heap size and used heap size are too close, the system might see thrashing due to GC activity. This increases latency.</td>
   </tr>
</table>

#### <a id="service-instance-UsedHeapSize"></a> Total Used Heap Size

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>serviceinstance.UsedHeapSize</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>Returns the total heap used across all instance members, in megabytes.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Every second</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>pulse</td>
   </tr>
   <tr>
      <th>Why a KPI?</th>
      <td>If the total heap size and used heap size are too close, the system might see thrashing due to GC activity. This increases latency.</td>
   </tr>
</table>

#### <a id="service-instance-UnusedHeapSizePercentage"></a> Total Available Heap Size as a Percentage

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>serviceinstance.UnusedHeapSizePercentage</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>Returns the proportion of total available heap across all instance members, expressed as a percentage.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>percent</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Every second</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>compount metric</td>
   </tr>
   <tr>
      <th>Warning Threshold</th>
      <td>40%</td>
   </tr>
   <tr>
      <th>Critical Threshold</th>
      <td>10%</td>
   </tr>
   <tr>
      <th>Suggested Actions</th>
      <td>If this is a spike due to eviction catching up with insert frequency, then customers need to keep a close watch that it should not hit the RED marker. If there is no eviction, then horizontal scaling is suggested.</td>
   </tr>
   <tr>
      <th>Why a KPI?</th>
      <td>If the total heap size and used heap size are too close, the system might see thrashing due to GC activity. This increases latency.</td>
   </tr>
</table>

### <a id="per-member-metrics"></a> Per Member Metrics

#### <a id="member-UsedMemoryPercentage"></a> Memory Used as a Percentage

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>member.UsedMemoryPercentage</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>RAM being consumed.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>percent</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Average over last 10 minutes</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>average</td>
   </tr>
   <tr>
      <th>Warning Threshold</th>
      <td>75%</td>
   </tr>
   <tr>
      <th>Critical Threshold</th>
      <td>85%</td>
   </tr>
</table>

#### <a id="member-GC-count"></a> Count of Java Garbage Collections

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>member.GarbageCollectionCount</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>The number of times that garbage has been collected.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Sum over last 10 minutes</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>count</td>
   </tr>
   <tr>
      <th>Warning Threshold</th>
      <td>Dependent on the IaaS and app use case.</td>
   </tr>
   <tr>
      <th>Critical Threshold</th>
      <td>Dependent on the IaaS and app use case.</td>
   </tr>
   <tr>
      <th>Suggested Actions</th>
      <td>Check the number of queries run against the system, which increases the deserialization of objects and increases garbage.</td>
   </tr>
   <tr>
      <th>Why a KPI?</th>
      <td>If the frequency of garbage collection is high, the system might see high CPU usage, which causes delays in the cluster.</td>
   </tr>
</table>

#### <a id="member-cpuusage"></a> CPU Utilization Percentage

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>member.HostCpuUsage</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>This member's process CPU utilization, expressed as a percentage.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>percent</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Average over last 10 minutes</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>average</td>
   </tr>
   <tr>
      <th>Warning Threshold</th>
      <td>85%</td>
   </tr>
   <tr>
      <th>Critical Threshold</th>
      <td>95%</td>
   </tr>
   <tr>
      <th>Suggested Actions</th>
      <td>If this is not happening with high GC activity, the system is reaching its limits. Horizontal scaling might help.</td>
   </tr>
   <tr>
      <th>Why a KPI?</th>
      <td>High CPU usage causes delayed responses and can also make the member non-responsive. This can cause the member to be kicked out of the cluster, potentially leading to data loss.</td>
   </tr>
</table>

#### <a id="member-get-avg-latency"></a> Average Latency of Get Operations

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>member.GetsAvgLatency</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>The average latency of cache get operations, in nanoseconds.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Average over last 10 minutes</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>average</td>
   </tr>
   <tr>
      <th>Warning Threshold</th>
      <td>Dependent on the IaaS and app use case.</td>
   </tr>
   <tr>
      <th>Critical Threshold</th>
      <td>Dependent on the IaaS and app use case.</td>
   </tr>
   <tr>
      <th>Suggested Actions</th>
      <td>If this is not happening with high GC activity, the system is reaching its limit. Horizontal scaling might help.</td>
   </tr>
   <tr>
      <th>Why a KPI?</th>
      <td>It is a good indicator of the overall responsiveness of the system. If this number is high, the service administrator should diagnose the root cause.</td>
   </tr>
</table>

#### <a id="member-put-avg-latency"></a> Average Latency of Put Operations

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>member.PutsAvgLatency</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>The average latency of cache put operations, in nanoseconds.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Average over last 10 minutes</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>average</td>
   </tr>
   <tr>
      <th>Warning Threshold</th>
      <td>Dependent on the IaaS and app use case.</td>
   </tr>
   <tr>
      <th>Critical Threshold</th>
      <td>Dependent on the IaaS and app use case.</td>
   </tr>
   <tr>
      <th>Suggested Actions</th>
      <td>If this is not happening with high GC activity, the system is reaching its limit. Horizontal scaling might help.</td>
   </tr>
   <tr>
      <th>Why a KPI?</th>
      <td>It is a good indicator of the overall responsiveness of the system. If this number is high, the service administrator should diagnose the root cause.</td>
   </tr>
</table>

#### <a id="member-jvm-pauses"></a> JVM pauses

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>member.JVMPauses</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>The quantity of JVM pauses.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Sum over 2 seconds</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>count</td>
   </tr>
   <tr>
      <th>Warning Threshold</th>
      <td>Dependent on the IaaS and app use case.</td>
   </tr>
   <tr>
      <th>Critical Threshold</th>
      <td>Dependent on the IaaS and app use case.</td>
   </tr>
   <tr>
      <th>Suggested Actions</th>
      <td>Check the cached object size; if it is greater than 1 MB, you may be hitting the limitation on JVM to garbage collect this object. Otherwise, you may be hitting the utilization limit on the cluster, and will need to scale up to add more memory to the cluster.</td>
   </tr>
   <tr>
      <th>Why a KPI?</th>
      <td>Due to a JVM pause, the member stops responding to "are-you-alive" messages, which may cause this member to be kicked out of the cluster.</td>
   </tr>
</table>

#### <a id="member-file-descriptor-limit"></a> File Descriptor Limit

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>member.FileDescriptorLimit</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>The maximum number of open file descriptors allowed for the member’s host operating system.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Every second</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>pulse</td>
   </tr>
   <tr>
      <th>Why a KPI?</th>
      <td>If the number of open file descriptors exceeds number available, it causes the member to stop responding and crash.</td>
   </tr>
</table>

#### <a id="member-total-file-descriptor-open"></a> Open File Descriptors

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>member.TotalFileDescriptorOpen</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>The current number of open file descriptors.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Every second</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>pulse</td>
   </tr>
   <tr>
      <th>Why a KPI?</th>
      <td>If the number of open file descriptors exceeds number available, it causes the member to stop responding and crash.</td>
   </tr>
</table>

#### <a id="member-file-descriptor-remaining"></a> Quantity of Remaining File Descriptors

<table>
   <tr><th colspan="2" style="text-align: center;"><br> <code>member.FileDescriptorRemaining</code><br><br></th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>The number of available file descriptors.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Suggested measurement</th>
      <td>Every second</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>compound metric</td>
   </tr>
   <tr>
      <th>Warning Threshold</th>
      <td>1000</td>
   </tr>
   <tr>
      <th>Critical Threshold</th>
      <td>100</td>
   </tr>
   <tr>
      <th>Suggested Actions</th>
      <td>Scale horizontally to increase capacity.</td>
   </tr>
   <tr>
      <th>Why a KPI?</th>
      <td>If the number of open file descriptors exceeds number available, it causes the member to stop responding and crash.</td>
   </tr>
</table>


### <a id="gateway-sender-metrics"></a> Gateway Sender and Gateway Receiver Metrics

These are metrics emitted through the CF Nozzle for gateway senders and gateway receivers.

#### <a id="gs-event-queue-size"></a> Queue Size for the Gateway Sender

<table>
   <tr><th colspan="2" style="text-align: center;"><code>gatewaySender.&lt;sender-id&gt;.EventQueueSize</code> </th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>The current size of the gateway sender queue. </td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>count</td>
   </tr>
</table>

#### <a id="gs-event-received-rate"></a> Events Received at the Gateway Sender

<table>
   <tr><th colspan="2" style="text-align: center;"><code>gatewaySender.&lt;sender-id&gt;.EventsReceivedRate</code> </th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>A count of the events coming from the region to which the gateway sender is attached. It is the count since the last time the metric was checked. The first time it is checked, the count is of the number of events since the gateway sender was created. </td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>count</td>
   </tr>
</table>

#### <a id="gs-events-queued-rate"></a> Events Queued by the Gateway Sender

<table>
   <tr><th colspan="2" style="text-align: center;"><code>gatewaySender.&lt;sender-id&gt;.EventsQueuedRate</code> </th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>A count of the events queued on the gateway sender from the region. This quantity of events might be lower than the quantity of events received, as not all received events are queued. It is a count since the last time the metric was checked. The first time it is checked, the count is of the number of events since the gateway sender was created.</td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>count</td>
   </tr>
</table>

#### <a id="gr-event-received-rate"></a> Events Received by the Gateway Receiver

<table>
   <tr><th colspan="2" style="text-align: center;"><code>gatewayReceiver.EventsReceivedRate</code> </th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>A count of the events received from the gateway sender which will be applied to the region on the gateway receiver's site. It is the count since the last time the metric was checked. The first time it is checked, the count is of the number of events since the gateway receiver was created. </td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>count</td>
   </tr>
</table>

### <a id="disk-metrics"></a> Disk Metrics

These are metrics emitted through the CF Nozzle for disks.

#### <a id="disk-writes-avg-latency"></a> Average Latency of Disk Writes

<table>
   <tr><th colspan="2" style="text-align: center;"><code>diskstore.DiskWritesAvgLatency</code> </th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>The average latency of disk writes in nanoseconds. </td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>time in nanoseconds</td>
   </tr>
</table>

#### <a id="total-disk-space"></a> Quantity of Bytes on Disk

<table>
   <tr><th colspan="2" style="text-align: center;"><code>diskstore.TotalSpace</code> </th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>The total number of bytes on the attached disk. </td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>count</td>
   </tr>
</table>

#### <a id="total-available-disk-space"></a> Quantity of Available Bytes on Disk

<table>
   <tr><th colspan="2" style="text-align: center;"><code>diskstore.UseableSpace</code> </th></tr>
   <tr>
      <th width="25%">Description</th>
      <td>The total number of bytes of available space on the attached disk. </td>
   </tr>
   <tr>
      <th>Metric Type</th>
      <td>number</td>
   </tr>
   <tr>
      <th>Measurement Type</th>
      <td>count</td>
   </tr>
</table>

### <a id="bosh-errand-report"></a> Total Memory Consumption

The BOSH `mem-check` errand calculates and outputs
the quantity of memory used across all PCC service instances.
This errand helps PCF operators monitor resource costs,
which are based on memory usage.

From the director, run a bosh command of the form:

``` pre
bosh -d <service broker name> run-errand mem-check
```

With this command:

``` pre
bosh -d cloudcache-service-broker run-errand mem-check
```

Here is an anonymized portion of example output from the `mem-check` errand
for a two cluster deployment:

``` pre
           Analyzing deployment xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx1...
           JVM heap usage for service instance xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx1
           Used Total = 1204 MB
           Max Total = 3201 MB

           Analyzing deployment xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2...
           JVM heap usage for service instance xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx2
           Used Total = 986 MB
           Max Total = 3201 MB

           JVM heap usage for all clusters everywhere:
           Used Global Total = 2390 MB
           Max Global Total = 6402 MB
```

### <a id="monitoring-prometheus"></a> Monitoring PCC Service Instances with Prometheus

Prometheus is one of various tools you can use to monitor services instances. It is a monitoring and alerting toolkit that allows for metric scraping.
You can use the [Firehose exporter](https://github.com/cloudfoundry-community/firehose_exporter) to export all the metrics from the Firehose,
which you can then graph with [Grafana](https://github.com/grafana/grafana) to monitor your PCC cluster.

Follow the instructions [here](https://github.com/pivotal-cf/prometheus-on-PCF) to deploy Prometheus alongside your PCF cluster.

Prometheus can be deployed on any IaaS.
You need to verify that the Firehose exporter job can talk to your UAA VM.
This might involve opening up firewall rules or enabling your VM to allow outgoing traffic.

![Grafana Example](grafana.png)

You can run queries on, and build a custom dashboard of, specific metrics that are important to you.

## <a id="upgrade"></a> Upgrading Pivotal Cloud Cache

Follow the steps below to upgrade PCC:

1. Download the new version of the tile from the Pivotal Network.
1. Upload the product to Ops Manager.
1. Click **Add** next to the uploaded product.
1. Click on the Cloud Cache tile and review your configuration options.
1. Click **Apply Changes**.

## <a id="update-plans"></a> Updating Pivotal Cloud Cache Plans

Follow the steps below to update plans in Ops Manager.

1. Click on the Cloud Cache tile.
1. Click on the plan you want to update under the **Information** section.
1. Edit the fields with the changes you want to make to the plan.
1. Click **Save** button on the bottom of the page.
1. Click on the **PCF Ops Manager** to navigate to the **Installation Dashboard**.
1. Click **Apply Changes**.

Plan changes are not applied to existing services instances until you run the `upgrade-all-service-instances` BOSH errand. You must use the BOSH CLI to run this errand. Until you run this errand, developers cannot update service instances.

Changes to fields that can be overridden by optional parameters, for example  `num_servers` or `new_size_percentage`, change the default value of these instance properties, but do not affect existing service instances. 

If you change the allowed limits of an optional parameter, for example the maximum number of servers per cluster, existing service instances in violation of the new limits are not modified.

When existing instances are upgraded, all plan changes are applied to them.

## <a id="uninstall"></a> Uninstalling Pivotal Cloud Cache

To uninstall PCC, follow the steps from below from the **Installation Dashboard**:

1. Click the trash can icon in the bottom-right-hand corner of the tile.
1. Click **Apply Changes**.

## <a id="troubleshooting"></a> Troubleshooting

###<a id='view-statistics'></a> View Statistics Files

You can visualize the performance of your cluster by downloading the statistics files from your servers.
These files are located in the persistent store on each VM.
To copy these files to your workstation, run the following command:

    `bosh2 -e BOSH-ENVIRONMENT -d DEPLOYMENT-NAME scp server/0:/var/vcap/store/gemfire-server/statistics.gfs /tmp`

See the Pivotal GemFire [Installing and Running VSD](http://gemfire.docs.pivotal.io/gemfire/tools_modules/vsd/running_vsd.html) topic for information about loading the statistics files into Pivotal GemFire VSD.

###<a id='smoke-test-failures'></a> Smoke Test Failures

#### Error: "Creating p-cloudcache SERVICE-NAME failed"
The smoke tests could not create an instance of GemFire. To troubleshoot
why the deployment failed, use the cf CLI to create a new service instance using the same plan and download the logs of the service deployment from BOSH.

#### Error: "Deleting SERVICE-NAME failed"
The smoke test attempted to clean up a service instance it created and failed to delete the service using the `cf delete-service` command. To troubleshoot this issue, run BOSH `logs` to view the logs on the broker or the service instance to see why the deletion may have failed.

#### Error: Cannot connect to the cluster SERVICE-NAME
The smoke test was unable to connect to the cluster.

To troubleshoot the issue, review the logs of your load balancer, and
review the logs of your CF Router to ensure the route to your PCC
cluster is properly registered.

You also can create a service instance and try to connect to it using the
gfsh CLI. This requires creating a service key.

#### Error: "Could not perform create/put on Cloud Cache cluster"
The smoke test was unable to write data to the cluster. The user may not have
permissions to create a region or write data.

#### Error: "Could not retrieve value from Cloud Cache cluster"
The smoke test was unable to read back the data it wrote. Data loss can happen
if a cluster member improperly stops and starts again or if the member machine
crashes and is resurrected by BOSH. Run BOSH `logs` to view the logs on the
broker to see if there were any interruptions to the cluster by a service
update.

###<a id='general-connectivity'></a> General Connectivity
#### Client-to-Server Communication

PCC Clients communicate to PCC servers on port 40404 and with
locators on port 55221.
Both of these ports must be reachable from the PAS (or Elastic Runtime) network to service the network.

#### Membership Port Range
PCC servers and locators communicate with each other using UDP and TCP. The current port range for this communication is `49152-65535`.

If you have a firewall between VMs, ensure this port range is open.
