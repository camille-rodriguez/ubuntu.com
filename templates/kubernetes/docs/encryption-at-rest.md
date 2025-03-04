---
wrapper_template: "kubernetes/docs/base_docs.html"
markdown_includes:
  nav: "kubernetes/docs/shared/_side-navigation.md"
context:
  title: "Encryption at rest"
  description: How to enable encryption-at-rest using Vault
keywords: juju, kubernetes, cdk, security, encryption, vault
tags: [operating, security]
sidebar: k8smain-sidebar
permalink: encryption-at-rest.html
layout: [base, ubuntu-com]
toc: False
---

**Kubernetes** has the concept of [secrets][] for managing sensitive information
needed by a cluster, such as usernames and passwords, encryption keys, etc.
Secrets can be managed independently of the pod(s) which need them and can be
made available to the pods that require them as needed.

By default, the secret data is stored in plaintext in **etcd**. **Kubernetes** does
support [encryption at rest][] for the data in **etcd**, but the key for that
encryption is stored in plaintext in the config file on the master nodes.  To
protect this key at rest, **CDK** can use [HashiCorp's Vault][] and [VaultLocker][] to
securely generate, share, and configure the encryption key used by **Kubernetes**.

## Using Encryption-at-Rest with CDK

To enable encryption-at-rest for **CDK**, simply deploy the [Vault charm][] (as
well as a database backend for it), and relate it to `kubernetes-master` via
the `vault-kv` relation endpoint.  The easiest way to do this is to deploy **CDK**
with the following overlay:

```yaml
applications:
  vault:
    charm: cs:~openstack-charmers-next/vault
    num_units: 1
  percona-cluster:
    charm: cs:percona-cluster
    num_units: 1
relations:
  - ['vault', 'percona-cluster']
  - ['vault:secrets', 'kubernetes-master:vault-kv']
```

To deploy **CDK** with this overlay - [download it][cdk-vault-overlay]), save it as, e.g.,
`cdk-vault-overlay.yaml`, and deploy with:

```
juju deploy charmed-kubernetes --overlay cdk-vault-overlay.yaml
```

Once **Vault** comes up (and any time it is rebooted), you will need to [unseal][]
it.

CDK will then automatically enable encryption for the secrets data stored in
**etcd**.

## Known Issues

This does not work on **LXD** at this time, due to security limitations preventing
charms from acquiring and managing the block devices and file systems needed to
implement this.  In the future, support for KMS, or encryption-as-a-service,
will remove this restriction.  In the meantime, **LXD** deployments can make use of
encryption at the level of the **LXD** storage pool, or even full-disk-encryption
on the host machine.

[cdk-vault-overlay]: https://raw.githubusercontent.com/juju-solutions/kubernetes-docs/master/assets/cdk-vault-overlay.yaml
[secrets]: https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/
[encryption at rest]: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
[HashiCorp's Vault]: https://www.vaultproject.io/
[VaultLocker]: https://github.com/openstack-charmers/vaultlocker
[Vault charm]: https://jujucharms.com/u/openstack-charmers-next/vault/
[unseal]: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/app-vault.html#initialize-and-unseal-vault
