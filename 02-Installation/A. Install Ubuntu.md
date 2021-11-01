 
#### Overview

Kubernetes is a tool for orchestrating and managing Docker containers at scale on on-prem server or across hybrid cloud environments. Kubeadm is a tool provided with Kubernetes to help users install a production ready Kubernetes cluster with best practices enforcement. This tutorial will demonstrate how one can install a Kubernetes Cluster on Ubuntu 20.04 with kubeadm.

#### Goals
There are two server types used in deployment of Kubernetes clusters:

- **Master:** A Kubernetes Master is where control API calls for the pods, replication controllers, services, nodes and other components of a Kubernetes cluster are executed.
- **Node:** A Node is a system that provides the run-time environments for the containers. A set of container pods can span multiple nodes.

#### Prerequisites Specifications

The minimum requirements for the viable setup are:
Memory: 2 GiB or more of RAM per machine
CPUs: At least 2 CPUs on the control plane machine.
Internet connectivity for pulling containers required (Private registry can also be used)
Full network connectivity between machines in the cluster – This is private or public
 
 
#### Installation

My Lab setup contains two servers. One control plane machine and one node to be used for running containerized workloads. You can add more nodes to suit your desired use case and load, for example using three control plane nodes for HA.

|Server Type | Server Hostname | Specs|
|------------|-----------------|-------|
|Master |k8s-master.example.com | 4GB Ram, 2vcpus|
|Worker1 |k8s-worker1.example.com | 2GB Ram, 1vcpus|
|Worker2 |k8s-worker2.example.com | 2GB Ram, 1vcpus|
 
#### Step 1: Install Kubernetes Servers

Provision the servers to be used in the deployment of Kubernetes on Ubuntu 20.04. The setup process will vary depending on the virtualization or cloud environment you’re using.

    $ sudo apt update -y 
    $ sudo apt -y upgrade && sudo systemctl reboot 

 
#### Step 2: Install kubelet, kubeadm and kubectl

Once the Servers are rebooted, add the Kubernetes repository for Ubuntu 20.04 to all the servers.

    $ sudo apt update
    $ sudo apt -y install curl apt-transport-https
    $ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    $ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

    $ sudo apt update
    $ sudo apt -y install vim git curl wget kubelet kubeadm  kubectl
    $ sudo apt-mark hold kubelet kubeadm kubectl docker-ce
    $ kubectl version --client && kubeadm version 

 
#### Step 3: Disable Swap

Turn off swap.

    $ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab 
    $ sudo swapoff -a

 
#### Configure sysctl.
 
    $ sudo modprobe overlay
    $ sudo modprobe br_netfilter
    $ sudo tee /etc/sysctl.d/kubernetes.conf <<EOF 
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    net.ipv4.ip_forward = 1
    EOF
    $ sudo sysctl --system


#### Install Docker. 
 
    $ sudo apt update
    $ sudo apt install docker.io
    $ sudo tee /etc/docker/daemon.json <<EOF
    {
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m"
    },
    "storage-driver": "overlay2"
    }
    EOF
    $ sudo systemctl enable --now docker kubelet

 
#### Initialize Kubernetes master server

Run these on the master node:

    kubernetes-master:~$ sudo kubeadm init
    kubernetes-master:~$ mkdir -p $HOME/.kube
    kubernetes-master:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    kubernetes-master:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config


 
#### Deploy a pod network
apply CNI Network :- 

    $ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    
    kubernetes-master:~$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
    
    $ kubectl get pods --all-namespaces
or 
    
    $ kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml 
    
    $ kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
 
#### Join the Kubernetes cluster
Run these from the worker node:

    kubernetes-worker:~$ sudo kubeadm join 192.168.122.219:6443 --token 1exb8s.2t4k3b5syfc3jfmo --discovery-token-ca-cert-hash sha256:72ad481cee4918cf2314738419356c9a402fb609263adad48c13797d0cba2341
    kubernetes-master:~$ kubectl get nodes
