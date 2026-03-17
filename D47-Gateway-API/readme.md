we'll use light weight Kubernetes Gateway API --> NGINX Gateway Fabric (NGF) 
- it supports core resources like GatewayClass, Gateway, HTTPRoute, GRPCRoute, TCPRoute, TLSRoute, and UDPRoute.

👉 Built by NGINX
👉 Designed specifically for Gateway API
👉 MUCH lighter than Istio

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

    3. Kong
        API Gateway + Ingress controller
        Strong plugin ecosystem:
            Auth
            Rate limiting
            Logging

        Works with:
            Ingress
            Gateway API

        👉 Think:
            “Ingress + API management”

    4. APISIX
        High-performance API Gateway (Apache)
        Built on NGINX + etcd
        Dynamic config (no reload needed)

        👉 Think:
            “Modern, dynamic API gateway (faster than Kong in some cases)”


Q: Difference between Kong and Istio?
    Most people fail here.

    ✔ Correct answer:
        Kong → north-south traffic (external → cluster)
        Istio → east-west traffic (service → service)
