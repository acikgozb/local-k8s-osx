# `local-k8s-osx`: Local Kubernetes Cluster on MacOS

This project is an implementation of a wonderfully written guide called [kubernetes-the-harder-way](https://github.com/ghik/kubernetes-the-harder-way).
It mainly serves as a learning material and a base playground for further experimentations.

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

## The "Implementation"

The reason why this project is called as _an implementation_ is that whilst the idea is the same with the original guide posted above, there are differences when it comes the overall networking setup of the cluster.
So, it's not really an exact copy of the guide.

The guide uses the subnet `192.168.1.0/24` to set up the cluster network.
However, this subnet is commonly occupied by ISPs, so it is a bit risky to assume that it is free for use.

The author mentions this in several different GitHub issues such as [this](https://github.com/ghik/kubernetes-the-harder-way/issues/16#issuecomment-2440799114).
So, the network infrastructure for this cluster is set up on a completely different subnet, which is `192.168.200.0/24`.

To dive deep into the networking part of the cluster, please refer to the [README](./vms/NETWORK-INFRASTRUCTURE.md) here.

The rest of the project follows more or less the same path with the guide, with slight differences:

- The overall directory structure is changed a little bit to show the intent of each script.
- The `setup` and `remove` scripts are added to the project to make it easier to play around with the cluster itself.

## TODO

- Writing a clear explanation about how to use this cluster in other projects as the main building block (via symlinks).
