# It collects metrics form "D16-requests-limits\metric-server.yaml" which is running in "kube-system" namespace.
# kubectl get all -n kube-system

# Autoscaling 
     Autoscaling in Kubernetes = automatic adjustment of resources based on demand.
     But Kubernetes has THREE completely different autoscalers.

| Autoscaler         | Scales WHAT?         |
| ------------------ | -------------------- |
| HPA                | Number of pods       |
| VPA                | CPU / Memory of pods |
| Cluster Autoscaler | Number of nodes      |


✅ 1️⃣ Horizontal Pod Autoscaler (HPA) ⭐ MOST IMPORTANT

     👉 Adds/removes pods.

     Used when:
          ✔ Traffic increases
          ✔ CPU spikes
          ✔ Load increases

     🔹 Mental Model
          More load → More pods
          Less load → Fewer pods

     Follow the steps OR go to : https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

     Step 1 — Install metrics-server (kind)
          ⚠ HPA needs metrics-server (very important)--> D16-requests-limits/metric-server.yaml
     
     Step 2 — Deployment (hpa-deploy.yaml)
          kubectl apply -f hpa-deployment.yaml

     Step 3 — Create HPA (hpa.yaml)
          kubectl apply -f hpa.yaml

                    OR

          kubectl autoscale deployment php-apache \
          --cpu-percent=50 \
          --min=1 \
          --max=10
     
     sTEP 4 -- Validate 
          > kubectl get hpa

     Step 5 --Increase the load
          > kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"

     step 6 -- watch scaling
          > kubectl get hpa php-apache 
          > kubectl get deployment php-apache

     step 7 -- Stop generating load

          To finish the example, stop sending the load.

          In the terminal where you created the Pod that runs a busybox image, terminate the load generation by typing <Ctrl> + C.

          Then verify the result state (after a minute or so):

          # type Ctrl+C to end the watch when you're ready
          kubectl get hpa php-apache --watch

✅ 2️⃣ Vertical Pod Autoscaler (VPA)

     👉 Adjusts CPU / Memory of pods.

     Used when:
          ✔ Wrong sizing
          ✔ Memory-heavy workloads
          ✔ CPU-heavy workloads

     VPA usually:
          ❌ Restarts pods
          ❌ Cannot resize live containers
     
     ⚠ VPA requires installation (not default)
          Step 1 — Clone autoscaler repo
               git clone https://github.com/kubernetes/autoscaler.git

          Step 2 — Go to VPA folder
               cd autoscaler/vertical-pod-autoscaler/

          Step 3 — Apply EVERYTHING (correct order handled)
               kubectl apply -f deploy/

     step1 : > kubectl apply -f vpa-deploy.yaml

     step 2: > kubectl apply -f vpa.yaml


     step 3 :Observe
          > kubectl describe vpa vpa-demo

          ✔ See recommendations
          ✔ Pods restart with new resources

          Wait ~1–2 minutes.
               You’ll see:

               ✔ CPU recommendation
               ✔ Memory recommendation

✅ 3️⃣ Cluster Autoscaler 🚀 INFRASTRUCTURE LEVEL

     👉 Adds/removes nodes.

     Used when:
          ✔ Pods Pending (no capacity)
          ✔ Scale cluster automatically

     Requires:
          ✔ AWS / GCP / Azure
          ✔ Node groups / ASG

     Mental Model :
          Pods Pending → Add node
          Nodes idle → Remove node