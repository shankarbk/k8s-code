# ClusterRole and ClusterRoleBindng

Kubernetes uses Role-Based Access Control (RBAC) to manage permissions. 
Key objects in this system are
    ✔ Role
    ✔ RoleBinding
    ✔ ClusterRole
    ✔ ClusterRoleBinding

    we have seen Role and RoleBinding earlier (D23a-RoleRoleBinding), lets look at ClusterRole and ClusterRoleBinding

🔹 ClusterRole
    A ClusterRole is just a set of permissions. That’s it.

    It answers:
        “What actions (verbs) are allowed on which resources ?”

    Its a cluster Wide scope

    Think:
        Role        → Permissions limited to ONE namespace
        ClusterRole → Permissions defined at CLUSTER scope

    Code file : cluster-role.yaml
        Meaning:
            ✔ Can read pods
            ✔ Across the cluster (not namespace-restricted)

            But notice something important:
                👉 ClusterRole itself grants NOTHING
                It’s just a definition.
                No subject yet.

🔹 ClusterRoleBinding
    A ClusterRoleBinding binds ClusterRole to user/service account/group.
    It answers:
        “Which user/service account/group gets this ClusterRole?”

    Code file : cluster-role-binding.yaml
        Meaning:
            ✔ User shankar
            ✔ Gets pod-reader permissions
            ✔ Across the cluster

🔥 A ClusterRole can be used in TWO ways:

1️⃣ ClusterRole + ClusterRoleBinding
    Grants permissions cluster-wide

    Example effects:
        ✔ Read pods in ALL namespaces
        ✔ Access nodes
        ✔ Access cluster-scoped resources

2️⃣ ClusterRole + RoleBinding
    Grants permissions inside ONE namespace

    code file : cluster-role-to-role-binding.yaml
        Meaning:
            ✔ Using ClusterRole rules
            ✔ BUT limited to dev namespace

🔥 ClusterRole ≠ Always cluster-wide
🔥 Binding type decides scope

🔹 When Do You Use ClusterRole?

    Use ClusterRole when permissions involve:
        ✔ Cluster-level resources (nodes, persistentvolumes, etc.)
        ✔ Multiple namespaces
        ✔ Reusable permission templates

    Example:
        "I want read-only access to pods in ANY namespace"
            → ClusterRole + ClusterRoleBinding

        "I want same permissions but only inside dev namespace"
            → ClusterRole + RoleBinding

🔹 Role vs ClusterRole
| Feature                       | Role    | ClusterRole   |
| ----------------------------- | ------- | ------------- |
| Namespace scoped?             | Yes     | No            |
| Can access nodes?             | No      | Yes           |
| Can be reused?                | Limited | Very reusable |
| Used with RoleBinding?        | Yes     | Yes           |
| Used with ClusterRoleBinding? | No      | Yes           |


🔹 RoleBinding vs ClusterRoleBinding
| Resource               | Scope     | Grants Permissions   |
| ---------------------- | --------- | -------------------- |
| **RoleBinding**        | Namespace | Inside ONE namespace |
| **ClusterRoleBinding** | Cluster   | Entire cluster       |
