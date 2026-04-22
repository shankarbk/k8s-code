## Using kind cluster locally, each node is a conatiner.
# SSH into the worker node worker1 and worker_node2 (because of 2 nodes schedulae may create the pods on any available node) and execute the below commands
    docker  exec -it <container-id> bash

    mkdir -p /mnt/data/mongodb-A
    mkdir -p /mnt/data/mongodb-B
    mkdir -p /mnt/data/mongodb-C
    mkdir -p /mnt/data/mongodb-D
    mkdir -p /mnt/data/mongodb-E

#  Create a Headless Service (mongodb-service.yaml)
    kubectl apply -f mongodb-service.yaml

# Create StorageClass (mongodb-storageclass.yaml)
    kubectl apply -f mongodb-storageclass.yaml

#  Create PersistentVolumes (mongodb-pv.yaml)
    kubectl apply -f mongodb-pv.yaml

# Create the StatefulSet (mongodb-statefulset.yaml)
    kubectl apply -f mongodb-statefulset.yaml

# Verify PersistentVolumeClaims
    Check that PVCs are created and bound: kubectl get pvc

# Test Data Persistence
Connect to the first MongoDB instance:
    kubectl exec -it mongodb-0 -- mongo

Create some test data:
    use testdb
    db.users.insert({name: "user1", email: "user1@example.com"})
    db.users.insert({name: "user2", email: "user2@example.com"})
    db.users.find()
    exit

Delete the Pod to test persistence:
    kubectl delete pod mongodb-0

Wait for the Pod to be recreated:
    kubectl get pods -w

Connect to the recreated Pod and verify data persistence:
    kubectl exec -it mongodb-0 -- mongo
    use testdb
    db.users.find()
    exit

You should see that your data is still there.

# Scale the StatefulSet
    Scale to 5 replicas:
        kubectl scale statefulset mongodb --replicas=5

    Verify the new Pods are created in order:
        kubectl get pods -w

# Clean Up
    When you're done with the exercise, clean up all resources:

        kubectl delete statefulset mongodb
        kubectl delete svc mongodb-service
        kubectl delete pvc --all
        kubectl delete pv --all
        kubectl delete sc mongodb-sc