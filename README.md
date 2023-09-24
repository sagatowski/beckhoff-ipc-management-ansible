# Introduction

So that others don't have to spend time trying to figure out how to automate the process of configuring Beckhoff IPCs (or other industrial PCs), I have created this repository which is an example of how to use the open source Ansible tool to automate the process of deploying a complete IPC configuration from one to many IPCs.

## What is Ansible?

Ansible is an open source tool that allows you to automate the configuration of one or more computers. It uses the concept of playbooks to define the configuration of a computer. A playbook is a YAML file that defines a series of tasks to execute on target systems. See the [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html) for more information on playbooks.

## Why use Ansible?

While you could use Powershell scripts to manually setup IPCs, it is more efficient to leverage a tool that has great support for automating the configuration of computers.

## Disclaimer

This repository is provided as an example of how to use Ansible to automate the configuration of IPCs. It is not intended to be a complete solution for configuring IPCs. It is intended to be a starting point for you to create your own solution. You will need to modify the example playbooks to meet your specific needs. Bear in mind that recommended security practices may have not been implemented in this example. You should review the example playbooks to ensure that they meet your security requirements.

## Getting Started

This project is setup per the recommendations from Ansible. The common role defines all common tasks to apply across all hosts. A couple of example secondary roles have been created, these could be different variants of machines on a factory floor.

To create new roles use the following command:
```
ansible-galaxy init <role name>
```

To assign roles to a particular group of hosts, edit the site.yml file and add the role to the list of roles for the group. e.g. if you added a new role `machine-variant-b` and you wanted to apply it to the host group with the same name of `machine-variant-b` you would add the role to the list of roles for that group:

```
- hosts: machine-variant-b
  roles:
    - common
    - machine-variant-b
```

## Minimal Infrastructure Requirements

To run this example you need to at least have a "control node", which can be your laptop or a server, and a "managed node", which is the IPC you want to configure. The control node is the computer that will run the Ansible playbooks. The managed node is the IPC that will be configured by the Ansible playbooks.

### Control Node Requirements

The control node must either be Windows running Windows Subsystem for Linux (WSL), or a Linux computer. This is required for Ansible. On the control node you must install Ansible. See the [Ansible documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) for more information on installing Ansible, but the basic steps are:

1. Install Python 3
2. Clone this repository into a folder on the control node
```
    git clone <repository url>
    cd <repository folder
```
3. Create a virtual environment for Ansible (this is optional but recommended)
```
    # run only once when setting up the virtual environment
    python3 -m venv venv
    # run each time you want to use the virtual environment
    source venv/bin/activate
```
4. Install Ansible and pywinrm using pip, for example: `pip install ansible pywinrm`
5. If you want to use the OPC-UA setup tasks, you will also need to install the asyncua library: `pip install asyncua`

### Managed Node Requirements

The managed node can be an Windows IPC running Windows 10 or newer, but for this example it is assumed that it is a Beckhoff IPC such as a CX or C series IPC. On each managed node you must run the following commands from Powershell to enable remote management:

```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
winrm quickconfig
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
winrm set winrm/config/service/auth '@{Basic="true"}'
```

You may need to run the following to get the interface index and set it to private if the prior winrm commands fail:
```
Get-NetConnectionProfile
Set-NetConnectionProfile -InterfaceIndex 5 -NetworkCategory Private
```

## Configuring a Vault

To follow best practices for storing passwords, you need to set up a vault that contains a few items. 

1. Create a file called `vault` in the `group_vars/all` directory.
```
ansible-vault create group_vars/all/vault
```
Set a password for the vault when prompted, and record this somewhere safe.
2. Add the following to the vault file:
```
vault_ansible_password: <password>
vault_ansible_become_password: <password>
vault_internal_nuget_repo: <repo>
tf6100_opc_ua_password: <password>
```
3. Encrypt the vault file by closing the editor

If you need to edit the file later, use `ansible-vault edit group_vars/all/vault` and enter the password when prompted.

Note that the above variables are referenced in the `group_vars/all/main.yml` file.

## Running the Playbooks

To run the playbooks, you need to run the following command from the root of the repository:
```
ansible-playbook -i staging.yml site.yml --ask-vault-pass
```

## About this Repository

In this repo I've defined a few roles that can be used to configure an IPC. The roles are defined in the `roles` directory. The `common` role is applied to all hosts. The `machine-variant-a` and `machine-variant-b` roles are applied to hosts that are in the `machine-variant-a` and `machine-variant-b` groups respectively. The `machine-variant-a` and `machine-variant-b` groups are defined in the `staging.yml` file.