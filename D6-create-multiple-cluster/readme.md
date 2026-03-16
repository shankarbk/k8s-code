## Kind cluster
It is a tool used to run Kubernetes clusters using Docker containers as nodes.
Instead of creating real VMs or cloud nodes, each Kubernetes node runs as a Docker container on your machine.

- very popular for local development, testing, CI pipelines, etc.

💥 Internally

    1️⃣ You run
        > kind create cluster
    2️⃣ Kind uses Docker to create containers.
    3️⃣ Each container acts as a Kubernetes node.

✅ Architecture:

    Your Laptop
        │
        ├── Docker
        │   ├── Container → Control Plane Node
        │   ├── Container → Worker Node
        │   └── Container → Worker Node
        │
        └── Kubernetes Cluster

    So in reality:
        Kubernetes Node = Docker Container

✅ Example

🔹Create a cluster (single cluster)
    kind create cluster --name demo

🔹Check nodes
    kubectl get nodes

Example output:

    NAME                 STATUS   ROLES           AGE
    demo-control-plane   Ready    control-plane   1m

✅ Multi-Node Kind Cluster Example
    Kind config file: kind-cluster-d26.yaml

    🔹Create cluster
        kind create cluster --name=cka --config kind-cluster-fianl.yaml

    Now cluster looks like:

        Control Plane
            │
        ┌───┴─────────┐
        Worker 1   Worker 2

✅ What does extraPortMappings actually do?
    - Kind runs each Kubernetes node as a Docker container.

    - By default, you cannot easily reach ports inside those containers from your host machine 
    (laptop), except for the Kubernetes API server (usually mapped to localhost:6443 
    automatically).

    - extraPortMappings tells Docker to create port forwarding rules between:
        hostPort  → your laptop / workstation
        containerPort → the Kind node container

        So when you write:
            containerPort: 31437
            hostPort: 8080

        It means:
            localhost:8080 on your computer
              ↓ forwarded by Docker
            port 31437 inside the Kind control-plane container

    🔹 Most common real-world use cases for your exact ports
        | Host Port | Container Port | Typical Usage in Kind |
        |-----------|---------------|-----------------------|
        | **8080** | `31437` | Very often used for HTTP services / NodePort / Ingress testing |
        | **8443** | `30478` | Commonly used for HTTPS services / secure Ingress |

    - These are not random numbers — many people/scripts choose high container ports 
    (30000–32767 range) to avoid conflicts with well-known ports inside the node, and then map 
    them to nice low numbers (80/443/8080/8443) on the host.


✅ When Kind is Used
    1️⃣ Local Kubernetes learning
    2️⃣ CI/CD Testing
    3️⃣ Kubernetes Controller Development
        Developers test:
            CRDs
            Operators
            Controllers

✅ Kind vs Minikube vs EKS
| Feature                | Kind | Minikube  | EKS |
| ---------------------- | ---- | --------  | --- |
| Runs locally           | YES  | YES       | NO  |
| Uses Docker containers | YES  | NO        |     |
| Uses VM                | NO   | YES       |     |
| Production use         | NO   | NO        | YES |
| Very fast              | YES  | NO        | —   |

✅Important DevOps Interview Line
    👉 Kind runs Kubernetes nodes as Docker containers instead of VMs.

✅ One Important Limitation
    Kind does not expose services externally easily, so sometimes we need port forwarding.

    Example:

        extraPortMappings:
            - containerPort: 80
            hostPort: 80

    This is important for:
        Ingress
        Gateway API
        LoadBalancer testing

Question:
    Is Kind using Docker as container runtime for Pods?

    Answer:
        ❌ No.

        Kind uses Docker only to run Kubernetes nodes, but inside the node Kubernetes uses containerd to run Pods.