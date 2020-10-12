# Adding Managed Clusters to ACM

This role will auto-import any number of Kubernetes clusters into ACM. The role leverages a variable called `acm` that contains information about the hub cluster and the managed clusters we will add to ACM.

# Using this Role

## acm_spoke_clusters Variable

The `acm_spoke_clusters` variable has a reference to the kubeconfig file for the hub cluster and also a list (called `spokes`) that references the name and kubeconfig file for each cluster we are bringing under management. An example is referenced below:

```yaml
acm:
  acm_hub_kubeconfig: /path/to/hub/kubeconfig
  spokes:
    - name: spoke-cluster-1
      kubeconfig: /path/to/spoke-cluster-1/kubeconfig
    - name: spoke-cluster-2
      kubeconfig: /path/to/spoke-cluster-2/kubeconfig
```

## Running in a Playbook

An example playbook that uses the `acm-add-spoke` role is shown below. The variable `acm` is statically defined, but could be generated dynamically at run time.

```yaml
---
- hosts: localhost
  roles:
    - role: acm-add-spoke
      vars:
        acm:
          acm_hub_kubeconfig: /path/to/hub/kubeconfig
          spokes:
            - name: spoke-cluster-1
              kubeconfig: /path/to/spoke-cluster-1/kubeconfig
            - name: spoke-cluster-2
              kubeconfig: /path/to/spoke-cluster-2/kubeconfig
```