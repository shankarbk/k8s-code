## Kubeadm
kubeadm is Kubernetes official command-line tool for bootstrapping (creating) a cluster in a standardized way.   
It’s designed to set up a minimum viable cluster quickly and correctly, following best practices.

Think of it as:
    “A cluster installer that sets up the Kubernetes control plane and joins nodes, but doesn’t manage workloads.”

    It does initialization, not ongoing operations.

🧠 What problem does kubeadm solve?
    Before kubeadm, setting up Kubernetes meant manually configuring:
        - Certificates
        - API server
        - Controller manager
        - Scheduler
        - etcd
        - kubelet configs
        - Networking prerequisites

    Lots of error-prone steps.

    kubeadm automates this using best practices.

🚀 What kubeadm actually does
    kubeadm helps you:
        ✅ Initialize a control plane node
        ✅ Generate certificates
        ✅ Start control plane components
        ✅ Configure kubelet
        ✅ Join worker nodes

    It does NOT:
        ❌ Install container runtime
        ❌ Install kubectl
        ❌ Install CNI plugin
        ❌ Manage cluster lifecycle

🏗 High-Level Architecture
    kubeadm creates the standard Kubernetes components:

    Control Plane:
        - kube-apiserver
        - kube-controller-manager
        - kube-scheduler
        - etcd

    Worker Nodes:
        - kubelet
        - kube-proxy

    All run as static pods managed by kubelet.

🛠️ Typical Workflow
    Install container runtime
    Install kubeadm + kubelet + kubectl

    Control Plane:
        → kubeadm init

    Workers:
        → kubeadm join

    Networking:
        → Apply CNI plugin

🎯 Kubeadm does not
    - install CNI
    - install container runtime 
    - provide GUI

    You should install this saperately.

⚡ Kubeadm is production ready and used by 
    - on-prem clusters --> Kubernetes running inside a company’s own infrastructure rather than a public cloud.
    - Bare-metal
    - Cloud VMs (Ex : EC2)

    🧠 what is bare metal cluster ?
        A bare metal cluster simply means:
            A Kubernetes cluster deployed directly on physical servers without a virtualization or cloud provider layer.
            No AWS EC2, no VMware, no VirtualBox.
            Just real machines.

        👉 The core idea
                In cloud environments:
                    Physical Server → Hypervisor → Virtual Machine → Kubernetes

                In bare metal:
                    Physical Server → Kubernetes

                Kubernetes nodes = actual hardware.

⚙️ Important kubeadm Commands

1️⃣ kubeadm init
    Used on the control plane node

    What it does:
        ✔ Preflight checks
        ✔ Generate certificates
        ✔ Create kubeconfig files
        ✔ Start control plane static pods
        ✔ Output join command

    Example:
        kubeadm init --pod-network-cidr=192.168.0.0/16

    Why CIDR?
        Because your CNI plugin needs IP space.

2️⃣ kubeadm join
    Used on worker nodes

    What it does:
        ✔ Connects worker nodes to cluster
        ✔ Validates token & CA hash
        ✔ Registers node with API server
        ✔ Starts kubelet

    Example:
        kubeadm join <api-server>:6443 \
        --token <token> \
        --discovery-token-ca-cert-hash sha256:<hash>

3️⃣ kubeadm reset
    Cluster cleanup.

    Deletes:
        ✔ Certificates
        ✔ kubelet configs
        ✔ Static pods

4️⃣ kubeadm upgrade
    Helps upgrade clusters to newer Kubernetes versions

🔐 Certificates & Security
    kubeadm automatically generates:
        - Cluster CA
        - API server certs
        - kubelet certs

    Without kubeadm → painful manual TLS setup

📦 Static Pods Concept (Important)
    kubeadm doesn’t run control plane components via systemd.

    Instead:
        👉 Writes YAML manifests into: /etc/kubernetes/manifests/
        kubelet sees them → launches pods.

        Example:
            kube-apiserver.yaml
            kube-scheduler.yaml
            etcd.yaml

    This is why:
        Even if API server dies, kubelet restarts it
        Nice self-healing property.

🧩 kubeadm vs kind vs managed clouds
| Tool            | Purpose                      |
| --------------- | ---------------------------- |
| kubeadm         | Production-grade DIY cluster |
| kind            | Local testing (Docker-based) |
| EKS / GKE / AKS | Managed control plane        |

    Important distinction:
        👉 kubeadm = real cluster setup
        👉 kind = simulated cluster