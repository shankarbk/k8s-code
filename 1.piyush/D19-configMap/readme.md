## SECTION 1
## Configmap (picked from copilot and gpt)

Its primary goal is to allows you to store configuration data separately from your application container images. making applications more portable and easier to manage across different environments instead of hard-coding values inside your image (bad practice), you externalize them so you can change config without rebuilding the container.

A ConfigMap is a Kubernetes API object used to store non-confidential configuration data in key-value pairs.

🧠 Why ConfigMaps exist ?
    Containers should be portable & reusable.

    If you hard-coded values inside your image:
        Changing config → rebuild image → redeploy → slow & messy

    With ConfigMap:
        Change config → update ConfigMap → restart pod (sometimes not even needed)

📦 Key Features
    ✔ Decouples config from code: Keeps container images clean and reusable.
    ✔ Flexible consumption: Pods can use ConfigMaps as:
        - Environment variables
        - Command-line arguments
        - Configuration files mounted in volumes
    ✔ Non-sensitive only: ConfigMaps are not encrypted or secure. Use Secrets for sensitive data.
    ✔ Size limit: ConfigMaps have a 1 MB limit. Larger configs should be stored in volumes.


🛠️ How to Create a ConfigMap
    You can create ConfigMaps either imperatively with kubectl or declaratively with YAML.
    
    ✔ Imperative (from literals):
        kubectl create configmap demo-config \
        --from-literal=database_host=172.138.0.1 \
        --from-literal=debug_mode=1 \
        --from-literal=log_level=verbose

    ✔ Imperative (from file)
        kubectl create configmap demo-config --from-file=app.properties

    ✔ Declarative (YAML manifest)

📦 Simple Example
    Create ConfigMap: db-config.yaml

        kubectl apply -f db-config.yaml

        ⚡ Example Usage
            Suppose your app needs a database host:
            - Local dev: DATABASE_HOST=localhost
            - Cloud: DATABASE_HOST=my-service
            Instead of rebuilding the container image for each environment, you store these values in a ConfigMap and mount them into the Pod.

🔌 How Pods Use ConfigMaps
    Three common ways:
        1️⃣ As Environment Variables (Most common)   
        2️⃣ Inject Entire ConfigMap   
        3️⃣ As Files (Very powerful)

    Switching Environments
        - For local dev, set key: DEV_DATABASE_HOST.
        - For cloud, set key: PROD_DATABASE_HOST.
        you’ll need to change the ConfigMap reference when deploying the Pod (or Deployment). But here’s the important nuance:
        You don’t touch the application code at all. The only thing you swap is the Kubernetes manifest that points to the right ConfigMap for the environment.

        🛠️ Best Practices
            - Separate manifests per environment: Keep deployment-dev.yaml, deployment-prod.yaml, etc., each pointing to the right ConfigMap.
                                                  OR  maintain separate configuration manifests(manifest-dev.yaml OR manifest-prod.yaml).

            - 👉 Here HELM came into picture (Refer : SECTION 2)
                Helm/Kustomize : Use templating tools to avoid duplicating manifests. For example, Helm lets you pass --set env=dev and automatically inject the correct  ConfigMap reference.

                A Helm values.yaml example where you can switch environments just by passing a flag, instead of editing YAML manually according to environments
            
            - CI/CD pipelines: Automate the selection of the correct manifest/config based on the target environment.

    1️⃣ As Environment Variables (Most common)
        dev-configmap.yaml
        prod-configmap.yaml

        dev-pod-as-env-var.yaml
        prod-pod-as-env-var.yaml

        To verify : whether the config map values are availabe inside the container as Environment variable
            kubectl logs dev-pod
            kubectl logs prod-pod

        This Approach is:
            ✔ Clean
            ✔ Simple
            ✔ Very common in exams & real projects
        
    2️⃣ Inject Entire ConfigMap
        Instead of mapping keys one by one

        dev-configmap.yaml
        dev-pod-inject-all-keys.yaml

        ✔ All keys become env vars

    3️⃣ As Files (Very powerful)
        Mount ConfigMap like a config file:

        dev-configmap.yaml
        dev-pod-as-file.yaml

        Inside container:
            /etc/config/APP_MODE
            /etc/config/LOG_LEVEL

        ✔ Perfect for apps needing config files
        ✔ Common for nginx, prometheus, etc.

🔄 What happens when ConfigMap changes?

| Usage Type    | Update Behavior                       |
| ------------- | ------------------------------------- |
| Env Vars      | Pod restart required                  |
| Mounted Files | Auto-updated (usually within seconds) |


🚨 Risks & Considerations
    - Not secure: Never store passwords or tokens in ConfigMaps.
    - Environment-specific configs: Ensure proper separation between dev, staging, and production.
    - Size constraints: Watch out for the 1 MB limit.

🎯 When to Use ConfigMap

    Use when config is:
        ✔ Environment-specific
        ✔ Frequently changing
        ✔ Non-sensitive


## SECTION 2 :
## complete Helm example that lets you switch environments (dev vs. cloud/prod) just by passing a flag, without editing YAML manually

Step 1: Helm Chart Structure
    my-app/
        ├── Chart.yaml
        ├── values.yaml
        ├── templates/
        │   ├── configmap.yaml
        │   └── pod.yaml

Step 2 : Create values.yaml

Step 3: Create templates/configmap.yaml

Step 4: Create templates/pod.yaml

Step 6: Deploying with Helm
    ✔ Dev environment (default):
        helm install my-app ./my-app --set env=dev

    ✔ Cloud environment
        helm install my-app ./my-app --set env=cloud

    Helm will automatically pick the right values from values.yaml based on the env flag.

🔑 Key Takeaway
- One chart, one codebase.
- Switch environments by passing --set env=dev or --set env=cloud.
- No manual YAML edits needed—Helm templates handle the substitution.
