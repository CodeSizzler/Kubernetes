# Exercise 1: Getting Started with Kubernetes on a Linux Virtual Machine

## Overview

This exericse  will guide you through setting up a Kubernetes cluster on a Linux VM and deploying your first Pod. Kubernetes is a **container orchestration platform** that automates deployment, scaling, and management of containerized applications.

### Key Kubernetes Components:

- **Nodes**: Machines that run workloads.

- **Pods**: The smallest deployable unit.

- **ReplicaSets**: Ensures a specified number of pods are running.

- **Services**: Expose applications internally or externally.

## Prerequisites

- A Linux VM (Ubuntu 20.04+ recommended)

- SSH access

- Basic knowledge of Linux commands

### Task 1: Installing Kubernetes

1. Run the below commands to install the dependencies that Kubernetes needs to function. 

  ```bash
  sudo apt update
  ```

  ```bash
  sudo apt install -y curl apt-transport-https
  ```

2. Run the below commands to install Docker, which will serve as our container runtime. 

  ```bash
  sudo apt install -y docker.io
  ```

  ```bash
  sudo systemctl enable --now docker
  ```

3. Run the below commands to install Kubernetes along with it's constituent tools and components (kubectl, kubelet, kubeadm). **Kubectl** is a command-line tool that is used to interact with Kubernetes clusters. **Kubelet** is a Kubernetes component that ensures that containers run in pods as defined by a Kubernetes cluster's control plane. **Kubeadm** is a command-line tool that makes it easy to fully setup Kubernetes clusters. 

  ```bash
  echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
  ```

  ```bash
  curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
  ```

  ```bash
  sudo apt update
  ```

  ```bash
  sudo apt install -y kubelet kubeadm kubectl
  ```

4. Run the below commands to disable swap functionality. This is necessary for kubelet to work and thus for Kubernetes as a whole to function.

  ```bash
  sudo swapoff -a
  ```

  ```
  sudo sed -i '/ swap / s/^/#/' /etc/fstab
  ```

### Task 2: Setting Up Kubernetes Cluster

1. Run the below command to initialize the Kubernetes Master (control plane) node. 

  ```bash
  sudo kubeadm init
  ```

2. Run the below commands to set up a kubeconfig file. This is used by kubectl to interact with a Kubernetes cluster. 

  ```bash
  mkdir -p $HOME/.kube
  ```

  ```bash
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  ```

  ```
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```

3. Run the below command to install the **Flannel** network plugin. This is used for pod communication. 

  ```bash
  kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
  ```

4. Run the below command to verify that your cluster is up and running. If the **STATUS** is **Ready**, Kubernetes is set up!

  ```bash
  kubectl get nodes
  ```


### Task 3: Working with Pods

1. Run the below command to open the **nano** text editor. 

  ```
  nano
  ```

2. Enter the following. This is a YAML file that defines a pod. Press **Ctrl + o** to save the file. Enter **pod.yaml** when prompted for the file name then press **Ctrl + x** to exit nano. 

  ```yaml
  # Save as pod.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-pod
  spec:
    containers:
      - name: nginx
        image: nginx
        ports:
          - containerPort: 80
  ```

3. Deploy the Pod by applying the yaml file using the below command. 

  ```bash
  kubectl apply -f pod.yaml
  ```

4. Verify the pod was created using the below command. 

  ```bash
  kubectl get pods
  ```
