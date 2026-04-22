## AlwaysAllow / AlwaysDeny
AlwaysAllow and AlwaysDeny are two of the simplest (and most extreme) authorization modes in Kubernetes.   
- They are built-in authorizers inside the kube-apiserver and are controlled via the --authorization-mode flag.   
- They exist almost exclusively for testing, development, debugging, and learning purposes — never use them in any real or shared cluster.
- These are only for testing / learning — never use AlwaysAllow in production.

Quick Comparison
## Kubernetes Authorization Modes Overview

| Mode | What It Does | Effect on Every Request | Real Usage | Security Impact |
|------|--------------|--------------------------|------------|-----------------|
| **AlwaysAllow** | Allows everything | → Permitted (HTTP 200/201/…) | Local dev, quick PoC, learning labs | Extremely dangerous — complete bypass of authorization |
| **AlwaysDeny** | Denies everything | → Forbidden (HTTP 403) | Testing that authorization is enforced | Very safe (but cluster becomes unusable) |
| **RBAC (normal mode)** | Decides based on Role / ClusterRole rules | Depends on user + verb + resource | Production & almost all real clusters | Correct model — least privilege possible |

1. AlwaysAllow – "let everyone do anything"
        kube-apiserver --authorization-mode=AlwaysAllow ...

    🧩 Behavior:

        ✔ The API server skips almost all authorization logic.
        ✔ As long as the request is authenticated (or anonymous access is allowed), it is accepted.
        ✔ Even if you have strict RBAC roles → ignored completely when AlwaysAllow is active.
        ✔ If you combine modes like "--authorization-mode=RBAC,AlwaysAllow" → it still behaves like pure AlwaysAllow (because AlwaysAllow short-circuits to "yes").

    Official warning from kubernetes.io (as of 2025):
        - This mode allows all requests, which brings security risks.
        - Use this authorization mode only if you do not require authorization for your API requests (for example, for testing).

    These are configured directly on the kube-apiserver flag (no YAML file needed).
        # === Always Allow everything === 
        # (extremely dangerous — use only for local dev / testing)
        kube-apiserver \
        --authorization-mode=AlwaysAllow \
        ...

        # === Always Deny everything ===
        # (useful to verify that authz is actually blocking things)
        kube-apiserver \
        --authorization-mode=AlwaysDeny \
        ...

        # Most common realistic combination (production style)
        kube-apiserver \
        --authorization-mode=Node,RBAC \
        ...

    - In kubeadm / kind / minikube setups → edit /etc/kubernetes/manifests/kube-apiserver.yaml
    - In managed clusters (EKS, GKE, AKS) → you usually cannot change this flag


2. AlwaysDeny – "block literally everything"
    kube-apiserver --authorization-mode=AlwaysDeny ...

    🧩 Behavior:

        ✔ Every request — even from cluster-admin, system:masters, kubelet, controller-manager — gets 403 Forbidden.
        ✔ Extremely useful when you want to prove that authorization is actually being enforced somewhere in your pipeline.


🛠️ How modes are evaluated (important!)
    Kubernetes supports multiple authorizers in a chain (comma-separated list):

        --authorization-mode=Node,RBAC
        --authorization-mode=Webhook
        --authorization-mode=AlwaysAllow               # ← dangerous shortcut

    Evaluation rule:

        - Go through the list in order
        - First authorizer that gives a clear allow or deny → decision made
        - If authorizer returns "no opinion" → next one is tried
        - AlwaysAllow almost always says "allow" → anything after it is ignored
        - AlwaysDeny almost always says "deny" → blocks everything

✔ AlwaysAllow → everything works
✔ AlwaysDeny → everything blocked