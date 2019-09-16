# Kubernetes essentials

> This course notes are **WIP**.

🎻 Linux Academy [course](https://linuxacademy.com/cp/modules/view/id/281) for understanding the Kubernetes container orchestration tool.

## Installation

### Cluster architecture

For this lesson, it's needed to spin up some servers for deploying our Kubernetes cluster. In this case, we are gonna need three servers where one of those would be the **Master Node** and the two other ones would be the **Node 1** and **Node 2**.

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

### Bootstrapping the Cluster

In this section, we will bootstrap the cluster on the **Kube master node**. Then, we will join each of the two worker nodes to the cluster, forming an actual multi-node Kubernetes cluster.

- On the **Kube master node,** initialize the cluster:

  ```bash
  sudo kubeadm init --pod-network-cidr=10.244.0.0/16
  ```

- Set up the local kubeconfig:

  ```bash
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

  Actually, you can find these last commands in the output of the cluster initialization logs.

- Verify that the cluster is responsive and that Kubectl is working:

  ```bash
  kubectl version
  ```

  You should get `Server Version` as well as `Client Version`. It should look something like this:

  ```bash
  Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.2", GitCommit:"17c77c7898218073f14c8d573582e8d2313dc740", GitTreeState:"clean", BuildDate:"2018-10-24T06:54:59Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}
  Server Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.2", GitCommit:"17c77c7898218073f14c8d573582e8d2313dc740", GitTreeState:"clean", BuildDate:"2018-10-24T06:43:59Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}
  ```

- The `kubeadm init` command should output a `kubeadm join` command containing a token and hash. Copy that command and run it with `sudo` on both **worker nodes**. It should look something like this:

  ```bash
  sudo kubeadm join $some_ip:6443 --token $some_token --discovery-token-ca-cert-hash $some_hash
  ```

- Verify that all nodes have successfully joined the cluster:

  ```bash
  kubectl get nodes
  ```

  You should see all three of your nodes listed. It should look something like this:

  ```bash
  NAME                      STATUS     ROLES    AGE     VERSION
  wboyd1c.mylabserver.com   NotReady   master   5m17s   v1.12.2
  wboyd2c.mylabserver.com   NotReady   <none>   53s     v1.12.2
  wboyd3c.mylabserver.com   NotReady   <none>   31s     v1.12.2
  ```

  > **Note:** The nodes are expected to have a STATUS of `NotReady` at this point.