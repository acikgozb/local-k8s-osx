# VM Definitions for Local K8s

This directory is responsible from defining the virtualization layer of our local Kubernetes cluster.

## Requirements

The VMs are provisioned via Vagrant, so you need to have it installed.
Refer to its [installation page](https://developer.hashicorp.com/vagrant/downloads) to get started.

Since the target machine is Darwin as of this writing, Vagrant uses `vmware_desktop` provider to provision the VMs.
Instead of `vmware_desktop`, you can use `vmware_fusion` as well, if you wish.

## Definitions

You can check the file `config.yml` to see the CPU and memory definitions for each VM.
The configuration is defined based on what the Kubernetes documentation sets as [minimum requirements](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#before-you-begin).

## Usage

To start the provisioning, all you need to do is:

```bash
# Run inside the directory "vms".
$ vagrant up
```

To verify the results, you can check the running VMs.
You should be able to see all VMs up and running, like this:

```bash
$ vagrant global-status
id       name          provider       state   directory
-------------------------------------------------------------------------------------
d60eff5  control-plane vmware_desktop running <git-clone-dir>/vms
a44e50b  node1         vmware_desktop running <git-clone-dir>/vms
0e2ab81  node2         vmware_desktop running <git-clone-dir>/vms
```

## Limits

Unfortunately, due to limited features of `vmware_provider`, it is not possible to assign private network subnets and static IP's to VMs.
This is documented in [here](https://developer.hashicorp.com/vagrant/docs/providers/vmware/known-issues#creating-network-devices).

This will be addressed probably during the provisioning step, but right now a manual lookup is needed to get the IP addresses.
One way of getting them is by using `ip address show`, here is an example with the control plane:

```bash
# Run inside the directory "vms".
$ vagrant ssh control-plane
vagrant@vagrant:~$ ip address show | grep "inet\\s"
inet 192.168.7.129/24
```
