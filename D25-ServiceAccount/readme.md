- To create service account :
    kbectl create sa jenkins-sa

    To get service accounts:
         kubectl get sa

    To describe service account:
        kubectl describe sa <sa-name>
    

- To create role for above ServiceAccount:
     kubectl create role jenkins-role --verb=list,get,watch --resource=pod

    To Describe secret:
         kubectl describe secret jenkins-sa-secret

- Create Rolebinding for above Role 
     kubectl create rolebinding jenkins-bind --role=jenkins-role --user=jenkins-sa

Task: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account
Step1 :- This command is used to create a Docker registry authentication secret in Kubernetes so that pods can pull images from a private registry.
     kubectl create secret docker-registry my-docker-registry-key --docker-server=<bits> --docker-username=<bits> --docker-password="<password>" --docker-email=<bits@gmail.com>

step2: Attach the Secret to your ServiceAccount:
     You must link the secret to ServiceAccount. This allows any Pod using this ServiceAccount to automatically inherit the credentials. 

     kubectl patch serviceaccount docker-sa -p '{"imagePullSecrets": [{"name": "my-docker-registry-key"}]}' 
                                             OR 
     refer: docker-sa.yaml


step3: Deploy the Pod
     In your Deployment or Pod manifest, specify your ServiceAccount. Kubernetes will now use the linked secret to pull the private image. 
     refer : pod.yaml
