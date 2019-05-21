---
title: "3. Failover an application"
weight: 3
keywords: cloud, backup, restore, snapshot, DR, migration, px-motion
description: Find out how to failover an application from one Kubernetes cluster to another.
---

In case of a disaster, where one of your Kubernetes clusters goes down and is inaccessible, you would want to failover the applications running on it to an operational Kubernetes cluster.

For this section, we will refer to,

* **Source Cluster** as the Kubernetes cluster which is down and where your applications were originally running. (In this example: `cluster_domain: us-east-1a`)
* **Destination Cluster** as the Kubernetes cluster where the applications will be failed over. (In this example: `cluster_domain: us-east-1b`)

In order to failover the application, you need to instruct Stork and Portworx that one of your Kubernetes clusters is down and inactive.

### Deactivate failed Cluster Domain

In order to initiate a failover, we need to first mark the source cluster as inactive.

#### Using storkctl

To deactivate a cluster domain run the below command on the destination cluster where Portworx is still running.

```text
storkctl deactivate clusterdomain us-east-1a --wait
```

By providing the `--wait` argument, the above command will wait until the deactivate command succeeds or timeouts. Every 5 seconds, the command will spit out the current state. A sample output of the above command looks like this:

```text
time="2019-05-20T18:45:41Z" Deactivating cluster domain us-east-1a. Current State: InProgress
time="2019-05-20T18:45:46Z" Deactivating cluster domain us-east-1a. Current State: InProgress
time="2019-05-20T18:45:51Z" Deactivating cluster domain us-east-1a. Current State: Failed. Reason: Cannot connect to endpoint.
time="2019-05-20T18:45:56Z" Deactivating cluster domain us-east-1a. Current State: InProgress
time="2019-05-20T18:46:01Z" Deactivating cluster domain us-east-1a. Current State: Success
```

Finally to check the current statuses of all the clusterdomains in your cluster you can run the following command:

```text
storkctl get clusterdomainsstatus
```

When a domain gets successfully deactivated the above command will show the following output:

```
NAME            ACTIVE         INACTIVE       CREATED
px-dr-cluster   [us-east-1b]   [us-east-1a]   09 Apr 19 17:12 PDT
```

You can see that the cluster domain `us-east-1a` is now **Inactive**


### Start the application on the destination cluster

In step 2., we migrated the applications to the destination cluster but the replica count was set to 0 for all the deployments and statefulsets, so that they do not run.
You can now scale the applications by setting the replica counts to the desired value.

Each application spec will have the following annotation `stork.openstorage.org/migrationReplicas` indicating what was the replica count for it on the source cluster.

Once the replica count is updated, the application would start running, and the failover will be completed.

You can run the following `storkctl` command to do that:

```text
storkctl activate migration -n migrationnamespace
```

`storkctl` will look for that annotation and scale it to the correct number automatically.

Let's make sure our application is up and running. List the pods with:

```text
kubectl get pods -n migrationnamespace
```

```output
NAME                     READY     STATUS    RESTARTS   AGE
mysql-5857989b5d-48mwf   1/1       Running   0          3m
```
