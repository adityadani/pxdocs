---
title: "Synchronous DR"
linkTitle: "Synchronous DR"
keywords: cloud, backup, restore, snapshot, DR, migration, px-metro
description: How to achieve DR across two kubernetes cluster spanning a metropolitan area network with a single stretch Portworx cluster.
---

Portworx supports synchronous DR when installed as a single stretch cluster across multiple Kubernetes clusters. The Kubernetes clusters need to span across a metropolitan area network with a maximum latency of 10ms.
This document explains how to install such a stretch Portworx cluster and achieve synchronous DR. It will demonstrate how to failover and failback applications between two Kubernetes clusters.
Here are the steps you should follow:
