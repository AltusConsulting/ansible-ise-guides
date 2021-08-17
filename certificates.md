# Certificate Management Guide

## Introduction

This guide explains how to use the Ansible ISE Collection to manage certificates.

## Goal

The goals of this guide are:

1. Install the Root CA public certificate in the trust store of an ISE node
2. Generate a Certificate Signing Request (CSR) for the system
3. Bind a CA signed digital certificate
4. Generate CSRs for other services (such as EAP authentication, Guest Access, etc) and install CA signed digital certificates for those services 


## Pre-requisites

It is recommended that you review the following guide before starting this one:

- [ISE Ansible Getting Started Guide](./ansible_start_guide.md)

## Sample playbook

A sample playbook is provided along with the Ansible ISE collection. The first playbook installs a Root CA certificate in the trust store of the ISE node:

```yaml
- name: Import certificate into ISE node
  cisco.ise.trusted_certificate_import:
    ise_hostname: 1.1.1.1
    ise_username: admin
    ise_password: C1sco12345
    data: "{{ lookup('file', item) }}"
    description: Root CA public certificate
    name: RootCert
    allowBasicConstraintCAFalse: true
    allowOutOfDateCert: false
    allowSHA1Certificates: true
    trustForCertificateBasedAdminAuth: true
    trustForCiscoServicesAuth: true
    trustForClientAuth: true
    trustForIseAuth: true
    validateCertificateExtensions: true
  with_fileglob:
    - "/Users/<username>/Downloads/RootCACert.pem"
```

Then, the playbook generates a Certificate Signing Request (CSR) to be signed by the Root CA:

```yaml
- name: Generate CSR
  cisco.ise.csr_generate:
    ise_hostname: 1.1.1.1
    ise_username: admin
    ise_password: C1sco12345
    allowWildCardCert: true
    subjectCommonName: ise.securitydemo.net
    subjectOrgUnit: Sample OU
    subjectOrg: Sample Org
    subjectCity: San Francisco
    subjectState: CA
    subjectCountry: US
    keyType: ECDSA
    keyLength: 1024
    digestType: SHA-256
    usedFor: MULTI-USE
  register: result

- name: Set ID value to variable
  ansible.builtin.set_fact:
    csr_id: "{{result['ise_response']['response'][0]['id']}}"

- name: Pause until the CSR has been signed by the CA
  pause:
```

Here we generated a "multi-use" CSR but you can generate CSRs for specific services, such as pxgrid, guest portal, AP Authentication, etc. Just set the `userdFor` parameter to one of the following values:
* ADMIN
* EAP-AUTH
* DTLS-AUTH
* PORTAL: 
* PXGRID
* SAML
* IMS

After generating the CSR, the execution of the playbook is paused to wait for the CSR to be manually signed by the CA. Note that the value of the CSR ID is captured in the variable `csr_id` to be used in the next task, where we bind the certificate signed by the CA.

```yaml
- name: Bind Signed Certificate
  cisco.ise.bind_signed_certificate:
    ise_hostname: 1.1.1.1
    ise_username: admin
    ise_password: C1sco12345
    admin: true
    allowExtendedValidity: true
    allowOutOfDateCert: true
    allowReplacementOfCertificates: true
    allowReplacementOfPortalGroupTag: true
    data: "{{ lookup('file', item) }}"
    hostName: ise.securitydemo.net
    name: My Signed Certificate
    validateCertificateExtensions: true
    id: "{{csr_id}}"
    eap: true
    radius: true
    pxgrid: true
    ims: true
    portal: true
  with_fileglob:
    - "/Users/rcampos/Downloads/RootCACert.pem" 
```

Run the Ansible playbook:

```cli
ansible-playbook -i hosts playbooks/certificates.yml
```

