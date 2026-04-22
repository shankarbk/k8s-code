## etcd Backup steps in on-prem k8s cluster
In an on-prem Kubernetes cluster (kubeadm based), etcd runs as a static pod on control-plane nodes. Backing it up means taking a snapshot of the etcd database.
Below are the standard etcd backup steps.

Step 1: 1. Check etcd pod and certificate locations
    First confirm etcd is running on the control-plane node.
        > kubectl get pods -n kube-system | grep etcd

    Login into the control plane : 
        for kind : docker exec -it cka-control-plane sh
        for cloud use ssh

    Check the etcd manifest to get certificate paths.
        > cat /etc/kubernetes/manifests/etcd.yaml

    Look for these parameters:
        --cert-file=/etc/kubernetes/pki/etcd/server.crt
        --key-file=/etc/kubernetes/pki/etcd/server.key
        --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

Step 2: Set environment variables (optional but easier)
    export ETCDCTL_API=3


Step 3: Take etcd snapshot (backup)
    
    First check: Does etcdctl exist in the node:
        > docker exec -it cka-control-plane sh ( SSH into control plane (master) node. -  if your using kubeadm/on-prem )
        > etcdctl version

    Verify certificate paths:
        In kubeadm clusters the paths are:
            /etc/kubernetes/pki/etcd/ca.crt
            /etc/kubernetes/pki/etcd/server.crt
            /etc/kubernetes/pki/etcd/server.key

        Check if they exist:
            > ls /etc/kubernetes/pki/etcd/
            If not present, get them from the etcd pod.

        Check etcd endpoint:
            > apt update && apt install -y net-tools
            > netstat -tulnp | grep 2379

    Common Kind-custer problems(Troubleshooting):
        ❌ Error: etcdctl: command not found
        Run using the etcd pod container:
            > kubectl -n kube-system exec -it etcd-cka-control-plane -- sh
    
        Then run:
            kubectl -n kube-system exec etcd-cka-control-plane -- \
            etcdctl --endpoints=https://127.0.0.1:2379 \
            --cacert=/etc/kubernetes/pki/etcd/ca.crt \
            --cert=/etc/kubernetes/pki/etcd/server.crt \
            --key=/etc/kubernetes/pki/etcd/server.key \
            snapshot save /tmp/etcd-backup.db

        Then copy it out if needed:
            kubectl cp kube-system/etcd-cka-control-plane:/tmp/etcd-backup.db ./etcd-backup.db
    
    Run on the control-plane node:
        etcdctl snapshot save /backup/etcd-snapshot.db \
        --endpoints=https://127.0.0.1:2379 \
        --cacert=/etc/kubernetes/pki/etcd/ca.crt \
        --cert=/etc/kubernetes/pki/etcd/server.crt \
        --key=/etc/kubernetes/pki/etcd/server.key

    Example output:
        Snapshot saved at /backup/etcd-snapshot.db

Step 4: Verify the backup
    > etcdctl snapshot status /backup/etcd-snapshot.db

    Example output:
    +---------+----------+------------+------------+
    | HASH    | REVISION | TOTAL KEYS | TOTAL SIZE |
    +---------+----------+------------+------------+
    | b714765 | 51465    | 1200       | 45 MB      |
    +---------+----------+------------+------------+

Step 5: Store backup safely
    Copy the backup to a remote location.
    Example:
        scp /backup/etcd-snapshot.db backup-server:/etcd-backups/

    or store in:
        NFS
        S3
        backup server
        versioned storage

Step 6: Automate backups (recommended)
    Create a cron job.
    Example:
        crontab -e
    
    Run every day at 1 AM:
        0 1 * * * ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +\%Y\%m\%d).db \
        --endpoints=https://127.0.0.1:2379 \
        --cacert=/etc/kubernetes/pki/etcd/ca.crt \
        --cert=/etc/kubernetes/pki/etcd/server.crt \
        --key=/etc/kubernetes/pki/etcd/server.key

Quick Summary:
| Step            | Command                   |
| --------------- | ------------------------- |
| Set API version | `export ETCDCTL_API=3`    |
| Take backup     | `etcdctl snapshot save`   |
| Verify backup   | `etcdctl snapshot status` |
| Store safely    | Copy to remote storage    |

