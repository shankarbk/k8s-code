✅ CRD (Custom Resource Definition)

    A CRD is how you teach Kubernetes a new type of object.

    By default Kubernetes understands things like:
        - Pod
        - Deployment
        - Service

    But what if you want Kubernetes to understand:
        - Database
        - Cache
        - Backup
        - FirewallRule

    👉 You create a CRD.

    Think of CRD as:
        📘 A blueprint / schema / class definition

    It defines:
        - Object name
        - API group
        - Fields (spec structure)
        - Validation rules

    Example Idea:
        Without CRD:
            ❌ Kubernetes doesn’t know what a Database is.

        With CRD:
            ✅ Kubernetes now accepts kind: Database.

✅ CR (Custom Resource)

    A CR is the actual object created using a CRD.

    Think of CR as:
        📄 An instance / actual object

    Like:
        CRD → class
        CR → object

Code reference : https://github.com/piyushsachdeva/CKA-2024/blob/main/Resources/Day49-CustomResources/README.md

Verify the Created Resources:
    # Check the custom resource instance
        kubectl get Foo

    # Check deployments created through the custom resource
        kubectl get deployments

get more details about the created deployment:
    Describe the deployment created by the controller
        kubectl describe deployment example-foo-deployment

Verify the CRD: 
    Check the fields of CRD
        kubectl explain Foo.spec.deploymentName
        kubectl explain Foo.spec.replicas

✅ What Happens Behind the Scenes?

    Important truth:

        ⚠️ CRD alone does nothing useful

            It just registers a new object type.
            To make it functional → you need a:

    ✅ Controller / Operator

        The controller:
            Watches CRs
            Takes action

✅ Why Kubernetes Designed This?

    Because Kubernetes is extensible.
    Vendors & tools add their own objects:

    Examples you’ve probably seen:

        Certificate (cert-manager)
        VirtualService (Istio)
        Prometheus (monitoring)

    Those are CRDs + CRs.

What Happens When You Create a CRD?
    CRD itself does NOT use any controller automatically.
    When you apply a CRD:
        kubectl apply -f crd.yaml

    Kubernetes API Server:
        ✅ Registers a new resource type
        ✅ Adds new REST endpoints
        ✅ Stores objects in etcd

    That’s it.
        ⚠️ NO business logic yet
        Kubernetes does NOT know what to DO with your CR.
        Because Kubernetes has no idea what your custom object means.
        CRD = Only schema + API extension (NOT behavior.)