### Troubleshooting
If there is a failure or you want more information about what resources were migrated you can describe the migration object using kubectl:
```
$ kubectl describe migration mysqlmigration
Name:         mysqlmigration
Namespace:    migrationnamespace
Labels:       <none>
Annotations:  <none>
API Version:  stork.libopenstorage.org/v1alpha1
Kind:         Migration
Metadata:
  Creation Timestamp:  2018-10-26T20:04:19Z
  Generation:          1
  Resource Version:    2148620
  Self Link:           /apis/stork.libopenstorage.org/v1alpha1/migrations/ctlmigration3
  UID:                 be63bf72-d95a-11e8-ba98-0214683e8447
Spec:
  Cluster Pair:       remotecluster
  Include Resources:  true
  Namespaces:
      migrationnamespace
  Selectors:           <nil>
  Start Applications:  true
Status:
  Resources:
    Group:      core
    Kind:       PersistentVolume
    Name:       pvc-34bacd62-d7ee-11e8-ba98-0214683e8447
    Namespace:
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      core
    Kind:       PersistentVolumeClaim
    Name:       mysql-data
    Namespace:  mysql
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
    Group:      apps
    Kind:       Deployment
    Name:       mysql
    Namespace:  mysql
    Reason:     Resource migrated successfully
    Status:     Successful
    Version:    v1
  Stage:        Final
  Status:       Successful
  Volumes:
    Namespace:                mysql
    Persistent Volume Claim:  mysql-data
    Reason:                   Migration successful for volume
    Status:                   Successful
    Volume:                   pvc-34bacd62-d7ee-11e8-ba98-0214683e8447
Events:
  Type    Reason      Age    From   Message
  ----    ------      ----   ----   -------
  Normal  Successful  2m42s  stork  Volume pvc-34bacd62-d7ee-11e8-ba98-0214683e8447 migrated successfully
  Normal  Successful  2m39s  stork  /v1, Kind=PersistentVolume /pvc-34bacd62-d7ee-11e8-ba98-0214683e8447: Resource migrated successfully
  Normal  Successful  2m39s  stork  /v1, Kind=PersistentVolumeClaim mysql/mysql-data: Resource migrated successfully
  Normal  Successful  2m39s  stork  apps/v1, Kind=Deployment mysql/mysql: Resource migrated successfully
```

## Advanced Operations

* [Migrating to GKE](gke)
* [Migrating to EKS](eks)
* [Configuring a namespace as a cluster namespace](cluster-admin-namespace)
<!--TODO:* [Configuring an external objectstore to be used for migration]-->