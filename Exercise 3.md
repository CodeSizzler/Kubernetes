# Exercise 3: Deploy and Configure Azure Kubernetes Service (AKS) with Security, Storage, and Networking

## Overview:

In this exercise, you will deploy an Azure Kubernetes Service (AKS) cluster in Azure and configure it with proper **networking**, **storage**, and **security** settings.

### Task 1: Deploy and Configure an Azure Kubernetes Service Cluster

1. Run the below command to install the Azure Command Line Interface (CLI). 

  ```
  curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
  ```

2. Run the below command to login to Azure. When prompted, navigate to the URL listed in the command's output in your local browser and enter the code shown in the output. When prompted, sign in using the Azure credentials located in the **Lab Environment** tab of the lab guide. 

  ```
  az login
  ```

3. Create a resource group in Azure by running the below command. 

  ```sh
  az group create --name AKSResourceGroup --location eastus
  ```

4. Deploy an AKS Cluster with role-based access control (RBAC) and Managed Identity enabled by running the below command. 

  ```sh
  az aks create \
    --resource-group AKSResourceGroup \
    --name MyAKSCluster \
    --node-count 3 \
    --enable-managed-identity \
    --enable-aad \
    --enable-azure-rbac \
    --generate-ssh-keys \
    --network-plugin azure
  ```

5. Run the below command to set a network policy for your Azure Kubernetes cluster. Network policies control how Pods communicate with each other and other network endpoints.

  ```sh
  az aks update \
    --resource-group AKSResourceGroup \
    --name MyAKSCluster \
    --network-policy calico
  ```

6. You will now create a Storage Class. These allow you to create Persistant Volumes for your AKS workloads. Storage volumes in AKS are often tied to the lifecycle of individual pods. Persistant Volumes are storage volumes that exist independently of pod lifecycles in AKS. Define a Storage Class by creating a file with the below contents in the **nano** text editor. Save the file as **storageclass.yaml**.  

  ```yaml
  # Save as storageclass.yaml
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    name: managed-premium
  provisioner: disk.csi.azure.com
  parameters:
    storageaccounttype: Premium_LRS
    kind: Managed
  allowVolumeExpansion: true
  reclaimPolicy: Retain
  ```

7. Deploy the Storage Class by applying the yaml file using the below command. 

  ```bash
  kubectl apply -f storageclass.yaml
  ```

8. You can now provision a Persistant Volume by creating a Persistent Volume Claim. Define a Persistent Volume Claim by creating a file with the below contents in the **nano** text editor. Save the file as **pvc.yaml**. 

  ```yaml
  # Save as pvc.yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: my-pvc
  spec:
    accessModes:
      - ReadWriteOnce
    storageClassName: managed-premium
    resources:
      requests:
        storage: 5Gi
  ```

9. Deploy the Persistent Volume Claim by applying the yaml file using the below command.

  ```
  kubectl apply -f pvc.yaml
  ```

10. Create a new pod with the persistant volume claim attached. Create a file with the below contents in the **nano** text editor. Save the file as **pod2.yaml**. 

  ```yaml
  # Save as pod2.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    volumes:
      - name: my-volume
        persistentVolumeClaim:
          claimName: my-pvc
    containers:
      - name: my-container
        image: nginx
        volumeMounts:
          - mountPath: "/mnt/data"
            name: my-volume
  ```

11. Deploy the pod by running the below command. 

  ```
  kubectl apply -f pod2.yaml
  ```

12. Verify the storage is mounted by running the below command. 

  ```
  kubectl describe pod my-pod
  ```

  ![Mounted Persistant Volume](images/MountedPV.png)

13. Enable Azure Defender for Kubernetes by running the below command. Azure Defender for Kubernetes enables threat detection, protection, and security monitoring for your Azure Kubernetes Service (AKS) clusters. 

  ```sh
  az security pricing create --name KubernetesService --tier Standard
  ```