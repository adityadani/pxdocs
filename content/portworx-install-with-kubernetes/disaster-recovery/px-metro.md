---
title: "PX-DR across metro Kubernetes cluster"
linkTitle: "PX-DR across metro Kubernetes cluster"
keywords: cloud, backup, restore, snapshot, DR, migration, px-motion
description: How to achieve DR across two kubernetes cluster spanning a metropolitan area network.
series: px-dr
---

The goal of this doc is to setup a single Portworx cluster that spans across multiple Kubernetes clusters and demonstrate failover and failback of applications between them.

## Pre-requisites
* **Kubernetes Clusters**: You should have at least two kubernetes clusters which are a part of the same metropolitan area network with a network latency of 500ms between them.
* **Version**: A single Portworx cluster of v2.1 or later release needs to be installed on both clusters. Also requires stork v2.2+ on both the clusters.
* **Secret Store** : Make sure you have configured a [secret store](/key-management) on both your clusters. This will be used to store the credentials for the objectstore.
* **Network Connectivity**: Ports between 9001 and 9020 should be open between the two Kubernetes clusters.
* **External Kvdb**: A kvdb like etcd or consul setup outside of the Kubernetes clusters.
* **Stork helper** : `storkctl` is a command-line tool for interacting with a set of scheduler extensions.
{{% content "portworx-install-with-kubernetes/disaster-recovery/shared/stork-helper.md" %}}

