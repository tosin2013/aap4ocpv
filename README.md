# Ansible Automation Platform for OpenShift VM Management

## Overview
This project provides a comprehensive solution for managing virtual machines (VMs) on OpenShift using Ansible Automation Platform (AAP) 2.5. It includes playbooks, scripts, and documentation for automating VM lifecycle operations and system management.

## Prerequisites
- OpenShift 4.18.4
- OpenShift Virtualization 4.18.1
- Ansible Automation Platform 2.5.0
- Python 3.9.21
- Ansible Core 2.14.17
- OpenShift CLI (oc) 4.18.4

## System Requirements
- Minimum 8 vCPUs
- Minimum 32GB RAM
- Minimum 100GB storage for database
- Minimum 20GB storage for Automation Controller
- Minimum 10GB storage for Automation Hub

## Environment Setup

### 1. Configure OpenShift Access
```bash
# Set the kubeconfig path
export KUBECONFIG=/home/lab-user/generated_assets/ocp4/auth/kubeconfig

# Verify OpenShift access
oc version
```

### 2. Verify Prerequisites
```bash
# Check OpenShift Virtualization
oc get csv -n openshift-cnv

# Check Ansible Automation Platform
oc get csv -n aap

# Verify Python and Ansible versions
python3 --version
ansible --version
```

### 3. Project Setup
```bash
# Clone the repository
git clone <repository-url>
cd <repository-name>

# Configure environment
./configure-env.sh

# Build execution environment
./build-ee.sh
```

### 4. AAP Installation
```bash
# Deploy AAP on OpenShift
ansible-playbook deploy-aap.yml
```

### 5. VM Management
```bash
# Create a new VM
ansible-playbook 01-create_vm.yml

# Start a VM
ansible-playbook 02-start_vm.yml

# Stop a VM
ansible-playbook 03-stop_vm.yml
```

## Project Structure
```
.
├── docs/                    # Documentation
│   ├── adr/                # Architecture Decision Records
│   ├── blog/               # Blog posts
│   └── _layouts/           # Documentation layouts
├── templates/              # VM and resource templates
├── vars/                   # Variable definitions
├── collections/           # Ansible collections
├── ee-build/             # Execution environment build files
└── *.yml                 # Ansible playbooks
```

## Key Components

### VM Lifecycle Management
- `01-create_vm.yml`: Create new VMs
- `02-start_vm.yml`: Start existing VMs
- `03-stop_vm.yml`: Stop running VMs
- `04-modify_cpu.yml`: Modify VM CPU resources
- `05-add_disk.yml`: Add storage to VMs
- `06-add_nic.yml`: Add network interfaces
- `07-take_snapshot.yml`: Create VM snapshots
- `08-restore_snapshot.yml`: Restore from snapshots
- `09-delete_vm.yml`: Remove VMs

### System Management
- `11-add_repo.yml`: Add software repositories
- `12-install_all_updates.yml`: System updates
- `13-install_httpd.yml`: Web server installation
- `14-install-eap.yml`: Enterprise Application Platform setup

### Provisioning
- `21-provision_vms.yml`: Complete VM provisioning
- `22-add_repos.yml`: Repository configuration
- `23-install_all_updates.yml`: System updates
- `24-install-pg.yml`: PostgreSQL installation
- `25-install-eap.yml`: EAP setup

### Patch Management
- `31-list_patches.yml`: List available patches
- `32-install_patches.yml`: Apply system patches

## Configuration

### Environment Variables
Required environment variables:
```bash
# OpenShift Configuration
export KUBECONFIG=/home/lab-user/generated_assets/ocp4/auth/kubeconfig

# Cluster Access
OPENSHIFT_FQDN=your-cluster.example.com
OPENSHIFT_USER=admin
OPENSHIFT_PASSWORD=your-password
```

### Ansible Configuration
The project uses:
- `ansible.cfg` for Ansible settings
- `ansible-navigator.yml` for execution environment configuration
- Collections in `/home/lab-user/.ansible/collections:/usr/share/ansible/collections`

## Troubleshooting

### Common Issues
1. AAP Installation
   ```bash
   # Check operator status
   oc get csv -n aap
   
   # Check AAP instance status
   oc get ansibleautomationplatform -n aap
   ```

2. VM Operations
   ```bash
   # Check VM status
   oc get vm -n your-namespace
   
   # Check VM events
   oc describe vm your-vm-name -n your-namespace
   ```

### Logs
```bash
# Check AAP operator logs
oc logs -n aap deployment/aap-operator

# Check automation controller logs
oc logs -n aap deployment/automation-controller
```

## Documentation
- [Architecture Documentation](docs/architecture-documentation.md)
- [Blog Series](docs/blog-series-plan.md)
- [Prerequisites](docs/aap-prerequisites.md)
- [OpenShift Environment](docs/openshift-environment.md)

## Contributing
1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License
[Add your license information here]

## Support
[Add support contact information here] 
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

