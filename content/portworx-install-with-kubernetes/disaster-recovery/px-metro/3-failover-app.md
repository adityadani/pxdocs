---
title: "3. Failover an application"
weight: 3
keywords: cloud, backup, restore, snapshot, DR, migration, px-motion
description: Find out how to failover an application from one Kubernetes cluster to another.
---

In case of a disaster, where one of your Kubernetes cluster goes down and is inaccessible you would want to failover the applications running on it to any other Kubernetes cluster that is operational. 

For this section we will refer to,

* **Source Cluster** as the Kubernetes cluster which is down and where your applications were originally running. (In this example: `cluster_domain: us-east-1a`)
* **Destination Cluster** as the Kubernetes cluster where the applications will be failed over. (In this example: `cluster_domain: us-east-1b`)

In order to failover the application, you need to instruct Stork and Portworx that one of your Kubernetes cluster is down and inactive. 

### Create a ClusterDomainUpdate object

 In a YAML, you will specify an object called a ClusterDomainUpdate and you will designate the cluster domain of the source cluster as inactive.

 ```bash
apiVersion: stork.libopenstorage.org/v1alpha1
kind: ClusterDomainUpdate
metadata:
 name: deactivate-us-east-1a
 namespace: kube-system
spec:
  # Name of the metro domain that needs to be activated/deactivated
  clusterdomain: us-east-1a
  # Set to true to activate cluster domain
  # Set to false to deactivate cluster domain
  active: false
 ```

In order to invoke from command-line, you will need to run the following

```
# kubectl create -f clusterdomainupdate.yaml
clusterdomainupdate "deactivate-us-east-1a-2" created
```

You can run the above command on any Kubernetes cluster which is alive. To validate that the command has succeeded you can check the status of all the cluster domains using storkctl

```
# storkctl get clusterdomainsstatus
NAME            ACTIVE         INACTIVE       CREATED
px-dr-cluster   [us-east-1b]   [us-east-1a]   09 Apr 19 17:12 PDT
```

You can see that the cluster domain `us-east-1a` is now **Inactive**

### Stop the application on the source cluster (if accessible)

If your source Kubernetes cluster is still alive and is accessible, we recommend you to stop the applications before failing them over to the destination cluster.

You can stop the applications from running by changing the replica count of your deployments and statefulsets to 0. In this way your application resources will persist
in Kubernetes, but the actual application would not be running.

```
# kubectl scale --replicas 0 deployment/mysql -n migrationnamespace
deployment "mysql" scaled
```

### Start the application on the destination cluster

In step 2. we migrated the applications to the destination cluster but the replica count was set to 0 for all the deployments and statefulsets, so that they do not run.
You can now scale the applications by setting the replica counts to the desired value. 

Each application spec, will have the following annotation `stork.openstorage.org/migrationReplicas` indicating what was the replica count for it on the source cluster.

Once the replica count is updated, the application would start running, and the failover will be completed.

You can use the following command to scale the application

```
# kubectl scale --replicas 1 deployment/mysql -n migrationnamespace
deployment "mysql" scaled
```


```
# kubectl get pods -n migrationnamespace
NAME                     READY     STATUS    RESTARTS   AGE
mysql-5857989b5d-48mwf   1/1       Running   0          3m
```