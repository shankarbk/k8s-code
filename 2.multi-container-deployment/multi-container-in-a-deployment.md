Below is the same multi-container example, but wrapped inside a Deployment (recommended for real workloads).

## We will deploy:
    2 replicas
    2 containers in the same Pod
    Shared volume
    Same Nginx + BusyBox (sidecar) pattern

## How This Works (Deployment Context)
✔ Deployment
    Ensures desired replicas are always running
    Handles self-healing & rolling updates
    Each replica = one Pod with two containers

Deployment
 ├── Pod-1
 │    ├── nginx
 │    └── busybox
 ├── Pod-2
 │    ├── nginx
 │    └── busybox

✔ Shared Volume Per Pod
    emptyDir is Pod-scoped
    Each Pod has its own isolated shared volume
    Containers inside the Pod can read/write freely

✔ Networking
    Containers communicate via: localhost
    No Service required for container-to-container communication

## Deploy It
kubectl apply -f multi-container-in-a-deployment.yaml

Verify:
kubectl get deployments
kubectl get pods

## Check Containers in a Pod
kubectl get pod -l app=multi-container-demo

Exec into nginx container:
kubectl exec -it <pod-name> -c nginx-container -- sh

Exec into busybox container:
kubectl exec -it <pod-name> -c busybox-container -- sh

## Expose the Application 
kubectl port-forward deployment/multi-container-deployment 8080:80

Open browser:
http://localhost:8080

## Or Create a Service

```
apiVersion: v1
kind: Service
metadata:
  name: multi-container-service
spec:
  selector:
    app: multi-container-demo
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```