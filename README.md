# aap4ocp

This is a collection of ansible tasks used for day 1 and day 2
administration of OpenShift Virtualization. It's designed to be run
from AAP - your mileage may vary running from Navigator or the
commandline.

## Setup

You'll need a couple of things for this automation to work.

### Service Account

First, you'll want to have a service account which AAP can use to run
the automation against your OpenShift cluster. Starting with OpenShift
4.11, tokens associated with service accounts have a scoped
duration. We'll create a service account which has cluster admin
privileges for now and we'll give it a token which expires in 90 days.

```
oc create sa aap-user -n aap
oc adm policy add-cluster-role-to-user cluster-admin aap-user -n aap
oc create token --duration 7776000s aap-user -n aap
```

For more information on creating a service account for AAP, see the official documentation: https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html/automation_controller_user_guide/controller-credentials#proc-controller-credential-create-openshift-account.

The output of the last command should be the token that we'll use to
create the credential in AAP.

### Credential in AAP

To create the credential in AAP, you'll need the following information

1. The API endpoint of the cluster
2. The token associated with the service account
3. Optionally the SSL/CA information.

For steps on creating a credential for your cluster, see:
https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html/automation_controller_user_guide/controller-credentials#controller-getting-started-create-credential

### Installation Disk

If you're making some Winders VMs or doing a Kickstart build, you'll
probably want to download an ISO to boot from for the install. In this case, you can create a DataVolume from an ISO like so:

```
apiVersion: cdi.kubevirt.io/v1beta1
kind: DataVolume
metadata:
  name: "install-disk"
spec:
  source:
      http:
         url: "https://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img" # S3 or GCS
         secretRef: "" # Optional
         certConfigMap: "" # Optional
  storage:
    resources:
      requests:
        storage: "64Mi"
```

For more information, see https://github.com/kubevirt/containerized-data-importer/blob/main/doc/datavolumes.md#source

