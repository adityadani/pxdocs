### Generate and Apply ClusterPair Spec

In Kubernetes you will define a trust object called **ClusterPair**. This object is required to communicate with the destination cluster. In a nutshell, it creates a pairing with the storage driver (_Portworx_) as well as the scheduler (Kubernetes) so that the volumes and resources can be migrated between clusters.
It is generated and used in the following way:

   * The **ClusterPair** spec is generated on the **destination** cluster.
   * The generated spec is then applied on the **source** cluster

#### Generate ClusterPair on the destination cluster

To generate the **ClusterPair** spec, run the following command on the **destination** cluster

```bash
storkctl generate clusterpair -n migrationnamespace remotecluster
```
Here, the name (remotecluster) is the Kubernetes object that will be created on the **source** cluster representing the pair relationship.

During the actual migration, you will reference this name to identify the destination of your migration

```
$ storkctl generate clusterpair -n migrationnamespace remotecluster
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
       <insert_storage_options_here>: ""
status:
  remoteStorageId: ""
  schedulerStatus: ""
  storageStatus: ""
```
