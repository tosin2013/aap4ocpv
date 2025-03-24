---
layout: post
title: "Quick Start: Setting Up Ansible Automation Platform for VM Management"
date: 2024-03-25
categories: [aap, openshift, virtualization]
tags: [ansible, automation, setup, configuration, quickstart]
series: "AAP on OpenShift VM Management"
---

## Table of Contents
1. [Introduction](#introduction)
2. [Architecture Overview](#architecture-overview)
3. [Prerequisites](#prerequisites)
4. [Initial Setup](#initial-setup)
5. [AAP Configuration Steps](#aap-configuration-steps)
6. [Testing the Setup](#testing-the-setup)
7. [Next Steps](#next-steps)

## Introduction
This quick start guide provides step-by-step instructions for setting up Ansible Automation Platform (AAP) on OpenShift for virtual machine management. This guide serves as the entry point for the complete series and focuses on getting you up and running quickly.

> **Note**: This is a quick start guide. For detailed information about AAP architecture, components, and advanced configurations, refer to the subsequent parts in this series.

## Architecture Overview
The following diagrams illustrate the key components and their interactions in our AAP setup for VM management:

### Component Architecture
![AAP Component Architecture](images/aap-architecture.png)
*Figure 1: High-level architecture of AAP components and their interactions*

### VM Management Flow
![VM Management Flow](images/vm-management-flow.png)
*Figure 2: Flow diagram showing VM lifecycle management process*

### Integration Points
![Integration Points](images/integration-points.png)
*Figure 3: Integration points between AAP, OpenShift, and KubeVirt*

## Quick Start Overview
This guide will help you:
1. Set up AAP on OpenShift
2. Configure basic components
3. Create initial job templates
4. Start managing VMs

For a comprehensive understanding of AAP and VM management, follow the complete series starting with [Part 1: Foundation and Architecture](part1-foundation.md).

## Prerequisites
Before starting, ensure you have:
- Access to an OpenShift cluster
- OpenShift CLI (`oc`) installed
- Basic understanding of Ansible and KubeVirt
- Red Hat subscription for AAP 2.5
- To get password for `ansible-admin` user, run `oc get secret -n aap ansible-admin-password -o jsonpath='{.data.password}' | base64 --decode`

## Initial Setup

### 1. Configure Execution Environment
See `ansible-navigator.yml`:
```yaml
---
ansible-navigator:
  execution-environment:
    container-engine: podman
    enabled: true
    image: quay.io/weishen/ocpv-ee:latest
    pull:
      policy: never
  logging:
    level: debug
```
* Under Execution Environment click on -> `Create Execution Environment`
* Click on -> `Create Execution Environment`
* Fill in the following:
  * Name: `ocpv-ee`
  * Image: `quay.io/weishen/ocpv-ee:latest`
  * Pull: `Always pull container before running`
  

### Execution Environment Architecture
![Execution Environment](images/execution-environment.png)
*Figure 4: Execution environment configuration and components*

### 2. Configure project in AAP
* Fork [aap4ocpv](https://github.com/tosin2013/aap4ocpv) then point the SCM URL to your fork
* Navigate to -> `Projects`
* Click on -> `Create Project`
* Fill in the following:
  * Name: `ocpv-project`
  * SCM Type: `Git`
  * SCM URL: `https://github.com/tosin2013/aap4ocpv.git`
  * SCM Branch: `main`

### 2. Configure Project Creation
Create `create_project.yml`:
```yaml
- name: Hello World Sample
  hosts: localhost
  vars:
    openshift_fqdn: https://api.ocp4.example.com:6443
    openshift_user: test1
    openshift_password: Redhat!23

  tasks:
    - name: Login to OpenShift
      redhat.openshift.openshift_auth:
        host: "{{ openshift_fqdn }}"
        username: "{{ openshift_user }}"
        password: "{{ openshift_password }}"
        validate_certs: false
      register: openshift_auth_results

    - name: Create namespace
      redhat.openshift.k8s:
        api_key: "{{ openshift_auth_results.openshift_auth.api_key }}"
        host: "{{ openshift_fqdn }}"
        validate_certs: false
        state: present
        definition:
          apiVersion: project.openshift.io/v1
          kind: Project
          metadata:
            name: wm-testing
```

### 3. Configure KubeVirt Inventory
Create `inventory.kubevirt.yml`:
```yaml
---
plugin: redhat.openshift_virtualization.kubevirt
host: https://api.ocp4.example.com:6443
validate_certs: false
```

### 4. Configure Kubernetes Credentials
Create `k8s_custom_creds.yml`:
```yaml
---
fields:
  - type: string
    id: kube_api
    label: Kubernetes API Endpoint
  - type: string
    id: kube_token
    label: Kubernetes API Token
    secret: true
required:
  - kube_api
  - kube_token
---
env:
  K8S_AUTH_HOST: "{{ kube_api }}"
  K8S_AUTH_API_KEY: "{{ kube_token }}"
```

## AAP Configuration Steps

### 1. Create Organization
1. Navigate to Administration → Organizations
2. Click "Add" button
3. Fill in:
   - Name: "vm-management"
   - Description: "Organization for VM management"

### 2. Configure Credentials
1. Navigate to Resources → Credentials
2. Click "Add" button
3. Select "Kubernetes Bearer Token"
4. Fill in:
   - Name: "kubevirt-credentials"
   - Organization: "vm-management"
   - Kubernetes API Endpoint: Your OpenShift API endpoint
   - Kubernetes API Token: Your service account token
   - Description: "Credentials for KubeVirt operations"

### 3. Create Project
1. Navigate to Resources → Projects
2. Click "Add" button
3. Fill in:
   - Name: "vm-management"
   - Organization: "vm-management"
   - SCM Type: "Git"
   - SCM URL: Your Git repository URL
   - SCM Branch: "main"
   - Update Options: Check "Clean" and "Delete on Update"

### 4. Configure Inventory
1. Navigate to Resources → Inventories
2. Click "Add" button
3. Fill in:
   - Name: "kubevirt-inventory"
   - Organization: "vm-management"
   - Variables: Paste contents of `inventory.kubevirt.yml`
   - Source: "Custom Script"
   - Update Options: Check "Overwrite" and "Update on Launch"

### 5. Create Execution Environment
1. Navigate to Administration → Execution Environments
2. Click "Add" button
3. Fill in:
   - Name: "ee-vm-management"
   - Image: "quay.io/weishen/ocpv-ee:latest"
   - Pull: "Never pull container before running"
   - Resource limits:
     - CPU: 1
     - Memory: 2Gi
   - Organization: "vm-management"
   - Description: "Execution environment for VM management"

## Testing the Setup

### 1. Verify Project Sync
1. Navigate to Resources → Projects
2. Select "vm-management"
3. Click "Sync" button
4. Check job status

### 2. Test Inventory
1. Navigate to Resources → Inventories
2. Select "kubevirt-inventory"
3. Click "Sources" tab
4. Click "Sync" button
5. Verify hosts are discovered

### 3. Test Credentials
1. Navigate to Resources → Credentials
2. Select "kubevirt-credentials"
3. Click "Test" button
4. Verify connection is successful

## Next Steps
1. Review [Part 1: Foundation and Architecture](part1-foundation.md) for detailed understanding
2. Proceed to [Part 2: Basic Operations](part2-basic-ops.md) for VM lifecycle management
3. Check [Troubleshooting Guide](troubleshooting.md) if you encounter issues

## Related Resources
- [Ansible Automation Platform Documentation](https://docs.ansible.com/automation-controller/latest/html/userguide/index.html)
- [OpenShift Virtualization Guide](https://docs.openshift.com/container-platform/4.18/virt/vm_management/vm_management.html)
- [KubeVirt Documentation](https://kubevirt.io/user-guide/docs/latest/operations/index.html) 