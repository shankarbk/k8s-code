## Gateway API

Gateway API is the next generation of Ingress.

Gateway API (gateway.networking.k8s.io) is the official next-generation Kubernetes API for L4/L7 routing. It is a set of CRDs that replaces/extends the older Ingress API with a more expressive, role-oriented, and extensible model.

✅ we'll use light weight Kubernetes Gateway API --> NGINX Gateway Fabric (NGF) 

Ingress problems:
    - Not expressive
    - Too many annotations
    - Hard to extend
    - Controller-specific hacks
    
    Gateway API solves this.

✅ Gateway API Architecture

                +----------------+
                |   GatewayClass |
                +----------------+
                         |
                         v
                +----------------+
                |     Gateway    |
                +----------------+
                         |
                         v
                +----------------+
                |    HTTPRoute   |
                +----------------+
                         |
                         v
                +----------------+
                |     Service    |
                +----------------+
                         |
                         v
                        Pods

✅ Components
| Object       | Purpose                   |
| ------------ | ------------------------- |
| GatewayClass | Defines controller        |
| Gateway      | Load balancer entry point |
| HTTPRoute    | Routing rules             |
| BackendRef   | Service destination       |

✅ Key Design Idea

    👉 Role separation: Infrastructure provider (GatewayClass) → Cluster operator (Gateway + listeners + policies) → App developer (Routes). Enables safe multi-tenancy.

    👉 Explicit attachment: Routes reference Gateway with parentRefs + sectionName (specific listener).

    👉 Listeners distinctness rules prevent conflicts.

    👉 AllowedRoutes + ReferenceGrant for cross-namespace security.
    
    👉 Extension points: HTTPRoute filters, BackendTLSPolicy, etc.

    👉 Much richer than Ingress: native gRPC, TLS passthrough, request/response modification, weighted backends, mirroring, etc., without annotations.

    Current status (as of 2026): Core (GatewayClass, Gateway, HTTPRoute) is GA in Standard channel since v0.5+. It is the future standard — many controllers (including F5) have fully migrated to it.

    🔹 Infrastructure admin creates ---> GatewayClass
    🔹 Platform team creates        ---> Gateway
    🔹 App developers creates       ---> HTTPRoute

✅ Core resources (all in gateway.networking.k8s.io group):

    🔹GatewayClass (GA, cluster-scoped): Defines a class of gateways (like StorageClass). Implemented by a specific controller (e.g., "nginx" for F5 NGF). Infrastructure provider role.

    🔹Gateway (GA): Requests infrastructure (load balancer/proxy) with Listeners (port, protocol HTTP/TLS/TCP/UDP, hostname, TLS config). Cluster operator role. One Gateway = one set of listeners.

    🔹HTTPRoute (GA), GRPCRoute (GA), TLSRoute (GA), TCPRoute/UDPRoute (alpha): Define routing rules that attach to a Gateway Listener via parentRefs. Application developer role. Uses routing discriminators (host, path, headers, SNI, etc.) so multiple Routes can safely share the same Listener.

💥 Sample example of Gateway API (hands-on compatible with KIND) 
    we'll use light weight Kubernetes Gateway API --> NGINX Gateway Fabric (NGF) 
    - it supports core resources like GatewayClass, Gateway, HTTPRoute, GRPCRoute, TCPRoute, TLSRoute, and UDPRoute.

    👉 Built by NGINX
    👉 Designed specifically for Gateway API
    👉 MUCH lighter than Istio

