---
title: "Disaster Recovery (DR)"
linkTitle: "Disaster Recovery (DR)"
keywords: cloud, backup, restore, snapshot, DR, migration
hidesections: true
---

This section describes Disaster Recovery (DR) options between *multiple* Kubernetes clusters when using Portworx.

There are 2 primary options for this:

1. Nodes are in the *same* region/datacenter
2. Nodes are across *different* regions/datacenters

Choosing between these options depends on how your cluster nodes are laid out. Below section helps you make that choice.

## 1. Nodes are in the same region/datacenter

![Portworx metro overview](/img/px-metro-overview.png)

### When

You should use this option when:

* Nodes in all your Kubernetes clusters are in the same region or datacenter. They could reside in separate zones or racks.
* The network latency between the nodes is low. Low here is defined as network latencies lower than 500ms.

### What

The option has the following characteristics:

* A single Portworx cluster that streches across multiple Kubernetes clusters.
* Portworx installation on all clusters use a common external key-value store (e.g etcd).
* Volumes are automatically replicated across the Kubernetes clusters as they shared the same Portworx fabric.

### How

Click on the section below for instructions on how to setup this option. 

{{< widelink url="/portworx-install-with-kubernetes/disaster-recovery/px-metro" >}}How to setup Synchronous DR{{</widelink>}}

## 2. Nodes are across different regions/datacenters

![Portworx Scheduled migration overview](/img/scheduled-migration-overview.png)

### When

You should use this option when:

* Nodes in all your Kubernetes clusters are in the different regions or datacenter.
* The network latency between the nodes is high.

### What

The option has the following characteristics:

* A separate Portworx cluster installation for each Kubernetes clusters.
* Portworx installations on each cluster can use their own key-value store (e.g etcd).
* Users create scheduled migrations of application and volumes between 2 clusters that are paired.

### How

Click on the section below to instructions on how to setup this option.

{{< widelink url="/portworx-install-with-kubernetes/disaster-recovery/px-metro" >}}How to setup Asynchronous DR{{</widelink>}}
