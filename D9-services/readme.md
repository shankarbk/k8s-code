## Label matching 
The Service selector labels must match the Pod labels (not the Deployment labels directly).
In Kubernetes, a Service finds backend pods using a label selector.
Those labels must exist on the Pod created by a Deployment.

How It Works:
    Service selector  ─────► matches ─────► Pod labels

🧠 Service does not talk to Deployment directly.

Architecture:
    Service
        │
        ▼
    Label selector
        │
        ▼
        Pods
        ▲
        │
    Deployment creates Pods

Example: 
    🔹sample-deployment.yaml
        Pod label --> labels:
                        app: nginx
    
    🔹sample-service.yaml
        Service selector --> selector:
                                app: nginx

    The Service will route traffic to those pods.

💥 To check service pointing to correct target pods (endpoints)
    > kubectl get endpoints nginx-service
        Result: <none> 
            --> Service will not connect to any pods.

👉 Important Rule (Deployment)
In a Deployment(sample-deployment.yaml):

    spec:
      selector:
        matchLabels:

    must match:
        template:
          metadata:
            labels:

    Otherwise Kubernetes will reject it.

🧩 Quick Summary
| Resource            | Label Used For                        |
| ------------------- | ------------------------------------- |
| Deployment selector | identifies pods managed by deployment |
| Pod labels          | actual labels on pods                 |
| Service selector    | selects pods to send traffic          |

🧠 Key rule:
        Service selector = Pod labels

⭐ The Service selector must match the labels on the Pods. Services use label selectors to discover and route traffic to Pods. Deployments create Pods with specific labels, and if those labels match the Service selector, the Service will forward traffic to those Pods.