Step 1: Install NGF + CRDs (exact commands from official docs):

        🔹Cluster required with the config : 
            > kind create cluster --config D6-create-multiple-cluster/kind-cluster-final.yaml ---> If it already exist Dont execute this

            Wait for nodes Ready: watch kubectl get nodes

        🔹Install Gateway API CRDs (standard channel, if not already):

            kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v2.4.2" | kubectl apply -f -

        🔹Install NGINX Gateway Fabric with exact NodePort override:

            helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
            --create-namespace -n nginx-gateway \
            --set nginx.service.type=NodePort \
            --set-json 'nginx.service.nodePorts=[{"port":31437,"listenerPort":80}, {"port":30478,"listenerPort":443}]'

                                        OR (Prefer the below one)

            helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric \
            --create-namespace -n nginx-gateway \
            --set nginx.service.type=NodePort \
            --set nginx.service.nodePorts[0].port=31437 \
            --set nginx.service.nodePorts[0].listenerPort=80 \
            --set nginx.service.nodePorts[1].port=30478 \
            --set nginx.service.nodePorts[1].listenerPort=443 \
            --set nginx.service.externalTrafficPolicy=Cluster

                👉To uninstall and delete the namespace :
                    helm uninstall ngf -n nginx-gateway
                    kubectl delete namespace nginx-gateway --force --grace-period=0


            🔹Any changes you did for above helm chart Re-apply Gateway and Routes (ensure they are in the same namespace, e.g., default):
                kubectl apply -f gateway.yaml
                kubectl apply -f cafe-routes.yaml
                
                Wait 30–90 seconds for the controller to reconcile.

            🔹Find the dynamically provisioned data-plane Service:
                # Look in the namespace where Gateway lives (usually default)
                > kubectl get svc --all-namespaces | grep nginx   # or grep -i gateway
                    # Example output might show:
                    # default   gateway-nginx   NodePort   10.96.xxx.xxx   <none>   80:30080/TCP,443:30443/TCP   2m

            
            - port: This is the nodePort (must be in 30000-32767 range, matches your KIND containerPort).
            - listenerPort: This is the port in your Gateway Listener spec (you have port: 80 in gateway.yaml).


        🔹Verify installation:
            kubectl get pods -n nginx-gateway          # should show 1/1 or 2/2 ready
            kubectl get svc -n nginx-gateway          # ngf-nginx-gateway-fabric should show NODEPORT 31080:80, 30443:443
            kubectl get gatewayclass nginx -o yaml

Step 2: Deploy the same cafe app (we did in 1-ingress/ directory).
        Create Gateway + HTTPRoutes (exact official example):

        Create : 

                gateway.yaml
                kubectl apply -f gateway.yaml

                cafe-routes.yaml
                kubectl apply -f cafe-routes.yaml

                cafe-app.yaml
                kubectl apply -f cafe-app.yaml

                Wait 60–90 seconds.

                Verify data-plane Service:
                    kubectl get svc gateway-nginx -n default -o wide
                                    OR
                    kubectl get svc -A | grep gateway-nginx

                Verify all the created resources:
                    > kubectl get gatewayclass
                    > kubectl get gateway
                    > kubectl get httproute

        Check Gateway status (very important):  
            kubectl get gateway gateway -o yaml
            kubectl describe gateway gateway

            Look for:
                status.conditions → Accepted: True, Programmed: True
                listeners.conditions → Accepted: True, Programmed: True, ResolvedRefs: True
                If Programmed: False or errors → check kubectl logs -n nginx-gateway -l app.kubernetes.io/name=nginx-gateway-fabric

        Test (no port-forward needed thanks to KIND mapping):
            curl --resolve cafe.example.com:8080:127.0.0.1 http://cafe.example.com:8080/coffee2
            curl --resolve cafe.example.com:8080:127.0.0.1 http://cafe.example.com:8080/tea2
                        OR

            kubectl port-forward svc/gateway-nginx 8080:80 -n default &
            curl http://127.0.0.1:8080/coffee2
            curl http://127.0.0.1:8080/tea2

        If Still No NodePort Shows / No Response
            Run this to confirm data-plane Service exists and has correct nodePort:
                > kubectl get svc -A -o wide | grep -E 'nginx|NodePort'

      -------------  WORKING -------------

✅ Ingress API vs Gateway API

| Aspect | Ingress API | Gateway API (with F5 NGF) |
|------|-------------|-----------------------------|
| **API style** | Single resource (`Ingress`) | Modular (`GatewayClass` + `Gateway` + `Routes`) |
| **Role separation** | Limited separation | Explicit separation (infra admin, platform operator, developer) |
| **Expressiveness** | Basic HTTP routing | Native rich routing, filters, policies, gRPC, traffic splitting |
| **Multi-tenancy** | Weak | Strong (`allowedRoutes`, `ReferenceGrant`, RBAC) |
| **Extensibility** | Controller-specific annotations | Standardized extension points and policies |
| **Future-proof** | Legacy API (still supported) | Official Kubernetes direction (GA core resources) |
| **Configuration** | Often annotation-heavy | Clean declarative YAML, no "magic" strings |

 ✅ Why choose Gateway API

    - It eliminates the "annotation hell" that plagues every Ingress controller.
    - True self-service for application teams while operators retain control.
    - Better security, observability, and portability across clouds/controllers.
    - F5's own NGF is the official, high-performance, production-ready implementation of Gateway API using the same NGINX data plane you already know 
      and trust.
    - Community and Kubernetes SIG are deprecating heavy reliance on Ingress in favor of Gateway API (many controllers are moving).
    - Easier migration path (see below) and long-term support from F5.

    Example:
        Traffic split
        Gateway API supports:
            90% → v1
            10% → v2