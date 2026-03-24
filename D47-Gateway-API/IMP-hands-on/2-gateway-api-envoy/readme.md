## Gateway api - Envoy
Envoy Gateway is an open-source, Kubernetes-native API Gateway and reverse proxy control plane that uses the high-performance Envoy Proxy as its data plane.   

It simplifies the configuration and management of Envoy for "north-south" (external-to-internal) traffic by leveraging the standard Kubernetes Gateway API. 

## Hands-on
Step 1 : Create a KIND cluster : D6-create-multiple-cluster\kind-cluster-final.yaml

Step 2: Install Envoy Gateway (includes Gateway API CRDs)
        Use the latest stable version (as of early 2026 it's v1.7.1 — check https://gateway.envoyproxy.io for updates)

        > helm install envoy oci://docker.io/envoyproxy/gateway-helm \
        --version v1.7.1 \
        -n envoy-gateway --create-namespace

        > kubectl wait --timeout=5m -n envoy-gateway deployment/envoy-gateway --for=condition=Available

Step 3 (Follow step 5): Apply the official quickstart (Gateway + sample HTTPRoute + demo app)
        > kubectl apply -f https://github.com/envoyproxy/gateway/releases/download/v1.7.1/quickstart.yaml -n default

        This creates:
            - A GatewayClass managed by Envoy Gateway
            - A Gateway listening on port 80
            - An HTTPRoute for www.example.com
            - A simple backend (returns /get responses)

Step 4: Test it (no external LoadBalancer needed in KIND)

        # Get the internal Envoy service name
            > export ENVOY_SERVICE=$(kubectl get svc -n envoy-gateway \
            --selector=gateway.envoyproxy.io/owning-gateway-namespace=default,gateway.envoyproxy.io/owning-gateway-name=eg \
            -o jsonpath='{.items[0].metadata.name}')

        # Port-forward (runs in background)
            > kubectl -n envoy-gateway port-forward service/${ENVOY_SERVICE} 8888:80 &

        # Test!
            > curl --verbose --header "Host: www.example.com" http://localhost:8888/get

            Result : You should see a JSON response from the demo backend. Success! 🎉

Step 5 : Play with your own resources (real hands-on)
        Create a simple custom setup (example below — apply with kubectl apply -f -):
            gateway.yaml
            kubectl apply -f gateway.yaml

            http-route.yaml
            kubectl apply -f http-route.yaml

            cafe-app.yaml
            kubectl apply -f cafe-app.yaml

        Then update the port-forward and curl with Host: myapp.local.
            > kubectl -n envoy-gateway port-forward service/${ENVOY_SERVICE} 8887:80 &
            Ex: kubectl -n envoy-gateway port-forward service/envoy-default-eg-e41e7b31 8887:80 &

            > curl --verbose --header "Host: www.myapp.local" http://localhost:8887/get

Cleanup:
    helm uninstall eg -n envoy-gateway
    kind delete cluster --name envoy-gateway-test
