---
title: "2. Pair the Kubernetes clusters and synchronize resources between them"
weight: 2
keywords: cloud, backup, restore, snapshot, DR, migration, px-motion
description: Find out how to pair your clusters and synchronize Kubernetes resources between them
---

## Pairing clusters
In order to failover an application running on one Kubernetes cluster to another Kubernetes cluster, we need to migrate the resources between them.
On Kubernetes you will define a trust object required to communicate with the other Kubernetes cluster called a ClusterPair. This creates a pairing between the scheduler (Kubernetes) so that all the Kubernetes resources can be migrated between them.
Throughout this section, the notion of source and destination clusters apply only at the Kubernetes level and does not apply to Storage, as you have a single Portworx storage fabric running on both the clusters.
As Portworx is stretched across them, the volumes do not need to be migrated. 

For reference,

* **Source Cluster** is the Kubernetes cluster where your applications are running
* **Destination Cluster** is the Kubernetes cluster where the applications will be failed over, in case of a disaster in the source cluster.

{{% content "portworx-install-with-kubernetes/disaster-recovery/shared/cluster-pair.md" %}}

In the generated **ClusterPair** spec, you will see an unpopulated *options* section. It expects options that are required to pair Storage. However, as we have a single storage fabric, this section is not needed.
You should delete the whole <insert_storage_options_here> section and save this to a file called clusterpair.yaml on the source cluster.

#### Apply the generated ClusterPair on the source cluster

On the **source** cluster create the clusterpair by applying the generated spec.

```text
kubectl apply -f clusterpair.yaml
```

### Verify the Pair status
Once you apply the above spec on the source cluster you should be able to check the status of the pairing using storkctl on the source cluster. 

```text
storkctl get clusterpair
```

On a successful pairing you should see the "Scheduler Status" as "Ready" and the "Storage Status" as "Not Provided"

```
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
In order to make the process schedulable and repeatable, you can write a YAML specification.

In that file, you will specify an object called `Migration`. This object will define the scope of the applications to move and decide whether to automatically start the applications.

Paste this to a file named `migration.yaml`.
```text
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

A few things to note:

* The option `includeVolumes` is set to false because the volumes are already present on the destination cluster as there is a single storage fabric
* The option `startApplications` is set to false so that the applications do not start when the resources are migrated. This is because the applications on the destination cluster should start only when the application fails over.


Next, you can invoke this migration manually from the command line:

```text
kubectl apply -f migration.yaml
```

or automate it through `storkctl`:
```text
storkctl create migration mysqlmigration --clusterPair remotecluster --namespaces migrationnamespace --includeResources -n migrationnamespace
```

#### Migration scope
Currently you can only migrate namespaces in which the object is created. You
can also designate one namespace as an admin namepace. This will allow an admin
who has access to that namespace to migrate any namespace from the source
cluster. Instructions for setting this admin namespace to stork can be found
[here](/portworx-install-with-kubernetes/migration/px-motion/cluster-admin-namespace)

### Monitoring Migration
Once the migration has been started using the above step, you can check the status of the migration using storkctl

The Stages of migration will go from Application→Final if successful. To get the current status of migration run the following command:

```text
storkctl get migration -n migrationnamespace
```

After a successful migration, you should see the following output
```
NAME             CLUSTERPAIR     STAGE     STATUS       VOLUMES   RESOURCES   CREATED               ELAPSED
mysqlmigration   remotecluster   Final     Successful   0/0       9/9         09 Apr 19 19:45 PDT   19s
```

Until now, we have only migrated the Kubernetes resources to the destination cluster. However, the replicas of the Application is set to 0, so that it does not start running. On the destination cluster
you can see that the application deployment is created but it is not running.

```
# kubectl get deployments -n migrationnamespace
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
mysql     0         0         0            0           5m
```
