# Network Device Management

## Introduction

It is possible to use the Cisco Identity Services Engine (ISE) to manage network devices.

## Goal

The goals of this guide are:

1. Create / Read / Update / Delete networkdevicegroup
2. Create / Read / Update / Delete networkdevice

## Pre-requisites

For this guide, it is needed that the engineer has basic knowledge of:

- [Ansible Getting Started Guide](./ansible-start-guide.md)

## Network Device Group

_Network Device Groups_ are managed using the [network_device_group_info](https://ciscoise.github.io/ansible-ise/main/html/plugins/network_device_group_info_module.html) and [network_device_group](https://ciscoise.github.io/ansible-ise/main/html/plugins/network_device_group_module.html) Ansible actions.

### network_device_group_info

This module is used to retrieve _Network Device Groups_ information. When the `id` parameter is passed, it will only retrieve one element, if not, then it will retrieve all the _Network Device Groups_.

The following example retrieves all _Network Device Groups_ and then just one identified by id `70e07b60-8bff-11e6-996c-525400b48521`.

```yaml
- name: Get all Network Device Group
  cisco.ise.network_device_group_info:
    ise_hostname: "{{ise_hostname}}"
    ise_username: "{{ise_username}}"
    ise_password: "{{ise_password}}"
    ise_verify: "{{ise_verify}}"
    page: 1
    size: 20
    sortasc: name
    filter: []
    filterType: AND

- name: Get all Network Device Group
  cisco.ise.network_device_group_info:
    ise_hostname: "{{ise_hostname}}"
    ise_username: "{{ise_username}}"
    ise_password: "{{ise_password}}"
    ise_verify: "{{ise_verify}}"
    id: 70e07b60-8bff-11e6-996c-525400b48521
```

### network_device_group

This action is used to create, update or delete a _Network Device Group_, where the action depends on the `state` parameter. The **absent** parameter will delete a _Network Device Group_ and the state **present** will create or update a _Network Device Group_, depending if the Network Device Group already existed based on the provided ID or name.

The following example creates a _Network Device Group_ and then deletes another based on the ID.

```yaml
- name: Create
  cisco.ise.network_device_group:
    ise_hostname: "{{ise_hostname}}"
    ise_username: "{{ise_username}}"
    ise_password: "{{ise_password}}"
    ise_verify: "{{ise_verify}}"
    state: present
    description: "..."
    name: Device Type#All Device Types#SDWAN
    othername: Device Type

- name: Delete by id
  cisco.ise.network_device_group:
    ise_hostname: "{{ise_hostname}}"
    ise_username: "{{ise_username}}"
    ise_password: "{{ise_password}}"
    ise_verify: "{{ise_verify}}"
    state: absent
    id: 6c4c3fd0-c95d-11eb-aee6-62e2dbfdcf7c
```

## Network Device

Network Device are managed using the [network_device_group_info](https://ciscoise.github.io/ansible-ise/main/html/plugins/network_device_info_module.html) and [network_device_group](https://ciscoise.github.io/ansible-ise/main/html/plugins/network_device_module.html) Ansible actions.

### network_device_info

This module is used to retrieve _Network Devices_ information. When the `id` parameter is passed, it will only retrieve one element, if not, then it will retrieve all the _Network Devices_.

The following example retrieves all _Network Devices_ and then just one identified by id `70e07b60-8bff-11e6-996c-525400b48521`.

```yaml
- name: Get all Network Device Groups
  cisco.ise.network_device_info:
    ise_hostname: "{{ise_hostname}}"
    ise_username: "{{ise_username}}"
    ise_password: "{{ise_password}}"
    ise_verify: "{{ise_verify}}"
    page: 1
    size: 20
    sortasc: name
    filter: []
    filterType: AND

- name: Get Network Device Group
  cisco.ise.network_device_info:
    ise_hostname: "{{ise_hostname}}"
    ise_username: "{{ise_username}}"
    ise_password: "{{ise_password}}"
    ise_verify: "{{ise_verify}}"
    id: 70e07b60-8bff-11e6-996c-525400b48521
```

### network_device

This action is used to create, update or delete a _Network Device_, where the action depends on the `state` parameter. The **absent** parameter will delete a _Network Device_ and the state **present** will create or update a _Network Device_, depending if the Network Device Group already existed based on the provided ID or name.

The following example creates a _Network Device_ and then deletes another one based on the name.

```yaml
- name: Create Network Device
  cisco.ise.network_device:
    ise_hostname: "{{ise_hostname}}"
    ise_username: "{{ise_username}}"
    ise_password: "{{ise_password}}"
    ise_verify: "{{ise_verify}}"
    state: present
    NetworkDeviceGroupList:
      - Location#All Locations
    NetworkDeviceIPList:
      - ipaddress: 1.2.3.4
    mask: 32
    authenticationSettings:
    networkProtocol: RADIUS
    radiusSharedSecret: C1sco12345
    description: ""
    name: SJC-10
    tacacsSettings:
    connectModeOptions: "OFF"
    sharedSecret: C1sco12345

- name: Delete by name
  cisco.ise.network_device:
    ise_hostname: "{{ise_hostname}}"
    ise_username: "{{ise_username}}"
    ise_password: "{{ise_password}}"
    ise_verify: "{{ise_verify}}"
    state: absent
    name: SJC-10
```
