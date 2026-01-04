## What’s Happening Here?
1️⃣ Single Pod
    Both containers run inside the same Pod, so:
        Same IP address
        Same localhost network
        Same lifecycle
    
2️⃣ Shared Volume (emptyDir)
```
volumes:
  - name: shared-logs
    emptyDir: {}
```
    Created when the Pod starts
    Deleted when the Pod is removed
    Used to share data between containers

3️⃣ Container Responsibilities
🟢 Nginx Container
    Serves files from: /usr/share/nginx/html
    Exposes port 80

🟡 BusyBox Container
    Writes text to: /logs/index.html
    This file is visible to nginx via shared volume

## Deploy the Pod
kubectl apply -f multi-container-in-a-pod.yaml

Check pod status: kubectl get pods

## Verify Containers Inside the Pod
kubectl describe pod multi-container-pod

Access nginx container:
kubectl exec -it multi-container-pod -c nginx-container -- sh

Access busybox container:
kubectl exec -it multi-container-pod -c busybox-container -- sh

## Access the Application:
Port-forward:
kubectl port-forward pod/multi-container-pod 8080:80

Open browser:
http://localhost:8080
You will see logs written by the BusyBox container, served by Nginx.

next : we'll see the same example using "deployment" 