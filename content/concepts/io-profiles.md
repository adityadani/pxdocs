---
title: IO profiles
description: Learn what IO profiles for Portworx volumes are.  
keywords: 
weight: 600
series: concepts
---

You can optimize the performance of your Portworx volumes by matching the type of workload you're running with a suitable IO profile. IO profiles change the how a Portworx volume interacts with the underlying storage disks to improve traffic for different workloads.

## Understand the different IO profiles

Portworx currently features the following IO profiles:

* auto
* db
* db_remote
* sequential
* random


### The auto profile
If you don't provide an IO profile, Portworx will default to `auto`. This profile attempts to automatically apply an IO profile that is most appropriate for the data patterns it sees. It does this by continuously analyzing the IO pattern of traffic in the background. 

### The db profile
Databases typically result in a large number of flush operations on the disk. Because a flush forces Portworx to wait until the data is synched on the disk, it can slow down traffic. When the `db` profile is active, Portworx batches flush operations for a quicker write response time. 

{{<info>}}
**NOTE:** 

* If there are not enough nodes online, Portworx will automatically disable this algorithm.
* The `db` profile only applies if you have a repl count of 2 or more. 
{{</info>}}

### The db_remote profile

This implements a write-back flush coalescing algorithm. This algorithm attempts to coalesce multiple `syncs` that occur within a 50ms window into a single sync. Coalesced syncs are acknowledged only after copying to all replicas. In order to do this, the algorithm requires a minimum replication (HA factor) of 2. This mode assumes all replicas do not fail (kernel panic or power loss) simultaneously in a 50 ms window. 

{{<info>}}
**NOTE:**

* If there are not enough nodes online, Portworx will automatically disable this algorithm.
* The `db_remote` profile only applies if you have a repl count of 2 or more. 
{{</info>}}

### The sequential profile

This profile optimizes the read-ahead algorithm for sequential workloads, such as backup operations. 

### The random profile

This profile records the IO pattern of recent access and optimizes the read-ahead and data layout algorithms for workloads involving short-term random patterns.

## Configure profiles

You can configure IO profiles through Kubernetes specs or at the Portworx level, depending on your needs and what operator you're running with.

For information about how to configure IO profiles for volumes on Kubernetes clusters, follow this link:
{{< widelink url="/portworx-install-with-kubernetes/storage-operations/io-profile-in-k8s/" >}}Configure IO profiles for Portworx volumes on Kubernetes{{</widelink>}}

For information about how to configure IO profiles from volumes using `pxctl`, follow this link:
{{< widelink url="/install-with-other/operate-and-maintain/performance-and-tuning/tuning" >}}Configure IO profiles from volumes using `pxctl`{{</widelink>}}