## What is a Kubernetes Volume?
    A Kubernetes volume is a directory accessible to containers in a Pod.

    Kubernetes Volumes provide a way for containers in a Pod to access storage that is decoupled from the container's ephemeral filesystem.
    Volumes are declared at the Pod level (spec.volumes) and mounted into one or more containers via volumeMounts (or volumeDevices for raw block).

    Unlike a container’s ephemeral filesystem, volumes allow data to persist across container restarts or be shared among containers.
    Without volumes → data disappears when container restarts.

🧠 Volume Lifecycle / Scope (Critical Concept)
    Volumes are primarily classified by how long they live.
    1️⃣ Ephemeral Volumes (Pod-scoped)
    2️⃣ Persistent Volumes (Cluster-scoped)

    1️⃣ Ephemeral Volumes (Pod-scoped)
        - Created when the Pod starts and destroyed when the Pod is deleted (or evicted).
        - Data is lost on Pod termination unless the backing storage is on the host (e.g., hostPath). Most common for temporary or injected data.

        Used for:
            ✔ Scratch space
            ✔ Cache
            ✔ Injected configs/secrets

        Examples:
            emptyDir
            configMap
            secret
            projected
            downwardAPI

    2️⃣ Persistent Volumes (Cluster-scoped)
        - Data survives Pod deletion
        - Backed by external storage

        Used for:
            ✔ Databases
            ✔ Stateful apps
            ✔ Shared storage

        Examples:
            persistentVolumeClaim (PVC)
            CSI-backed storage (EBS / EFS / etc.)

🧩 Common Volume Types 
1. emptyDir (Ephemeral – most common scratch space)
2. hostPath (Semi-persistent, node-local)
3. configMap (Ephemeral – config injection)
4. secret (Ephemeral – sensitive data)
5. downwardAPI (Ephemeral – pod metadata injection)
6. projected (Ephemeral – combine multiple sources)
7. nfs (Persistent – network file share)
8. persistentVolumeClaim (Persistent – general durable storage)
9. csi (Modern standard for cloud/external storage)

1. emptyDir (Ephemeral – most common scratch space)
    Temporary empty directory on the node (disk or Memory tmpfs). Shared by all containers in the Pod. Deleted when Pod is removed.
    ✔ Fast
    ✔ Simple
    ❌ Data lost on Pod deletion
    
    Code file : emptydir.yaml

2. hostPath (Semi-persistent, node-local)
    Mounts a file/directory from the host node's filesystem. Data survives Pod deletion but Pod must schedule on the same node.
    Not recommended in production (security & portability issues).
    ✔ Useful for debugging / system agents
    ❌ Breaks portability
    ❌ Security risk
    ❌ Pod tied to node

    Code file : hostPath.yaml

3. configMap (Ephemeral – config injection)
    Inject configuration as files (read-only).
    ✔ No rebuild needed
    ✔ Dynamic config
    ❌ Not for large data

    Code file : configmap.yaml

4. secret (Ephemeral – sensitive data)
    Like configMap but for Secrets (always read-only, base64-encoded).
    (Example is almost identical to configMap, just change configMap: → secret: and reference a Secret object.)
    ✔ Stored in tmpfs (memory)
    ✔ Secure
    ❌ Size limits

    Code file : secret.yaml
    
5. downwardAPI (Ephemeral – pod metadata injection)
    Exposes Pod/ container fields (labels, annotations, IP, resource limits, etc.) as read-only files.
    Useful for logging / monitoring agents.

    Code file : downward-api.yaml
     
6. projected (Ephemeral – combine multiple sources)
    Maps secret + configMap + downwardAPI + serviceAccountToken into one directory.

    Code file : projected.yaml

7. nfs (Persistent – network file share)
    Mounts an existing NFS share. Supports multi-writer. Contents preserved when Pod is deleted.

8. persistentVolumeClaim (Persistent – general durable storage)
    The standard way to consume any PV (local, NFS, CSI, cloud, etc.).
    PVC = request for storage.

    ✔ Data survives Pod restart
    ✔ Cloud-backed storage
    ✔ Production workloads

    Code file : pvc.yaml

9. csi (Modern standard for cloud/external storage)
    Used directly for ephemeral CSI volumes or via PVC for persistent. 
    All major providers (AWS, Azure, GCP, etc.) now require CSI drivers.

🧩 Access Modes (Often Confused with Scope)
    These describe **how storage can be mounted**, NOT lifecycle.
    Important for cloud storage behavior.

    | Mode                    | Meaning               |
    | ----------------------- | --------------------- |
    | **ReadWriteOnce (RWO)** | One node can mount    |
    | **ReadOnlyMany (ROX)**  | Many nodes read-only  |
    | **ReadWriteMany (RWX)** | Many nodes read-write |


🧩 Reclaim policy
    🔹What is Reclaim Policy?
        Reclaim Policy = What happens to the storage AFTER the PVC is deleted
        It applies to the PersistentVolume (PV), not directly to the PVC.

        Think:
            👉 Pod dies → nothing special
            👉 PVC deleted → reclaim policy decides fate of disk

    🔹Available Reclaim Policies
        |           Policy           |          What Happens               |
        | -------------------------- | ----------------------------------- |
        | **Delete**                 | Underlying storage is deleted       |
        | **Retain**                 | Storage kept, manual cleanup needed |
        | **Recycle** *(deprecated)* | Old wipe & reuse behavior           |

        Only Delete and Retain matter today.

        1️⃣ Delete Policy (Most Common in Cloud)
            Behavior:
                ✔ PVC deleted → PV deleted → Disk deleted

            Used when:
                ✔ Dynamic provisioning
                ✔ Temporary workloads
                ✔ Most EKS setups

            Example(YAML):
                persistentVolumeReclaimPolicy: Delete

            Real-world implication
                ✔ Easy automation
                ✔ No orphaned disks
                ❌ Dangerous for databases if PVC accidentally removed

            Classic production horror story.

        2️⃣ Retain Policy (Safer, Manual Control)

            Behavior:
                ✔ PVC deleted → PV becomes Released
                ✔ Disk remains intact

            Admin must:
                ✔ Manually delete disk
                ✔ Or rebind PV

            Example(YAML):
                persistentVolumeReclaimPolicy: Retain

            Used when:
                ✔ Critical data
                ✔ Backup / forensic recovery
                ✔ Compliance requirements

            Tradeoff:
                ✔ Safer for data
                ❌ Requires ops discipline
    
    👉 Refer : pv-pvc.md

