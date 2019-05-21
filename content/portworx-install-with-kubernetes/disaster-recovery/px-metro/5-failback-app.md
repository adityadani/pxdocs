---
title: "4. Failback an application"
weight: 4
keywords: cloud, backup, restore, snapshot, DR, migration, px-motion
description: Find out how to failback an application from the backup Kubernetes cluster to the original one.
---

Once your unhealthy Kubernetes cluster is back up and running, the Portworx nodes in that cluster will not immediately rejoin the cluster. They will stay in
`Out of Quorum` state until you explicitly **Activate** this Cluster Domain.

After this domain is marked as **Active** you can failback the applications if you want.

For this section, we will refer to,

* **Source Cluster** as the Kubernetes cluster which is back online and where your applications need to failback to. (In this example: `cluster_domain: us-east-1a`)
* **Destination Cluster** as the Kubernetes cluster where the applications will be failed over. (In this example: `cluster_domain: us-east-1b`)

### Reactivate inactive Cluster Domain

In order to initiate a failback, we need to first mark the source cluster as active.

#### Using storkctl

In order to activate the source cluster run the following `storkctl` command from the Kubernetes cluster which is **Active**: 

```text
storkctl activate clusterdomain us-east-1a --wait
```

By providing the `--wait` argument, the above command will wait until the activate command succeeds or timeouts. Every 5 seconds, the command will spit out the current state. A sample output of the above command looks like this:

```text
time="2019-05-20T18:45:41Z" Activating cluster domain us-east-1a. Current State: InProgress
time="2019-05-20T18:45:46Z" Activating cluster domain us-east-1a. Current State: InProgress
time="2019-05-20T18:45:51Z" Activating cluster domain us-east-1a. Current State: Failed. Reason: Cannot connect to endpoint.
time="2019-05-20T18:45:56Z" Activating cluster domain us-east-1a. Current State: InProgress
time="2019-05-20T18:46:01Z" Activating cluster domain us-east-1a. Current State: Success
```

Finally to check the current statuses of all the clusterdomains in your cluster you can run the following command:

```text
storkctl get clusterdomainsstatus
```

```output
NAME            ACTIVE                    INACTIVE   CREATED
px-dr-cluster   [us-east-1a us-east-1b]   []         09 Apr 19 17:13 PDT
```

You can see that the cluster domain `us-east-1a` is now **Active**

### Stop the application on the destination cluster

On the destination cluster, where the applications were failed over in Step 3, you need to stop them so that we can failback to the source cluster.

You can stop the applications from running by changing the replica count of your deployments and statefulsets to 0.

```text
kubectl scale --replicas 0 deployment/mysql -n migrationnamespace
```

### Start back the application on the source cluster
After you have stopped the applications on the destination cluster, let's jump to the source cluster. Here, we would want to start back the applications by editing the replica count.

```text
kubectl scale --replicas 1 deployment/mysql -n migrationnamespace
```

Lastly, let's check that our application is running:

```text
kubectl get pods -n migrationnamespace
```

```output
NAME                     READY     STATUS    RESTARTS   AGE
mysql-5857989b5d-48mwf   1/1       Running   0          3m
```

```text
kubectl scale --replicas 0 deployment/mysql -n migrationnamespace
```