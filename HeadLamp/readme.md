## HeadLamp
Headlamp is an easy-to-use and extensible Kubernetes web UI.
Headlamp was created to blend the traditional feature set of other web UIs/dashboards (i.e., to list and view resources) with added functionality.

Headlamp can be used in-cluster, where it's accessed through a web browser, or as a desktop application (using the information defined in the user's kubeconfig).

Step 1 : installation via helm
    https://headlamp.dev/docs/latest/installation/in-cluster/

    # first add our custom repo to your local helm repositories
    helm repo add headlamp https://kubernetes-sigs.github.io/headlamp/

    # now you should be able to install headlamp via helm
    helm install my-headlamp headlamp/headlamp --namespace kube-system

    Verify : kubectl get po -n kube-system

Step 2: we used kind cluster port forwarding is required 
    >kubectl port-frward -n kube-system service/my-headlamp 8080:80 --address 0.0.0.0

Step 3: Open browser : 
    http://localhost:8080/
    then Click on Generate token 
    follow this document (execute the commands) in new terminal : https://headlamp.dev/docs/latest/installation/#create-a-service-account-token

step 4 : copy the token and paste it on browser password


