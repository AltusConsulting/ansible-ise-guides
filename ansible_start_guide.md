# Ansible - Getting started

## Introduction

It is possible to use Ansible to interact with Cisco Identity Services Engine (ISE) API.

## Goal

The goals of this guide are:

1. Install the Cisco ISE Ansible collection
2. Execute a playbook

## Installation

`ansible-galaxy collection install cisco.ise`

## Usage

The collection needs the following variables defined to connect to ISE:

- **ise_hostname**: ISE IP Address or Fully Qualified Domain Name (FQDN)
- **ise_username**: ISE administrative username
- **ise_password**: ISE password
- **ise_verify**: Boolean variable to verify the TLS certificate.

## Network device

The following example retrieves all the network devices from ISE:

### File: network_device.yml

```yaml
- hosts: ise_servers
  gather_facts: no
  tasks:
    - name: Get all network devices
      cisco.ise.network_device_info:
        ise_hostname: 198.18.133.27
        ise_username: admin
        ise_password: C1sco12345
        ise_verify: False
```

### File: hosts

```ini
[ise_servers]
198.18.133.27
```

### CLI

```cli
ansible-playbook -i hosts network_device.yml
```

## Documentation

The documentation for all the available modules can be found [here](https://ciscoise.github.io/ansible-ise/).
