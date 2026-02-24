# install prometheus in kind cluster via helm

1. create namespace in k8s --> 
    > kubectl create ns monitoring

2. create hem repo and update it 
    > helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    > helm repo update

3. create a file (custom_kube_prometheus_stack.yml)

4. Deploy the chart into a new namespace "monitoring"

    > helm install my-prometheus prometheus-community/prometheus --version 28.9.1 -n monitoring -f ./custom_kube_prometheus_stack.yml

5. check the installed pods
    > kubectl get po -n monitoring

6. chececk all the services for prometheus
    > kubectl get svc -n monitoring

    expose the "my-prometheus-server" service, check how does prometheus-server looks like. to do this 
    # Step 1. change the "ClusterIP" service of my-prometheus-server to NodePort service.
                (This step is Optional in KindCluster, but required in In a cloud/bare-metal cluster because nodes are real machines on your network.)
        > kubectl expose service my-prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext -n monitoring
        Explanation:

            kubectl expose service my-prometheus-server: This tells Kubernetes to look up the existing service named my-prometheus-server and use its pod selectors to create a new service.

            --type=NodePort: Sets the service type to NodePort. This opens a specific port (usually between 30000–32767) on every node in your cluster, allowing external traffic to reach the service via any node's IP.

            --target-port=9090: Specifies the port the Prometheus container is actually listening on. Traffic sent to the service will be forwarded to this port on the pods.

            --name=prometheus-server-ext: The name of the newly created service.

            -n monitoring: Executes the command within the monitoring namespace.

        Traffic Flow Summary:
            External Client: Connects to NodeIP:NodePort (you can find the assigned port using kubectl get svc -n monitoring).
            NodePort: Receives the traffic on the node's physical port and forwards it to the service's internal port.
            Pod: The service routes the traffic to the target-port (9090) on the Prometheus pod. 

        Because of kind cluster, you need to forward the port:
            Explanation :
                Your service:
                    prometheus-server-ext   NodePort   80:32100/TCP

                means:
                    👉 Prometheus is exposed on node port 32100
                    But in kind, nodes are Docker containers — not your host machine.

                    So opening:
                        http://localhost:32100
                        usually does NOT work unless you mapped ports when creating the cluster.

                Use Port-Forward : 
                    > kubectl port-forward svc/prometheus-server-ext 9090:80 -n monitoring
                    Then open:  http://localhost:9090

    # Step 2. port forwarding (works on Kind Cluster - without creating "prometheus-server-ext" service):
        kubectl port-forward svc/my-prometheus-server 9090:80 -n monitoring
        http://localhost:9090

    
    Why Step 1 is optional ?
        (Optional in KindCluster, but required in In a cloud / bare-metal cluster - works because nodes are real machines on your network.)

        In a cloud / bare-metal cluster:
            Browser → NodeIP:NodePort → Service → Pod
            works because nodes are real machines on your network.

        But in kind:
            👉 Nodes are Docker containers
            👉 NodeIP = Docker internal network (172.x.x.x)
            👉 Your host browser cannot see that network

            So NodePort is technically “outside the cluster”……but still inside Docker. That’s the key twist.
        
        ✅ Why ClusterIP + Port-Forward Still Works
            Port-forward is not normal networking.

            Flow is:
                Browser → localhost → kubectl → API Server → Pod
                It tunnels traffic through Kubernetes control plane.

            Meaning:
                ✔ No NodePort needed
                ✔ No NodeIP needed
                ✔ No Docker networking needed

        ✅ NodePort in kind Only Works If Ports Were Mapped
            NodePort becomes usable from host only when you mapped ports at cluster creation:

            Example:
                extraPortMappings:
                - containerPort: 32100
                hostPort: 32100

            Without this:
            🚫 NodePort is trapped inside Docker network

        ✅ Why Engineers Rarely Use NodePort with kind

            In local dev:
                ✔ Port-forward → simplest
                ✔ Ingress → cleaner
                ✔ NodePort → mostly pain + confusion

            NodePort shines in:
                ✔ Bare metal
                ✔ Cloud VMs
                ✔ Real node networking

        ✅ Mental Model Upgrade (Useful Insight)

            Think of kind as:
                👉 Kubernetes inside Docker sandbox

            So:
                ✔ ClusterIP → inside K8s
                ✔ NodePort → inside Docker
                ✔ Port-forward → bypasses everything

# install Grafana in kind cluster via helm

1. create namespace in k8s --> 
    > kubectl create ns visualization

2. create hem repo and update it 
    > helm repo add grafana-community https://grafana-community.github.io/helm-charts/
    > helm repo update

