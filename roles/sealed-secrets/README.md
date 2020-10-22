# Kubernetes Sealed Secrets

This role will deploy Bitnami's "Sealed Secrets" for Kubernetes on OpenShift. You can read more about sealed secrets [here](https://github.com/bitnami-labs/sealed-secrets). This documentation assumes you are already familiar wih sealed secrets. The role (`sealed-secrets`) requires you to provide your own RSA keypair for the sealed secrets controller to use. There are several advantages to doing this, but mainly we want to be able to create `SealedSecrets` custom resources *before* our target cluster is provisioned (ultimately to support a GitOps methodology for our cluster configuration).

# Using the Role

It is assumed you will leverage this projects existing roles `namespace` and `tls-secrets` to create a namespace and also a TLS secret that contains an RSA keypair for the sealed secrets controller. This secret will be used to encrypt and ultimately decrypt the secrets on the target cluster. As mentioned above, since we are providing this RSA keypair to the controller we can leverage the public key directly to create SealedSecret CRs without calling the Kubernetes API directly on the target cluster.

## Creating the RSA Keypair

To generate the keypair, we will leverage the `openssl` command. This can be done as follows:

```shell
$ openssl req -x509 -nodes -newkey rsa:4096 -keyout /path/to/cluster.key -out /path/to/cluster.crt -subj "/CN=sealed-secret/O=sealed-secret"
```

Ultimately we will base64 encode these files and add them to a TLS secret on the target cluster. The will be achieved through the use of the `tls-secrets` role as shown in the next section.

## Creating Namespace and TLS Secrets

As mentioned in the README.md at the root of this repository, you will need a variables file to provide some information about your cluster to the `post-install.yaml` playbook. An example for the cluster `acm-managed-1` is shown below:

```yaml
---
oc_bin: "/home/chris/bin/oc"
oc_kubeconfig: "/home/chris/upi/acm-managed-1/auth/kubeconfig"

cluster_name: acm-managed-1

namespaces:
  - name: sealed-secrets
```

You will also need to create an Ansible vault that describes the TLS secrets you wish to create for the RSA keypair. For the `crt` and `key` fields, use the `base64` command to encode the files generated above and store the output of the command under the appropriate variable in the vault. An example of using the `base64` command to encode the private key is shown below:

```shell
$ base64 -w0 /path/to/cluster.key
```

Take the output of that command and assign it as the value of the `key` (`tls_secrets.acm-managed-1[].key`) variable in the vault. Be sure to encode and store the `crt` value (`/path/to/cluster.crt`) as well.

```yaml
---
tls_secrets:
  acm-managed-1:
    - name: sealed-secrets-custom-key
      namespace: sealed-secrets
      crt: LS0tLS1CRUdJTiB=
      key: LS0tEtFWS0tLS0tCk=
      labels:
        - key: sealedsecrets.bitnami.com/sealed-secrets-key
          value: active
        - key: manager
          value: controller
        - key: operation
          value: Update
```

The sealed secrets controller will look for RSA keypairs under any secret with the label `sealedsecrets.bitnami.com/sealed-secrets-key: active`.

## Deploying the Sealed Secrets Controller

To deploy the sealed secrets controller, run the `post-install.yaml` playbook as follows (adjust the paths to your vault and cluster variables file accordingly):

```shell
$ ansible-playbook --ask-vault-pass -e @tls-secrets.yaml -e @vars/acm-managed-1.yaml --tags namespaces,tls-secrets,sealed-secrets post-install.yaml
```