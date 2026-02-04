✅ STEP 1 — Create ServiceAccount
     kubectl create sa docker-sa

     Verify:
     kubectl get sa docker-sa


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
