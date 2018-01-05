ansible-role-consul-ca
======================

This role is used in [Kubernetes the Not So Hard Way With Ansible (at Scaleway) - Part 8 - Ingress with Traefik](https://www.tauceti.blog/post/kubernetes-the-not-so-hard-way-with-ansible-at-scaleway-part-8/). It creates a CA (certificate authority) for Consul certificates and the certificates needed to secure communication between the Consul daemons and Traefik proxy used in the blog post mentioned above. This role is meant to be used for exactly this purpose but it should be useable for other use cases too as it has no dependencies besides [CFSSL](https://github.com/cloudflare/cfssl) PKI toolkit binaries installed (which you would need anyway's).

Versions
--------

I tag every release and try to stay with [semantic versioning](http://semver.org) (well kind of...). If you want to use the role I recommend to checkout the latest tag. The master branch is basically development while the tags mark stable releases. But in general I try to keep master in good shape too. A tag `r1.0.0_v1.0.0` means this is release 1.0.0 of this role and it's meant to be used with Consul version 1.0.0. If the role itself changes `rX.Y.Z` will increase. If the Consul version changes `vX.Y.Z` will increase. This allows to tag bugfixes and new major versions of the role while it's still developed for a specific Consul release.

Changelog
---------
**r1.0.0_v1.0.2**

- only Git tag update for Consul v1.0.2

**r1.0.0_v1.0.0**

- Initial Ansible role.

Requirements
------------

This role needs [CFSSL](https://github.com/cloudflare/cfssl) PKI toolkit binaries installed. You can use [ansible-role-cfssl](https://github.com/githubixx/ansible-role-cfssl) to install CFSSL locally on your machine. If you want to store the generated certificates and CA's locally or on a network share specify the role variables below in `group_vars/all.yml` e.g.

Role Variables
--------------

This playbook has quite a few variables. But that's mainly information needed for the certificates.

```
# Where to store the CA and certificate files
consul_ca_conf_directory: "{{ '~/consul/ssl' | expanduser }}"
consul_ca_certificate_owner: "root"
consul_ca_certificate_group: "root"
```

`consul_ca_conf_directory` tells Ansible where to store the CA's and certificate files. To enable Ansible to read the files in later runs you should specify a user and group in `consul_ca_certificate_owner` / `consul_ca_certificate_group` which has permissions (in most cases this will be the user you use on your workstation).

```
# Expiry for Consul root certificate
ca_consul_expiry: "87600h"
```
`ca_consul_expiry` sets the expiry date for Consul root CA.

```
# Certificate authority for Consul certificates
ca_consul_csr_cn: "Consul"
ca_consul_csr_key_algo: "rsa"
ca_consul_csr_key_size: "2048"
ca_consul_csr_names_c: "DE"
ca_consul_csr_names_l: "The_Internet"
ca_consul_csr_names_o: "Consul"
ca_consul_csr_names_ou: "BY"
ca_consul_csr_names_st: "Bayern"
```
This variables are used to create the CSR (certificate signing request) of the CA (certificate authority) which we use to sign certifcates for Consul.

```
# CSR parameter for Consul certificate
consul_csr_cn: "server.dc1.consul"
consul_csr_key_algo: "rsa"
consul_csr_key_size: "2048"
consul_csr_names_c: "DE"
consul_csr_names_l: "The_Internet"
consul_csr_names_o: "Consul"
consul_csr_names_ou: "BY"
consul_csr_names_st: "Bayern"
```

This variables are used to create the CSR (certificate signing request) for the certificate that is used to secure the Consul communication. One note about `consul_csr_cn`: Consul want's the certs and the servers to be `server.<data_center>.consul` as common name (cn) value (also see [Consul: Adding TLS to Consul using Self Signed Certificates](http://russellsimpkins.blogspot.de/2015/10/consul-adding-tls-using-self-signed.html). So in Consul's `config.json` you have a parameter `datacenter` which is often `dc1` by default. E.g. if you have `"datacenter": "par1"` in Consul's configuration specified the first value in `consul_csr_cn` should be `server.par1.consul`. You can specify additional common names afterwards separated by comma's. Even wildcards are possible e.g. `consul_csr_cn: "server.dc1.consul,*.example.com"` but still the first value should be as mentioned.

Example Playbook
----------------

```
- hosts: consul_instances

  roles:
    - githubixx.consul-ca
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
