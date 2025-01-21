# `local-k8s-osx`: Local Kubernetes Cluster on MacOS

This project is an implementation of a wonderfully written guide called [kubernetes-the-harder-way](https://github.com/ghik/kubernetes-the-harder-way).
It mainly serves as a learning material and a base playground for further experimentations.

Whilst the guide is heavily used, this project is not an exact copy of it.
There are certain key differences, especially regarding the networking part.

## Disclaimer

Here are some disclaimers about this project:

- This project is _NOT_ a replacement for several tools that does the same thing, such as `minikube`.
  Tools like `minikube` make the cluster bootstrapping really easy and straightforward to allow people to focus on other things, e.g. application deployments.

- This project is aimed for the people who are curious about how a basic cluster is set up and how the main K8s components interact with each other.

## Requirements

- This project is written for **arm64 MacOS hosts only**.
  If you want to have a Linux version, you can refer to the guide I shared.

- The required tools will be installed when you run the setup script.

- The project heavily utilizes `tmux` to set up different sessions for VMs and SSH connections.
  Whilst `tmux` is installed with the installation script, having a basic understanding of the tool would be really helpful to make the most out of this project.

- The default subnet configured for the cluster is `192.168.200.0`.
  If this subnet is occupied on your host, you need to change the network configuration.
  You can check the [network infrastructure README](./vms/NETWORK-INFRASTRUCTURE.md) to see how it is configured.

## Setting Up the Cluster

The installation is pretty straightforward:

```bash
# Clone the repository.
git clone git@github.com:acikgozb/local-k8s-osx.git
cd ./local-k8s
# Run the setup script, `sudo` is not needed.
./setup
```

A couple of notes:

- The installation takes a while simply because it downloads Ubuntu 24.04 cloud image to the host machine and the required K8s binaries on 7 VMs.
- Since some of the configurations require elevated privileges (`dnsmasq`, `vmnet_socket`, `nfsd`), the script will use `sudo` in some parts during the installation.

Once you see the message `local-k8s-setup: the installation is completed.`, you can start navigating within the cluster.

## Removing the Cluster

If you want to wipe out everything that belongs to the cluster, run the `remove` script.

```bash
cd ./local-k8s
# No need to run it with `sudo`, like the setup script.
./remove
```

When the removal process completes, you will see the message `local-k8s-rm: the cluster is removed successfully.`.

## The Implementation

For the implementation, please refer to the individual README's which explain each part of the project:

- [Network infrastructure](./vms/NETWORK-INFRASTRUCTURE.md),
- [Provisioning and configuring VMs](./vms/VMs.md),
- [TLS and Kubeconfigs to secure the cluster](),
- [Installing and configuring the binaries on nodes](),
- [Configuring the host machine]()

## TODO

- Writing a clear explanation about how to use this cluster in other projects as the main building block (via symlinks).
