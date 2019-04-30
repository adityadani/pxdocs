---
title: Encrypting Kubernetes PVCs with AWS KMS
weight: 1
keywords: Portworx, Amazon, AWS KMS, containers, storage, encryption
description: Instructions on using AWS KMS with Portworx for encrypting PVCs
noicon: true
series: aws-secret-uses
series2: k8s-pvc-enc
hidden: true
---

{{% content "key-management/shared/intro.md" %}}

### Encryption using per volume secrets

In this method each volume will use its own unique passphrase to encrypt the volume. Portworx uses the AWS KMS apis to generate a Data Encryption Key. This key will then be used for enctypting and decrypting your volumes.


{{<info>}}
**NOTE** This is the recommended method for encrypting volumes when you want to take a cloud backup of an encrypted volume or migrate encrypted volumes between multiple clusters
{{</info>}}

#### Step 1: Create a Storage Class

{{% content "key-management/shared/enc-storage-class-spec.md" %}}

#### Step 2: Create a Persistent Volume Claim

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql-data
  annotations:
    volume.beta.kubernetes.io/storage-class: px-secure-sc
spec:
  storageClassName: px-mysql-sc
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi

```

If you do not want to specify the `secure` flag in the storage class, but you want to encrypt the PVC using that Storage Class, then create the PVC as below

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: secure-pvc
  annotations:
    px/secure: "true"
spec:
  storageClassName: portworx-sc
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```
Note the `px/secure: "true"` annotation on the PVC object.

### Encryption using named secrets

{{<info>}}
{{% content "key-management/aws-kms/shared/warning-note.md" %}}
{{</info>}}

#### Step 1: Creating Named Secrets {#creating-named-secrets}

{{% content "key-management/aws-kms/shared/named-secrets.md" %}}

#### Step 2: Create a Storage Class

{{% content "key-management/shared/enc-storage-class-spec.md" %}}

#### Step 3: Create a Persistent Volume Claim

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql-data
  annotations:
    px/secret-name: mysecret
    volume.beta.kubernetes.io/storage-class: px-secure-sc
spec:
  storageClassName: px-mysql-sc
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi

```

Take a note of the annotation `px/secret-name: mysecret`. This specific annotation indicates Portworx to use the named secret `mysecret` generated using the Portworx CLI to encrypt the volume. In this case it will **NOT**  create a new AWS Data Key for this volume. If the annotation is not provided then Portworx will use the per volume encryption workflow as described in the previous sections.

{{<info>}}
**NOTE** A single named secret can be used for encrypting multiple volumes.
{{</info>}}

Again, if your Storage Class does not have the `secure` flag set, but you want to encrypt the PVC using the same Storage Class, then add the annotation `px/secure: "true"` to the above PVC.

### Encryption using cluster wide secret

{{% content "key-management/aws-kms/shared/cluster-wide-intro.md" %}}

#### Step 1: Set the cluster wide secret key

{{% content "key-management/aws-kms/shared/cluster-wide-secret.md" %}}

#### Step 2: Create a Storage Class

{{% content "key-management/shared/enc-storage-class-spec.md" %}}

#### Step 3: Create a Persistent Volume Claim

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql-data
  annotations:
    px/secret-name: default
    volume.beta.kubernetes.io/storage-class: px-secure-sc
spec:
  storageClassName: px-mysql-sc
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi

```

Take a note of the annotation `px/secret-name: default`. This specific annotation indicates Portworx to use the default secret to encrypt the volume. In this case it will **NOT**  create a new passphrase for this volume and NOT use per volume encryption. If the annotation is not provided then Portworx will use the per volume encryption workflow as described in the previous section.

Again, if your Storage Class does not have the `secure` flag set, but you want to encrypt the PVC using the same Storage Class, then add the annotation `px/secure: "true"` to the above PVC.
