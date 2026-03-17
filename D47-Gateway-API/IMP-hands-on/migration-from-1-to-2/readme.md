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
        > go install github.com/kubernetes-sigs/ingress2gateway@latest
                            OR 
        download binary from GitHub releases (Works for windows) ---> https://github.com/kubernetes-sigs/ingress2gateway/releases

Step 3: Export existing Ingress resources:
        Makesure your ingress exists in cluster : kubectl get ingress
        > kubectl get ingress --all-namespaces -o yaml > ingresses.yaml  
        makesure this converted file is in "UTF-8" format.
            If not, follow the below steps to convert it to UTF-8(follow any one of the below step)
            1. in VS Code:
                Open file → bottom-right click encoding → "Save with Encoding" → UTF-8.

            2. in PowerShell (quick probe):
                Get-Content ingresses.yaml | Set-Content -Encoding utf8 ingresses-fixed.yaml
            
            3. Notepad++ (easiest & reliable)
                Open ingresses.yaml in Notepad++.
                Menu: Encoding → Convert to UTF-8 (without BOM).
                File → Save (or Save As to a new file like ingresses-utf8.yaml).


step 4: Convert to Gateway API resources (NGINX provider handles F5-specific annotations where possible):
        To convert your exported Ingress resources (from ingresses.yaml) for the NGINX provider to Gateway API resources, use:
        > ingress2gateway print --providers=nginx --input-file ingresses.yaml > gateway-resources.yaml

Step 5: Install F5 NGINX Gateway Fabric (replace the old controller)
        Cluster required with the config : 
            > kind create cluster --config D6-create-multiple-cluster/kind-cluster-final.yaml ---> If it already exist Dont execute this

        > kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v2.4.2" | kubectl apply -f -

        > helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway \
        --set nginx.service.type=NodePort \
        --set-json 'nginx.service.nodePorts=[{"port":31437,"listenerPort":80}]'

Step 6: Apply the converted resources
        >kubectl apply -f gateway-resources.yaml
            (If the GatewayClass name is different, edit it to nginx or match your NGF GatewayClass.)

Step 7: Clean up old controller & resources:
        helm uninstall nginx-ingress -n nginx-ingress   # or kubectl delete -f old manifests
        kubectl delete -f cafe-ingress.yaml

Step 8: Verify & test:
        kubectl get gateway,httproute
        kubectl describe gateway gateway
        curl --resolve cafe.example.com:8080:127.0.0.1 http://cafe.example.com:8080/tea1