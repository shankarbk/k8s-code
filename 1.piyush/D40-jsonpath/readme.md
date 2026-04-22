## JSONPath
JSONPath in Kubernetes is a query language used with kubectl to extract specific fields from Kubernetes API objects in JSON format.

JSONPath in Kubernetes is used to extract specific fields from Kubernetes API objects (like Pod, Service, Node) when using kubectl.
It helps you filter and format output instead of printing the entire YAML/JSON.   

Most commonly used with:
    kubectl get <resource> -o jsonpath='{expression}'

Flow diagram: 

      User
        │
        │  request (> kubectl get po)
        ▼
      kubectl
        │
        │  REST API request
        ▼
-----------------------------
|        API Server         |
-----------------------------
        │
        │  JSON Payload
        ▼
      kubectl
        │
        │  Human Readable Output
        ▼
      User

The user interacts with the Kubernetes cluster using kubectl, which sends requests to the API Server and receives responses in JSON format.
kubectl then processes the JSON response and converts it into a human-readable output before displaying it to the user.

✅ Why JSONPath is used in Kubernetes

    By default:
        kubectl get pod nginx -o yaml
        Shows the entire YAML, which is large.

    If you only need the Pod IP, JSONPath helps:
        kubectl get pod nginx -o jsonpath='{.status.podIP}'

    Output:
        10.244.1.5

    So JSONPath = Query language for JSON objects.

✅ Basic JSON structure of a Kubernetes object
Example Pod JSON (simplified):
```
{
 "metadata": {
   "name": "nginx-pod",
   "namespace": "default"
 },
 "status": {
   "podIP": "10.244.1.5"
 },
 "spec": {
   "nodeName": "worker-node1"
 }
}
```
To access values:
    | Field     | JSONPath                |
    | --------- | ----------------------- |
    | Pod name  | `{.metadata.name}`      |
    | Namespace | `{.metadata.namespace}` |
    | Pod IP    | `{.status.podIP}`       |
    | Node name | `{.spec.nodeName}`      |

✅ Common JSONPath Examples (Interview Important)

1️⃣ Get Pod Name
    kubectl get pod nginx -o jsonpath='{.metadata.name}'

    Output : nginx

2️⃣ Get Pod IP
    kubectl get pod nginx -o jsonpath='{.status.podIP}'

3️⃣ Get Node where Pod is running
    kubectl get pod nginx -o jsonpath='{.spec.nodeName}'

4️⃣ Get all Pod names in namespace
    kubectl get pods -o jsonpath='{.items[*].metadata.name}'
    
    Output : nginx redis mysql

    Explanation:
        items[*] → all objects
        metadata.name → pod name

5️⃣ Get Pod names with IP
    kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name} {.status.podIP}{"\n"}{end}'

    Output:
        nginx   10.244.1.3
        redis   10.244.1.4
        mysql   10.244.1.5

    Important keywords::
        | Keyword  | Meaning           |
        | -------- | ----------------- |
        | `range`  | loop through list |
        | `end`    | end loop          |
        | `{"\n"}` | new line          |

6️⃣ Get container image from pod
    kubectl get pod nginx -o jsonpath='{.spec.containers[*].image}'

    Output : nginx:1.25

7️⃣ Filter specific container
    kubectl get pod nginx -o jsonpath='{.spec.containers[0].image}'

    Here:
        [0] → first container

8️⃣ Get all node internal IPs
    kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

    This is a filter condition.
        Meaning: @.type == InternalIP

✅ JSONPath syntax cheat sheet
| Syntax        | Meaning          |
| ------------- | ---------------- |
| `{.field}`    | access field     |
| `{.items[*]}` | all items        |
| `{.items[0]}` | first item       |
| `{range}`     | loop             |
| `{end}`       | end loop         |
| `?()`         | filter condition |

✅Real DevOps / Interview Example
    Get all pods with node name
        kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name} {.spec.nodeName}{"\n"}{end}'

    Output
        nginx    worker1
        redis    worker2
        mysql    worker1

⭐ Pro DevOps Tip
    When learning JSONPath, always run:
    > kubectl get pod nginx -o json --> returns the entire object (here its 'pod') in json format.

    Then identify the field and write the path.