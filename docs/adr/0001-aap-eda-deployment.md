# ADR 0001: Ansible Automation Platform EDA Deployment

## Status
Proposed

## Context
We need to deploy Ansible Automation Platform (AAP) on OpenShift with a focus on Event-Driven Ansible (EDA) capabilities. The deployment should be automated and follow best practices for OpenShift deployments.

## Decision
We will deploy AAP using the following approach:

1. **Namespace Setup**
   - Create a dedicated namespace `aap` for AAP components
   - Ensure proper RBAC permissions

2. **Operator Installation**
   - Install the AAP operator using OLM
   - Use the latest stable version compatible with OpenShift 4.18

3. **AAP Instance Configuration**
   - Deploy a simplified AAP instance with the following configuration:
     ```yaml
     cat <<EOF > /tmp/aap-instance.yaml
     apiVersion: aap.ansible.com/v1alpha1
     kind: AnsibleAutomationPlatform
     metadata:
       name: ansible
       namespace: aap
     spec:
       # Platform
       image_pull_policy: IfNotPresent
       # Components
       controller:
         disabled: false
       eda:
         disabled: true
       hub:
         disabled: false
         storage_type: file
         file_storage_storage_class: ocs-storagecluster-cephfs
         file_storage_size: 100Gi
       lightspeed:
         disabled: true
    EOF
     ```
     oc apply -f /tmp/aap-instance.yamlx

4. **Storage Configuration**
   - Use OpenShift Container Storage (OCS) for persistent storage
   - Storage class: `ocs-storagecluster-cephfs`
   - Storage size: 100Gi for hub storage

5. **Component Selection**
   - Enable only Event-Driven Ansible (EDA)
   - Disable other components (Controller, Hub, Lightspeed)
   - This simplifies the deployment and reduces resource requirements

6. **Verification**
   - Verify operator installation
   - Check AAP instance status
   - Validate EDA component deployment
   - Verify routes and access

## Consequences

### Positive
- Simplified deployment focusing on EDA capabilities
- Reduced resource requirements by disabling unused components
- Automated deployment process using Ansible
- Clear verification steps

### Negative
- Limited to EDA functionality only
- No access to Controller or Hub features
- Requires proper storage configuration
- May need additional configuration for specific EDA use cases

## Implementation Notes
- Ensure proper storage class availability
- Monitor resource usage
- Plan for future scaling if needed
- Document any specific EDA configurations required 