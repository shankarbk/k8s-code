## CNI   

✅ What is CNI?   
    CNI (Container Network Interface) is a standard for configuring networking for containers in Kubernetes.   

    In simple words:   
        CNI allows Pods to get IP addresses and communicate with other Pods, Services, and external networks.   
        Without CNI, Pods would start but would not have networking.

✅ Why Kubernetes Needs CNI ?
    Kubernetes itself does not implement networking. Instead, it delegates networking to CNI plugins.

    CNI plugins handle:
        - Assigning IP address to Pods
        - Setting up network routes
        - Connecting Pods across nodes
        - Managing network policies

✅ How CNI Works (Pod Creation Flow)
    When a Pod is created, the following happens:

    1️⃣ Pod Scheduled
        Kubernetes scheduler assigns the Pod to a node.

    2️⃣ Kubelet Calls Container Runtime
        Runtime (containerd / Docker / CRI-O) starts creating the container.

    3️⃣ CNI Plugin is Triggered
        Runtime calls the CNI plugin.

    4️⃣ Network Setup
        Plugin:
            - Assigns a Pod IP
            - Creates virtual network interface
            - Configures routes and bridge

    5️⃣ Pod Ready
        Pod can now communicate with:
            - other Pods
            - Services
            - Internet

    Ex: By usimg "Calico" CNI, When a Pod is created

        Pod Created
            ↓
        Scheduler assigns node
            ↓
        Kubelet calls container runtime
            ↓
        Runtime calls Calico CNI plugin
            ↓
        Calico assigns Pod IP
            ↓
        Virtual interface created
            ↓
        Pod gets network connectivity


✅ Popular CNI Plugins   
| CNI Plugin      | Description                                 |
| --------------- | ------------------------------------------- |
| **Calico**      | Most popular, supports **Network Policies** |
| **Flannel**     | Simple overlay networking                   |
| **Cilium**      | Advanced networking using **eBPF**          |
| **Weave Net**   | Easy multi-node networking                  |
| **AWS VPC CNI** | Used in **Amazon EKS**                      |


✅ Kubernetes Networking Rule (Important)
    Kubernetes networking follows 3 key rules:
        1️⃣ Every Pod gets its own IP
        2️⃣ Pods can communicate without NAT (Network Address Translation)
        3️⃣ Pods across nodes can communicate

    CNI plugins implement these rules.

## NAT [Network Address Translation] - (Additional information)
✅ what is NAT ? 
    NAT (Network Address Translation) is a networking technique used to modify IP addresses in packet headers while they pass through a router or firewall.

    In simple terms:
        NAT allows multiple devices in a private network to share a single public IP address to access the internet.

✅ Why NAT is Needed

    Private networks use private IP addresses that cannot communicate directly with the internet.

    Examples of private IP ranges:
        10.0.0.0 – 10.255.255.255
        172.16.0.0 – 172.31.255.255
        192.168.0.0 – 192.168.255.255

    So when a device inside the network wants to access the internet, NAT converts the private IP to a public IP.

✅ Where NAT is Used
    - Home routers
    - Corporate networks
    - Cloud NAT gateways
    - Kubernetes nodes (for outbound internet access)

⚙️ How NAT Works
    - A device in a private network (e.g., 192.168.1.10) sends a request to the internet.
    - The NAT-enabled router replaces the private IP with its public IP (e.g., 203.0.113.5).
    - The router assigns a unique port number to track the session.
    - When the response comes back, NAT maps the port back to the correct private IP.