💡 Important notes for production clusters
    - Always backup from one control-plane node
    - Ensure etcdctl version matches etcd version
    - Backup before cluster upgrades
    - Keep multiple snapshot versions

## etcd Restore steps in on-prem k8s cluster
1️⃣ Why etcd Restore is Needed

    etcd stores the entire Kubernetes cluster state:
        Pods
        Deployments
        Services
        ConfigMaps
        Secrets
        Nodes

    If etcd is corrupted or lost, the cluster state disappears, so we restore from a snapshot backup

2️⃣ High-Level Restore Workflow
                +-------------------+
                |   Kubernetes API  |
                +---------+---------+
                          |
                          v
                +-------------------+
                |        etcd       |
                | (cluster state DB)|
                +---------+---------+
                          |
                          v
                +-------------------+
                |   etcd snapshot   |
                |  etcd-backup.db   |
                +---------+---------+
                          |
                 RESTORE FROM SNAPSHOT
                          |
                          v
                +-------------------+
                |  restored etcd DB |
                +---------+---------+
                          |
                          v
                Update kube-apiserver
                to use restored data

3️⃣ Step-by-Step etcd Restore (kubeadm cluster)
    Assume snapshot file: /tmp/etcd-backup.db

    Step 1 — Stop kube-apiserver
        Since etcd is used by API server.
        Move the static pod manifest:
            mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/

        Kubelet automatically stops the pod.

    Step 2 — Restore etcd snapshot
        > ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db \
        --data-dir=/var/lib/etcd-restore

        Output example:
            Restoring snapshot...

        This creates a new etcd data directory.

    Step 3 — Backup old etcd data
        mv /var/lib/etcd /var/lib/etcd-old

    Step 4 — Move restored data
        mv /var/lib/etcd-restore /var/lib/etcd

        Now etcd will use the restored database.

    Step 5 — Update etcd static pod (if required)

        Open:
            /etc/kubernetes/manifests/etcd.yaml

        Check:
            --data-dir=/var/lib/etcd

        Ensure it points to the restored directory.

    Step 6 — Start kube-apiserver again

        Move manifest back:
            mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

        Kubelet automatically recreates the pod.

    Step 7 — Verify cluster

        Check nodes:
            kubectl get nodes

        Check pods:
            kubectl get pods -A

        Cluster should return to the state at snapshot time.

4️⃣ Full Restore Architecture
    Before Disaster
    ----------------

        kube-apiserver
            |
            v
          etcd
            |
            v
        /var/lib/etcd


    After Restore
    --------------

        kube-apiserver
            |
            v
          etcd
            |
            v
        /var/lib/etcd
            ^
            |
        snapshot restore
            |
        /tmp/etcd-backup.db

⭐ Important Interview Points ⭐
    If asked "How does Kubernetes restore etcd?", answer like this:

    1️⃣ Take etcd snapshot backup
    2️⃣ Stop kube-apiserver
    3️⃣ Restore snapshot using etcdctl snapshot restore
    4️⃣ Replace etcd data directory
    5️⃣ Restart control plane components

✅ Common Production Practices
| Practice                 | Why                        |
| ------------------------ | -------------------------- |
| Scheduled etcd backups   | Prevent cluster state loss |
| Store backups externally | Node failure protection    |
| Version match etcdctl    | Avoid restore errors       |
| Test restore regularly   | Disaster readiness         |

✅ Most Important CKA Tip
    In CKA exam, restore usually requires:
        ETCDCTL_API=3 etcdctl snapshot restore snapshot.db
        Then update etcd.yaml data-dir.
    
    Many people forget this step.

✅ Simple way to remember
    Backup → Snapshot
    Restore → New data-dir
    Restart → Control plane

✅ Comparison: On-Prem vs EKS etcd backup
| Feature           | kubeadm / On-prem       | EKS         |
| ----------------- | ----------------------- | ----------- |
| etcd access       | Yes                     | No          |
| SSH control plane | Yes                     | No          |
| etcd snapshot     | `etcdctl snapshot save` | Not allowed |
| Backup method     | etcd snapshot           | Velero / S3 |
| Managed by        | Admin                   | AWS         |

✅ Simple memory trick for interviews
    On-Prem Kubernetes → Backup ETCD
    EKS / AKS / GKE → Backup Kubernetes resources