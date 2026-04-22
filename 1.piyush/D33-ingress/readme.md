In general ingress means --> requests coming into your application.

## K8s Ingress 
✅ What is K8s ingress ?   
    Ingress in Kubernetes is an API object that manages external access to services inside the cluster, usually via HTTP or HTTPS.   
    It acts as a smart entry point, allowing you to define routing rules based on hostnames, paths, and TLS, but it requires an Ingress Controller (like NGINX or Traefik) to actually enforce those rules.


    In simple words:
        Ingress allows users from the internet to access Kubernetes services using a single entry point (domain or URL path).

✅ Why We Need Ingress
    Without Ingress, to expose our applications to users/external world we use:
        - NodePort
        - LoadBalancer

    Problems:
    | Issue                  | Explanation                              |
    | ---------------------- | ---------------------------------------- |
    | Too many LoadBalancers | Each service creates its own external IP |
    | Expensive              | Cloud LoadBalancers cost money           |
    | Hard to manage         | No central routing                       |

    Ingress solves this by acting like a smart router for HTTP traffic.

🔑 What Ingress Does
    - Exposes services externally: 
            Provides a single IP or domain to access multiple services.
    - Routing rules: 
            Directs traffic based on hostnames (e.g., app.example.com) or paths (e.g., /api, /shop).
    - TLS/SSL termination: 
            Handles HTTPS connections securely.
    - Load balancing: 
            Distributes traffic across multiple backend pods.
    - Virtual hosting: 
            Supports multiple domains on the same cluster.


✅ How Ingress Works
    Request flow:

        User Request (example.com)
                ↓
        External Load Balancer
                ↓
        Ingress Controller
                ↓
        Ingress Rules
                ↓
        Kubernetes Service
                ↓
            Pod

    Example:
        example.com/app1 → service-app1
        example.com/app2 → service-app2

    One IP can route to multiple services.

⚙️ Components of Ingres
    1️⃣ Ingress Resource
        - A YAML object that defines rules for routing external traffic.
        - Example: route /api to one service and /shop to another.

    2️⃣ Ingress Controller
        - The actual implementation that enforces the rules.
        - Popular controllers: NGINX Ingress Controller, Traefik, HAProxy, Kong.
        - Cloud providers often have their own (e.g., AWS ALB Ingress Controller)

⚠️ Ingress alone does nothing without an Ingress Controller.

✅ Popular Ingress Controllers

| Controller                     | Description                     |
| ------------------------------ | ------------------------------- |
| **NGINX Ingress Controller**   | Most widely used                |
| **Traefik**                    | Dynamic routing                 |
| **HAProxy**                    | High-performance load balancing |
| **AWS ALB Ingress Controller** | Used in EKS                     |
| **Istio Gateway**              | Service mesh ingress            |

✅ Key Features of Ingress

    Ingress supports:
        - HTTP/HTTPS routing
        - TLS termination (SSL)
        - Load balancing
        - Path-based routing
        - Host-based routing
        - Central entry point

Simple Example:   

```
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    name: my-app-ingress
    spec:
    rules:
    - host: myapp.com
        http:
        paths:
        - path: /
            pathType: Prefix
            backend:
            service:
                name: my-service
                port:
                number: 80

Meaning: myapp.com → my-service → pods
```

✅ Core Routing Types (Official Ingress Spec):

| Routing Type           | Description                        | Example                      |
| ---------------------- | ---------------------------------- | ---------------------------- |
| **Host-based routing** | Route traffic based on domain name | `app.example.com → service1` |
| **Path-based routing** | Route traffic based on URL path    | `example.com/api → service2` |

✅Additional Capabilities (Provided by Ingress Controllers)
    Depending on the Ingress Controller (like NGINX, Traefik, HAProxy), more features are available:
    | Feature                     | Description                                       |
    | --------------------------- | ------------------------------------------------- |
    | **TLS / HTTPS termination** | Handle SSL certificates                           |
    | **Load balancing**          | Distribute traffic across pods                    |
    | **URL rewrite**             | Modify request paths                              |
    | **Authentication**          | Basic auth / OAuth                                |
    | **Rate limiting**           | Control request rate                              |
    | **Header-based routing**    | Route based on HTTP headers (controller-specific) |
    | **Canary deployments**      | Gradual traffic shifting                          |


    Kubernetes Ingress primarily supports host-based and path-based routing, while advanced features like TLS termination, authentication, and rate limiting depend on the Ingress Controller implementation.

💡 Important concept (many people miss this):
    Ingress itself does not route traffic — the Ingress Controller performs the routing.

    Ingress (rules)
       ↓
    Ingress Controller (NGINX / ALB / Traefik)
       ↓
    Service
       ↓
    Pods

Conclusion :
    Ingress is a Kubernetes API object that manages external HTTP/HTTPS access to services in a cluster using routing rules, typically implemented by an Ingress Controller.

🧠 Gateway API: 
    Kubernetes is evolving toward the "Gateway API", which offers more flexibility and is recommended for new deployments.

## hands-on demo of deploying the NGINX Ingress Controller on a kind cluster
Step 1: Create a kind cluster

Step 2: Install NGINX Ingress Controller
    Better to know this : https://www.youtube.com/watch?v=ExUGVIOrNbE

    The easiest way is to apply the official f5:
        https://docs.nginx.com/nginx-ingress-controller/
        Ex: helm install nginx-ingress oci://ghcr.io/nginx/charts/nginx-ingress --version 2.4.4 -n kube-system

Step 3: Verify Installation:
    kubectl get pods -n kube-system

Step 4: Deploy Sample Applications:
    Let’s create two simple services:
        app1.yaml
        app2.yaml

    apply them :
        kubectl apply -f app1.yaml
        kubectl apply -f app2.yaml

Step 5: Create Ingress Resource:
    ingress.yaml

    Apply it: kubectl apply -f ingress.yaml

    verify : kubectl get ingress

step 6: Get Ingress Controller External IP
    kubectl get svc -n kube-system

Step 7: Test Access:
    Get Node IP : 
        kubectl get nodes -o wide

    Update /etc/hosts (NOT the ClusterIP): 
        Since kind runs inside Docker, you’ll need to map local.test to <INGRESS-IP> in your hosts file:
        - On Windows: open --> C:\Windows\System32\drivers\etc\hosts
                    add INGRESS-IP from step 6 --> <INGRESS-IP>  local.test

Step 8: Use NodePort to access
    curl http://local.test:32292/app1

    Now test:
        curl http://local.test/app1
        # Hello from App1

        curl http://local.test/app2
        # Hello from App2

    ✅ Result
    You now have:
    - NGINX Ingress Controller running in kind.
    - Two apps (app1, app2) exposed via a single domain (local.test).
    - Path-based routing (/app1, /app2).

