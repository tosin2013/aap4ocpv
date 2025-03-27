---
layout: post
title: "Setting Up Ansible Automation Platform 2.5 on OpenShift"
date: 2024-03-24
categories: [aap, openshift, virtualization]
tags: [ansible, automation, kubevirt]
series: "AAP on OpenShift VM Management"
part: 1
next: /blog/part2-basic-ops
---

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Environment Setup](#environment-setup)
4. [AAP Installation](#aap-installation)
5. [Configuration](#configuration)
6. [Testing](#testing)
7. [Next Steps](#next-steps)

## Introduction
In this first part of our series, we'll cover the foundational setup of Ansible Automation Platform (AAP) 2.5 on OpenShift. This setup will serve as the basis for all our VM management operations.

## Prerequisites
Before we begin, ensure you have:
- Access to an OpenShift cluster
- OpenShift CLI (`oc`) installed
- `kubectl` installed
- Basic understanding of OpenShift concepts

## Environment Setup
1. **OpenShift Cluster Requirements**
   - Minimum 4 CPU cores
   - 9GB RAM minimum
   - 35GB storage minimum
   - OpenShift 4.x

2. **Resource Requirements**
   ```yaml
   # Example resource requirements
   resources:
     requests:
       cpu: "4"
       memory: "9Gi"
     limits:
       cpu: "8"
       memory: "16Gi"
   ```

## AAP Installation
1. **Create Namespace**
   ```bash
   oc new-project ansible-automation-platform
   ```

2. **Install Operator**
   ```yaml
   # operator-subscription.yaml
   apiVersion: operators.coreos.com/v1alpha1
   kind: Subscription
   metadata:
     name: ansible-automation-platform
     namespace: ansible-automation-platform
   spec:
     channel: 'stable-2.5'
     name: ansible-automation-platform-operator
     source: redhat-operators
     sourceNamespace: openshift-marketplace
   ```

3. **Deploy AAP Instance**
   ```yaml
   # aap-instance.yaml
   apiVersion: aap.ansible.com/v1alpha1
   kind: AnsibleAutomationPlatform
   metadata:
     name: aap-instance
     namespace: ansible-automation-platform
   spec:
     controller:
       disabled: false
     hub:
       disabled: false
     eda:
       disabled: false
   ```

## Configuration
1. **Project Setup**
   ```yaml
   # project-config.yaml
   apiVersion: project.openshift.io/v1
   kind: Project
   metadata:
     name: vm-management
     labels:
       purpose: vm-management
   ```

2. **Credential Configuration**
   ```yaml
   # credentials.yml
   ---
   - name: Configure OpenShift credentials
     kubernetes.core.k8s:
       state: present
       definition:
         apiVersion: v1
         kind: Secret
         metadata:
           name: openshift-credentials
         type: Opaque
         data:
           token: "{{ openshift_token | b64encode }}"
   ```

## Testing
1. **Verify Installation**
   ```bash
   oc get pods -n ansible-automation-platform
   ```

2. **Access AAP UI**
   ```bash
   oc get route -n ansible-automation-platform
   ```

## Next Steps
In the next post, we'll cover:
- Basic VM operations
- Resource management
- Network configuration
- Storage setup

## Related Resources
- [Red Hat Ansible Automation Platform Documentation](https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform)
- [OpenShift Virtualization Documentation](https://docs.openshift.com/container-platform/4.18/virt/virt-getting-started.html)
- [KubeVirt Documentation](https://kubevirt.io/user-guide/docs/latest/administration/intro-to-kubevirt.html) 