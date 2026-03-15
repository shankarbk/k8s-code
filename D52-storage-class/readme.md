## StorageClass   
A StorageClass in k8s is an API object, that defines how storage should be provisioned dynamically.
It acts like template for creating PersistantVolumns(PVs)

**StorageClass = Blueprint for volumes**

Instead of manually creating disks (PersistentVolumes) by K8s Admins, you tell Kubernetes:
    “When a pod asks for storage of this type, create it like this.”

🧠 Why StorageClass Exists
    
    🔹Without StorageClass:
        1. Admin creates PersistentVolume (PV) manually (static provisioning)
        2. Developer creates PersistentVolumeClaim (PVC)
        3. Kubernetes binds PVC → PV

        This is static and painful at scale.

    🔹With StorageClass:
        1. Developer creates PVC
        2. Kubernetes automatically provisions storage
        3. PV is created dynamically (dynamic provisioning)

        Much cleaner.

🧱 Core Idea

    A StorageClass defines:
        • Which storage backend to use
        • How volumes should be created
        • Policies for lifecycle & binding

📦 Basic Example:
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
    name: fast-storage
provisioner: ebs.csi.aws.com
parameters:
    type: gp3
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

🔎 Key Fields Explained

    1️⃣ provisioner
        Defines who creates the storage.   
        Its the plugin or driver responsible for creating the actual storage dynamically by communicationg to the storage backends (AWs EBS, Azure)
        This providioner is attached to the cluster.

        Examples:
            • ebs.csi.aws.com → AWS EBS
            • kubernetes.io/gce-pd → GCP Disk
            • kubernetes.io/no-provisioner → Local / static

    2️⃣ parameters
        Backend-specific configuration.
        Driver-specific configuration like disk type, IOPS, encryption

        Example for AWS:

            parameters:
                type: gp3
                fsType: ext4

        Example for Azure:

            parameters:
                skuName: Premium_LRS

    3️⃣ reclaimPolicy
        Controls what happens when PVC is deleted.
            • Delete → Disk is destroyed 💥
            • Retain → Disk survives 🛟

        Production systems often use Retain.

    4️⃣ volumeBindingMode
        Very important, often misunderstood.
            • Immediate → Volume created instantly
            • WaitForFirstConsumer → Waits until Pod scheduling

        Why wait?
            Because storage may be zone-aware.
            Without waiting → storage provisioned in wrong zone → pod fails.

    5️⃣ allowVolumeExpansion
        Allows resizing PVC later.
            kubectl edit pvc myclaim

        Without this → resize fails.

🚀 How It Works in Real Life

    Step 1 → Create StorageClass

            Admin defines storage behavior. (just defining, nothing is created)
            sc.yaml

    Step 2 → Developer Creates PVC
            pvc.yaml

    Step 3 → Kubernetes Magic ✨
            Kubernetes:
                1. Reads StorageClass
                2. Calls provisioner
                3. Creates the actual volume (e.g. AWS EBS gp3, GCP pd-ssd, Azure premium disk…)
                4. Creates PV automatically
                5. Binds PVC → PV

                No admin intervention (fully automated.).

🎯 Mental Model (Very Useful)
| Kubernetes Object | Real-World Analogy   |
| ----------------- | -------------------- |
| StorageClass      | Disk creation policy |
| PVC               | Disk request         |
| PV                | Actual disk          |

🧩 volumeBindingMode choices (2025–2026 best practice)
| Mode | When PV is Created | Topology Aware | Recommendation Today |
|------|--------------------|----------------|----------------------|
| **Immediate** | PV is created immediately when the PVC is created | No | Avoid if possible |
| **WaitForFirstConsumer** | PV is created only when the first Pod using the PVC is scheduled | Yes | Strongly recommended |

⚡Default StorageClass
    Many clusters set one StorageClass as default: 
        > kubectl get storageclass

    You see something like:
        NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
        gp3 (default)   ebs.csi.aws.com         Delete          WaitForFirstConsumer   true                   128d

    If a PVC does not specify storageClassName, it gets the default one.
    To create a non-default class:

        metadata:
          name: ultra-fast-nvme
            annotations:
                storageclass.kubernetes.io/is-default-class: "false"

🧩 Real-World Usage Patterns
Typical setups:
    ✔ Fast SSD → Databases
    ✔ Cheap HDD → Logs / backups
    ✔ Encrypted → Sensitive data
    ✔ Zone-aware → Stateful apps



🧠 Static Vs Dynamic Provisioning
| Feature           | Static Provisioning                       | Dynamic Provisioning                  |
| ----------------- | ----------------------------------------- | ------------------------------------- |
| setup             | Admin pre-creates PVs                     | K8s Automatically creates PVs         |
| Flexibility       | Limited, Fixed size                       | Highly-Flexible, on-demand            |
| Developer Effort  | Must match PVC to PV                      | Just request via PVC                  |
| Use case          | Legacy systems, Predictable workloads     | Modern, Scalable, Cloud-native apps   | 