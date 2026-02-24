## authorization and its modes in k8s
   In Kubernetes, authorization determines what an authenticated user, service account, or node is allowed to do.

🧠 First: Authentication vs Authorization

    Kubernetes always performs two distinct checks:

        ✅ 1️⃣ Authentication → “Who are you?”
            ✔ Verifies identity
            ✔ Token / cert / OIDC / etc.

        ✅ 2️⃣ Authorization → “What are you allowed to do?”
            ✔ Verifies permissions
            ✔ Happens after authentication

    Example flow:
        Every request to the Kubernetes API server goes through authorization checks. By default, access is denied unless explicitly allowed.

        kubectl delete pod
            ↓
        Authentication → valid user? ✅
            ↓
        Authorization → allowed to delete pods? ❓
    
    Kubernetes evaluates:
        ✔ User / ServiceAccount
        ✔ Verb (get, list, create, delete…)
        ✔ Resource (pods, deployments…)
        ✔ Namespace / scope

🔐 Kubernetes Authorization Modes   
    1️⃣ RBAC (Role-Based Access Control)    
    2️⃣ ABAC (Attribute-Based Access Control)   
    3️⃣ Node Authorization   
    4️⃣ Webhook Authorization   
    5️⃣ AlwaysAllow / AlwaysDeny

|       Mode      |                 Description                 |                   Typical Use Case                    |
| --------------- | ------------------------------------------- |------------------------------------------------------ |
|       RBAC      | Grants permissions based on roles and bindings. Most widely used in production. | Fine-grained access control for users, groups, and service accounts. |
|       ABAC      | Uses policies defined in JSON files with attributes like user, group, and resource. | Legacy clusters; less flexible and harder to maintain. |
| Node Authorization | Special authorizer for kubelets (nodes). Ensures nodes can only access resources they need (e.g., Pods on that node, Secrets for those Pods). | Securing node-level operations. |
| Webhook Authorization | Delegates authorization decisions to an external service via HTTP callbacks. | Integrating custom or enterprise-grade authorization systems. |

NOTE : In KIND, you can easily test RBAC
       Other modes require API server configuration changes


1️⃣ RBAC (Role-Based Access Control) → MOST IMPORTANT
    ✔ Industry standard
    ✔ Used everywhere
    ✔ What you’ll use 99% of the time

    🧠 Concept
        Permissions assigned via:

        ✔ Role / ClusterRole → defines rules
        ✔ RoleBinding / ClusterRoleBinding → assigns rules

    Why RBAC wins
        ✔ Flexible
        ✔ Granular
        ✔ Namespace-aware
        ✔ Cloud-provider standard

    Ex: RBAC directory
        kubectl apply -f RBAC/role.yaml
        kubectl apply -f RBAC/role-binding.yaml

        Verify:
            kubectl get role
            kubectl get rolebinding

        Test : 
            kubectl auth can-i list pods --as shankar --> ✅ Yes
            kubectl auth can-i delete pods --as shankar --> ❌ Delete → denied

2️⃣ ABAC (Attribute-Based Access Control)
    Old model.
    Permissions defined via policy file

    Why rarely used
        ❌ Hard to manage
        ❌ Static file
        ❌ No dynamic updates

    Ex: ABAC directory

3️⃣ Node Authorization
    Used internally by Kubernetes.
        ✔ Controls what kubelets (nodes) can access
        ✔ Ensures node only reads its own pods/secrets

    👉 Security isolation mechanism

    You rarely configure this manually.
    The Node Authorizer runs inside the kube-apiserver (control plane), NOT on worker nodes.

4️⃣ Webhook Authorization
    Delegates decision to external system.
    Flow:
        Request → Kubernetes → External Auth Server → Allow/Deny

    Used when:
        ✔ Enterprise IAM
        ✔ Custom policy engines
        ✔ Advanced security setups

5️⃣ AlwaysAllow / AlwaysDeny
    Exactly what they sound like.
        ✔ Testing
        ✔ Debugging
        ✔ Never production

🧠 How Kubernetes Evaluates Authorization
    Kubernetes can run multiple authorizers together.
    Logic:
        If ANY authorizer approves → Request Allowed
        If ALL deny → Request Denied

🚨 Very Common Confusion
❓ “Authorization vs Admission Controller”

    ✔ Authorization → Permission check
    ✔ Admission → Policy / mutation / validation

Example:
    ✔ Authorization: "Can you create pods?"
    ✔ Admission: "Pod allowed with these security settings?"

❓ how to check local node authorizers in kind ?
    You cannot directly “kubectl get node-authorizer”
    Because the Node Authorizer is an API server internal component, not a resource.

    ✅ Check API Server Authorization Modes
        In KIND, the API server runs inside the control-plane container.
        Run:
            docker ps

            Find your KIND control-plane container: cka-control-plane

            exec into conainer : docker exec -it cka-control-plane bash
                inside container : 
                    cd /etc/kubernetes/manifests
                    ls -lrt
                    cat kube-scheduler.yaml

                    Look for: --authorization-mode=Node,RBAC

                    ✔ If you see Node → Node Authorizer is enabled ✅

