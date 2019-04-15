---
title: "4. Failback an application"
weight: 4
keywords: cloud, backup, restore, snapshot, DR, migration, px-motion
description: Find out how to failback an application from the backup Kubernetes cluster to the original one.
---

Once your unhealthy Kubernetes cluster is back up and running, you would want to failback all the applications that were originally running on it

For this section we will refer to,

* **Source Cluster** as the Kubernetes cluster which is back online and where your applications need to failback to. (In this example: `cluster_domain: us-east-1a`)
* **Destination Cluster** as the Kubernetes cluster where the applications will be failed over. (In this example: `cluster_domain: us-east-1b`)

In order to failback the application, you need to instruct Stork and Portworx that your source Kubernetes cluster is now back up and is active

### Create a ClusterDomainUpdate CRD

Start by creating a new file named `clusterdomainupdate.yaml`. In this file, let's specify an object called a ClusterDomainUpdate and designate the cluster domain of the source cluster as active

 ```text
 apiVersion: stork.libopenstorage.org/v1alpha1
kind: ClusterDomainUpdate
metadata:
 name: activate-us-east-1a
 namespace: kube-system
spec:
  # Name of the metro domain that needs to be activated/deactivated
  clusterdomain: us-east-1a
  # Set to true to activate cluster domain
  # Set to false to deactivate cluster domain
  active: true
 ```

In order to invoke from command-line, you will need to run the following

```text
# kubectl create -f clusterdomainupdate.yaml
clusterdomainupdate "activate-us-east-1a" created
```

You can run the above command from any Kubernetes cluster. To validate that the command has succeeded you can do the following checks

```
# storkctl get clusterdomainsstatus
NAME            ACTIVE                    INACTIVE   CREATED
px-dr-cluster   [us-east-1a us-east-1b]   []         09 Apr 19 17:13 PDT
```

You can see that the cluster domain `us-east-1a` is now **Active**

### Stop the application on the destination cluster

On the destination cluster, where the applications were failed over in Step 3, you need to stop them so that we can failback to the source cluster.

You can stop the applications from running by changing the replica count of your deployments and statefulsets to 0. 

```text
kubectl scale --replicas 0 deployment/mysql -n migrationnamespace
deployment "mysql" scaled
```


### Start back the application on the source cluster
After you have stopped the applications on the destination cluster, let's jump to the source cluster. Here, we would want to start back the applications by editing the replica count.

```
# kubectl scale --replicas 1 deployment/mysql -n migrationnamespace
deployment "mysql" scaled
```


```
# kubectl get pods -n migrationnamespace
NAME                     READY     STATUS    RESTARTS   AGE
mysql-5857989b5d-48mwf   1/1       Running   0          3m
```
