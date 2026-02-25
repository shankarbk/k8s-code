## CNI - Container Network Interface
Container Network Interface (CNI) is a specification and framework that Kubernetes uses to **manage networking for containers**.   
It defines how network interfaces are created, configured, and cleaned up when containers start or stop.   

Kubernetes itself does not implement pod networking. It delegates that job to a CNI plugin.   

**CNI is the glue between Kubernetes and networking.** Without a CNI   
plugin, pods cannot talk to each other or the outside world

✅ Why Kubernetes needs CNI ?

    Every pod must:
        ✔ Get an IP address
        ✔ Talk to other pods
        ✔ Reach Services
        ✔ Access the outside world (if allowed)

    But Kubernetes doesn’t know how your network should work. Data center? Cloud VPC? Overlay network?
    That’s where CNI plugins come in.

📘 What CNI Does ?   
    - Assigns IP addresses to pods so they can communicate.   
    - Configures networking between pods across nodes.   
    - Manages connectivity to external networks.   
    - Provides plugins that implement different networking models (overlay, underlay, etc.).

✅ Key Kubernetes networking rule (often misunderstood)

    Kubernetes defines a networking model:
        ✔ Every pod gets its own IP
        ✔ Pods talk without NAT
        ✔ Nodes talk to pods directly

    CNI plugins must implement this model.
    K8s just define i want all these networking models, the CNI plugin should have all these.

✅  Popular CNI plugins (real-world examples)   
        Different plugins solve networking differently:

| Plugin             | Strategy                       |
| ------------------ | ------------------------------ |
| **Flannel** | Simple overlay network, easy to set up. |
| **Calico** | Advanced networking with policy enforcement and BGP routing. |
| **Weave Net** | Mesh-based networking, auto-discovery of peers.|
| **Cilium** | eBPF-powered networking with strong security and observability. |
| **Amazon VPC CNI** | Native VPC IPs (EKS) (Pods get real VPC IPs) |

Choosing the right plugin depends on your cluster’s needs—simplicity (Flannel), security (Calico/Cilium), or advanced routing (Kube-Router).

⚙️ How It Works
- Pod Creation : When a pod is scheduled, the kubelet calls the container runtime.
- CNI Plugin Execution : The runtime invokes the CNI plugin to set up networking.
- Network Setup : The plugin assigns an IP, configures routes, and ensures connectivity.
- Pod Deletion : The plugin cleans up the network resources.

FLOW :
```
When a pod is scheduled
        |
        v
Kubelet calls the container runtime
        |
        v
The runtime invokes the CNI plugin
        |
        v
Network Setup
(Assigns IP, configures routes, ensures connectivity)
        |
        v
Pod is active and running
        |
        v
Pod Deletion
        |
        v
CNI plugin cleans up network resources
(Releases IP and removes virtual interfaces)             
```

✅ Where CNI lives on a node

    On each node:   
        /etc/cni/net.d/      ← CNI config   
        /opt/cni/bin/        ← CNI binaries

✅ Without CNI:
    🚫 Pods would exist
    🚫 But have no network

⚖️ Why CNI Matters   
| Feature             | Benefit                       |
| ------------------ | ------------------------------ |
| **Standardization** | Provides a consistent interface for different networking solutions. |
| **Flexibility** | Lets you choose plugins based on performance, security, or simplicity. |
| **Scalability** | Ensures pods can communicate across nodes and clusters.|
| **Security** | Supports network policies to control traffic between pods.|


## Hands On:
Let’s build a minimal, observable, hands-on example you can run in your cluster.

# Ex 1 : Frontend ↔ Backend ↔ Database isolation
    full-application-piy.yaml
    network-policy-piy.yaml

# Ex 2 :
Goal:
    ✔ Pods can talk by default
    ✔ Then we apply a Calico NetworkPolicy
    ✔ Traffic gets blocked
    ✔ Then selectively allowed

Step 1 — Create a test namespace
    kubectl create ns policy-demo

Step 2 — Create two pods
    pods.yaml
    kubectl apply -f pods.yaml

    Verify :
        kubectl get pods -n policy-demo

Step 3 — Create the Service
    kubectl expose pod nginx --namespace policy-demo --port 80 --name nginx

    Verify :
        kubectl get svc -n policy-demo

Step 4 — Verify connectivity (baseline)
    Exec into busybox:
        kubectl exec -it busybox -n policy-demo -- sh
        wget -qO- nginx

    Result :
        ✔ Should return nginx HTML.
        Important baseline truth:
        Without policies → everything allowed

Step 5 — Apply a DENY policy
    deny.yaml
    kubectl apply -f deny.yaml

    This policy says:
        ❌ Deny ALL ingress to nginx pods

Step 6 — Test again
    kubectl exec -it busybox -n policy-demo -- sh
    wget -qO- nginx

    Result:
        💥 Now it hangs / fails
        Why?
            Because NetworkPolicies are whitelist-based.
            The moment a pod is selected by any policy:
            ✔ Only explicitly allowed traffic works
            ❌ Everything else blocked

Step 7 — Allow traffic from busybox
    Now we refine access.
    allow.yaml
    kubectl apply -f allow.yaml

Step 8 — Test again
    kubectl exec -it busybox -n policy-demo -- sh
    wget -qO- nginx

    Result :
        ✔ Works again.

step 8 : clear all resource
        we have created all these resources in policy-demo namespace.
        so delete that namespace by : kubectl delete ns policy-demo

✅ What you just experienced (this is key)
    This tiny demo proves core Calico/K8s policy behavior:
        ✔ Policies are additive
        ✔ No “deny rules” inside policy
        ✔ Default = allow
        ✔ Policy present = implicit deny

✅ Important Calico-specific superpower (worth knowing)
    Calico extends beyond basic K8s NetworkPolicy.

    You can also:
        ✔ Match by namespace
        ✔ Match by IP blocks
        ✔ Control egress
        ✔ Use GlobalNetworkPolicy
        ✔ Use explicit deny rules (Calico-only)

    Code :
        calico-only-deny.yaml
    
    Kubernetes native policy cannot do this.

✅ Classic troubleshooting tip (real-world useful)
    If policy “doesn’t work”, check:
        kubectl get networkpolicy -n policy-demo
        kubectl describe networkpolicy deny-nginx -n policy-demo

    And confirm Calico is enforcing:
        kubectl get pods -n kube-system | grep calico