3. Deploy the chart into a new namespace "visualization"
    > helm install my-grafana grafana-community/grafana --version 11.1.7 -n visualization

4. check the pods running related to grafana in the "visualization" namespace 
    > kubectl get pod -n visualization

5. Get your 'admin' user password (The instruction shown after helm install step)
    > kubectl get secret --namespace visualization my-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
    Result : 
        --> EmmGBCgnRWPCP77IE5TCuO5cbPinvZJO2bgHQcnm
        --> admin

5. chececk all the services for Grafana
    > kubectl get svc -n visualization

6. Expose the service as did for Grafana (step 1) and (step2)
    > kubectl port-forward svc/my-grafana 9095:80 -n visualization
    > http://localhost:9095

7. Create data souce 
    Create a prometheus as data source for grafana (prometheus <----- Grafana)
    Home page --> Add data source --> (grafana supports multiple datasource) --> select prometheus --> provide the IP address of the prometheus in "connection" (http://my-prometheus-server.monitoring.svc.cluster.local)
        Q: why "http://my-prometheus-server.monitoring.svc.cluster.local" instead of "http://localhost:9090/" ?
        Because :
            👉 localhost inside Grafana ≠ your machine’s localhost

            ✅ What’s Actually Happening
                You did:
                    ✔ Grafana → http://localhost:9095 (browser)
                    ✔ Prometheus → http://localhost:9090 (browser)

                    Works fine individually.

                But when Grafana connects to:

                    http://localhost:9090
                    Grafana interprets this as: 👉 Grafana container's localhost. NOT your host machine.
            
                Network view:
                    Browser → Host localhost → OK
                    Grafana → localhost → Grafana pod itself

                    Grafana is running inside Kubernetes, not on your laptop.
                    So it looks for Prometheus inside its own pod, where nothing is listening.

            ✅ Correct Way to Connect Grafana → Prometheus (Inside Cluster)
                Use the Kubernetes service name (from Kubernetes DNS), not localhost.
                Your service: my-prometheus-server
                Correct URL inside Grafana:
                    http://my-prometheus-server.monitoring.svc.cluster.local

                                            OR
                    
                            http://my-prometheus-server.monitoring.svc:80

                                            OR

                                http://my-prometheus-server:80
                                (if Grafana is in same namespace)

                The Kubernetes Builds the DNS Name : <service-name>.<namespace>.svc.cluster.local
                                

8. create a dashboard (Optional)
    simplest way to use grafana as a beginer is 
    Home --> Dashboard --> Import Dashboard --> use the id 3662 --> load

    What is Dashboard ID 3662?  
        3662 is a public dashboard from Grafana’s official dashboard library.
        It’s basically a prebuilt visualization template for Prometheus metrics.

    What Does This Dashboard Show?
        It visualizes Prometheus self-metrics, like:

        ✔ Prometheus uptime
        ✔ Scrape durations
        ✔ Target status
        ✔ TSDB stats
        ✔ Query performance
        ✔ Memory usage

        Meaning:
        👉 It monitors Prometheus itself, not your cluster apps.

    ✅ Why People Use Dashboard 3662
        Because it’s:

        ✔ Standard
        ✔ Well-designed
        ✔ No need to build panels manually
        ✔ Good sanity check after setup
        ✔ Helps detect Prometheus issues early

        In real environments, this dashboard is almost always present.

    ✅ Important Clarification (Common Confusion)

        Dashboard 3662 ≠ Kubernetes monitoring dashboard.

            It is for:
            👉 Prometheus server health

            NOT:
                👉 Nodes / Pods / CPU / Memory / etc.
                For Kubernetes you’d use dashboards like:

            ✔ 6417 → Kubernetes cluster monitoring
            ✔ 315 → Node Exporter
            ✔ 1860 → Node metrics
            ✔ 8588 → Kube-state-metrics dashboards

    ✅ What Happens When You Import ID 3662

        Grafana downloads:
            ✔ Dashboard JSON
            ✔ Panel definitions
            ✔ Queries
            ✔ Layout

        Then maps queries to your Prometheus datasource.

9. Expose the "my-prometheus-kube-state-metrics" service as did for Grafana (step 1) and (step2)

    > kubectl get svc -n monitoring
        --> my-prometheus-kube-state-metrics =  which gives lot of information about cluster
    
    > kubectl port-forward svc/my-prometheus-kube-state-metrics 9097:8080 -n monitoring

    > http://localhost:9097
        --> click on Metrics (which gives lot of info) create a new datasouce for this to visualise this properly

        You can't create data source for my-prometheus-kube-state-metrics directly
