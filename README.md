# aap-ocp-rolling-upgrade

A helper playbook to gracefully cordon a node running Ansible Automation Controller and Ansible Automation job for rolling upgrade of OCP worker nodes with no outage to Ansible Automation Controller

!!!CAUTION: THIS IS A POC USE AT YOUR OWN RISK!!!

## Prerequisites

Set the following environment variable before running the playbook:

`CONTROLLER_HOSTNAME`: The hostname of the Ansible Automation Controller
`CONTROLLER_USERNAME`: The username of the Ansible Automation Controller
`CONTROLLER_PASSWORD`: The password of the Ansible Automation Controller

`KUBECONFIG`: The path to the kubeconfig file of the OCP cluster

Note: Please verify that your kubeconfig is pointing to the correct cluster!

## Prepare the OCP cluster

1. Create a MachineConfigPool 
2. Pause the MachineConfigPool
3. Upgrade your OCP cluster to the desired version (since the MachineConfigPool for worker are paused, the upgrade will only affect the Master nodes)

## Alternative steps

There is a provided playbook which does the following:

1. run machine_config_pool.yml
2. wait for master node(s) to finish upgrading
3. continue with next steps

## Run the playbook

```bash
ansible-playbook machine_config_pool.yml -e desire_version=<OCPVERSION>
ansible-playbook upgrade.yml -e kube_worker_node=<node_name> -e automation_controller_namespace=<namespace>
```

The playbook will do the following:

1. Taint the node and prevent new pods from being scheduled on the node.
2. If any Ansible Automation Controller pods are running on the node it will disable the corresponding instance in the Automation Controller.
3. If the Ansible Automation Controller instance is controlling any jobs it will wait for the jobs to finish.
4. If any Ansible Automation Job is running on the node it will wait for the job to finish.

Once the playbook is completed you can resume the MachineConfigPool and the node will be upgraded.
