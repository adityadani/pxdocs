---
title: "PX-Motion with stork on Kubernetes"
linkTitle: "PX-Motion with stork"
keywords: cloud, backup, restore, snapshot, DR, migration, px-motion
description: How to migrate stateful applications on Kubernetes
series: px-motion
noicon: true
aliases:
  - /cloud-references/migration/migration-stork.html
  - /cloud-references/migration/migration-stork
---

## Pre-requisites
* **Version**: The source AND destination clusters need the v2.0 or later
release of PX-Enterprise on both clusters. As future releases are made, the two
clusters can have different PX-Enterprise versions (e.g. v2.1 and v2.3). Also requires stork v2.0+ on the source cluster.
* **Secret Store** : Make sure you have configured a [secret store](/key-management) on both your clusters.
This will be used to store the credentials for the objectstore.
* **Network Connectivity**: Ports 9001 and 9010 on the destination cluster should be
reachable by the source cluster.
* **Stork helper** : `storkctl` is a command-line tool for interacting with a set of scheduler extensions.
{{% content "portworx-install-with-kubernetes/disaster-recovery/shared/stork-helper.md" %}}


## Pairing clusters
On Kubernetes you will define a trust object required to communicate with the destination cluster called a ClusterPair. This creates a pairing 
with the storage driver (Portworx) as well as the scheduler (Kubernetes) so that the volumes and resources, can be migrated between
clusters.

### Get cluster token from destination cluster
On the destination cluster, run the following command from one of the Portworx nodes to get the cluster token:
```bash
pxctl cluster token show
```

{{% content "portworx-install-with-kubernetes/disaster-recovery/shared/cluster-pair.md" %}}


#### Update ClusterPair with storage options

In the generated **ClusterPair** spec, you will need to add Portworx clusterpair information under spec.options. The required options are:

   1. **ip**: IP of one of the Portworx nodes on the destination cluster
   2. **port**: Port on which the Portworx API server is listening for requests.
      Default is 9001 if not specified
   3. **token**: Cluster token generated in the [previous step](#get-cluster-token-from-destination-cluster)

The updated **ClusterPair** should look like this:
```
apiVersion: stork.libopenstorage.org/v1alpha1
kind: ClusterPair
metadata:
  creationTimestamp: null
  name: remotecluster
  namespace: migrationnamespace
spec:
  config:
      clusters:
        kubernetes:
          LocationOfOrigin: /etc/kubernetes/admin.conf
          certificate-authority-data: <CA_DATA>
          server: https://192.168.56.74:6443
      contexts:
        kubernetes-admin@kubernetes:
          LocationOfOrigin: /etc/kubernetes/admin.conf
          cluster: kubernetes
          user: kubernetes-admin
      current-context: kubernetes-admin@kubernetes
      preferences: {}
      users:
        kubernetes-admin:
          LocationOfOrigin: /etc/kubernetes/admin.conf
          client-certificate-data: <CLIENT_CERT_DATA>
          client-key-data: <CLIENT_KEY_DATA>
  options:
      ip: <ip_of_remote_px_node>
      port: <port_of_remote_px_node_default_9001>
      token: <token_generated_from_destination_cluster>>
status:
  remoteStorageId: ""
  schedulerStatus: ""
  storageStatus: ""
```
Copy and save this to a file called clusterpair.yaml on the source cluster.

#### Apply the generated ClusterPair on the source cluster

On the **source** cluster create the clusterpair by applying the generated spec.
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
remotecluster      Ready            Ready              26 Oct 18 03:11 UTC
```

## Migrating Volumes and Resources
Once the pairing is configured, applications can be migrated repeatedly to the destination cluster.

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
  # There will be an annotation with "stork.openstorage.org/migrationReplicas" on the destinationto store the replica count from the source.
  startApplications: true
  # List of namespaces to migrate
  namespaces:
  - migrationnamespace
```

You can now invoke or schedule the now defined migration. This step is automateable or can
be user invoked. In order to invoke from the command-line, run the following
steps:

```
kubectl apply -f migration.yaml
```

#### Using storkctl
You can also start a migration using storkctl:
```
$ storkctl create migration mysqlmigration --clusterPair remotecluster --namespaces migrationnamespace --includeResources --startApplications -n migrationnamespace
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
```
$ storkctl get migration -n migrationnamespace
NAME            CLUSTERPAIR     STAGE     STATUS       VOLUMES   RESOURCES   CREATED
mysqlmigration  remotecluster   Volumes   InProgress   0/1       0/0         26 Oct 18 20:04 UTC
```

The Stages of migration will go from Volumes→ Application→Final if successful.
```
$ storkctl get migration -n migrationnamespace
NAME            CLUSTERPAIR     STAGE     STATUS       VOLUMES   RESOURCES   CREATED
mysqlmigration  remotecluster   Final     Successful   1/1       3/3         26 Oct 18 20:04 UTC
```

{{% content "portworx-install-with-kubernetes/disaster-recovery/shared/migration-common.md" %}}