## role and role binding in k8s
In Kubernetes, Roles and RoleBindings are part of the RBAC (Role-Based Access Control) system. They define what actions are allowed and who can perform them.

✅ Role → "What can be done?"
    A Role defines permissions within a specific namespace

    It answers:
        “What operations are allowed on which resources?”

    Ex: role.yaml
    ✔ Meaning:
        Inside namespace dev, this Role allows:
            • get pods
            • list pods
            • watch pods

        But NOT delete / create / update

✅ RoleBinding → "Who can do it?"
    A RoleBinding attaches the Role to a user, group or ServiceAccount.

    It answers:
        Give permission to user, group or ServiceAccount.

    Ex: role-binding.yaml
    
    Code Explanation:
        Can subjects belong to different namespaces (default and dev)?
            Yes. Perfectly legal.

        A RoleBinding’s namespace controls:
            ✅ Where permissions apply
            ❌ NOT where subjects must live

        Your binding:
            metadata:
                namespace: dev

        Means:
            👉 Permissions are granted inside dev namespace only
            Regardless of where the subjects exist.

  ✅ Your Specific Case
        You bound:
            ✔ User → shankar
            ✔ ServiceAccount → docker-sa in default

        Result:
            Both can access resources in namespace dev

        🧠 Critical Rule
            RoleBinding namespace = Permission scope
            NOT subject scope.

    ✅ What actually happens?
        | Subject        | Namespace                  | Effective Access |
        | -------------- | -------------------------- | ---------------- |
        | User `shankar` | (users are cluster-scoped) | Pods in `dev` ✅  |
        | SA `docker-sa` | `default`                  | Pods in `dev` ✅  |

    ServiceAccount location does NOT restrict binding usage.
    This is very common in real clusters.
        Example:
            👉 CI/CD SA in default namespace which controlling workloads in dev

    🚨 Common Beginner Mistake
        People assume:
            ❌ "ServiceAccount must be in same namespace" ---> Wrong.

        The Only requirement is:
            ✔ You must explicitly specify namespace when SA is not availabe in binding namespace (our service account 'docker-sa' is in default namespace, and our binding namespace is dev)

            Which you did correctly:
                - kind: ServiceAccount
                  name: docker-sa
                  namespace: default   ✅ correct


    ✅ roleRef Check (Important)
        This must exist (The Role must exist in dev, we're creating rolebinding in dev namespace):
            kind: Role
            name: pod-reader-role
            namespace: dev

            Because:
                👉 RoleBinding cannot reference Role from another namespace
                    If pod-reader-role lives in default → this binding will FAIL
