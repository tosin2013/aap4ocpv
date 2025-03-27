---
layout: post
title: "Implementing Basic VM Lifecycle Operations with AAP"
date: 2024-03-25
categories: [aap, openshift, virtualization]
tags: [ansible, automation, kubevirt, vm-lifecycle]
series: "AAP on OpenShift VM Management"
part: 2
previous: /blog/part1-foundation
next: /blog/part3-advanced-config
---

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [VM Creation](#vm-creation)
4. [VM Operations](#vm-operations)
5. [Resource Management](#resource-management)
6. [Testing](#testing)
7. [Next Steps](#next-steps)

## Introduction
In this second part of our series, we'll explore the basic VM lifecycle operations using Ansible Automation Platform on OpenShift. We'll cover creating, starting, stopping, and deleting VMs, along with basic resource management.

## Prerequisites
Before proceeding, ensure you have:
- AAP 2.5 installed and configured (from Part 1)
- OpenShift Virtualization enabled
- Access to VM templates
- Required credentials configured

## VM Creation
1. **Basic VM Template**
   ```yaml
   # templates/rhel-vm.yml
   apiVersion: kubevirt.io/v1
   kind: VirtualMachine
   metadata:
     name: example-vm
   spec:
     running: false
     template:
       spec:
         domain:
           devices:
             disks:
               - name: containerdisk
                 disk:
                   bus: virtio
               - name: cloudinitdisk
                 disk:
                   bus: virtio
         resources:
           requests:
             memory: 1Gi
             cpu: 1
           limits:
             memory: 2Gi
             cpu: 2
   ```

2. **Creating a VM**
   ```yaml
   # 01-create_vm.yml
   ---
   - name: Create VM
     kubernetes.core.k8s:
       state: present
       definition: "{{ lookup('template', 'templates/rhel-vm.yml') | from_yaml }}"
   ```

## VM Operations
1. **Starting a VM**
   ```yaml
   # 02-start_vm.yml
   ---
   - name: Start VM
     kubernetes.core.k8s:
       state: present
       definition:
         apiVersion: kubevirt.io/v1
         kind: VirtualMachine
         metadata:
           name: "{{ vm_name }}"
         spec:
           running: true
   ```

2. **Stopping a VM**
   ```yaml
   # 03-stop_vm.yml
   ---
   - name: Stop VM
     kubernetes.core.k8s:
       state: present
       definition:
         apiVersion: kubevirt.io/v1
         kind: VirtualMachine
         metadata:
           name: "{{ vm_name }}"
         spec:
           running: false
   ```

3. **Deleting a VM**
   ```yaml
   # 09-delete_vm.yml
   ---
   - name: Delete VM
     kubernetes.core.k8s:
       state: absent
       definition:
         apiVersion: kubevirt.io/v1
         kind: VirtualMachine
         metadata:
           name: "{{ vm_name }}"
   ```

## Resource Management
1. **CPU Management**
   ```yaml
   # 04-modify_cpu.yml
   ---
   - name: Modify VM CPU
     kubernetes.core.k8s:
       state: present
       definition:
         apiVersion: kubevirt.io/v1
         kind: VirtualMachine
         metadata:
           name: "{{ vm_name }}"
         spec:
           template:
             spec:
               domain:
                 cpu:
                   cores: "{{ cpu_cores }}"
   ```

2. **Storage Management**
   ```yaml
   # 05-add_disk.yml
   ---
   - name: Add Disk to VM
     kubernetes.core.k8s:
       state: present
       definition: "{{ lookup('template', 'templates/add-disk.yml') | from_yaml }}"
   ```

## Testing
1. **Verify VM Status**
   ```bash
   oc get vm
   oc get vmi
   ```

2. **Check Resource Usage**
   ```bash
   oc adm top vm
   ```

## Next Steps
In the next post, we'll cover:
- Advanced VM configuration
- Network interface management
- Snapshot operations
- Performance optimization

## Related Resources
- [OpenShift Virtualization VM Management](https://docs.openshift.com/container-platform/4.18/virt/vm_management/vm_management.html)
- [KubeVirt VM Operations](https://kubevirt.io/user-guide/docs/latest/operations/virtual_machines.html)
- [Ansible Kubernetes Collection](https://docs.ansible.com/ansible/latest/collections/kubernetes/core/k8s_module.html) 