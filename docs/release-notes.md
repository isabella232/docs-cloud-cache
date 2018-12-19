---
title: Pivotal Cloud Cache Release Notes
owner: Cloud Cache Engineers
---

## <a id='v140'></a>v1.4.0

**Release Date:**  May 14, 2018

Features included in this release:

- PCC is now running Pivotal Gemfire 9.3.0.
- PCC now supports connecting more than two clusters via WAN.
This facilitates implementing design patterns such as Hub-and-Spoke.
- The new `mem-check` BOSH errand generates a report containing
the current data consumption by PCC service instances and total JVM sizes.
- PCC now supports Pivotal Application Service (PAS) 2.1.
- PCC selects minimum default values for VM types on PAS 2.1 and 2.0.

Known issues in this release:

- If you upgrade to PCC v1.3 as part of the process of upgrading to this
1.4 release, and you created service keys on PCC before you installed v1.3:
delete and recreate the service keys so that users are properly
assigned roles for authentication and authorization within the cluster.
Then, rebind all your apps.
For information about how to perform these tasks,
see [Delete a Service Key](https://docs.pivotal.io/pivotalcf/2-0/devguide/services/service-keys.html),
[Create Service Keys](accessing-instance.html#create-service-key),
and [Bind an App to a Service Instance](using-pcc.html#bind-service).

- A Pulse topology diagram might not be accurate and might show more members than are actually in the cluster. However, the numerical value displayed on the top bar is accurate.

## <a id="16x"></a>Release Notes for Earlier Versions

For v1.3.x versions of PCC,
see [Release Notes](http://docs.pivotal.io/p-cloud-cache/1-3/release-notes.html) in the v1.3 version of this documentation.

For v1.2.x versions of PCC,
see [Release Notes](http://docs.pivotal.io/p-cloud-cache/1-2/release-notes.html) in the v1.2 version of this documentation.

For v1.1.x versions of PCC,
see [Release Notes](http://docs.pivotal.io/p-cloud-cache/1-1/release-notes.html) in the v1.1 version of this documentation.

For v1.0.x versions of PCC,
see [Release Notes](http://docs.pivotal.io/p-cloud-cache/1-0/release-notes.html) in the v1.0 version of this documentation.
