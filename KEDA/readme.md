To set up [KEDA](https://artifacthub.io/packages/helm/kedacore/keda) (Kubernetes Event-Driven Autoscaling) in a local kind (Kubernetes in Docker) cluster, you can follow this hands-on guide.   
KEDA allows you to scale workloads to zero based on external events like cron schedules, message queues, or metrics.   

## 1. Create a kind Cluster
    

## 2. Install KEDA
The most common way to install KEDA is via Helm. This adds the necessary Custom Resource Definitions (CRDs) and controllers to your cluster.  
```
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
kubectl create namespace keda

# Install KEDA into a dedicated namespace
helm install keda kedacore/keda --namespace keda --version 2.19.0
```

> kubectl get pods -n keda

## 3. Deploy a Sample Workload
Deploy a simple NGINX server that we will eventually autoscale.

> kubectl apply -f nginx-deployment.yaml

## 4. Configure a ScaledObject (Cron Example) 
A ScaledObject defines how KEDA should scale your application.   
The Cron scaler is easy to test in a local environment because it doesn't require external message queues.

> Apply it: kubectl apply -f scaledobject.yaml 

## 5. Observe Scaling
KEDA works alongside the standard Horizontal Pod Autoscaler (HPA). You can watch your deployment scale as the cron schedule activates: 
    - Scale to Zero: If it's currently outside the "start" and "end" window, KEDA will scale the ***nginx-deployment*** to 0 replicas.
    - Check Status: Use ***kubectl get scaledobject*** and ***kubectl get hpa*** to see KEDA managing the scaling logic.
