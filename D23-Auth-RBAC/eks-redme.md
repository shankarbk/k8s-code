## Amazon EKS

    Amazon EKS uses RBAC as the primary authorization mode.

    ✅ Default Authorization in EKS
        EKS API server runs with: authorization-mode = Node,RBAC

        Meaning:
            ✔ RBAC → handles user & workload permissions
            ✔ Node Authorizer → secures kubelet/node communication

    🧠 The AWS Twist (Critical Understanding)
        Unlike vanilla Kubernetes:
            EKS does NOT manage users directly.
            Authentication comes from AWS IAM, not Kubernetes users.

        Flow:
            IAM Identity → Authentication → RBAC Authorization

    ✅ How this actually works
        1️⃣ Authentication → AWS IAM
            IAM user / role:
                ✔ Authenticated via AWS
                ✔ Mapped into Kubernetes identity
            
            Done using:
                aws-auth ConfigMap

            Example:
                mapRoles:
                - rolearn: arn:aws:iam::123456789:role/DevRole
                username: dev-user
                groups:
                    - developers

            👉 IAM Role → Kubernetes Username + Groups

        2️⃣ Authorization → RBAC
            Then RBAC decides:
                ✔ Can dev-user list pods?
                ✔ Can dev-user delete deployments?

            Using:
                ✔ Role / ClusterRole
                ✔ RoleBinding / ClusterRoleBinding

    ✅ Node Authorizer in EKS

        Same as standard Kubernetes:

            ✔ Controls kubelet permissions
            ✔ Prevents node privilege escalation

        You rarely interact with this directly.

What EKS does NOT use by default:
| Mode               | Status in EKS |
| ------------------ | ------------- |
| ABAC               | Not used      |
| Webhook Authorizer | Not used      |
| AlwaysAllow/Deny   | Not used      |


AWS sticks to:

    ✅ IAM for authentication
    ✅ RBAC for authorization

Q: “Which authorization modes are enabled by default in EKS?”
    ✅ RBAC + Node Authorizer

Q: “How does EKS authorization differ from standard Kubernetes?”
    ✅ Authentication via AWS IAM
    ✅ Authorization via RBAC

✔ IAM → Who are you
✔ RBAC → What can you do