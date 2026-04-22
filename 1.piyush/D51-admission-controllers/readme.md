# Check enabled admission controller plugin 
    1. Check api server pod
    > kubectl get po -n kube-system

    2.  get the podname and get yaml format of that pod
        > kubectl get po -n kube-system kube-controller-manager-cka-control-plane -o yaml
            check for "--enable-admission-plugins=NodeRestriction"  
            default "NodeRestriction" plugin is enabled 
            (reference : https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)

try to create an image in a namespace which is not exists (Ex: test namespace)
    > kubectl run nginx-pod --image=nginx -n test
    it failed becaue "NamespaceAutoProvision" is not enabled. (Error from server (NotFound): namespaces "test" not found)

    now enable the "NamespaceAutoProvision" in example1

## steps to change in kind cluster (my laptop)
1. docker ps
2. docker exec -it <container-id>> bash ---> cka-control-plane
3. cd /etc/kubernetes/manifests
4. optional (if vi editor is missing) - apt-get update && apt-get install -y vim
5. ls -lrt
6. vi kube-apiserver.yaml
7. add this content - ValidatingAdmissionPolicy and NamespaceAutoProvision (refer the image : k8s-admission-controllers-changes.png)
8. Restart the server
    mv kube-apiserver.yaml /tmp
    mv /tmp/kube-apiserver.yaml .


## Ex:
# 1.  Scenario in a pipeline create a namespace on the fly if its not exists
    we need to enable "NamespaceAutoProvision" plugin 
    (reference : https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)

    edit yor conatrolplane pod's yaml availabe in "kube-system" namespace.
    (control-plane is part of your kube-system)
    change this line 
        from :
            - --enable-admission-plugins=NodeRestriction
        to :
            - --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision

        restart the server (simply move the "kube-apiserver.yaml" from other directory, and then move it back to its original place)

    now try :
        > kubectl run nginx-pod --image=nginx -n test 
            It will work

# Instead of building a webhook server (certs, HTTPS, pain 😈), we’ll use:
# ValidatingAdmissionPolicy (native Kubernetes feature) - Much cleaner for learning.

# 1️⃣ Validating Admission Controller Example
Reject any Pod that does NOT define resource limits.(Very realistic production rule.)
Reference : https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#validatingadmissionpolicy


    Step 1 — Create Admission Policy(policy-1.yaml)
        > kubectl apply -f policy-1.yaml
        👉 What this means:
            ✔ Trigger on Pod creation
            ✔ Check every container
            ✔ Ensure .resources.limits exists

    Step 2 — Bind the Policy ( policy-bind-1.yaml - Policy alone does nothing until bound.)

        ✔ Action = Deny
       
        > kubectl apply -f policy-bind-1.yaml

    Step 3 — Try Creating BAD Pod (bad-pod.yaml)
        > kubectl apply -f bad-pod-1.yaml

        Result : Admission controller blocked it.

    Step 4 — Try GOOD Pod
        > kubectl apply -f good-pod-1.yaml

        Result :  ✔ Pod created successfully
    
    🧠 What Just Happened
        Flow:
            kubectl apply
                ↓
            API Server
                ↓
            Admission Policy evaluates
                ↓
            Allowed / Denied

        No controller
        No reconciliation loop
        Pure request-time enforcement

        This is EXACTLY how real clusters enforce:
            ✔ Resource rules
            ✔ Security constraints
            ✔ Label requirements
            ✔ Naming conventions

        Always scope carefully.

            Example safer match:
            ```
                namespaceSelector:
                    matchLabels:
                        env: prod
            ```

    ✅ What This Demonstrates

        ✔ Cannot modify
        ✔ Only Allow / Deny

        👉 Pure validation.

# 2️⃣ Mutating Admission Controller Example
# Automatically add resource limits if missing
# Classic platform engineering pattern.
    Step 1 — Create Admission Policy(policy-2.yaml)

    Step 2 — Bind the Policy ( policy-bind-2.yaml - Policy alone does nothing until bound.)

    step 3 - pod-2.yaml

        Input Pod (User)

        ```
        containers:
            - name: app
            image: nginx
        ```

        After Mutation (Stored)

        ```
        containers:
        - name: app
        image: nginx
        resources:
            limits:
            cpu: "500m"
            memory: "256Mi"
        ```

        ✔ User didn’t specify
        ✔ Cluster injected defaults

        ✅ What This Demonstrates
            ✔ Object modified
            ✔ Request still allowed

            👉 Mutation.

Conclusion:
    ✔ Mutating → Add defaults
    ✔ Validating → Enforce correctness

    User Pod → Mutate → Validate → Store

    Mutation first. Validation second.

    Simple Memory Hook(To Rememver):
        Mutating = “Let me fix it”
        Validating = “This is illegal”

# 3️⃣ Working with Built-in Admission Controllers
Exercise : ResourceQuota (Validating Controller)
    ResourceQuota prevents resource overconsumption - like having a spending limit on your credit card.

    Step 1: Create a namespace with a resource quota (kubectl apply -f resource-quota-demo.yaml)

    Step 2: Apply the configuration (kubectl apply -f resource-quota-demo.yaml)

    Step 3: Try to create a pod that exceeds the quota (big-pod.yaml)

    Step 4: Apply and observe the error (kubectl apply -f big-pod.yaml)
    Error:
        Error from server (Forbidden): error when creating "big-pod.yaml": pods "big-pod" is forbidden: failed quota: compute-quota: must specify limits.cpu for: big-container; limits.memory for: big-container

Exercise 2: LimitRanger (Mutating Controller)
    LimitRanger automatically adds resource limits - like a helpful assistant that fills out forms for you.

    Step 1: Create a LimitRange (limit-range-demo.yaml)

    Step 2: Apply the LimitRange (kubectl apply -f limit-range-demo.yaml)

    Step 3: Create a pod without specifying resources (simple-pod.yaml)

    Step 4: Apply and check the pod (kubectl apply -f simple-pod.yaml)

        kubectl get pod simple-pod -n quota-demo -o yaml | grep -A 10 resources


