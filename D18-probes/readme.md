## Health Probes 
In Kubernetes, Health Probes are mechanisms that let the kubelet check the health of containers running inside Pods. They ensure that applications are functioning correctly and help Kubernetes decide whether to restart, stop, or route traffic to a container. 
Without probes:
    👉 Kubernetes assumes container is healthy if process is running.

With probes:
    👉 Kubernetes actively tests container state.

🔑 Types of Health Probes
| Probe Type      | Purpose                       | Behavior                       |
| --------------- | ----------------------------- | ------------------------------ |
| Liveness Probe  | “Are you alive?” (Detects if the container is still alive)   | If it fails, Kubernetes restarts the container. Useful for catching deadlocks or hung processes.|
| Readiness Probe | “Can you serve traffic?” (Checks if the container is ready to serve requests) |  If it fails, the Pod is removed from the Service load balancer until it passes again. Prevents traffic from being sent to an unready app. |
| Startup Probe   | “Have you finished starting?” (Ensures the application has started successfully) | Used for slow-starting apps. Until this probe succeeds, Kubernetes won’t run liveness or readiness checks.|

✅ 1️⃣ Liveness Probe → Deadlock / Freeze Detector

    Purpose:
        👉 Detect broken container that is running but useless

    If liveness fails:
        ❌ Container restarted

    Example Scenario
        App:
            ✔ Process running
            ❌ App stuck / deadlocked

        Without liveness:
            👉 Pod looks healthy forever 😈

        With liveness:
            👉 Restarted automatically ✅

        Ex:
            livenessProbe:
                httpGet:
                    path: /health
                    port: 80
                initialDelaySeconds: 5
                periodSeconds: 10

            Meaning:
                ✔ Check every 10s
                ✔ If fails repeatedly → restart

✅ 2️⃣ Readiness Probe → Traffic Controller ⭐ VERY IMPORTANT

    Purpose:
        👉 Decide if pod receives traffic

    If readiness fails:
        ❌ Pod NOT restarted
        ❌ Pod REMOVED from Service endpoints

    Huge difference.

    Example Scenario
        App:
            ✔ Alive
            ❌ DB connection not ready

        Without readiness:
            👉 Traffic sent → errors 😈

        With readiness:
            👉 No traffic until ready ✅

        Ex:
            readinessProbe:
                httpGet:
                    path: /ready
                    port: 80
                initialDelaySeconds: 5
                periodSeconds: 5

    🔥 Critical Insight (Exam Favorite)
    
        Readiness failure:
            ✔ Pod = Running
            ✔ Container = Running
            ✔ BUT no traffic

✅ 3️⃣ Startup Probe → Slow App Protector

    Purpose:
        👉 Protect slow-starting containers

    Without startup probe:
        👉 Liveness may kill slow apps 😈

    With startup probe:
        👉 Liveness disabled until startup succeeds

    Ex:
        startupProbe:
            httpGet:
                path: /startup
                port: 80
            failureThreshold: 30
            periodSeconds: 10

        Meaning:
            ✔ Allow 5 minutes to start

⚙️ Probes can be defined in the Pod spec (.spec.containers[].livenessProbe, .readinessProbe, .startupProbe).

🔥 How kubelet Executes Probes
- Kubelet runs probes via:

    ✔ HTTP request : Kubernetes sends an HTTP request to a specified endpoint.
        Ex:
            httpGet:
                path: /health
                port: 8080


    ✔ TCP socket check : Kubernetes attempts to open a TCP connection (Checks port open).
        Ex:
            tcpSocket:
            port: 3306

        Used for DBs.

    ✔ Exec command : Kubernetes runs a command inside the container and checks the exit code.
        Ex:
            exec:
                command:
                - cat
                - /tmp/healthy

- Parameters like initialDelaySeconds, periodSeconds, timeoutSeconds, and failureThreshold control probe timing and sensitivity.

🔥 Golden Mental Model

    Liveness → Survival check
    Readiness → Traffic check
    Startup → Boot-time protection

🔥 Real Production Usage

    ✔ Liveness → Detect deadlocks
    ✔ Readiness → Avoid bad traffic
    ✔ Startup → Slow JVM / ML apps

✅ LIVENESS PROBE EXAMPLES 
    Liveness = “Restart me if broken”

    1️⃣ Liveness + HTTP (liveness-http.yaml)

        kubectl apply -f liveness-http.yaml

        ✔ Kubelet calls http://pod-ip/
        ✔ If nginx dies → restart

    2️⃣ Liveness + TCP

        kubectl apply -f liveness-tcp.yaml

        ✔ Checks if port 80 open
        ✔ Simple but effective

    3️⃣ Liveness + EXEC

        kubectl apply -f liveness-exec.yaml

        ✔ File exists → healthy
        ✔ Delete file → restart 😈

        Test:
            kubectl exec live-exec -- rm /tmp/healthy

✅ READINESS PROBE EXAMPLES
    Readiness = “Send traffic only if ready”.
    NO restarts.

    1️⃣ Readiness + HTTP

        kubectl apply -f readyness-http.yaml

        Check:
            kubectl get pod ready-http

    2️⃣ Readiness + TCP

        kubectl apply -f readyness-tcp.yaml

        ✔ Port open → ready

    3️⃣ Readiness + EXEC

        kubectl apply -f readyness-exec.yaml

        Initially:
            ❌ NotReady

        Make ready:
            kubectl exec ready-exec -- touch /tmp/ready
        
            ✔ Pod becomes Ready ✅

✅ STARTUP PROBE EXAMPLES

    Startup = “Give me time to boot”
    Protects from premature liveness failures.

    1️⃣ Startup + HTTP

        kubectl apply -f startup-http.yaml

        ✔ Allows slow startup

    2️⃣ Startup + TCP

        kubectl apply -f startup-tcp.yaml

        ✔ Wait for port availability

    3️⃣ Startup + EXEC

        kubectl apply -f startup-exec.yaml

        ✔ Probe fails initially
        ✔ After 10s → succeeds ✅
        ✔ Liveness starts afterwards

🔥 Golden Debug Commands 😈
    kubectl describe pod <pod-name>

    Watch probe failures:
        kubectl get events --sort-by=.metadata.creationTimestamp


🔥 Mental Reinforcement
| Probe Type | Failure Effect      |
| ---------- | ------------------- |
| Liveness   | Restart             |
| Readiness  | Remove from Service |
| Startup    | Delay liveness      |