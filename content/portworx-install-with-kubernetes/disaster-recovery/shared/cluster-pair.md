### Get cluster token from destination cluster
On the destination cluster, run the following command from one of the Portworx nodes to get the cluster token:
   `/opt/pwx/bin/pxctl cluster token show`

### Generate ClusterPair spec
Get the **ClusterPair** spec from the destination cluster. This is required to migrate Kubernetes resources to the destination cluster.
You can generate the template for the spec using `storkctl generate clusterpair -n migrationnamespace remotecluster` on the destination cluster.
Here, the name (remotecluster) is the Kubernetes object that will be created on the source cluster representing the pair relationship.
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
