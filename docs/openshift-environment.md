# OpenShift Environment Documentation

## Overview
This document provides detailed information about the OpenShift environment used in this project, including cluster configuration, virtualization setup, and integration points with Ansible Automation Platform.

## Cluster Information
- **OpenShift Version**: 4.18.4
- **Kubernetes Version**: v1.31.6
- **OpenShift Virtualization**: 4.18.1
- **Storage**: OpenShift Container Storage (OCS)
- **Additional Operators**:
  - Red Hat OpenShift GitOps 1.15.1
  - Red Hat OpenShift Pipelines 1.18.0

## Access Configuration

### Kubeconfig Setup
```bash
# Set the kubeconfig path
export KUBECONFIG=/home/lab-user/generated_assets/ocp4/auth/kubeconfig

# Verify access
oc version
```

### Required Permissions
- Cluster admin access for AAP installation
- Project admin access for VM management
- Access to OpenShift Virtualization resources

## OpenShift Virtualization

### Components
- KubeVirt Operator
- HyperConverged Cluster Operator
- Containerized Data Importer

### Storage Configuration
- Storage class: `ocs-storagecluster-cephfs` for file storage
- Storage class: `ocs-storagecluster-ceph-rbd` for database storage

### Network Configuration
- Ingress controller configured
- DNS resolution for cluster domain
- Network policies for component communication

## Integration Points

### Ansible Automation Platform
- AAP Operator deployed in `aap` namespace
- Integration with OpenShift authentication
- Access to OpenShift API for VM management

### Monitoring and Logging
- OpenShift monitoring stack
- Log aggregation
- Metrics collection

## Resource Requirements

### Cluster Resources
- Minimum 8 vCPUs
- Minimum 32GB RAM
- Minimum 100GB storage for database
- Minimum 20GB storage for Automation Controller
- Minimum 10GB storage for Automation Hub

### VM Resources
- CPU and memory allocation based on workload
- Storage allocation from OCS
- Network configuration through OpenShift Virtualization

## Troubleshooting

### Common Commands
```bash
# Check cluster status
oc get nodes
oc get clusterversion

# Check OpenShift Virtualization
oc get csv -n openshift-cnv
oc get hyperconverged -n openshift-cnv

# Check storage
oc get storageclass
oc get pv
oc get pvc

# Check networking
oc get networkpolicies
oc get routes
```

### Log Collection
```bash
# Collect cluster logs
oc adm must-gather

# Check operator logs
oc logs -n openshift-cnv deployment/kubevirt-hyperconverged-operator

# Check VM logs
oc logs -n openshift-cnv deployment/virt-handler
```

## Security Considerations

### Network Policies
- Required policies for AAP components
- VM network isolation
- Service mesh configuration

### Authentication
- OpenShift authentication integration
- Service account configuration
- Token management

## Maintenance

### Updates
- OpenShift cluster updates
- Operator updates
- Storage updates

### Backup and Recovery
- Cluster backup procedures
- VM backup procedures
- Data backup procedures

## References
- [OpenShift Documentation](https://docs.openshift.com/container-platform/4.18/welcome/index.html)
- [OpenShift Virtualization Documentation](https://docs.openshift.com/container-platform/4.18/virt/virt-getting-started.html)
- [OpenShift Container Storage Documentation](https://docs.openshift.com/container-platform/4.18/storage/container_storage_interface/persistent-storage-csi-odf.html)
- [OpenShift Monitoring Documentation](https://docs.openshift.com/container-platform/4.18/monitoring/monitoring-overview.html) 