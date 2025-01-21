# VM Configuration and Provisioning

Before configuring the local Kubernetes cluster, we need to have some hardware (or virtual ones) to run the cluster on.
This part is handled mainly by QEMU.

The author of the [original guide](https://github.com/ghik/kubernetes-the-harder-way) explains QEMU beautifully, and I certainly know a lot less than the author himself, so I leave the explanation to him.

This document outlines the details of the overall provisioning and configuration of the VMs.

## Table of Contents

<!--toc:start-->

- [Step 1: The QEMU Command](#step-1-the-qemu-command)
- [Step 2: `cloud-init`](#step-2-cloud-init)
- [Step 3: Launch!](#step-3-launch)
- [Step 4: SSH](#step-4-ssh)
<!--toc:end-->

## <a id='step-1-the-qemu-command' /> Step 1: The QEMU Command

The VM provisioning step is probably the most complicated step in the entire project.
This is due to the fact that a _lower-level_ tool such as QEMU is used to provision the VMs, instead of other solutions like Vagrant.

Nonetheless, the idea is a lot similar to Vagrant, we just create and pass the configurations in a more verbose way.

The main work is done via the QEMU command listed in [launch-vm](./launch-vm), all the other scripts under this directory is written to pass the correct values and have the correct infrastructure state before the command gets executed.

Here is a breakdown of each argument and the differences between the original guide (if there is any):

- `-nographic`: This is used to disable the GUI for the VMs. Since we primarily work through SSH in a separate `tmux` session, GUI is not needed.

- `-machine`: This is used to specify that we want to instruct QEMU that we need **virtualization**, not **emulation**.

- `-accel`: This is used to specify the hypervisor that QEMU should work on. In MacOS, this is `HVF`, aka. Hyperviser Framework.

- `-cpu`: This is used to specify the CPU type of the VM. This part is different than the guide in a way that `host` did not work properly. Since the QEMU documentation marks `host` as KVM only, a different CPU type is used instead.

- `-smp`: This is used to specify the amount of CPU cores we want to have.

- `-m`: This is used to specify the amount of memory we want to have.

- `-device, -netdev`: These flags are used as pairs to specify the network device of the VM. In the guide `-nic` is used as a shorthand, but the longer version is preferred here to see the actual underlying values.

- `-bios`: This is used to assign a BIOS to the VM, to boot our custom Ubuntu 24.04 image.

- `-hda`: This is used to pass the image we want to run on the VM.

- `-drive`: This is used to pass the `cloud-init` configuration ISO to the VM, to trigger the initial configuration.

For more details about the flags, I would suggest you to read the original guide.

## <a id='step-2-cloud-init' /> Step 2: `cloud-init`

`cloud-init` is a tool that is used to automatically configure an instance during the initialization phase.
Even though the name sounds like it is intended for cloud providers only, that is not the case.

In this project, `cloud-init` is mainly used to:

- Authorize the host machine to access each VM via SSH.
- Install several packages by default.

There is also a networking configuration done through `cloud-init`.
This configuration is done to make the `gateway` act as a load balancer on a virtual IP, as discussed in the [network infrastructure README](./NETWORK-INFRASTRUCTURE.md).

In short, the virtual IP is distributed to both `gateway` and `control` nodes, but the ARP communication of `control` nodes is disabled for the virtual IP, making `gateway` the only owner of the IP to the outside world.

## <a id='step-3-launch' /> Step 3: Launch!

The final step of the VM provisioning is to combine previous parts to create the virtualized hardware.
This is done in [launch-all](./launch-all) script.

Essentially, the `cloud-init` configurations for each VM are turned into an ISO, and then each QEMU command is executed in a tmux session called `kubenet-vms`.

## <a id='step-4-ssh' /> Step 4: SSH

Both launching and SSH connections highlight the productivity boost that comes via `tmux`.

The SSH setup is mainly handled in [ssh-all](./ssh-all) for all VMs.

During the `setup` script, this is executed right after when the QEMU commands are fired for all VMs.
The cool part about this is, it is not necessary to wait the whole boot process to finish.
We just need to wait until the VMs are ready to accept TCP 22 connections from the host machine.

When they are ready, a separate tmux session called `kubenet-ssh` is created with multiple windows for each node _group_:

- One for `gateway`,
- One for `control` nodes,
- One for `worker` nodes,
- One for both `control` and `worker` nodes.

With the SSH connections are set for each VM, the overall provisioning and initial configuration of the VMs are done.

At this point, the local Kubernetes cluster has all the (virtualized) hardware and the network configuration.

From this point onwards, everything is about Kubernetes now.
