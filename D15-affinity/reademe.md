## requiredDuringSchedulingIgnoredDuringExecution and preferredDuringSchedulingIgnoredDuringExecution will affect the existing pods ?
Short answer:
    👉 No — they do NOT affect already running pods.

That long phrase actually tells you the behavior:

    DuringScheduling → Only when pod is being scheduled

    IgnoredDuringExecution → Ignored after pod is running

🔹 What “IgnoredDuringExecution” REALLY means

    Once a pod is placed on a node:
        ✅ Kubernetes will NOT re-evaluate the affinity rule
        ✅ Kubernetes will NOT evict the pod
        ✅ Kubernetes will NOT move the pod

    Even if the rule becomes invalid later.

🔥 Why Kubernetes behaves this way

    Imagine if Kubernetes constantly revalidated:
        ❌ Labels change → Pods restart randomly
        ❌ Chaos everywhere
        ❌ Cluster instability

    Kubernetes prioritizes stability over strict rule enforcement.

🔹 When DO pods get affected?

    Only in these cases:
        ✔ Pod restart
        ✔ Pod reschedule
        ✔ Node failure
        ✔ Eviction
        ✔ Rolling update

    Because scheduling happens again.
    Then rules apply again.

🚨 Critical Mental Model

    Affinity rules = Scheduling-time rules

    NOT runtime enforcement rules.

🔥 Compare with Taints (Important contrast)

    Taints with: NoExecute
        👉 CAN evict running pods

    Example:
        kubectl taint node node1 key=value:NoExecute

    Result:

        ❌ Pods without toleration → evicted
        This is runtime enforcement.
    
    Affinity is NOT.

🔥 Easy Memory Trick 😈

    Affinity = Placement logic
    Taints = Node enforcement logic

    Affinity → “Where should I go?”
    Taint → “Who is allowed to stay?”

✅ Exam-style tricky question

    Pod uses requiredDuringSchedulingIgnoredDuringExecution
    Node label removed after scheduling
    What happens?

    Correct answer:
        ✅ Pod continues running