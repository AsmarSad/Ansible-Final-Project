# Ansible-Final-Project

## Project Structure
```
├── ansible.cfg
├── inventory
│   ├── dynamic_inventory.yml
│   └── dynamic_inventory.yml.j2
├── k8s_setup.yml
├── README.md
├── roles
│   ├── configure_nodes
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── handlers
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   ├── dynamic_inventory
│   │   └── tasks
│   │       └── main.yml
│   └── provision_nodes
│       └── tasks
│           └── main.yml
├── secrets.yml
└── vars
    ├── production.yml
    └── staging.yml
```
# secrets.yml
`secrets.yml` contains our sensitive data like digital ocean token. This is what file looks like:
```
do_token: <digital_ocean_token>
```
We encrypted it using Ansible-Vault.


# k8s_setup.yml
This file defines the main playbook for setting up the Kubernetes nodes, including provisioning the nodes on DigitalOcean and configuring them for Kubernetes.
## First Task: Provision Kubernetes Nodes
✅First of all, this task is executed on local host (Where we run this ansible file).

✅ It takes variables based on environment (Production or Staging)

✅ Then role is specified: provision_nodes.
Responsible for provisioning the Kubernetes nodes (master and worker) on DigitalOcean.

### Roles: Provision_Nodes
This role is responsible for provisioning Kubernetes nodes on DigitalOcean (master and worker).

✅ First, we upload our ssh to Digital ocean. We will be able to connect to our machines via ssh keys.

✅ Then we take ssh fingerprint from Digital Ocean and assign it to ssh_fingerprint variable using `set_fact:`

✅ Create master and worker nodes seperately assigning parameters (size, image, ssh_keys and so on). We create it with ssh keys that we retrieved in previous task. We don't use password as it is not considered secure. 

✅ Combine all nodes to a list and pass them to seperate inventory files (master and workers) for future use.

### Roles: Dynamic_Inventory

This task is responsible for generating a dynamic inventory for Ansible from a template.

✅ This module generates an Ansible inventory file from a Jinja2 template (`dynamic_inventory.yml.j2`) and saves it to `inventory/dynamic_inventory.yml`.




# Second Task: Configure Kubernetes Nodes

✅ First, we gather facts (like OS, network settings) from the nodes.

✅ Then we use this parameter 
`ansible_ssh_common_args:` to be sure that we can connect our nodes 

✅ We call our second role to config nodes.

## Role: Configure_Nodes
This role configures the Kubernetes nodes by installing necessary packages and ensuring the required services are running.

### Tasks
✅ We ensure Kubernetes apt keyring directory exists.

✅ Download Kubernetes apt key and add Kubernetes apt repository.

✅ We update apt cache and install necessary kubernetes packeges. In this task we use retry. System retries installation if it fails.

✅ Ensure services are enabled and running

### Handlers

✅ This handler restarts service whenever necessary.

### Defaults
In defaults, we contain default version for Kubernetes to install.


# Commands to run Ansible playbook

## Staging
`ansible-playbook k8s_setup.yml -e env=staging --ask-vault-pass`

## Production
`ansible-playbook k8s_setup.yml -e env=production --ask-vault-pass`



