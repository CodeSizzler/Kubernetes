# Exercise 2: Deployment and Scaling in Kubernetes

## Overview

This exercise will guide you through working with ReplicaSets, Deployments, and manual scaling in Kubernetes. 

## Task 1: Creating and Deploying a ReplicaSet

1. ReplicaSets are used by Kubernetes to ensure that a specified number of identical replica pods are running at all times. If one pod crashes or fails in any way, Kubernetes will automatically create a new pod. Define a ReplicaSet by creating a file with the below contents in the **nano** text editor. Save the file as **replicaset.yaml**. 

  ```yaml
  # Save as replicaset.yaml
  apiVersion: apps/v1
  kind: ReplicaSet
  metadata:
    name: nginx-replicaset
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
          - name: nginx
            image: nginx
            ports:
              - containerPort: 80
  ```

2. Deploy the ReplicaSet by applying the yaml file using the below command.

  ```bash
  kubectl apply -f replicaset.yaml
  ```

3. Verify the ReplicaSet was created using the below command. 

  ```bash
  kubectl get replicaset
  ```



## Task 2: Exposing your ReplicaSet as a Service

1. Exposing your ReplicaSet as a Service makes it accessible so that traffic can reach the pods it controls. Define a service definition for your ReplicaSet by creating a yaml file with the below contents in the **nano** text editor. Save the file as **service.yaml**. 

  ```yaml
  # Save as service.yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
  spec:
    selector:
      app: nginx
    ports:
      - protocol: TCP
        port: 80
        targetPort: 80
    type: LoadBalancer
  ```

2. Now simply apply the ReplicaSet service using the below command.

  ```bash
  kubectl apply -f service.yaml
  ```

3. Run the below command to verify the prescense of your service. Your ReplicaSet is now accessible. 

  ```bash
  kubectl get services
  ```



## Task 3: Creating a Kubernetes Deployment

1. A Kubernetes deployment is used to manage ReplicaSets, enabling functionality such as scaling, rolling updates, and rollbacks. Define a Kubernetes deployment by by creating a yaml file with the below contents in the **nano** text editor. Save the file as **deployment.yaml**.

  ```yaml
  # Save as deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
          - name: nginx
            image: nginx
            ports:
              - containerPort: 80
  ```

2. Now simply apply the deployment using the below command.

  ```bash
  kubectl apply -f deployment.yaml
  ```

3. Run the below command to verify the prescense of your deployment.

  ```bash
  kubectl get deployments
  ```



## Task 4: Scaling your Deployment

1. To scale a Kubernetes deployment is to increase or decrease the number of running pods in order to efficiently handle varying workloads. Run the below command to scale up your kubernetes deployment to 5 replica pods. 

  ```bash
  kubectl scale deployment nginx-deployment --replicas=5
  ```



2. Run the below command to scale down your kubernetes deployment to 2 replica pods. 

  ```bash
  kubectl scale deployment nginx-deployment --replicas=2
  ```

3. If you run the below command again, you will see the results of your scaling. 

  ```
  kubectl get deployments
  ```
