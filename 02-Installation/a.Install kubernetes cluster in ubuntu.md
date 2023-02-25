 
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
|Master |devops-lab.example.com | 4GB Ram, 2vcpus|
|Worker1 |k8s-worker1.example.com | 2GB Ram, 1vcpus|
|Worker2 |k8s-worker2.example.com | 2GB Ram, 1vcpus|
 
#### Step 1: Install Kubernetes Servers

Provision the servers to be used in the deployment of Kubernetes on Ubuntu 20.04. The setup process will vary depending on the virtualization or cloud environment you’re using.

    root@devops-lab:~# sudo apt update -y 
    root@devops-lab:~# sudo apt -y upgrade && sudo systemctl reboot 
        
#### Step 2: Install kubelet, kubeadm and kubectl

Once the Servers are rebooted, add the Kubernetes repository for Ubuntu 20.04 to all the servers.

    root@devops-lab:~# sudo apt -y install curl apt-transport-https
    root@devops-lab:~# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    root@devops-lab:~# echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    root@devops-lab:~# apt update
    root@devops-lab:~# apt -y install vim git curl wget kubelet kubeadm  kubectl 
    root@devops-lab:~# apt-mark hold kubelet kubeadm kubectl 
    root@devops-lab:~# kubectl version --client && kubeadm version 

 
#### Step 3: Disable Swap

Turn off swap.

    root@devops-lab:~# sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab 
    root@devops-lab:~# sudo swapoff -a

 
#### Step 4: load kernel module & Configure sysctl.
 
    root@devops-lab:~# cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
    overlay
    br_netfilter
    EOF

    root@devops-lab:~#  sudo modprobe overlay
    root@devops-lab:~#  sudo modprobe br_netfilter

 Setup required sysctl params, these persist across reboots.

    root@devops-lab:~# cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.ipv4.ip_forward                 = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    EOF

 Apply sysctl params without reboot

    root@devops-lab:~# sudo sysctl --system
    root@devops-lab:~# sudo apt update -y

#### Step 5: Add CRI-O Kubic repository

Add the Kubic repository which host binary packages for Debian based systems. If using CRI-O with Kubernetes, install the version matching Kubernetes version you’ll setup.

If your Kubernetes version is 1.26, install CRI-O version 1.23. We’ll start by adding the APT repository which contains CRI-O packages:


    root@devops-lab:~# OS=xUbuntu_22.04
    root@devops-lab:~# CRIO_VERSION=1.26
    root@devops-lab:~# echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
    root@devops-lab:~# echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$CRIO_VERSION/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION.list


Once the repository is added to your system, import GPG key:

    root@devops-lab:~# curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION/$OS/Release.key | sudo apt-key add -
    root@devops-lab:~# curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key add -

#### Step 6: Install CRI-O 

When repository is added, update apt cache and install CRI-O on Ubuntu.

    root@devops-lab:~# sudo apt update
    root@devops-lab:~# sudo apt install cri-o cri-o-runc

    Accept installation prompt with y key.
    The following additional packages will be installed:
    conmon containers-common
    Suggested packages:
    cri-o-runc | runc containernetworking-plugins
    The following NEW packages will be installed:
    conmon containers-common cri-o
    0 upgraded, 3 newly installed, 0 to remove and 94 not upgraded.
    Need to get 20.2 MB of archives.
    After this operation, 99.4 MB of additional disk space will be used.
    Do you want to continue? [Y/n] y

#### Step 7: Checking the version of CRI-O installed on Ubuntu:

    root@devops-lab:~# apt show cri-o
    Package: cri-o
    Version: 1.23.0~0
    Priority: optional
    Section: devel
    Maintainer: Peter Hunt <haircommander@fedoraproject.org>
    Installed-Size: 98.3 MB
    Depends: libgpgme11, libseccomp2, conmon, containers-common (>= 0.1.27), tzdata
    Suggests: cri-o-runc | runc (>= 1.0.0), containernetworking-plugins
    Replaces: cri-o-1.19, cri-o-1.20, cri-o-1.21
    Homepage: https://github.com/cri-o/cri-o
    Download-Size: 19.9 MB

#### Step 8: Start and enable crio service:

    root@devops-lab:~# sudo systemctl enable crio.service
    root@devops-lab:~# sudo systemctl start crio.service
    Service status can be checked with the command:
    root@devops-lab:~# systemctl status crio
    ● crio.service - Container Runtime Interface for OCI (CRI-O)
        Loaded: loaded (/lib/systemd/system/crio.service; enabled; vendor preset: enabled)
        Active: active (running) since Sun 2020-06-07 20:16:50 CEST; 37s ago
        Docs: https://github.com/cri-o/cri-o
    Main PID: 2461 (crio)
        Tasks: 13
        Memory: 7.7M
        CGroup: /system.slice/crio.service
                └─2461 /usr/bin/crio

    Jun 07 20:16:50 ubuntu systemd[1]: Starting Container Runtime Interface for OCI (CRI-O)...
    Jun 07 20:16:50 ubuntu systemd[1]: Started Container Runtime Interface for OCI (CRI-O).
    Step 4: Using CRI-O on Ubuntu22.04|20.04|18.04

The command line tool crioctl can be installed through cri-tools package.

    root@devops-lab:~# apt install cri-tools
    Check existence of crictl command:
    root@devops-lab:~# sudo crictl info
    {
    "status": {
        "conditions": [
        {
            "type": "RuntimeReady",
            "status": true,
            "reason": "",
            "message": ""
        },
        {
            "type": "NetworkReady",
            "status": false,
            "reason": "NetworkPluginNotReady",
            "message": "Network plugin returns error: Missing CNI default network"
        }
        ]
    }
    }
#### Step 9: pull the container images 

    root@devops-lab:~# sudo kubeadm config images pull --cri-socket /run/containerd/containerd.sock --kubernetes-version v1.26.0


 
#### Step 10: Initialize Kubernetes master server

Run these on the master node:

    root@devops-lab:~# sudo kubeadm init   --pod-network-cidr=10.244.0.0/16   --upload-certs    --control-plane-endpoint=MASTERIP  --cri-socket /var/run/crio/crio.sock

    root@devops-lab:~# mkdir -p $HOME/.kube
    root@devops-lab:~#  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    root@devops-lab:~#  sudo chown $(id -u):$(id -g) $HOME/.kube/config


 
#### Deploy a pod network

apply CNI Network :- 

    root@devops-lab:~# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml
    root@devops-lab:~# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml 
    root@devops-lab:~# kubectl get pods --all-namespaces

  
#### If coreDNS not creating container then delete the pod and redeploy
    root@devops-lab:~#  kubectl delete pod   coredns-78fcd69978-zml9m -n kube-system
    

#### Join the Kubernetes cluster
Run these from the worker node:

    kubernetes-worker:~$ sudo kubeadm join 192.168.122.219:6443 --token 1exb8s.2t4k3b5syfc3jfmo --discovery-token-ca-cert-hash sha256:72ad481cee4918cf2314738419356c9a402fb609263adad48c13797d0cba2341

#### Go to master and check the node.
    kubernetes-master:~$ kubectl get nodes
