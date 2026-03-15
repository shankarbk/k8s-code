## Pod Priority & Preemption in Kubernetes

In Kubernetes, Pod Priority and Preemption ensures that critical workloads run even when the cluster does not have enough resources.   
If resources are full, Kubernetes can evict lower-priority pods to schedule higher-priority pods.   

Pod Priority:
    Defines scheduling importance using PriorityClass numeric value.

Preemption:
    Allows scheduler to evict(remove) lower priority pods to schedule higher priority pods **when resources are insufficient.**

✅ The Problem It Solves

    Imagine a cluster with limited CPU/memory.

        Cluster Capacity:
            CPU: 8 cores
            Memory: 16GB

        Running pods:
            pod-A (2 CPU)
            pod-B (2 CPU)
            pod-C (2 CPU)
            pod-D (2 CPU)

            Total = 8 CPU (cluster full)

    Now a critical pod arrives:
        payment-service (4 CPU)

        Scheduler cannot place it because the cluster is full.
        Without priority → pod stays Pending.
        With Priority + Preemption → Kubernetes can remove low priority pods and run the critical one.

✅ Concepts
| Concept    | Meaning                                                           |
| ---------- | ----------------------------------------------------------------- |
| Priority   | Determines **importance of a pod**                                |
| Preemption | **Killing lower priority pods** to schedule a higher priority pod |

Priority influences the schedular (Kube Schedular). not the Kubelet.

✅ Create PriorityClass
    First we define a PriorityClass:
    low-priority-class.yaml
    high-priority-class.yaml

    Higher number = higher priority

✅ Using Priority in a Pod
    imaging we have low priority pods are running --> low-priority-pod.yaml
    Then Schedule high priority pod --> high-priority-pod.yaml

    🔹What Happens During Preemption:
        Scenario:

            Cluster resources full (Imagine Our cluster is running with full capacity)
            Running pods:
                batch-job-1 (low priority)
                batch-job-2 (low priority)
                batch-job-3 (low priority)

            New pod arrives:
                payment-service (high priority)

            Scheduler logic:
                1. Scheduler checks cluster → no resources
                2. Looks for lower priority pods
                3. Selects victims
                4. Evicts them
                5. Schedules high priority pod

            Result:
                payment-service → Running
                batch-job-1 → Terminated
                batch-job-2 → Terminated

    🔹Architecture Flow:

        High Priority Pod Created
                │
                ▼
        Kubernetes Scheduler
                │
                ▼
        Cluster Full?
                │
            YES
                │
                ▼
        Find Lower Priority Pods
                │
                ▼
        Preempt (Evict) Victim Pods
                │
                ▼
        Schedule High Priority Pod

✅ Built-in Kubernetes Priority Classes

    Kubernetes already includes critical classes:

    | PriorityClass           | Purpose               |
    | ----------------------- | --------------------- |
    | system-cluster-critical | Core cluster services |
    | system-node-critical    | Node level components |

    Example:
        CoreDNS
        kube-proxy
        metrics server

    These must always run.

🎯 PDB (PodDisruptionBudget)

    A PodDisruptionBudget defines:
        How many pods must remain available during voluntary disruptions.

        - PodDisruptionBudget defines minimum available or maximum unavailable pods during voluntary disruptions to prevent downtime during maintenance.
        
        🔹What is voluntary disruptions ? 
            Voluntary disruptions in Kubernetes are intentional, planned actions taken by administrators or users that temporarily make application pods 
            unavailable.

    Keyword:
        🚨 Voluntary disruptions (very important)

        NOT crashes
        NOT node failures
        NOT OOM kills

    What Counts(considered) as Voluntary Disruption?
        ✔ Node drain (kubectl drain)
        ✔ Cluster upgrade
        ✔ Manual pod deletion
        ✔ Autoscaler scale-down
        ✔ Rolling updates

        ❌ Pod crash → NOT covered
        ❌ Node failure → NOT covered
        ❌ Hardware failure → NOT covered

        👉 PDB does not protect against failures
        👉 It protects against our own actions

    🧠 Why PDB Exists

    Without PDB:
        Drain node → All pods evicted → Outage

    With PDB:
        Drain node → Kubernetes checks budget → Prevents unsafe eviction

    Ex: pdb.yaml
        Code meaning : At least 2 pods must stay running

        Alternative Form :
            Instead of minAvailable, you can use:
                maxUnavailable: 1

            Meaning: Only 1 pod allowed to be down

    🚨 Common Confusion #1
        "PDB guarantees uptime" ---> ❌ False.
        If pods crash → PDB irrelevant.

        PDB ≠ High Availability
        PDB = Safe maintenance guardrail

    🚨 Common Confusion #2

        "PDB blocks everything" ----> ❌ False.

        PDB only blocks evictions, not:
            ✔ Scaling
            ✔ Scheduling
            ✔ Failures
            ✔ Pod restarts

    ⚔️ Critical Interaction With Preemption (Great interview question.)
        This is where it gets interesting.

        During preemption:
            Scheduler tries evicting pods.

        BUT…
            👉 If PDB would be violated → Eviction blocked.

        Result:
            ✔ High-priority pod stays Pending 😈
            Yes — PDB can stop priority preemption.
        
        Explanation: 
            Assume the cluster is running wit full capacity by using CPU and Memory.
            For one of the low priority pod running with the PDB is set as  maxUnavailable: 1 
            when preemption tries to evict this POD, the preemption is blocked/failed.
            Because it voilating the PDB.

        🎯 Real Scenario: 
            pdb-deployment.yaml

            Application runs 3 pods (Replica count → 3 pods)
            PDB ensures at least 2 pods must always stay running (PDB: minAvailable: 2)
            During maintenance, Kubernetes can evict only 1 pod at a time (Allowed disruption: ✔ Only 1 pod can go down)
                
            Try draining node with 2 pods:
                ❌ Drain blocked
                ❌ Eviction denied

            👉 What Happens During Node Drain:
                Example command: kubectl drain node-1 --ignore-daemonsets
                Scenario:
                    node-1
                        ├── my-app-pod-1
                        ├── my-app-pod-2

                    Kubernetes checks the PDB

                Result:
                    Evict pod-1 ✅
                    pod-2 must stay running ❌

                Drain pauses until a new pod starts on another node.

                To Verify: kubectl get pdb

        ✅ Real Production Use Case
                Common workloads that use PDB:
                    API servers
                    Databases
                    Microservices with replicas
                    Critical services

It ensures cluster upgrades or node drains don't cause downtime.
    🧠 Advanced Insight (Interview Gold)
        PDB works via:
            ✔ Eviction API
            ✔ NOT Delete API

        Meaning:
            kubectl delete pod → bypasses PDB 😈
            kubectl drain → respects PDB ✅

    🚨 Common Production Mistake
        Setting:
            minAvailable: 100%

        Result:
            ❌ Nodes cannot drain
            ❌ Upgrades stuck
            ❌ Cluster operations break

        Classic DevOps pain.

💥 One important interview tip
    PDB works only when replicas ≥ 2

    If:
        replicas = 1
        minAvailable = 1

    Node drain will never succeed.