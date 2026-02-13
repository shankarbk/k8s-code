we'll use light weight Kubernetes Gateway API --> NGINX Gateway Fabric (NGF) 
- it supports core resources like GatewayClass, Gateway, HTTPRoute, GRPCRoute, TCPRoute, TLSRoute, and UDPRoute.

👉 Built by NGINX
👉 Designed specifically for Gateway API
👉 MUCH lighter than Istio

What it actually does :
    Gateway API Objects → NGINX Gateway Controller → NGINX Config.

    Same mental model as Ingress — but modern API.

Install Flow :
    you'd install

    1️⃣ Gateway API CRDs
    2️⃣ NGINX Gateway Controller

STEP 1️⃣ — Gateway API CRDs
    install : kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml


    delete: kubectl delete -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml

    ✔ Installs core resources:
        Gateway
        GatewayClass
        HTTPRoute
        Without this → NGF cannot function.

    Verify the installation of CRDs(bash):
        kubectl get crds | grep gateway

STEP 2️⃣ — Install NGINX Gateway Fabric via Helm

    helm install nginx-gateway oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
    --create-namespace \
    -n nginx-gateway

    ✔ Installs:
        Controller
        GatewayClass nginx
        Config objects

✅ WHY THIS ORDER?

    Because Kubernetes works like this:
    👉 CRDs must exist BEFORE resources using them

    Always think (To Remember):

        Foundation → Extensions → Controller

            ✔ Gateway API CRDs → foundation
            ✔ NGF CRDs → extensions
            ✔ NGF Deploy → controller

            Once installation cmplated, it creates a namespace "nginx-gateway"

Some other options of Gateway API:
    1. Traefik
        ✅ Lightweight
        ✅ Simple
        ✅ Popular in dev setups
    2. Istio
        Very powerful… but:
            ❌ Heavy
            ❌ More RAM/CPU
            ❌ Sidecars
            ❌ Overkill for simple local tests

            Use Istio if learning:

            ✔ Service mesh
            ✔ mTLS
            ✔ Traffic shaping
            ✔ Observability

- It creates a saperate Namespace for nginx --> nginx-gateway
    > kubectl get ns

- to verify the pod of nginx-gateway
    > kubectl get po -n nginx-gateway

## Work
1.  check for the getway class name 
 > kubectl get gc
     this above command shows the "controller name"

if getway class not exists create one by (gateway-class.yaml)

2. create gateway (gateway.yaml)

3. create service and the respective pod for ui-app (core-ui)
kubectl apply -f .\core-ui-pod.yaml 
kubectl apply -f .\core-ui-svc.yaml

4. crate route (http-route.yaml)
kubectl apply -f .\http-route.yaml 

## Troubleshooting (i'm using kind cluster)

✅ Your KIND Port Mapping

    You mapped:

        extraPortMappings:
        - containerPort: 30001
        hostPort: 30001


        Meaning:

        ✅ Laptop:30001 → KIND Control Plane Container:30001

        ✔ This part is correct.

✅ Your Gateway Listener (D47-Gateway-API\gateway.yaml)
    listeners:
        - name: http
        protocol: HTTP
        port: 30001


    Meaning:

        ✅ NGINX should listen on port 30001 inside cluster

    ✔ Also logically correct.

👉 Is NGINX actually listening on port 30001 inside the container?
    This is the common misconception.

        Because: Gateway Listener Port ≠ Container Port Automatically

🧠 VERY IMPORTANT Gateway API Reality

    Your Gateway:
        port: 30001

    Means:

        ✔ Logical listener port
        ✔ Gateway abstraction layer

    BUT…

        👉 Underlying NGINX Service / Pod must expose that port

🎯 The Missing Piece (Most Likely Issue)

    NGINX Gateway Fabric usually creates:

        Service → Port 80
        Container → Port 80

    NOT 30001 🚨

        Even if Gateway says 30001.

✅ Let’s Verify (Critical Step)

    Run 👇

    kubectl get svc -n nginx-gateway


You’ll likely see something like:

    NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
    nginx-gateway   ClusterIP   10.96.191.98   <none>        443/TCP   4h36m


🚨 The Real Issue (Crystal Clear Now)

    Your Gateway listener:

        port: 30001


    BUT your NGINX service:

        ONLY exposes 443


    👉 There is NO port 30001
    👉 There is NO port 80
    👉 Only HTTPS (443)

    So traffic to localhost:30001 → 💥 Nothing listening

🧠 Critical Insight (Very Important)

NGINX Gateway Fabric is currently configured as:

    ✅ HTTPS ONLY MODE

    Meaning:

    ✔ NGINX listens on 443
    ✔ Service exposes 443
    ✔ HTTP listener won't work automatically

🎯 Why Your Gateway Still Shows "Programmed=True"

    Because Gateway API config is VALID ✅

    BUT…

    👉 Controller cannot map listener 30001 → No matching service port 🚨

    Config valid ≠ Traffic reachable

🎯 Why NGF Defaults to 443

    Modern gateway design:

    ✔ Secure by default
    ✔ TLS-first model
    ✔ Production-oriented defaults