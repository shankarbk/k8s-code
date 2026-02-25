## Namespace
A Kubernetes namespace is simply a logical partition inside a cluster. 

Cluster = Physical infrastructure
Namespace = Logical folder inside cluster

✅ What problem does a namespace solve?

    Without namespaces:
        Everything lives together in one giant pool.
        Pods, services, deployments — all mixed.
        That becomes messy very quickly.

    With namespaces:
        You can organize resources cleanly.

    Example:
        dev → Development workloads
        test → Testing workloads
        prod → Production workloads

        All inside the same cluster.

✅ Why do companies rely heavily on namespaces?

    They allow:
        ✔ Multi-team sharing of clusters
        ✔ Environment separation
        ✔ Access control
        ✔ Cost/resource control
        ✔ Clean management

    Instead of running 10 clusters, run one cluster + multiple namespaces.
    Much cheaper, much simpler.

✅ Default namespaces you always see (kubectl get ns)

    default → Where things go if you don’t specify
    kube-system → Core Kubernetes components
    kube-public → Publicly readable resources
    kube-node-lease → Node heartbeat data


🧠 Fundamental Concept
    👉 Namespaces are NOT created on nodes.

    A Namespace is:
        ✅ A logical grouping construct
        ✅ Stored in etcd
        ✅ Managed by kube-apiserver
        ✅ Exists at cluster control plane level
        ✅ Cluster-scoped resource

        ❌ NOT tied to any node
        ❌ NOT scheduled anywhere
        ❌ NOT running on worker machines

🔎 Where does a namespace “live”?
    Technically:
        👉 In etcd database
            kubectl → kube-apiserver → etcd

        Nodes are never involved.

🎯 Mental Model Correction

    Nodes run:
        ✔ Pods
        ✔ Containers
        ✔ Kubelet workloads

    Control plane manages:
        ✔ Namespaces
        ✔ Deployments
        ✔ Services
        ✔ RBAC
        ✔ All API objects

    Namespace = API object, not workload (application).