## Installing Portworx
In this mode of operation, a single portworx cluster will stretch across multiple Kubernetes clusters. To install Portworx on each of these Kubernetes clusters, you will need to generate
a separate Portworx kubernetes manifest file for each of them using [Portworx Spec Generator]((https://edge-install.portworx.com/))

While generating the spec file for each Kubernetes cluster, make sure you provide the same values for the following arguments:

* **Cluster ID** (Portworx install argument: `-c`)
* **Kvdb Endpoints**  (Portworx install argument: `-k`)

Specifying the same **ClusterID** and **Kvdb Endpoints** in each Kubernetes manifest file, ensures that a single Portworx cluster will stretch across multiple of your Kubernetes clusters.


### Specifying cluster domain
A cluster domain identifies a subset of nodes from the stretch Portworx cluster, that are a part of the same failure domain. In this case your Kubernetes 
clusters are separated across a metropolitan area network and we wish to achieve DR across them. So each Kubernetes cluster and its nodes are one cluster domain. This cluster domain
information needs to be explicitly specified to Portworx through the `-cluster_domain` install argument.

Once you have generated the Kubernetes manifest file, add the `cluster_domain` argument in the args section of the daemonset

```bash
      containers:
        - name: portworx
          image: portworx/oci-monitor:2.1
          imagePullPolicy: Always
          args:
            ["-k", "etcd:http://100.26.199.167:2379", "-c", "px-dr-cluster", "-cluster_domain", "us-east-1a", "-a", "-secret_type", "k8s",
             "-x", "kubernetes"]

```

The value for `cluster_domain` needs to be unique for each Kubernetes cluster. You can name your cluster domains with arbitrary names like `datacenter1` OR `datacenter2`
identifying where your Kubernetes clusters are running. You can name them based of your cloud provider's zone names such as `us-east-1a`, `us-east-1b`.

{{<info>}}
**NOTE:** Only deploy Portworx in this mode, when your Kubernetes clusters are separated by a metropolitan area network with a network latency of 500ms or less. It is not recommended to run in this mode, if your Kubernetes clusters are across regions like AWS regions `us-east` and `us-west`
{{</info>}}

A cluster domain and in turn the set of nodes which are a part of this domain are associated with either of the following states:
* **Active State**: Nodes from an active cluster domain participate in the cluster activities
* **Inactive State**: Nodes from an inactive cluster domain do not participate in the cluster activities and remain in `Out of Quorum` state.

Once the Kubernetes manifest is applied on both the clusters, Portworx will form the stretch cluster. You can run the following commands to ensure Portworx is successfully installed in a stretch fashion.

**Kubernetes Cluster 1 running in `us-east-1a`**

```
# kubectl get nodes -owide
NAME                            STATUS    ROLES     AGE       VERSION   EXTERNAL-IP      OS-IMAGE                       KERNEL-VERSION   CONTAINER-RUNTIME
ip-172-20-33-100.ec2.internal   Ready     node      3h        v1.11.9   18.205.7.13      Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
ip-172-20-36-212.ec2.internal   Ready     node      3h        v1.11.9   18.213.118.236   Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
ip-172-20-48-24.ec2.internal    Ready     master    3h        v1.11.9   18.215.34.139    Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
ip-172-20-59-248.ec2.internal   Ready     node      3h        v1.11.9   34.200.228.223   Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
```

**Kubernetes Cluster 2 running in `us-east-1b`**

```
# kubectl get nodes -owide
NAME                            STATUS    ROLES     AGE       VERSION   EXTERNAL-IP     OS-IMAGE                       KERNEL-VERSION   CONTAINER-RUNTIME
ip-172-40-34-140.ec2.internal   Ready     node      3h        v1.11.9   34.204.2.95     Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
ip-172-40-35-23.ec2.internal    Ready     master    3h        v1.11.9   34.238.44.60    Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
ip-172-40-40-230.ec2.internal   Ready     node      3h        v1.11.9   52.90.187.179   Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
ip-172-40-50-47.ec2.internal    Ready     node      3h        v1.11.9   3.84.27.79      Debian GNU/Linux 9 (stretch)   4.9.0-7-amd64    docker://17.3.2
```

A single Portworx cluster running across both the Kubernetes clusters

```
# kubectl exec -it portworx-d6rk7 -n kube-system /opt/pwx/bin/pxctl status
Status: PX is operational
License: Trial (expires in 31 days)
Node ID: 04de0858-4081-47c3-a2ab-f0835b788738
        IP: 172.40.40.230
        Local Storage Pool: 1 pool
        POOL    IO_PRIORITY     RAID_LEVEL      USABLE  USED    STATUS  ZONE            REGION
        0       LOW             raid0           150 GiB 9.0 GiB Online  us-east-1b      us-east-1
        Local Storage Devices: 1 device
        Device  Path            Media Type              Size            Last-Scan
        0:1     /dev/xvdf       STORAGE_MEDIUM_SSD      150 GiB         09 Apr 19 22:57 UTC
        total                   -                       150 GiB
        Cache Devices:
        No cache devices
Cluster Summary
        Cluster ID: px-dr-cluster
        Cluster UUID: 27558ed9-7ddd-4424-92d4-5dfe2af572e0
        Scheduler: kubernetes
        Nodes: 6 node(s) with storage (6 online)
        IP              ID                                      SchedulerNodeName               StorageNode     Used    Capacity        Status  StorageStatus   Version         Kernel          OS
        172.20.33.100   c665fe35-57d9-4302-b6f7-a978f17cd020    ip-172-20-33-100.ec2.internal   Yes             0 B     150 GiB         Online  Up              2.1.0.0-cb23fd1 4.9.0-7-amd64   Debian GNU/Linux 9 (stretch)
        172.40.50.47    bbb2f11d-c6ad-46e7-a52f-530f313869f3    ip-172-40-50-47.ec2.internal    Yes             0 B     150 GiB         Online  Up              2.1.0.0-cb23fd1 4.9.0-7-amd64   Debian GNU/Linux 9 (stretch)
        172.40.34.140   a888a08e-0596-43a5-8d02-6faf19e8724c    ip-172-40-34-140.ec2.internal   Yes             0 B     150 GiB         Online  Up              2.1.0.0-cb23fd1 4.9.0-7-amd64   Debian GNU/Linux 9 (stretch)
        172.20.36.212   7a83c652-ffaf-452f-978c-82b0da1d2580    ip-172-20-36-212.ec2.internal   Yes             0 B     150 GiB         Online  Up              2.1.0.0-cb23fd1 4.9.0-7-amd64   Debian GNU/Linux 9 (stretch)
        172.20.59.248   11e0656a-45a5-4a5b-b4e6-51e130959644    ip-172-20-59-248.ec2.internal   Yes             0 B     150 GiB         Online  Up              2.1.0.0-cb23fd1 4.9.0-7-amd64   Debian GNU/Linux 9 (stretch)
        172.40.40.230   04de0858-4081-47c3-a2ab-f0835b788738    ip-172-40-40-230.ec2.internal   Yes             0 B     150 GiB         Online  Up (This node)  2.1.0.0-cb23fd1 4.9.0-7-amd64   Debian GNU/Linux 9 (stretch)
Global Storage Pool
        Total Used      :  0 B
        Total Capacity  :  900 GiB
root@aditya-dev:~/kops#
```

### Cluster Domains Status 

When stork is deployed with Portworx enabled with cluster domains it creates a new CRD ClusterDomainsStatus. This can be used to find out the current state of all the cluster domains.
Each Kubernetes cluster will have this CRD registerd and you can use the following commands to get the current status

```
# storkctl get clusterdomainsstatus
NAME            ACTIVE                    INACTIVE   CREATED
px-dr-cluster   [us-east-1a us-east-1b]   []         09 Apr 19 17:12 PDT
```

You can also check the status of the ClusterDomainsStatus object

```
# kubectl describe clusterdomainsstatus -n kube-system
Name:         px-dr-cluster
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  stork.libopenstorage.org/v1alpha1
Kind:         ClusterDomainsStatus
Metadata:
  Creation Timestamp:  2019-04-10T00:12:53Z
  Generation:          1
  Resource Version:    62704
  Self Link:           /apis/stork.libopenstorage.org/v1alpha1/clusterdomainsstatuses/px-dr-cluster
  UID:                 62865df5-5b25-11e9-a3ff-12e335b4e36e
Status:
  Active:
    us-east-1b
    us-east-1a
  Inactive:  <nil>

```

Once your Portworx stretch cluster is installed, you can proceed to next step.

## Pairing clusters
In order to failover an application running on one Kubernetes cluster to another Kubernetes cluster, we need to migrate the resources between them.
On Kubernetes you will define a trust object required to communicate with the other Kubernetes cluster called a ClusterPair. This creates a pairing between the scheduler (Kubernetes) so all the Kubernetes resources, 
can be migrated between them. Throughout this section the notion of source and destination clusters apply only at the Kubernetes level and does not apply to Storage, as you have a single Portworx storage fabric running on both the clusters.
As Portworx is stretched across them, the volumes do not need to be migrated. 

For reference,

* **Source Cluster** is the Kubernetes cluster where your applications are running
* **Destination Cluster** is the Kubernetes cluster where the applications will be failed over, in case of a disaster in the source cluster.

{{% content "portworx-install-with-kubernetes/disaster-recovery/shared/cluster-pair.md" %}}

In the generated **ClusterPair** spec, you will see an unpopulated *options* section. It expects options that are required to pair Storage. However as we have a single storage fabric this section is not needed.
You can delete the line `<insert_storage_options_here>` and proceed ahead.
Save this to a file called clusterpair.yaml on the source cluster.

### Create the ClusterPair
On the source cluster create the clusterpair by applying the generated spec.
```
$ kubectl apply -f clusterpair.yaml
clusterpair.stork.libopenstorage.org/remotecluster created
```

### Verify the Pair status
Once you apply the above spec on the source cluster you should be able to check the status of the pairing. On a successful pairing, you should
see the "Storage Status" and "Scheduler Status" as "Ready" using storkctl on the
source cluster:
```
$ storkctl get clusterpair
NAME               STORAGE-STATUS   SCHEDULER-STATUS   CREATED
remotecluster      NotProvided      Ready              09 Apr 19 18:16 PDT
```

## Migrating Resources
Once the pairing is configured, applications can now failover from one cluster to another. In order to achieve that we need to 
migrate the Kubernetes resources to the destination cluster

### Starting Migration

#### Using a spec file
In order to make the process schedulable and repeatable, you can write a YAML
specification. In that YAML, you will specify an object called a Migration.
In the specification, you will define the scope of the 
applications to move and decide whether to automatically start the applications.
Here, create a migration and save as migration.yaml.
```
apiVersion: stork.libopenstorage.org/v1alpha1
kind: Migration
metadata:
  name: mysqlmigration
  namespace: migrationnamespace
spec:
  # This should be the name of the cluster pair created above
  clusterPair: remotecluster
  # If set to false this will migrate only the Portworx volumes. No PVCs, apps, etc will be migrated
  includeResources: true
  # If set to false, the deployments and stateful set replicas will be set to 0 on the destination.
  # If set to true, the deployments and stateful sets will start running once the migration is done
  # There will be an annotation with "stork.openstorage.org/migrationReplicas" on the destinationto store the replica count from the source.
  startApplications: false
  # If set to false, the volumes will not be migrated
  includeVolumes: false
  # List of namespaces to migrate
  namespaces:
  - migrationnamespace
```

From the above Migration spec note:

    * the option `includeVolumes` is set to false, because the volumes are already present on the destination cluster as there is a single Storage fabric

    * the option `startApplications` is set to false, so that the applications do not start when the resources are migrated. We want to start the applications on the destination cluster only when we want to failover the application.


You can now invoke or schedule the now defined migration. This step is automateable or can
be user invoked. In order to invoke from the command-line, run the following
steps:

```
kubectl apply -f migration.yaml
```

#### Using storkctl
You can also start a migration using storkctl:
```
$ storkctl create migration mysqlmigration --clusterPair remotecluster --namespaces migrationnamespace --includeResources -n migrationnamespace
Migration mysqlmigration created successfully
```

#### Migration scope
Currently you can only migrate namespaces in which the object is created. You
can also designate one namespace as an admin namepace. This will allow an admin
who has access to that namespace to migrate any namespace from the source
cluster. Instructions for setting this admin namespace to stork can be found
[here](cluster-admin-namespace)

### Monitoring Migration
Once the migration has been started using the above step, you can check the status of the migration using storkctl

The Stages of migration will go from Application→Final if successful.
```
$ storkctl get migration -n migrationnamespace
NAME             CLUSTERPAIR     STAGE     STATUS       VOLUMES   RESOURCES   CREATED               ELAPSED
mysqlmigration   remotecluster   Final     Successful   0/0       9/9         09 Apr 19 19:45 PDT   19s
```

Until now, we have only migrated the Kubernetes resources to the destination cluster. However the replicas of the Application is set to 0, so that it does not start running. On the destination cluster
you can see the application deployment is created but it is not running.

```
# kubectl get deployments -n migrationnamespace
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
mysql     0         0         0            0           5m
```

## Disaster Recovery

In case of a disaster, where your source Kubernetes cluster goes down or is inaccessible you would want to failover the applications to the destination (backup/standby) cluster. The following section
describes the steps to achieve that

### Failover an Application

In this section we will failover an application from the source Kubernetes cluster to the destination Kubernetes cluster.
In order to failover the application, you need to instruct Stork and Portworx that one of your Kubernetes cluster is down and inactive. 

#### Create a ClusterDomainUpdate CRD

 In a YAML, you will specify an object called a ClusterDomainUpdate and we will designate the cluster domain of the source cluster as inactive

 ```
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

You can run the above command on any Kubernetes cluster which is alive. To validate that the command has succeeded you can do the following checks

```
# storkctl get clusterdomainsstatus
NAME            ACTIVE         INACTIVE       CREATED
px-dr-cluster   [us-east-1b]   [us-east-1a]   09 Apr 19 17:12 PDT
```

You can see that the ClusterDomainsStatus is updated to reflect that cluster domain `us-east-1a` is **Inactive**

#### Start the application on the destination cluster

When we migrated the applications to the destination cluster, the replica count was set to 0 for all the deployments and statefulsets. You can edit them 
and set the replica counts to the desired value. 

Each application spec, will have the following annotation `stork.openstorage.org/migrationReplicas` indicating what was the replica count for it on the source cluster.

Once the replica count is 
```
# kubectl get pods -n migrationnamespace
NAME                     READY     STATUS    RESTARTS   AGE
mysql-5857989b5d-48mwf   1/1       Running   0          3m
```

### Failback an Application

In this section we will failback the previous application from the destination Kubernetes cluster back to the source Kubernetes cluster.
In order to failover the application, you need to instruct Stork and Portworx that one of your source Kubernetes cluster is back up and is active

#### Create a ClusterDomainUpdate CRD

 In a YAML, you will specify an object called a ClusterDomainUpdate and we will designate the cluster domain of the source cluster as active

 ```
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

```
# kubectl create -f clusterdomainupdate.yaml
clusterdomainupdate "activate-us-east-1a" created
```

You can run the above command on any Kubernetes cluster which is alive. To validate that the command has succeeded you can do the following checks

```
# storkctl get clusterdomainsstatus
NAME            ACTIVE                    INACTIVE   CREATED
px-dr-cluster   [us-east-1a us-east-1b]   []         09 Apr 19 17:13 PDT
```

You can see that the ClusterDomainsStatus is updated to reflect that cluster domain `us-east-1a` is back **Active**

#### Start back the application on the source cluster
On the destination cluster, where the application is currently running, edit the application deployment/statefulset and set the replica count to 0
This will stop the application from running. At the same time you will need to start the applications back on the source cluster.

Each application spec, will have the following annotation `stork.openstorage.org/migrationReplicas` indicating what was the replica count for it on the source cluster.

Once the replica count is 
```
# kubectl get pods -n migrationnamespace
NAME                     READY     STATUS    RESTARTS   AGE
mysql-5857989b5d-48mwf   1/1       Running   0          3m
```

