## PV and PVC
    Persistent Volumes (PV) and Persistent Volume Claims (PVC) solve one core problem:
        👉 Pods are ephemeral, but data often must survive pod restarts.

🌊 The Problem Kubernetes Needed to Solve
    Pods can die, be rescheduled, recreated — and when they do, their container filesystem disappears.

    If your app writes:
        database files
        uploaded images
        logs
        caches

    …that data would vanish without persistent storage.

    **Kubernetes separates storage management from application usage**

    That’s where PV & PVC come in.

🧱 Persistent Volume (PV)
    A Persistent Volume is:
        👉 Actual storage provided to the cluster.

    Think of it as:
        “A piece of disk made available to Kubernetes”

    It could be backed by:
        EBS / cloud disk
        NFS
        iSCSI
        local disk
        Ceph / network storage

    Important idea:
        PV is a cluster resource, not tied to any pod.

    Code Example : pv-pvc/pv.yaml
        This says:
            ✔ Provide 1Gi storage
            ✔ Use local node path /mnt/data

🧾 Persistent Volume Claim (PVC)
    A PVC is:
        👉 A request for storage by a user / pod.

    Think of it as:
        “I don’t care where storage comes from — I just need 5Gi”

    PVC is created by developers, not cluster admins.

    Code Example : pv-pvc/pvc.yaml
        This says:
            ✔ Request 500Mi storage
            ✔ Match PV with storageClass manual

🔗 How PV & PVC Work Together
    Kubernetes acts like a matchmaker.
    PVC asks for storage → Kubernetes finds matching PV → They bind together.

    Matching Rules
        PVC must match PV based on:
            ✔ Capacity (>= requested)
            ✔ AccessMode
            ✔ StorageClass

        If match found: PVC → Bound → PV

🏗 Real-World Analogy (Most Helpful)

    Cluster admin builds → PV

    Developer requests → PVC

    Kubernetes assigns matching PV by → Binding

🚀 Using PVC Inside a Pod
    Pods never use PV directly. --> They use PVC.

    Example code : pv-pvc/pod.yaml
        ✔ Pod mounts PVC
        ✔ PVC already bound to PV
        ✔ Data persists across pod restarts

    Real flow : pv-pvc/pod.yaml ---> pv-pvc/pvc.yaml ---> pv-pvc/pv.yaml ---> (clustter)
            pod → pvc → pv

🔄 Reclaim Policy (Critical Concept)

    What happens when PVC is deleted? --> Defined in PV
    
    | Policy               | Behavior                     |
    | -------------------- | ---------------------------- |
    | Retain               | Keep data (manual cleanup)   |
    | Delete               | Remove storage automatically |
    | Recycle (deprecated) | Wipe & reuse                 |

⚡ Static vs Dynamic Provisioning
    Two ways PVs appear.
    1️⃣ Static Provisioning
        Admin manually creates PV.
        PVC waits → binds.
        Old-school, rigid.

    2️⃣ Dynamic Provisioning (Modern K8s : StorageClass)
        PVC triggers automatic storage creation.
        Requires StorageClass.
        PVC → StorageClass → Cloud disk created → Bound.
        Much more scalable.

🚨 Common Beginner Confusion

    ❌ “Why pod won't mount PV directly?”
        Because Kubernetes enforces: 👉 Loose coupling & abstraction
        PVC allows flexibility & dynamic provisioning.

Next Section : storage-class.md