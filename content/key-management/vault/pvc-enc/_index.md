---
title: Encrypting Kubernetes PVCs with Vault
weight: 1
keywords: Portworx, Hashicorp, Vault, containers, storage, encryption
description: Instructions on using Vault with Portworx for encrypting PVCs in Kubernetes
noicon: true
series: vault-secret-uses
series2: k8s-pvc-enc
hidden: true
---

{{% content "key-management/shared/intro.md" %}}

There are two ways in which Portworx volumes can be encrypted and are dependent on how a secret passphrase is provided to PX.

### Encryption using Storage Class

In this method, PX will use the cluster wide secret key to encrypt PVCs.

#### Step 1: Set a cluster wide secret

{{% content "key-management/shared/set-cluster-wide-secret.md" %}}

{{% content "key-management/shared/storage-class-encryption.md" %}}

### Encryption using PVC annotations

In this method, each PVC can be encrypted with its own secret key.

#### Step 1: Create a Storage Class

{{% content "key-management/shared/enc-storage-class-spec.md" %}}

#### Step 2: Create a PVC with annotation

{{% content "key-management/shared/other-providers-pvc-encryption" %}}

__Important: Make sure secret `your_secret_key` exists in Vault__

### Encryption per Namespace

In this method, all the PVCs from a namespace will be encrypted using the same secret key. In order to achieve this we will do the following:

* In the storage class we will set the following two fields:
    * **secret: "true"**: This indicates that the PVCs created from this storage class are encrypted.
    * **secret_key: "your_secret_key"**: This indicates that the PVCs using this storage class will use the key **your_secret_key** from Vault
* We will define **ResourceQuotas** on the namespace in which PVCs are created so that only the above Storage Class will be used.


#### Step 1: Create a Storage Class

Create a storage class with the `secure` parameter set to `true`.
```text
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: px-secure-sc
provisioner: kubernetes.io/portworx-volume
parameters:
  secure: "true"
  secret_key: "your_secret_key"
  repl: "3"
```

__Important: Make sure secret `your_secret_key` exists in Vault__

To create a **shared encrypted volume** set the `shared` parameter to `true` as well.

#### Step 2: Create a ResourceQuota object for the PVC's Namespace

Create a resource quota that disallows any other storage classes to be used in this namespace

```text
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-storageclass-quotas
  namespace: dev
spec:
  hard:
    <storageclass-1-name>.storageclass.storage.k8s.io/persistentvolumeclaims: 0
    <storageclass-2-name>.storageclass.storage.k8s.io/persistentvolumeclaims: 0
    <storageclass-3-name>.storageclass.storage.k8s.io/persistentvolumeclaims: 0
```

With the above ResourceQuota definition we are disallowing PVC creations using
storage classes `<storageclass-1-name>, <storageclass-2-name>, <storageclass-3-name>` in the `dev` namespace.

__Note: In the above ResourceQuota we have excluded the `px-secure-sc` storage class as we want to use it to create encrypted PVCs.__

#### Step 3: Create an encrypted PVC

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: secure-mysql-pvc
  namespace: dev
spec:
  storageClassName: px-secure-sc
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```