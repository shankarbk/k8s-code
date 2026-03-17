## Migrate the above F5 NGINX Ingress to Gateway API – detailed hands-on steps (KIND)
✅Prerequisites
    Have the NIC + cafe-ingress.yaml from section 2 running.
    KIND cluster running.

Step 1: Install a Gateway controller OR install the Gateway API CRDs manually .
        Reference document links:
            - [Installing a Gateway controller](https://gateway-api.sigs.k8s.io/guides/getting-started/#installing-a-gateway-controller)
            - [Installing Gateway API](https://gateway-api.sigs.k8s.io/guides/getting-started/#installing-gateway-api)

        If you're done hands-on on "2-gateway-api". skip this step.

Step 2 : Install the conversion tool (kubernetes-sigs/ingress2gateway with NGINX provider)
        Reference link : [Upgrading to Gateway](https://kubernetes.io/blog/2023/10/25/introducing-ingress2gateway/#upgrading-to-gateway)
        > go install github.com/kubernetes-sigs/ingress2gateway@v0.1.0
            # Or download binary from GitHub releases


        