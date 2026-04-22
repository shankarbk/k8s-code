# Test linux capability (linux-capability.yaml)
# kubectl apply -f .\linux-capability.yaml
Check for : NET_ADMIN capability (securityContext -->"add" )

    step 1: sh into the pod
        kubectl exec -it busybox-pod-secure-demo-pod-name -- sh
    (inside pod)
        step 2: ip link show - used to display information and the status of all network interfaces (links) on the system
        step 3: ip link add dummy0 type dummy - add dummy network interfaces
        step 4: check again , whether its added or not 
        > ip link show

Lets try to mount the file (securityContext -->"drop" ): Drops all other capabilites which results the Permission error
    step 1: sh into the pod
        kubectl exec -it busybox-pod-secure-demo-pod-name -- sh
    (inside pod)
        step 2: mount the file
            > mkdir /tmp/test - creates a directory "test"
            > mount -t tmpfs tmpfs /tmp/test - mount the directory
        Result :
            mount: permission denied (are you root?)

# Test Security context (security-context.yaml)
# kubectl apply -f .\security-context.yaml
    step 1: sh into the pod
            > kubectl exec -it secure-demo-pod-name -- sh
        (inside pod)
    step 2: check the process id running 
        > ps
            as defined in YAML : user is running on id 1000
    step 3: redirect to volume mounted (refer YAML file)
        > cd /data
    step 3 : check the process id of demo
        > ls -lrt : as defined in YAML, file ownership group (fsGroup) is running on 2000
    step 4: check all the id
        > id : uid=1000 gid=3000 groups=2000,3000
    
    step 4: denied permissions check
        > ping 127.0.0.1
            PING 127.0.0.1 (127.0.0.1): 56 data bytes
            ping: permission denied (are you root?)

# Security context : Test the pod as root user (root-user-test.yaml)
# kubectl apply -f .\root-user-test.yaml
    step 1: check the id of root user
        kubectl exec -it root-user-test -- id
        Result : uid=0(root) gid=0(root) groups=0(root),10(wheel)

Pod security standards
    1. Privileged
    2. Baseline
    3. Restricted

    Privileged --> Unrestricted policy, providing the widest possible level of permissions. This policy allows for known privilege escalations.
    Baseline --> Minimally restrictive policy which prevents known privilege escalations. Allows the default (minimally specified) Pod configuration.
    Restricted --> Heavily restricted policy, following current Pod hardening best practices.

# Create the Restricted Namespace and configure the "Restrictd" pod security standard
    step 1 : create Namespace
        > Kubectl create ns restricted-ns

    step 2 : apply pod security standard label ("Restricted") to this namespace
        > kubectl label namespace restricted-ns \
        pod-security.kubernetes.io/enforce=restricted \
        pod-security.kubernetes.io/audit=restricted \
        pod-security.kubernetes.io/warn=restricted

    step 3: check appliued labels
        > kubectl get ns --show-labels

    step 4: Create a pod, thet is compliant with "restricted" standard (restricted-compliant-pod.yaml)
        > kubectl apply -f restricted-compliant-pod.yaml

        check the created pod : kubectl get po -n restricted-ns
    
    step 5: Create a pod, thet is compliant with "restricted" standard (restricted-non-compliant-pod.yaml)
        > kubectl apply -f restricted-non-compliant-pod.yaml
        It shows the error :
            Error from server (Forbidden): error when creating ".\\restricted-non-compliant-pod.yaml": pods "security-violations" is forbidden: violates PodSecurity "restricted:latest": privileged (container "violation-container" must not set securityContext.privileged=true), allowPrivilegeEscalation != false (container "violation-container" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "violation-container" must set securityContext.capabilities.drop=["ALL"]; container "violation-container" must not include "NET_ADMIN", "SYS_ADMIN" in securityContext.capabilities.add), restricted volume types (volume "host-root" uses restricted volume type "hostPath"), runAsNonRoot != true (pod or container "violation-container" must set securityContext.runAsNonRoot=true), runAsUser=0 (container "violation-container" must not set runAsUser=0), seccompProfile (pod or container "violation-container" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
