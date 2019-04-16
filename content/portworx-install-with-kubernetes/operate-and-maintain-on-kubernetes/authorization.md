---
title: Authorization
description: Manage Portworx authoriztion in Kubernetes
keywords: portworx, kubernetes, security, authorization, jwt, shared secret
weight: 100
series: k8s-op-maintain
---

Before proceeding with this installation, please review the
[Security](/concepts/authorization) model used by Portworx.

## Enabling authorization
_NOTE: The following will be a cluster level interruption event while all the
nodes in the system come back online with security enabled_

To enable authorization you must simply edit your Portworx `yaml` configuration
to add the appropriate information. You must first create a Kubernetes Secret
which holds the values of the environment variables. Then populate the
environment variables required from your Secret. Here is an example of how to
setup an environment variable from a Secret:

* Create a secret:

```text
kubectl create secret generic mysecret \
  --from-literal=system-secret='RmlqRSfh9'
```

* Then we can access the key as follows:

```text
...
  - name: "PORTWORX_AUTH_SYSTEM_KEY"
    valueFrom:
      secretKeyRef:
        name: mysecret
        key: system-key
...
```

### Example
In the following example shows a how to enable Portworx authorization to verify
self-signed tokens created by a TA with the issuer called `myissuer` and using a
shared secret to generate the signature.

* Save the sensitive information in a secret

```text
kubectl create secret generic mysecret \
  --from-literal=system-secret='RmlqRSfh9' \
  --from-literal=shared-secret='hnuiUDFHf' \
  --from-literal=stork-secret='hn23nfsFD'
```

* The Portworx `yaml` configuration would look like this:

```text
...
  image: openstorage/stork:2.2.0
  env:
    - name: "PX_SHARED_SECRET"
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: stork-secret

...
  image: portworx/oci-monitor:2.1.0
  args:
  [..."--jwt-issuer", "myissuer", ...]
  env:
    - name: "PORTWORX_AUTH_JWT_SHAREDSECRET"
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: shared-secret
    - name: "PORTWORX_AUTH_SYSTEM_KEY"
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: system-key
    - name: "PORTWORX_AUTH_STORK_KEY"
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: stork-secret
...
```

## Creating volumes
Portwox authorization provides a method of protection for creating volumes
through Kubernetes. Users must provide a token when requesting volumes. These
tokens must be saved in a Secret, normally in the same namespace as the PVC.

The key in the Secret which holds the token must be named `auth-token`.

Then the annotations of the PVC can be used to point to the secret holding the
token. The following table shows the annotation keys used to poing to the
secret:

| Name | Description |
| ---- | ----------- |
| `openstorage.io/auth-secret-name` | Name of the secret which has the token |
| `openstorage.io/auth-secret-namespace` | Optional key which contains the namespace of the secret reference by `auth-secret-name`. If omitted, the namespace of the PVC will be used as default |

Here is an example:

* Create a secret with the token:

```text
kubectl create secret generic px-secret \
  -n default --from-literal=auth-token=ey..hs
```

* Create a PVC request for a 2Gi volume with the appropriate authorization:

```text
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-auth
  annotations:
    volume.beta.kubernetes.io/storage-class: portworx-sc
    openstorage.io/auth-secret-name: px-secret
    openstorage.io/auth-secret-namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

## Stork
When using CRDs consumed by Stork, you must use the same authorization model
described above for the PVCs. Here is an example:

```text
apiVersion: volumesnapshot.external-storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: mysql-snap1
  annotations:
    openstorage.io/auth-secret-name: px-secret
    openstorage.io/auth-secret-namespace: default
spec:
  persistentVolumeClaimName: mysql-data
```

## References

For more information on Kubernetes Secret which holds the environment variables See [Kubernetes
Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data)
