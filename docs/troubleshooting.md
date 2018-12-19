---
title:  Troubleshooting
---

Here are problems and fixes related to using PCC.

- **Problem:**  An error occurs when creating a service instance or
when running a smoke test.
The service creation issues an error message that starts with

    ```
    Instance provisioning failed: There was a problem completing your request.
    ```

    GemFire server logs at
    `/var/vcap/sys/log/gemfire-server/gemfire/server-<N>.log`
    will contain a disk-access error with the string

    ``` 
    A DiskAccessException has occurred
    ``` 

    and a stack trace similar to this one that begins with

    ``` 
    org.apache.geode.cache.persistence.ConflictingPersistentDataException
        at org.apache.geode.internal.cache.persistence.PersistenceAdvisorImpl.checkMyStateOnMembers(PersistenceAdvisorImpl.java:743)
        at org.apache.geode.internal.cache.persistence.PersistenceAdvisorImpl.getInitialImageAdvice(PersistenceAdvisorImpl.java:819)
        at org.apache.geode.internal.cache.persistence.CreatePersistentRegionProcessor.getInitialImageAdvice(CreatePersistentRegionProcessor.java:52)
        at org.apache.geode.internal.cache.DistributedRegion.getInitialImageAndRecovery(DistributedRegion.java:1178)
        at org.apache.geode.internal.cache.DistributedRegion.initialize(DistributedRegion.java:1059)
        at org.apache.geode.internal.cache.GemFireCacheImpl.createVMRegion(GemFireCacheImpl.java:3089)
    ``` 

    **Cause of the Problem:** The PCC VMs are underprovisioned;
    the quantity of disk space is too small.

    **Solution:** Use Ops Manager to provision VMs of at least
    the minimum size.
    See [Configure Service Plans](operator.html#plan-config)
    for minimum-size details.

