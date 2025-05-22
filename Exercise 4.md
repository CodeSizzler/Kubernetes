# Exercise 4: Configure Monitoring and Auto-Scaling in AKS

## Overview:

In this exercise, you will configure **Azure Monitor**, **Prometheus**, and **Auto-Scaling (HPA & Cluster Autoscaler)** for an AKS cluster.

### Task 1: Configure and Auto-Scale your Azure Kubernetes Service Cluster

1. Run the below command to set a variable for a log analytics workspace name. Replace **<WORKSPACE_NAME>** with a unique name.

  ```sh
  WORKSPACE_NAME="<WORKSPACE_NAME>"
  ```

2. Run the below command to create a log analytics workspace resource in Azure. 

  ```sh
  az monitor log-analytics workspace create --resource-group AKSResourceGroup --workspace-name $WORKSPACE_NAME --location eastus
  ```

3. Enable Azure Monitor for AKS by running the below command. Azure Monitor is a cloud-based monitoring service that collects, analyzes, and visualizes telemetry data from Azure as well as on-premises resources.

  ```sh
  az aks enable-addons \
    --addons monitoring \
    --resource-group AKSResourceGroup \
    --name MyAKSCluster \
    --workspace-resource-id "$(az monitor log-analytics workspace show --resource-group AKSResourceGroup --workspace-name $WORKSPACE_NAME --query id -o tsv)"
  ```

4. Prometheus is an alerting and monitoring toolkit used by containerized environments such as Kubernetes. To install Prometheus we must first install Helm, a package manager that Kubernetes uses to install complex applications like Prometheus. Run the below command to install Helm. 

  ```sh
  curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
  ```

5. Run the below commands to add Prometheus to the Helm Repository.

  ```sh
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
  helm repo update
  ```

6. Set your Kubernetes context to your AKS cluster by running the below command. 

  ```
  az aks get-credentials --resource-group AKSResourceGroup --name MyAKSCluster --overwrite-existing
  ```

7. Install Prometheus by running the below command. If prompted, navigate to the URL listed in the command's output and enter the code shown in the output. When prompted, sign in using the Azure credentials located in the **Lab Environment** tab of the lab guide. 

  ```sh
  helm install prometheus prometheus-community/kube-prometheus-stack
  ```

8. Run the below command to forward a local port (9090) to a port (9090) on the Prometheus service running inside your Kubernetes cluster. This allows you to access the Prometheus web interface on your local machine. Press **Ctrl + c** to exit.          

  ```sh
  kubectl port-forward svc/prometheus-kube-prometheus-prometheus 9090:9090
  ```



9. You will now enable Horizontal Pod Autoscaler (HPA). This is used to automatically scale the number of pods in a deployment based on CPU or memory usage. We will begin by deploying a sample app. Define the sample app by creating a file with the below contents in the **nano** text editor. Save the file as **sample-app.yaml**. 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: sample-app
        image: k8s.gcr.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: sample-app
spec:
  ports:
  - port: 80
  selector:
    app: sample-app
```

10. Deploy the sample app by applying the yaml file using the below command. 

  ```
  kubectl apply -f sample-app.yaml
  ```

11. Define the Horizontal Pod Autoscaler (HPA) by creating a file with the below contents in the **nano** text editor. Save the file as **hpa.yaml**. This will automatically scale the **sample-app** that you created based on CPU usage. When CPU usage is greater than 50%, your deployment's pods will increase.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: sample-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: sample-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

12. Deploy the HPA by applying the yaml file using the below command. 

  ```
  kubectl apply -f hpa.yaml
  ```

13. Simulate an increase in the lab virtual machine's CPU workload by running the below command. 

  ```sh
  kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://sample-app; done"
  ```

14. Open a new terminal window and log into the lab virtual machine. Run the below command. You should see the number of replicas increase as the CPU load increases. You can then close the new terminal window. In the original terminal windows, press **Ctrl + c** then **Enter** to stop the workload simulation. 

  ```sh
  kubectl get hpa sample-app-hpa --watch
  ```


15. You will now enable Cluster Autoscaler on your AKS cluster. Cluster Autoscaler automatically adjusts the number of nodes in a Kubernetes cluster based on the demand for resources. When pods are in a pending state due to insufficient resources, it adds nodes. It also removes underutilized nodes. Run the below command to enable Cluster Autoscaler on your AKS cluster.

  ```sh
  az aks update \
    --resource-group AKSResourceGroup \
    --name MyAKSCluster \
    --enable-cluster-autoscaler \
    --min-count 2 \
    --max-count 5
  ```

16. Run the below command to increase the number of running pods of your sample app deployment. 

  ```sh
  kubectl scale deployment sample-app --replicas=20
  ```

17. Run the below command to to see your node count slowly increase via the Cluster Autoscaler. Press **Ctrl + c** then **Enter** to return to the original command prompt. 

  ```sh
  kubectl get nodes --watch
  ```
