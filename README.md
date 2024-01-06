Ansible script to create k8s cluster via qemu vm's

This role requires these variables to exist in inventory:

# 1. Users
```yaml
users: 
- name: denis
  full_name: Denis
  passwd: $6$rounds=2048$ababababab
  pub_key: ssh-rsa KEY EMAIL
```

The passwd and pub_key parameters take your hashed password and your SSH public key so that you are able to access the server after provisioning.
To generate a new password hash, use mkpasswd -m sha-512 -R 2048 (Included in the whois package).

# 2. VMs
```yaml
virtual_machines: 
- name: u18-svr-001
  cpu: 1
  mem: 1024
  disk: 10G
  bridge: br10
```
This is a list of virtual machines that should be built.

Bridge: this is the name of the bridge device that references the vlan the server will be connected to. This must already exist on the KVM host.

The virtual machine can also be built with a static IP address:

```yaml
- name: u18-svr-002
  cpu: 1
  mem: 512 
  disk: 5G
  bridge: br10 
  net: 
    ip: 10.11.12.13/24
    gateway: 10.11.12.1 
    domain: gablogianartcollection.org
    dns: 
      - 1.1.1.1
      - 9.9.9.9
```

This variable takes exactly one IP address, domain, and gateway; and two or more DNS servers.

Default Values
If no value is supplied, the default settings will be used:

```yaml
CPU: 1 core
Memory: 512 MB
Disk: 5GB .qcow image
Bridge: Default libvirt NAT network
Network: DHCP
```

How it works
This role works by downloading the Ubuntu 20.04 cloud image, then converting it to Copy-On-Write format and cloning it for each virtual machine defined.

When VMs are launched, they are given a small secondary disk image that includes a cloud-config file. This file is generated from a template that includes all the users and system parameters that are defined in inventory. There is also a network-config file that contains network metadata in Netplan-V2 format.

Before running this, ensure that networking is correctly configured on the KVM hosts.