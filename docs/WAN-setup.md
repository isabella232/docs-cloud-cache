---
title: Set Up WAN-Separated Service Instances
---

Two service instances may form a single distributed system across a WAN.
The interaction of the two service instances may follow one of 
the patterns described within the section on
[Design Patterns](design-patterns.html).

Call the two service instances A and B.
The GemFire cluster within each service instance uses an identifier
called a `distributed_system_id`.
This example assigns `distributed_system_id = 1` to cluster A
and `distributed_system_id = 2` to cluster B.
GemFire gateway senders provide the communication path and construct that
propagates region operations from one cluster to another.
On the receiving end are GemFire gateway receivers. 
Creating a service instance also creates gateway receivers.

<p class="note"><strong>Note</strong>: To set up more than two service
instances across a WAN,
set up the interaction between the first two service instances A and B 
following the directions in either
<a href="./WAN-bi-setup.html">Set Up a Bidirectional System</a> or
<a href="./WAN-uni-setup.html">Set Up a Unidirectional System</a>,
as appropriate.
After that,
set up the interaction between service instance A and another
service instance (called C) following the directions in either
<a href="./WAN-addl-bi-setup.html">Set Up an Additional Bidirectional Interaction</a> or
<a href="./WAN-addl-uni-setup.html">Set Up an Additional Unidirectional Interaction</a>,
as appropriate.
</p>