💥 Volumes with Respect to Amazon EKS (AWS-managed Kubernetes)
    EKS supports all standard Kubernetes volume types above. However:

        ✔ In-tree awsElasticBlockStore (old EBS) is deprecated/removed → always use the Amazon EBS CSI driver (ebs.csi.aws.com or in EKS Auto Mode ebs.csi.eks.amazonaws.com).

        ✔ Recommended drivers are installed as EKS add-ons (best practice for security & auto-updates).

    🔹 EBS (Elastic Block Store)

        Default persistent storage.
            ✔ Backed by disks
            ✔ High performance
            ✔ Supports RWO only

        Requirements:
            👉 Install EBS CSI Driver

        Behavior:
            ✔ Pod must run in same AZ as EBS volume
            ❌ Cannot mount across nodes

        Use case:
            ✔ Databases
            ✔ Stateful apps

        Code file : EKS EBS Example (most common)
            eks-ebs-storageclass.yaml
                ✔ Creates gp3 EBS volumes
                ✔ Encrypted
                ✔ Expandable

            eks-ebs-pvc.yaml
                ✔ Requests a 10Gi gp3 EBS volume
                ✔ Dynamically provisioned

    🔹 EFS (Elastic File System)
        Shared storage.
            ✔ RWX supported
            ✔ Multiple Pods / Nodes
            ✔ Network filesystem

        Use case:
            ✔ Shared uploads
            ✔ Multi-replica apps

    🔹 FSx (Advanced workloads)

        Used for:
            ✔ High-performance / Windows / Lustre

        Code file : eks-efs.yaml
        
| Storage | CSI Driver | Access Modes | Best For | Multi-AZ? | Dynamic Provisioning|
| ------- | ---------- | ------------ | -------- |---------- |-------------------- |
| EBS (block) | EBS CSI | Mostly RWO (io2 multi-attach limited RWX) | Databases | high-performance single-pod | No (AZ-bound) | Yes |
| EFS (NFS-like) | EFS CSI | RWX | "Shared file storage |  web content |  CI caches | Yes (regional) | Yes (via access points) | 
| FSx for Lustre | FSx CSI | RWX | "HPC/ML | high-throughput | Yes | Yes | 

⚖️ EKS Volume-Specific Nuances (Very Important)
    ⚠ emptyDir in EKS
        Backed by:
            ✔ Node disk
            ✔ Or memory (if medium: Memory)

        If node dies → data gone.
        Works same as vanilla Kubernetes.

    ⚠ hostPath in EKS
        ✔ Works on EC2 nodes
        ❌ NOT allowed on Fargate

        Why?
            Fargate = no node access.

    ⚠ EBS Constraints
        EBS = AZ-locked.

        Implication:
            ❌ Pod rescheduled to another AZ → volume attach fails

        Solution:
            ✔ Use topology-aware scheduling
            ✔ Use StatefulSets properly

    ⚠ EFS Behavior
        ✔ Network-based → works across nodes
        ✔ Slightly higher latency than EBS

        Tradeoff = flexibility vs performance.

    ⚠ IAM Permissions (Unique to EKS)
        CSI drivers require IAM roles.

        Modern approach:
            ✔ IRSA (IAM Roles for Service Accounts)

        Without IAM → PVC provisioning fails.
            Classic real-world failure pattern.

📘 Mental Model That Prevents Mistakes

    Ask three questions:
        ✅ Does data need to survive Pod restart?
            → Yes → PVC

        ✅ Does storage need sharing across Pods?
            → Yes → RWX → EFS

        ✅ Is workload latency-sensitive?
            → Yes → EBS


⚙️ Recommendations for EKS:
    - Always install CSI drivers via EKS add-ons (eksctl or console).
    - Use gp3 for general EBS, io2 for high IOPS.
    - For multi-AZ shared storage → prefer EFS.
    - Monitor with AWS Console or kubectl get pv,pvc,sc.
    - For raw block or very high performance → use volumeMode: Block in PVC.

This covers the core concepts and practical usage in both vanilla Kubernetes and EKS.
## My code working
Follow the below commands in sequence  to understand:
kubectl apply -f .\redis.yaml
kubectl get pods
kubectl exec -it redis-pod -- sh
    ls -lrt
    df -hk
    cd /data/redis
    ls -lrt
    echo "hi from k8s" > test.txt
    cat test.txt
    apt-ge updata && apt-get install procps -y
    ps -aux
    kill -9 1 
    <!-- we have killed the redis process(container), but the volume persist, because its attached to the pod (not container) -->
    ps -aux
    ls -lrt
    exit
kubectl delete po redis-pod
kubectl apply -f .\redis.yaml
kubectl exec -it redis-pod -- sh
    ls -lrt
    cd /data/redis
    <!-- 
    Check the test.txt file - its deleted now, because we have recreated the pod. 
    This volume is attached to the pod
    -->