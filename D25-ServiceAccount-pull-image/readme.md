## Service Account
A ServiceAccount in Kubernetes is a special type of account designed for non-human identities. 
Unlike user accounts (which represent people), service accounts are meant for applications, pods, and system components that need to interact with the Kubernetes API securely

Think of it like:
    👉 User account → identity for humans    
    👉 ServiceAccount → identity for pods/apps

🧠 Why do we need ServiceAccounts?
    - Security: Prevents applications from using human credentials.
    - Granularity: Each app can have its own identity and permissions.
    - Automation: System components (like controllers) use service accounts to interact with the cluster.
    - Best Practice: Always assign dedicated service accounts to workloads instead of relying on the default one.

    - Pods often need to talk to the Kubernetes API:
        - Read ConfigMaps / Secrets
        - Query other resources
        - Interact with controllers
        - Access cloud integrations (IRSA in EKS, etc.)

        But Kubernetes is secure by default.
        Every API request must be authenticated + authorized.

        So Pods need credentials → That’s what ServiceAccounts provide.

⚙️ What exactly is a ServiceAccount?

    A ServiceAccount is:
        ✅ An identity object in Kubernetes
        ✅ Namespaced
        ✅ Used by Pods
        ✅ Mapped to RBAC permissions


🚀 What happens when a Pod uses a ServiceAccount?

    When a Pod runs with a ServiceAccount:
    Kubernetes automatically:
        ✅ Generates a token
        ✅ Mounts it inside the Pod
        ✅ Allows the Pod to authenticate to API server

    Inside container:
        /var/run/secrets/kubernetes.io/serviceaccount/
            ├── token
            ├── ca.crt
            └── namespace

        That token = Pod’s credentials.

📘 Key Characteristics

    - Namespaced: Each service account belongs to a specific namespace.
    - Identity Provider: Pods can use a service account to authenticate with the API server.
    - Automatic Mounting: By default, Kubernetes mounts a token (JWT) and CA certificate into pods that use a service account.
    - Default Service Account: Every namespace has a default service account automatically created.

🔐 ServiceAccount + RBAC (MOST IMPORTANT)

    ServiceAccount alone = identity only
    Permissions come from:
        ✅ Role / ClusterRole
        ✅ RoleBinding / ClusterRoleBinding

    Flow:
        Pod → ServiceAccount → RBAC Binding → Permissions
    
    No binding → No access.

✅ Pods should NOT use default SA in production
    Why?
        Because:
            ✔ Default SA often gets excessive permissions
            ✔ Hard to audit
            ✔ Security risk

    Best practice:
        👉 One SA per application

✅ ServiceAccount token = short-lived (modern K8s)
    Older K8s → long-lived Secrets
    Newer K8s → projected tokens (safer)


✅ Cloud IAM Integration

    In managed clusters:
        EKS → IRSA
        GKE → Workload Identity
        AKS → Managed Identity

    ServiceAccount → mapped to Cloud IAM role.(very powerfull)

🧩 Mental Model (Easy Memory Trick)

    Think of ServiceAccount as:
        👉 "Pod’s login user"

    Just like:
        Human logs in → User account
        Pod logs in → ServiceAccount

