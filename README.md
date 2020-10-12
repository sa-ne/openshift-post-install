# Post Installation/Day Two Tasks for OpenShift 4

These playbooks are designed to automate common "day two" OpenShift configuration tasks. This repo is still a work in progress.

## Background

The following capabilities are included:

|Role|Capabilities|
|:---|:---|
|acm-add-spoke|Add managed clusters to ACM.|
|apiserver|Replace TLS certificate.|
|ingress-operator|Replace default IngressController TLS certificate. Adjust replicas. Adjust pod placement and apply infrastructure node tolerations.|
|install-operators|Create Namespace, OperatorGroup and Subscription resources for a list of operators.|
|label-nodes|Assign a set of labels and taints to nodes.|
|localvolumes|Use the Local Storage Operator to create LocalVolume resources and verify associated PersistentVolumes.|
|oauth-ldap|Configure the cluster OAuth resource to leverage LDAP. Assign a list of users the cluster-admin role.|
|proxy|Create additional CA trust ConfigMap resource. Update cluster Proxy resource to include additional CA trust bundle.|
|registry|Create registry PVC for persistent storage. Adjust replicas. Adjust pod placement and apply infrastructure node tolerations.|
|storagecluster|Verify OCS, Rook/Ceph and Noobaa operator deployments. Create StorageCluster resource (OCS deployment). Apply universal toleration for CSI DaemonSet resources.|
|tls-secret|Create TLS Secret resources.|
|worker-mcp|Create additional user defined worker MachineConfigPools (e.g. nfra and ocs).|

# Using the Playbooks

Each grouping of tasks is broken out into an Ansible role. See the invidual README.md files for each role for more information on usage.

## Vaults

When using the `oauth-ldap` role, you will need to create at least one vault (`vault.yaml`) which supports the following variables:

|Variable|Description|
|:---|:---|
|oauth\_ldap\_bind\_user|LDAP bind username|
|oauth_ldap_bind_pass|LDAP bind password|

Example vault contents:

```yaml
---
oauth_ldap_bind_user: uid=ocp-service-account,cn=users,cn=accounts,dc=umbrella,dc=local
oauth_ldap_bind_pass: changeme
```

If you plan on customizing the TLS certificates in your cluster, create another vault (`tls-secrets.yaml` for example) which supports the following variables:

|Variable|Description|
|:---|:---|
|tls_secrets|A list of dictionaries container information about TLS certificates (name, key, certificate, etc)

Example vault contents:

```yaml
tls_secrets:
- name: api-cert
  namespace: openshift-config
  cert: |
    -----BEGIN CERTIFICATE-----
    p7WhF8pNtu0hebflAGSSufP7piMFTfnue32vuEUZ4bcW/VUFZRGXl8LKNK5j39Ih
    S4eahQGeyqoPPi7mlowg2lKF5u/MQg/RtOOTDSOiZqINUoj1tHV7NAlGQIJ7i3gZ
    ATKADskOsYX7BBy67uAQ+tf4a7csBw==
    ...
    -----END CERTIFICATE-----
    -----BEGIN CERTIFICATE-----
    p7WhF8pNtu0hebflAGSSufP7piMFTfnue32vuEUZ4bcW/VUFZRGXl8LKNK5j39Ih
    S4eahQGeyqoPPi7mlowg2lKF5u/MQg/RtOOTDSOiZqINUoj1tHV7NAlGQIJ7i3gZ
    ATKADskOsYX7BBy67uAQ+tf4a7csBw==
    ...
    -----END CERTIFICATE-----
  key: |
    -----BEGIN PRIVATE KEY-----
    p7WhF8pNtu0hebflAGSSufP7piMFTfnue32vuEUZ4bcW/VUFZRGXl8LKNK5j39Ih
    S4eahQGeyqoPPi7mlowg2lKF5u/MQg/RtOOTDSOiZqINUoj1tHV7NAlGQIJ7i3gZ
    ATKADskOsYX7BBy67uAQ+tf4a7csBw==
    ...
    -----END PRIVATE KEY-----
- name: wildcard-cert
  namespace: openshift-ingress
  cert: |
    -----BEGIN CERTIFICATE-----
    p7WhF8pNtu0hebflAGSSufP7piMFTfnue32vuEUZ4bcW/VUFZRGXl8LKNK5j39Ih
    S4eahQGeyqoPPi7mlowg2lKF5u/MQg/RtOOTDSOiZqINUoj1tHV7NAlGQIJ7i3gZ
    ATKADskOsYX7BBy67uAQ+tf4a7csBw==
    ...
    -----END CERTIFICATE-----
    -----BEGIN CERTIFICATE-----
    p7WhF8pNtu0hebflAGSSufP7piMFTfnue32vuEUZ4bcW/VUFZRGXl8LKNK5j39Ih
    S4eahQGeyqoPPi7mlowg2lKF5u/MQg/RtOOTDSOiZqINUoj1tHV7NAlGQIJ7i3gZ
    ATKADskOsYX7BBy67uAQ+tf4a7csBw==
    ...
    -----END CERTIFICATE-----
  key: |
    -----BEGIN PRIVATE KEY-----
    p7WhF8pNtu0hebflAGSSufP7piMFTfnue32vuEUZ4bcW/VUFZRGXl8LKNK5j39Ih
    S4eahQGeyqoPPi7mlowg2lKF5u/MQg/RtOOTDSOiZqINUoj1tHV7NAlGQIJ7i3gZ
    ATKADskOsYX7BBy67uAQ+tf4a7csBw==
    ...
    -----END PRIVATE KEY-----
```