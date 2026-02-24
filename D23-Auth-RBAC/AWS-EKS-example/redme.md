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

🎯 Scenario (Realistic EKS Example)

    Let’s say:
        ✔ IAM user: dev-user
        ✔ Needs access ONLY to dev namespace
        ✔ Can only view pods

    ✅ Step 1 — IAM → Kubernetes Mapping (aws-auth ConfigMap)
        EKS first needs to recognize the IAM identity.
        This happens in: aws-auth ConfigMap

        code : aws-auth-configMap.yaml

            👉 Meaning:
                IAM User → Kubernetes User
                Added to → dev-group

            Without this step → RBAC won’t work.

    ✅ Step 2 — Create Kubernetes Role (Namespace Permissions)

        code : role.yaml
            We restrict permissions inside namespace dev.
            👉 Allows:
                ✔ View pods
                ❌ Cannot delete
                ❌ Cannot modify

            ONLY inside dev.

    ✅ Step 3 — RoleBinding (Assign Permissions)
        Now bind IAM-mapped group → Role.

        code : role-binding.yaml
            👉 Meaning:
                Anyone in dev-group →
                Can view pods →
            ONLY in namespace dev.

    🔥 The Most Important EKS Insight
        EKS does NOT replace Kubernetes RBAC

        It simply plugs in:
            ✔ AWS IAM → Authentication
            ✔ Kubernetes RBAC → Authorization

    🎯 Common Real-World Pattern in EKS
        Instead of Roles, many teams use:
            ✔ ClusterRole (predefined)
            ✔ RoleBinding (namespace restriction)

        Example: Use built-in view role
            
                roleRef:
                    kind: ClusterRole
                    name: view

            Same effect → cleaner setup.

    🚀 Production-Style Example (Very Common)
        
        Dev team:
            ✔ IAM Role → mapped to group developers

        RBAC:
            ✔ RoleBinding → namespace dev

        subjects:
        - kind: Group
        name: developers

        👉 Scales better than per-user mapping.

Example scenario : Allow IAM role to allow pods in EKS to access AWS services (e.g., S3).

     Step 1: Create an IAM Role for EKS (IRSA)
This IAM role will allow pods in EKS to access AWS services (e.g., S3).
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": ["arn:aws:s3:::my-demo-bucket"]
    }
  ]
}


- Attach this policy to an IAM role.
- Trust policy must allow the EKS OIDC provider to assume the role:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/<OIDC_PROVIDER>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "<OIDC_PROVIDER>:sub": "system:serviceaccount:default:my-app-sa"
        }
      }
    }
  ]
}


📌 Step 2: Create a Kubernetes ServiceAccount annotated with IAM Role
This links the IAM role to the Kubernetes ServiceAccount via IRSA.
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: default
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/EKS-S3-Access-Role


📌 Step 3: Create a Kubernetes Role and RoleBinding
This controls Kubernetes API access (not AWS). For example, allow listing pods:

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-app-sa
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io



⚡ How it all ties together
    - IAM Role (IRSA) → Grants AWS permissions (e.g., S3 access).
    - ServiceAccount annotation → Connects IAM role to Kubernetes pods.
    - Role + RoleBinding → Grants Kubernetes API permissions (e.g., list pods).

    So your pod using my-app-sa can both:
        - Call AWS APIs (via IAM role).
        - Interact with Kubernetes resources (via RoleBinding).
