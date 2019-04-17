# Kubernetes essentials

🎻 Linux Academy [course](https://linuxacademy.com/cp/modules/view/id/281) for understanding the Kubernetes container orchestration tool.

## Installation

### Cluster architecture

For this lesson, it's needed to spin up some servers for deploying our Kubernetes cluster. In this case, that we don't have the capability of having three server easily, we are gonna pretend to have three servers where one of those would be the **Master Node** and the two other ones would be the **Node 1** and **Node 2**.

Everyone of these three nodes must have the following requirements:

- [Docker](https://www.docker.com/)
- [Kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/)
- [Kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

Also, apart from that, the **Master node** requires to have a [Control Plane](https://kubernetes.io/docs/concepts/#kubernetes-control-plane) properly installed.

### Installing Docker

A *container runtime* is the software that actually runs the containers. Kubernetes supports several other container runtimes (such as [rkt](https://coreos.com/rkt/) and [containerd](https://containerd.io/)) but Docker is the most popular. The first step in building our cluster is to install Docker on all three servers:

- Add the Docker repository GPG key:

  ```bash
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  ```

- Add the Docker repository:

  ```bash
  sudo add-apt-repository \
     "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) \
     stable"
  ```

- Reload the apt sources list:

  ```bash
  sudo apt-get update
  ```

- Install Docker:

  ```bash
  sudo apt-get install -y docker-ce
  ```

- Prevent auto-updates for the Docker package:

  ```bash
  sudo apt-mark hold docker-ce
  ```

- Verify that docker is working:

  ```bash
  sudo docker version
  ```

### Installing Kubeadm, Kubelet, and Kubectl

The point of this subsection is to install the different and needed component of Kubernetes. Before that, let's explain what are these components.

- **Kubeadm**: this is a tool which automates a large portion of the process of setting up a cluster. It will make our job much easier.
- **Kubelet**: the essential component of Kubernetes that handles running containers on a node. Every server that will be running containers needs kubelet.
- **Kubectl**: command-line tool for interacting with the cluster once it is up. We will use this to manage the cluster.

Once understood these three components, let's install them on all three servers:

- Add the Kubernetes repository GPG key:

  ```bash
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
  ```

- Add the Kubernetes repository:

  ```bash
  cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
  deb https://apt.kubernetes.io/ kubernetes-xenial main
  EOF
  ```

- Reload the apt sources list:

  ```bash
  sudo apt-get update
  ```

- Install Kubernetes:

  ```bash
  sudo apt-get install -y kubelet=1.12.7-00 kubeadm=1.12.7-00 kubectl=1.12.7-00
  ```

- Prevent auto-updates for the Kube package:

  ```bash
  sudo apt-mark hold kubelet kubeadm kubectl
  ```

- Verify that Kubeadm is working:

  ```bash
  kubeadm version
  ```