🎯 Example Scenario 
    We create a Pod that:
        ✅ Can list pods via Kubernetes API
        ✅ Uses ServiceAccount authentication
        ✅ Works only because of RBAC

    We’ll build this scenario:
        👉 Create a ServiceAccount
        👉 Give it permission to read Pods
        👉 Run a Pod using that ServiceAccount
        👉 Verify access from inside container

    step 1 : Create ServiceAccount
        sa.yaml
        kubectl apply -f sa.yaml
        

    step 2: create role
        role.yaml
        kubectl apply -f role.yaml

    step 3: Bind Role to ServiceAccount
        rb.yaml
        kubectl apply -f rb.yaml

    step 4 : Create Test Pod Using ServiceAccount
        pod.yaml
        kubectl apply -f pod.yaml

    Step 5 : Exec Into Pod
        kubectl exec -it sa-test-pod -- sh

    Step 6 : Inspect ServiceAccount Token
        cd /var/run/secrets/kubernetes.io/serviceaccount/
        ls

        You’ll see:
            token
            ca.crt
            namespace

    Step 7: Call Kubernetes API (REAL AUTH TEST)
        Inside container:
            TOKEN=$(cat token)
          
          TRY THIS : 
               curl -s \
               --header "Authorization: Bearer $TOKEN" \
               --cacert ca.crt \
               https://kubernetes.default.svc/api/v1/pods

               Problem:
                    👉 You granted a Role (namespaced)
                    👉 But your API request is interpreted as cluster-scope

               Resolution :
                    Your curl call: https://kubernetes.default.svc/api/v1/pods
                    This endpoint = ALL pods in ALL namespaces
                    Equivalent to: kubectl get pods --all-namespaces
                    Which requires: ✅ ClusterRole, not Role.

                    Grant Cluster-wide access (c-role.yaml,c-role-bind.yaml)
                                        OR 
                    Change the API endpoint to "https://kubernetes.default.svc/api/v1/namespaces/default/pods"--> below WORKING example

          WORKING :
               curl -s \
               --header "Authorization: Bearer $TOKEN" \
               --cacert ca.crt \
               https://kubernetes.default.svc/api/v1/namespaces/default/pods

            Result : 🎉 SUCCESS → JSON output of pods

            Now Pod can list pods ✅

        🚨 Now Let’s Prove RBAC Works
            Try something forbidden:
                curl -s \
                --header "Authorization: Bearer $TOKEN" \
                --cacert ca.crt \
                https://kubernetes.default.svc/api/v1/nodes

                Result: Forbidden
                Because:
                    ✔ Role only allowed pods
                    ✔ Nodes not permitted

    🧠 What Just Happened Internally
        curl → API Server
                ↓
        Token Authentication (ServiceAccount)
                ↓
        RBAC Authorization
                ↓
        Allowed / Denied

        Optional : Check which SA pod uses (Verification From Outside)
            kubectl get pod sa-test-pod -o yaml | grep serviceAccount

    🚀 Real-World Usage

        This pattern is used for:
            ✔ Controllers
            ✔ Operators
            ✔ CI/CD Pods
            ✔ External integrations
            ✔ Cloud IAM mappings (IRSA)

# My working setup - coreui           
✅ STEP 1 — Create ServiceAccount
     kubectl create sa docker-sa

     Verify:
     kubectl get sa docker-sa

     Here we created a service account but no RBAC


✅ STEP 2 — Create Docker Registry Secret (FIXED)
     Your image: sbkqubits/core-ui:1.0
     So registry server MUST be: sbkqubits

     kubectl create secret docker-registry my-docker-registry-key --docker-server=docker.io --docker-username=sbkq --docker-password=dckr_pat_m_fZx --docker-email=sbk.q@gmail.com

     kubectl create secret docker-registry my-docker-registry-key \
     --docker-server=docker.io \
     --docker-username=sbkqubits \
     --docker-password=<PASTE_ACCESS_TOKEN_HERE> \
     --docker-email=sbk.q@gmail.com



     Verify:

     kubectl describe secret my-docker-registry-key

✅ STEP 3 — Attach Secret to ServiceAccount
          kubectl patch serviceaccount docker-sa -p '{"imagePullSecrets":[{"name":"my-docker-registry-key"}]}'


     Verify:
          kubectl describe sa docker-sa

     Expected output:
          Image pull secrets:  my-docker-registry-key

✅ STEP 4 — Create Pod YAML (Corrected)

apiVersion: v1
kind: Pod
metadata:
  name: private-image-pod
spec:
  serviceAccountName: docker-sa
  containers:
  - name: core-ui-app
    image: sbkqubits/core-ui:1.0
    imagePullPolicy: Always
    ports:
    - containerPort: 80

     Apply:
     kubectl apply -f pod.yaml

     ✅ STEP 5 — Verify Pod Status
          kubectl get pods
          Expected:
               private-image-pod   Running

