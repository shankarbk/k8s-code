## Webhook Authorization

Requires:
    ✔ External authorization server
    ✔ API server configuration

KIND Config Example(YAML):
    authorization-mode: Webhook
    authorization-webhook-config-file: /etc/kubernetes/auth-webhook.yaml

Webhook config(YAML):
    apiVersion: v1
    kind: Config
    clusters:
    - name: auth-server
    cluster:
        server: http://host.docker.internal:8080/authorize

Requires you to build an HTTP server returning(JSON):
    {
    "apiVersion": "authorization.k8s.io/v1",
    "status": {
        "allowed": true
        }
    }

✔ Used in enterprise / custom IAM
✔ Overkill for learning unless security-focused