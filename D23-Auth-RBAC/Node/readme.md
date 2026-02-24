## Node Authorization (Already active in KIND)

Good news:
    ✔ KIND enables Node Authorizer by default

You don’t manually test this easily because it controls:
    ✔ kubelet → API server communication


What it enforces

    A node can:
        ✔ Read its own pods
        ✔ Read related secrets/configmaps

    ❌ Cannot read everything

How to observe indirectly:
    kubectl get clusterrole system:node -o yaml
    
    You’ll see node permissions.
    🧠 This is internal security plumbing