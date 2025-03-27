# Ansible Automation Platform Prerequisites

## Overview
This document outlines the prerequisites and installation process for Ansible Automation Platform (AAP) 2.5 on OpenShift. The installation is performed using the AAP Operator, which manages the lifecycle of all AAP components.

## Cluster Requirements

### OpenShift Version
- OpenShift 4.14 or later
- OpenShift Virtualization 4.18.0 or later

### Resource Requirements
- Minimum 8 vCPUs
- Minimum 32GB RAM
- Minimum 100GB storage for database
- Minimum 20GB storage for Automation Controller
- Minimum 10GB storage for Automation Hub

### Storage Requirements
- Storage class: `ocs-storagecluster-cephfs` for file storage
- Storage class: `ocs-storagecluster-ceph-rbd` for database storage

### Network Requirements
- Ingress controller configured
- DNS resolution for cluster domain
- Network policies allowing communication between AAP components

## Installation Process

### 1. Prerequisites
- OpenShift CLI (oc) installed
- Cluster admin access
- Storage classes available
- Namespace created
- `pip3 install kubernetes openshift`
- `ansible-galaxy collection install  kubernetes.core`

### 2. Operator Installation
- Create OperatorGroup in the target namespace
- Create Subscription for AAP Operator
- Wait for operator to be ready

### 3. AAP Instance Creation
- Create AnsibleAutomationPlatform custom resource
- Configure resource requirements for components
- Configure storage requirements
- Wait for instance to be ready

### 4. Validation
- Verify operator status
- Verify AAP instance status
- Verify component deployments
- Verify routes and access

## Troubleshooting

### Common Issues
1. Operator Installation
   ```bash
   oc get csv -n aap
   oc describe csv aap-operator -n aap
   ```

2. AAP Instance
   ```bash
   oc get ansibleautomationplatform -n aap
   oc describe ansibleautomationplatform aap -n aap
   ```

3. Component Status
   ```bash
   oc get automationcontroller -n aap
   oc get automationhub -n aap
   oc get eda -n aap
   ```

4. Pod Status
   ```bash
   oc get pods -n aap
   oc describe pod <pod-name> -n aap
   oc logs <pod-name> -n aap
   ```

### Maintenance Commands
- Check operator logs:
  ```bash
  oc logs -n aap deployment/aap-operator
  ```
- Check component logs:
  ```bash
  oc logs -n aap deployment/automation-controller
  oc logs -n aap deployment/automation-hub
  oc logs -n aap deployment/eda-server
  ```
- Check routes:
  ```bash
  oc get routes -n aap
  ```

## Notes
- The namespace must be set to `aap` in the variables file
- Resource requirements can be adjusted based on workload needs
- Storage requirements can be adjusted based on content needs
- Network policies may need to be configured based on security requirements

## References
- [OpenShift Virtualization Installation Guide](https://docs.openshift.com/container-platform/4.18/virt/install/installing-virt.html)
- [OpenShift Container Storage Documentation](https://docs.openshift.com/container-platform/4.18/storage/container_storage_interface/persistent-storage-csi-rbd.html)
- [Ansible Automation Platform Installation Guide](https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.5/html/installing_and_configuring_ansible_automation_platform_on_openshift/index)
- [OpenShift Monitoring Documentation](https://docs.openshift.com/container-platform/4.18/monitoring/monitoring-overview.html) 