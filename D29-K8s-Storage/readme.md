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