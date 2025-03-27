---
layout: post
title: "Advanced VM Configuration and Resource Management"
date: 2024-03-26
categories: [aap, openshift, virtualization]
tags: [ansible, automation, kubevirt, advanced-config]
series: "AAP on OpenShift VM Management"
part: 3
previous: /blog/part2-basic-ops
next: /blog/part4-system-config
---

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Advanced CPU Configuration](#advanced-cpu-configuration)
4. [Storage Backend Integration](#storage-backend-integration)
5. [Network Interface Management](#network-interface-management)
6. [Snapshot Operations](#snapshot-operations)
7. [Testing](#testing)
8. [Next Steps](#next-steps)

## Introduction
In this third part of our series, we'll dive into advanced VM configuration options, including CPU optimization, storage backend integration, network interface management, and snapshot operations.

## Prerequisites
Before proceeding, ensure you have:
- Basic VM operations working (from Part 2)
- Access to storage backends (ODF, NetApp Trident, or LVMS)
- Network policies configured
- Sufficient cluster resources

## Advanced CPU Configuration
1. **CPU Pinning**
   ```yaml
   # templates/vm-cpumem.yml
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
             dedicatedCpuPlacement: true
             features:
               - name: "{{ feature_name }}"
                 policy: "{{ policy }}"
   ```

2. **NUMA Configuration**
   ```yaml
   # NUMA configuration example
   spec:
     template:
       spec:
         domain:
           cpu:
             numa:
               guestMappingPassthrough: {}
   ```

## Storage Backend Integration
1. **OpenShift Data Foundation**
   ```yaml
   # templates/add-datavolume.yml
   apiVersion: cdi.kubevirt.io/v1beta1
   kind: DataVolume
   metadata:
     name: "{{ dv_name }}"
   spec:
     source:
       pvc:
         namespace: openshift-storage
         name: "{{ pvc_name }}"
     pvc:
       storageClassName: ocs-storagecluster-ceph-rbd
   ```

2. **NetApp Trident**
   ```yaml
   # NetApp storage class example
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: netapp-storage
   provisioner: csi.trident.netapp.io
   parameters:
     backendType: "ontap-nas"
   ```

3. **LVMS Configuration**
   ```yaml
   # LVMS storage class example
   apiVersion: storage.k8s.io/v1
   kind: StorageClass
   metadata:
     name: lvms-storage
   provisioner: topolvm.io
   parameters:
     storage: "10Gi"
   ```

## Network Interface Management
1. **Adding Network Interfaces**
   ```yaml
   # 06-add_nic.yml
   ---
   - name: Add Network Interface
     kubernetes.core.k8s:
       state: present
       definition: "{{ lookup('template', 'templates/add-nic.yml') | from_yaml }}"
   ```

2. **Network Policies**
   ```yaml
   # Network policy example
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: vm-network-policy
   spec:
     podSelector:
       matchLabels:
         kubevirt.io/vm: "{{ vm_name }}"
     policyTypes:
       - Ingress
       - Egress
   ```

## Snapshot Operations
1. **Creating Snapshots**
   ```yaml
   # 07-take_snapshot.yml
   ---
   - name: Create VM Snapshot
     kubernetes.core.k8s:
       state: present
       definition: "{{ lookup('template', 'templates/vm-snapshot.yml') | from_yaml }}"
   ```

2. **Restoring Snapshots**
   ```yaml
   # 08-restore_snapshot.yml
   ---
   - name: Restore VM Snapshot
     kubernetes.core.k8s:
       state: present
       definition: "{{ lookup('template', 'templates/restore-snapshot.yml') | from_yaml }}"
   ```

## Testing
1. **Verify Storage Integration**
   ```bash
   oc get storageclass
   oc get pvc
   ```

2. **Check Network Configuration**
   ```bash
   oc get networkpolicies
   oc get pods -l kubevirt.io/vm
   ```

3. **Test Snapshot Operations**
   ```bash
   oc get vmsnapshots
   oc get vmsnapshotcontents
   ```

## Next Steps
In the next post, we'll cover:
- System configuration
- Repository management
- Update strategies
- Application installation

## Related Resources
- [OpenShift Data Foundation](https://docs.openshift.com/container-platform/4.18/storage/container_storage_interface/persistent-storage-csi-odf.html)
- [NetApp Trident](https://netapp-trident.readthedocs.io/en/stable-v23.10/kubernetes/operations/tasks/backends/ontap/nas.html)
- [LVMS Documentation](https://github.com/topolvm/topolvm)
- [KubeVirt Network Configuration](https://kubevirt.io/user-guide/docs/latest/operations/virtual_machines.html#network-configuration) 