🚫 IMPORTANT: You Do NOT Need Role or RoleBinding

     RBAC controls Kubernetes API access, not image pulling.

     So this part is unnecessary:

     kubectl create role docker-role ...
     kubectl create rolebinding ...


     Image pulling uses registry authentication, not Kubernetes RBAC.

🧪 STEP 7 — Validate Credentials Manually (Strongly Recommended)

     From any machine with Docker:

     docker login sbkqubits
     docker pull sbkqubits/core-ui:1.0


     If this fails → Kubernetes will also fail.

     Fix credentials first.

🧠 FINAL CHECKLIST

     ✔ Secret exists
     ✔ Secret type = dockerconfigjson
     ✔ Secret attached to ServiceAccount
     ✔ Pod uses that ServiceAccount
     ✔ Registry name matches image prefix
     ✔ Pod recreated after changes


🧠 Why Docker Pull Works but Kubernetes Fails

     Your laptop has cached credentials from: docker login


     Kubernetes does not use your laptop credentials.
     It only uses what is inside the Secret.

     So both must be correct.

✅ Final Checklist

✔ Secret recreated with token
✔ docker.io used as server
✔ Secret attached to ServiceAccount
✔ Pod recreated

## Why it’s required

✅ STEP 1 — Create ServiceAccount

A ServiceAccount (SA) represents an identity for processes running inside Pods.

When a Pod runs, Kubernetes assigns it a ServiceAccount.
That ServiceAccount tells Kubernetes:

👉 “Which permissions and which secrets this Pod is allowed to use.”

Without a ServiceAccount:

     Pod uses the default ServiceAccount.

     Default SA usually has no image pull secrets attached.

     Kubernetes won’t know what credentials to use for private images.

In short:

     ServiceAccount = identity of the Pod.

✅ STEP 2 — Create Docker Registry Secret

Private container registries require username/password or token.

Kubernetes cannot pull private images unless credentials are stored in a Secret.

This secret stores:
     Registry address (docker.io)
     Username
     Password / token
     Encoded securely

In short:

     Secret = stores registry login credentials.

Without this:

     Kubernetes tries to pull anonymously

     Registry rejects request → 401 Unauthorized

✅ STEP 3 — Attach Secret to ServiceAccount

Creating a secret alone is not enough.

Kubernetes must know:

     👉 Which Pods are allowed to use this secret?

By attaching the secret to the ServiceAccount:

     Any Pod using that ServiceAccount

     Automatically inherits the registry credentials

So when kubelet pulls the image:
     Pod → ServiceAccount → imagePullSecret → Registry

In short:

     This step connects identity (SA) with credentials (Secret).

## To Access the application 
STEP 1 — Add Label to Pod

     Service finds Pods using labels.
     kubectl apply -f core-ui-pod.yaml

STEP 2 — Create Service YAML  
     kubectl apply -f core-ui-service.yaml


STEP 3 — Verify Service
kubectl get svc core-ui-service

STEP 4 — Access Application
Find node IP: kubectl get nodes -o wide

Open browser:

     http://<NODE-IP>:30080

     Why NodePort Did NOT Work in kind

          kind (Kubernetes in Docker) runs nodes as Docker containers.
          So this IP: 172.18.0.4 is an internal Docker network IP, not reachable directly from your laptop browser.
          That’s why: http://172.18.0.4:30080  ❌
          times out.

     ✅ Correct Ways to Access Services in kind
          ✅ OPTION 1 (Best) — Use localhost with NodePort

          Try:
          http://localhost:30080
          Kind maps NodePorts to localhost automatically.

     ✅ OPTION 2 — Port Forward (What You Did)
          kubectl port-forward pod/core-ui-pod 8080:80
          Access:
          http://localhost:8080

          ✔ Works always
          ✔ Easiest for labs

     ✅ OPTION 3 — Port Forward Service
          kubectl port-forward svc/core-ui-service 8080:80
          Access:
          http://localhost:8080

🧠 Why NodePort?

     Exposes app outside cluster

     Easy for labs / practice

     No cloud load balancer needed
