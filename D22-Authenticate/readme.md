## KubeConfig
🧠 What is KubeConfig

    Kubeconfig is a YAML file that stores cluster connection details, user credentials, and contexts used by kubectl (and other Kubernetes clients) to connect to a cluster.
    
    It defines clusters, users, contexts, and authentication details so you can securely interact with multiple Kubernetes environments without hardcoding credentials or endpoints.

    Theis YAML file used by kubectl to know : WHICH cluster → WHICH user → WHICH permissions

    Without it, kubectl is blind.
    
    The kubeconfig file stored on a client machine (the OS/Machine, where you installed k8s):
        Default location: ~/.kube/config
    
    FOr every kubectl command to api server :
        ✅ It reads your kubeconfig file
        👉 You are authenticated every time.
        👉 But you don’t manually pass credentials each time.
            Because kubeconfig handles that automatically.

    It stores:
        ✔ Clusters (servers)
        ✔ Users (credentials)
        ✔ Contexts (cluster + user combination)

🔑 Key Components of a KubeConfig
    A typical KubeConfig file has four main sections

| Section         |              Purpose                      |
| --------------- | ----------------------------------------- |
| clusters        | (Where to connect) Defines the Kubernetes API server endpoint and certificate authority (CA) data for secure communication. |
| users           | (Who are you) Stores authentication info (client certificates, tokens, or cloud provider credentials). |
| contexts        | (Use THIS user on THIS cluster) Combines a cluster + user + namespace into a single "context" for easy switching. |
| current-context | Specifies which context is currently active. By default all kubectl commands run against this context. |

👉 Refer sample example kubeconfig-example.yaml

⚡ How It Works
    ✔ When you run kubectl get pods, kubectl looks at the current-context in your KubeConfig.
    ✔ That context tells it which cluster to talk to, which user credentials to use, and which namespace to default to.

🚨 Best Practices
    - Keep it secure: 
        KubeConfig often contains sensitive tokens or certificates. Treat it like a password.

    - Multiple clusters: 
        You can merge multiple KubeConfig files (e.g., dev, staging, prod) into one using
        KUBECONFIG=~/.kube/config:~/.kube/config-prod kubectl config view --merge --flatten > ~/.kube/config

    - Do not share blindly: 
        Only use KubeConfig files from trusted sources, as malicious files can expose secrets or execute harmful commands

    - Do not commit kubeconfig file to version control system

❓ Why kubectl config?
    Because:
        👉 You are managing kubectl configuration,
        not querying cluster resources.

    Wrong mental model: kubectl get-contexts ❌

    Correct: kubectl config get-contexts ✅

    Because Contexts live in your local file, not the cluster.


🧩 Real-World Analogy (Sticky Memory Trick)
    Imagine:
        ✔ Cluster = Website URL
        ✔ User = Login credentials
        ✔ Context = Saved login profile

🔄 Common Commands :

    See contexts : kubectl config get-contexts

    Switch context : kubectl config use-context dev-context

    View full config : kubectl config view

    Rename context : kubectl config rename-context old new

    See current context : kubectl config current-context

    set default namespace to the context : kubectl config -set-context --current --namespace=<namespace-name>
