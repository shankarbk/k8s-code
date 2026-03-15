## Pod Priority & Preemption in Kubernetes

In Kubernetes, Pod Priority and Preemption ensures that critical workloads run even when the cluster does not have enough resources.   
If resources are full, Kubernetes can evict lower-priority pods to schedule higher-priority pods.   

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

