# Ansible automation

This automation contains several playbooks to automate the installation of IdM on RHEL 9.

Also to create VMs on Azure.

NOTE: The instructions below are expected to work on a RHEL9 machine. If you are using a different O.S, you may need to adapat several steps.

## Prepare Env

### Python dependencies

Install python dependencies:

```bash
pip install -r requirements.txt
```
### Ansible dependencies

Configure `ansible.cfg` with your [Red Hat Automation Hub token credentials](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/managing_automation_content/managing-cert-valid-content#assembly-creating-tokens-in-automation-hub).

```bash
cp ansible ansible-example.cfg ansible.cfg
vi ansible.cfg
```
Now install Ansible collections:
```bash
ansible-galaxy collection install -r requirements-rhel.yml
```
### Azure credentials

Configure your Azure credentials for the `az` cli and Ansible to work:
```bash
cp azure-example.env azure.env
```
Add your env vars to the file and source it:
```bash
vi azure.env
source azure.env
```

Login into Azure using the `az` client:
```bash
az login --service-principal -u $CLIENT_ID -p $PASSWORD --tenant $TENANT
```

### OpenShift crendentials

For Ansible to be able to interact with OpenShift, it consumes the kubeconfig at `~/.kube/config`.
So we need to login into the OpenShift cluster so the `oc` generates the kubeconfig automatically.

The user we login into OpenShift needs to have `cluster-admin` permissions.
```bash
oc login url --token=.....
```
## Deploy Azure VM for IdM

Assuming you already have an ARO cluster deployed and all the required Azure resources already
created: Resource Group, Network, Subnets, etc.

Create an Ansible inventory and fill the variables that are marked with `<CHANGE_ME>`.
```bash
cp inventory-example.ini inventory.ini
vi inventory.ini
``` 
Run the `deploy-idm-vm-azure.yaml` playbook:
```bash
ansible-navigator -vvv --ee false -m stdout run -i inventory --pae false deploy-idm-vm-azure.yaml
```
This Ansible playbook will create a RHEL9 VM in Azure, with a public IP and network peering between IdM network and ARO network,
for bidirectional communication between IdM and ARO nodes, required by the ACME protocol.

## Configure IdM and patch ARO OpenShift DNS and CA

Run the `site.yaml` playbook:
```bash
ansible-navigator -vvv --ee false -m stdout run -i inventory --pae false `site.yaml`
 
