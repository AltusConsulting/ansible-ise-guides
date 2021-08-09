# Backup and Restore Guide

## Introduction

This guide explains how to use the Ansible ISE Collection to create a repository of any type (DISK, FTP, SFTP, etc), how to store backup configuration files in it and how to restore a node's configuration from a backup.

## Goal

The goals of this guide are:

1. Create a repository
2. Create a configuration backup and store it on the repository
3. Restore a configuration from a previously created backup
4. Schedule a backup to be executed at a future time

## Pre-requisites

It is recommended that you review the following guide before starting this one:

- [ISE Ansible Getting Started Guide](./ansible_start_guide.md)

## Sample playbooks

Three backup-related sample playbooks are provided with the Ansible collection: `backup_create.yml`, `backup_restore.yml` and `backup_schedule.yml`.

### `backup_create.yml`
The first playbook, `backup_create.yml`, executes the following tasks:
* Creates a repository
* Creates a backup
* Shows the status of the backup task.

The playbook creates a repository called `myRepo` by using the `cisco.ise.repository` module. In this case, we're creating a repository of type `DISK` and path `/`.

```yaml
- name: Create a repository
  cisco.ise.repository:
    ise_hostname: 1.1.1.1
    ise_username: admin
    ise_password: C1sco12345
    state: present
    name: myRepo
    password: MyP@ssworD
    path: /
    protocol: DISK
```

Then, the playbook uses the `cisco.ise.backup_config` module to request the creation of the backup inside the repository `myRepo`. We register the response from ISE inside the variable `result`. This response contains the task ID for the creation of the backup.

```yaml
- name: Create backup
  cisco.ise.backup_config:
    ise_hostname: 1.1.1.1
    ise_username: admin
    ise_password: C1sco12345
    backupEncryptionKey: My3ncrypt1onKey
    backupName: myBackup
    repositoryName: myRepo
  register: result
```
In order to visualize the task ID returned by ISE after requesting the creation of the backup, we use the `cisco.ise.tasks_info` module to retrieve the status of the task and the `debug` module to show it:

```yaml
- name: Get Tasks by id
  cisco.ise.tasks_info:
    ise_hostname: 1.1.1.1
    ise_username: admin
    ise_password: C1sco12345
    taskId: "{{result.ise_response.response.id}}"
  register: task_status

- name: Show task status
  debug:
    msg: "{{task_status}}"
```

Run the Ansible playbook:

```cli
ansible-playbook -i hosts playbooks/backup_create.yml
```

### `backup_restore.yml`

The second playbook `backup_restore.yml` takes the name of a backup file and the name of the repository it's in and restores the corresponding configuration:

```yaml
- name: Restore backup
  cisco.ise.backup_restore:
    ise_hostname: 1.1.1.1
    ise_username: admin
    ise_password: C1sco12345
    backupEncryptionKey: My3ncryptionkey
    restoreFile: myBackup-CFG10-210806-2232.tar.gpg 
    repositoryName: myRepo
    restoreIncludeAdeos: True
```

Run the Ansible playbook:

```cli
ansible-playbook -i hosts playbooks/backup_restore.yml
```

### `backup_schedule.yml`
The third playbook `backup_schedule.yml` schedules a configuration backup at a predefined date and time.

```yaml
- name: Create a backup every Saturday at midnight
  cisco.ise.backup_schedule_config:
    ise_hostname: 1.1.1.1
    ise_username: admin
    ise_password: C1sco12345
    backupDescription: mybackup
    backupEncryptionKey: My3ncryptionkey
    repositoryName: myRepo
    backupName: myBackup
    startDate: 01/01/2022
    endDate: 12/31/2022
    frequency: WEEKLY
    weekDay: SAT
    time: 12:00 AM
    status: ENABLE

- name: Create a backup on the 15th of each month
  cisco.ise.backup_schedule_config:
    ise_hostname: 1.1.1.1
    ise_username: admin
    ise_password: C1sco12345
    backupDescription: mybackup
    backupEncryptionKey: My3ncryptionkey
    repositoryName: myRepo
    backupName: myBackup
    startDate: 01/01/2022
    endDate: 12/31/2022
    frequency: MONTHLY
    monthDay: 15
    time: 12:00 AM
    status: ENABLE
```

Following are the possible values for each option:
* frequency: ONCE, DAILY, WEEKLY, MONTHLY
* weekDay: MON, TUE, WED, THU, FRI, SAT, SUN. This option is valid only when the frequency is set to WEEKLY
* monthDay: from 1 to 28. This option is valid only when the frequency is set to MONTHLY
* status: ENABLE, DISABLE

Run the Ansible playbook:

```cli
ansible-playbook -i hosts playbooks/backup_schedule.yml
```
