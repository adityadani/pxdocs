Generate a new KMS Data Key by running the following command:

```text
/opt/pwx/bin/pxctl secrets aws generate-kms-data-key --secret_id default
```

The above command generates an AWS KMS Data Key and associates it with the name `default`. 
Now set the cluster secret key using the following command

```text
/opt/pwx/bin/pxctl secrets set-cluster-key --secret default
```

This command needs to be run just once for the cluster.
