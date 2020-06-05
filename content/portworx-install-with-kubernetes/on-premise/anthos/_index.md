---
title: Portworx on Anthos
linkTitle: Anthos
weight: 4
keywords: Install, on-premise, anthos, kubernetes, k8s, air gapped
description: How to install Portworx on Anthos
---

Portworx has been certified with the following Anthos versions:

* Anthos 1.1
* Anthos 1.2
* Anthos 1.3 


## Architecture

{{% content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-shared-arch.md" %}}

## Installation

This topic explains how to install Portworx with Kubernetes on Anthos. Follow the steps in this topic in order.

{{<info>}}Run these steps from the anthos admin station or any other machine which has kubectl access to your cluster.{{</info>}}

{{% content "shared/cloud-references-auto-disk-provisioning-vsphere-vsphere-install-common.md" %}}

#### Generate the spec file

Now generate the spec with the following curl command.

{{<info>}}Observe how curl below uses the environment variables setup up above as query parameters.{{</info>}}

```text
export VER=$(kubectl version --short | awk -Fv '/Server Version: /{print $3}')
curl -fsL -o px-spec.yaml "https://install.portworx.com/{{% currentVersion %}}?kbver=$VER&csida=true&c=portworx-demo-cluster&b=true&st=k8s&csi=true&vsp=true&ds=$VSPHERE_DATASTORE_PREFIX&vc=$VSPHERE_VCENTER&s=%22$VSPHERE_DISK_TEMPLATE%22"
```

{{% content "shared/portworx-install-with-kubernetes-4-apply-the-spec.md" %}}

## Known issues

### vSphere 6.5

* **Issue**: If you are running on VMware vSphere  6.5 or lower, when you delete a worker node VM, all disks attached to the VM will get deleted. This is the default vSphere behavior. Deletion of the disks will cause Portworx cluster to loose quorum.
* **Workaround**: Upgrade to vSphere 6.7.3 and use Portworx 2.5.1.3 or higher
 