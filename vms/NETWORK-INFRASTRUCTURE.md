# Network Infrastructure

<!--toc:start-->

- [Step 1: Deciding the IPs](#step-1-deciding-the-ips)
- [Step 2: The DHCP Server](#step-2-the-dhcp-server)
- [Step 3: Creating The Infrastructure](#step-3-creating-the-infrastructure)
- [Changing The Default Subnet](#changing-the-default-subnet)
<!--toc:end-->

The [original guide](https://github.com/ghik/kubernetes-the-harder-way) uses the subnet `192.168.1.0/24` to set up the cluster network.
However, this subnet is commonly occupied by ISPs, so it is a bit risky to assume that it is free for use.

The author mentions this in several different GitHub issues such as [this](https://github.com/ghik/kubernetes-the-harder-way/issues/16#issuecomment-2440799114).

So, the network infrastructure for this cluster is set up on a completely different subnet, which is `192.168.200.0/24`.

This document outlines how the overall network is set up for the cluster.

## <a id='step-1-deciding-the-ips' /> Step 1: Deciding the IPs

First, we need to figure out how we should use the subnet itself before diving into anything.

The obvious thing at this point is that the cluster will have 7 VMs:

- The `gateway` VM, which will be the main access point to the cluster from the host machine.
- The `control0-2` VMs, which will be the control nodes of the Kubernetes cluster.
- The `worker0-2` VMs, which will be the worker nodes of the Kubernetes cluster.

In order to have a single, reproducible configuration of the cluster, we need to assign **static IPs** to these VMs.
Otherwise, everytime this project is installed on a host machine, we would need to configure the Kubernetes components to use the latest IP addresses given to the nodes.
So we need to decide 7 IPs by default.

To assign static IPs, we need to have a custom **DHCP server** that distributes the IPs to the nodes.
Having a floating DHCP server does not help, though.
We need to make the DHCP server listen the main interface that is created by `vmnet_socket` under a specific IP called the `gateway`.
Keep in mind that this is different than the `gateway` VM.
So, we need to decide another IP address that will be used by the network interface created by `vmnet_socket`.

For the last IP address, I need to go back to the responsibility of the `gateway` VM.
It will be used as the main access point, meaning that it will act like a **load balancer** in front of the control nodes.
For this to happen, it needs to have another IP address.

At the end, we need **9** IP addresses.
In this project, the IP addresses are decided as follows:

| IP address       | Ownership              |
| ---------------- | ---------------------- |
| `192.168.200.1`  | `vmnet_socket`         |
| `192.168.200.3`  | `gateway`              |
| `192.168.200.4`  | `control0`             |
| `192.168.200.5`  | `control1`             |
| `192.168.200.6`  | `control2`             |
| `192.168.200.7`  | `worker0`              |
| `192.168.200.8`  | `worker1`              |
| `192.168.200.9`  | `worker2`              |
| `192.168.200.11` | `gateway` (virtual IP) |

## <a id='step-2-the-dhcp-server' /> Step 2: The DHCP Server

Like it is mentioned above, a DHCP server is needed in order to assign static IPs.
In this project, `dnsmasq` is used as a DHCP server (also for DNS as well).

By default, `dnsmasq` reads its configurations from `/opt/homebrew/etc/dnsmasq.conf`.
So, this project contains a custom dnsmasq configuration that is appended to the main configuration file when the setup script is executed.

You can see the custom configuration in [dnsmasq.conf]() file, which is where the IPs we decide above is used.

During the removal process, the custom configuration is removed from `/opt/homebrew/etc/dnsmasq.conf`, preventing persistent changes made on the host machine.

## <a id='step-3-creating-the-infrastructure' /> Step 3: Creating The Infrastructure

The main script that is executed for the network infrastructure is called [setup-dnsmasq]().

Unfortunately, working on MacOS brings some challenges in this area.
Whenever a new interface is created by `vmnet_socket` (same with QEMU), the default DHCP and DNS server of MacOS called `mDNSResponder` starts listening the interface's IP.
When this happens, the `dnsmasq` fails to listen the interface because there is already another process listening the same IP/port combination.

So, a workaround solution is implemented:

1. Instead of letting `vmnet_socket` create the network interface **first**, it is created manually instead.
   By creating the interface via `ifconfig`, we avoid triggering `mDNSResponder`.

2. `dnsmasq` is restarted manually to start listening the newly created network interface.

3. The network interface is destroyed manually via `ifconfig`.
   This is done simply because `vmnet_socket` expects to have no network interface that occupies the IP address given by the `--gateway` option.
   So, if we do not remove the interface, the socket process does not start.

4. When we destroy the interface, we make the `dnsmasq` enter into a malfunctioned state.
   The interface it listens does not exist anymore.
   So, in order to fix this issue, the service is restarted once `vmnet_socket` creates the interface.

## <a id='changing-the-default-subnet' /> Changing The Default Subnet

If you wish to use another subnet for the cluster, you can change the `$kubenet_nw_bits` to a different value, such as `192.168.100`.
This variable holds the network bits of the cluster subnet, and it is used by `dnsmasq` to define the DHCP pool.

However, changing the value is not enough to change the _entire_ configuration of the cluster.
Here are the effected places:

- The configurations of the Kubernetes components use hardcoded values, so they need to be changed as well.
  This is a limitation brought by executing the scripts via SSH in the `setup` script, and if there is a fix for this I would love to know!
- The `setup-dnsmasq` script uses pattern matching to understand when to restart `dnsmasq`. Since the pattern has special characters, it needed to be hardcoded.

Hopefully, these effected places will be fixed in a future update. Until that happens, you need to update these parts as well.
Then, you can use another subnet for your cluster.
