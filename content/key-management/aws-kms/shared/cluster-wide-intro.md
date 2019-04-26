The behavior of cluster wide secrets has changed from **Portworx version 2.1**. If you have volumes that were created using older Portworx versions, those volumes will still seamlessly work with newer Portworx versions. However new volumes created using cluster wide secret will have to follow the procedure mentioned below.

In this method a default cluster wide secret will be set for the Portworx cluster. Such a secret will be referenced by the user and Portworx as **default** secret. Any PVC request referencing the secret name as `default` will use this cluster wide secret as a passphrase to encrypt the volume.

{{<info>}}
{{% content "key-management/aws-kms/shared/warning-note.md" %}}
{{</info>}}