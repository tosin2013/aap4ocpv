---
layout: post
title: "System Configuration and Update Management"
date: 2024-03-27
categories: [aap, openshift, virtualization]
tags: [ansible, automation, system-config, updates]
series: "AAP on OpenShift VM Management"
part: 4
previous: /blog/part3-advanced-config
next: /blog/part5-multi-vm
---

## Table of Contents
1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Repository Management](#repository-management)
4. [System Updates](#system-updates)
5. [Application Installation](#application-installation)
6. [Patch Management](#patch-management)
7. [Testing](#testing)
8. [Next Steps](#next-steps)

## Introduction
In this fourth part of our series, we'll focus on system configuration and update management for VMs running on OpenShift. We'll cover repository management, system updates, application installation, and patch management.

## Prerequisites
Before proceeding, ensure you have:
- VMs configured (from Parts 2 and 3)
- Access to Red Hat repositories
- Required subscription credentials
- Network access to repositories

## Repository Management
1. **Adding Repositories**
   ```yaml
   # 11-add_repo.yml
   ---
   - name: Add Repository
     kubernetes.core.k8s_exec:
       namespace: "{{ namespace }}"
       pod: "{{ pod_name }}"
       command: |
         subscription-manager register --username={{ rhn_username }} --password={{ rhn_password }}
         subscription-manager attach --pool={{ pool_id }}
         subscription-manager repos --enable={{ repo_id }}
   ```

2. **Repository Configuration**
   ```yaml
   # Repository configuration example
   repositories:
     - name: rhel-8-for-x86_64-baseos-rpms
       enabled: true
       gpgcheck: true
     - name: rhel-8-for-x86_64-appstream-rpms
       enabled: true
       gpgcheck: true
   ```

## System Updates
1. **Updating All Packages**
   ```yaml
   # 12-install_all_updates.yml
   ---
   - name: Install All Updates
     kubernetes.core.k8s_exec:
       namespace: "{{ namespace }}"
       pod: "{{ pod_name }}"
       command: |
         dnf update -y
         dnf clean all
   ```

2. **Update Strategy**
   ```yaml
   # Update strategy example
   update_strategy:
     schedule: "0 2 * * *"  # Run at 2 AM daily
     reboot: "if-needed"
     notifications:
       email: "admin@example.com"
   ```

## Application Installation
1. **HTTP Server Installation**
   ```yaml
   # 13-install_httpd.yml
   ---
   - name: Install HTTP Server
     kubernetes.core.k8s_exec:
       namespace: "{{ namespace }}"
       pod: "{{ pod_name }}"
       command: |
         dnf install -y httpd
         systemctl enable httpd
         systemctl start httpd
   ```

2. **Enterprise Application Platform**
   ```yaml
   # 14-install-eap.yml
   ---
   - name: Install EAP
     kubernetes.core.k8s_exec:
       namespace: "{{ namespace }}"
       pod: "{{ pod_name }}"
       command: |
         dnf install -y java-11-openjdk-devel
         dnf install -y jboss-eap-8
   ```

## Patch Management
1. **Listing Available Patches**
   ```yaml
   # 31-list_patches.yml
   ---
   - name: List Available Patches
     kubernetes.core.k8s_exec:
       namespace: "{{ namespace }}"
       pod: "{{ pod_name }}"
       command: |
         dnf check-update
   ```

2. **Installing Patches**
   ```yaml
   # 32-install_patches.yml
   ---
   - name: Install Patches
     kubernetes.core.k8s_exec:
       namespace: "{{ namespace }}"
       pod: "{{ pod_name }}"
       command: |
         dnf update -y --security
   ```

## Testing
1. **Verify Repository Access**
   ```bash
   oc exec -it <pod-name> -- dnf repolist
   ```

2. **Check System Updates**
   ```bash
   oc exec -it <pod-name> -- dnf check-update
   ```

3. **Verify Application Installation**
   ```bash
   oc exec -it <pod-name> -- systemctl status httpd
   oc exec -it <pod-name> -- java -version
   ```

## Next Steps
In the next post, we'll cover:
- Multi-VM operations
- Bulk provisioning
- Application stack deployment
- Resource optimization

## Related Resources
- [Red Hat Subscription Management](https://access.redhat.com/documentation/en-us/red_hat_subscription_management)
- [RHEL System Updates](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_basic_system_settings/managing-software-with-dnf_configuring-basic-system-settings)
- [JBoss EAP Documentation](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform)
- [OpenShift Virtualization VM Management](https://docs.openshift.com/container-platform/4.18/virt/vm_management/vm_management.html) 