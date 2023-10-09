# Resource Quota 

A resource quota, defined by a ResourceQuota object, provides constraints that limit aggregate resource consumption per namespace. It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be consumed by resources in that namespace.

Resource quotas work like this:

    1. Different teams work in different namespaces. This can be enforced with RBAC.

    1. The administrator creates one ResourceQuota for each namespace.

    1. Users create resources (pods, services, etc.) in the namespace, and the quota system tracks usage to ensure it does not exceed hard resource limits defined in a ResourceQuota.

    1. If creating or updating a resource violates a quota constraint, the request will fail with HTTP status code 403 FORBIDDEN with a message explaining the constraint that would have been violated.

    1. If quota is enabled in a namespace for compute resources like cpu and memory, users must specify requests or limits for those values; otherwise, the quota system may reject pod creation. Hint: Use the LimitRanger admission controller to force defaults for pods that make no compute resource requirements.
    

## Create a namespace

Create a namespace so that the resources you create in this exercise are isolated from the rest of your cluster.

    $ kubectl create namespace quota-example

## Create a ResourceQuota

Here is a manifest for an example ResourceQuota:
     
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: mem-cpu-demo
    spec:
      hard:
        requests.cpu: "1"
        requests.memory: 1Gi
        limits.cpu: "2"
        limits.memory: 2Gi

Create the ResourceQuota:

    $ kubectl apply -f https://k8s.io/examples/admin/resource/quota-mem-cpu.yaml --namespace=quota-example

View detailed information about the ResourceQuota:

    $ kubectl get resourcequota mem-cpu-demo --namespace=quota-example --output=yaml

The ResourceQuota places these requirements on the quota-mem-cpu-example namespace:

    1. For every Pod in the namespace, each container must have a memory request, memory limit, cpu request, and cpu limit.
    1. The memory request total for all Pods in that namespace must not exceed 1 GiB.
    1. The memory limit total for all Pods in that namespace must not exceed 2 GiB.
    1. The CPU request total for all Pods in that namespace must not exceed 1 cpu.
    1. The CPU limit total for all Pods in that namespace must not exceed 2 cpu.

## Create a Pod

Here is a manifest for an example Pod:

    apiVersion: v1
    kind: Pod
    metadata:
      name: quota-mem-cpu-demo
    spec:
      containers:
      - name: quota-mem-cpu-demo-ctr
        image: nginx
        resources:
          limits:
            memory: "800Mi"
            cpu: "800m"
          requests:
            memory: "600Mi"
            cpu: "400m"

Create the Pod:

    $ kubectl apply -f quota-pod.yaml --namespace=quota-example

Verify that the Pod is running and that its (only) container is healthy:

    $ kubectl get pod quota-mem-cpu-demo --namespace=quota-example

Once again, view detailed information about the ResourceQuota:

    $ kubectl get resourcequota mem-cpu-demo --namespace=quota-example --output=yaml

The output shows the quota along with how much of the quota has been used. You can see that the memory and CPU requests and limits for your Pod do not exceed the quota.

    status:
      hard:
        limits.cpu: "2"
        limits.memory: 2Gi
        requests.cpu: "1"
        requests.memory: 1Gi
      used:
        limits.cpu: 800m
        limits.memory: 800Mi
        requests.cpu: 400m
        requests.memory: 600Mi

If you have the jq tool, you can also query (using JSONPath) for just the used values, and pretty-print that that of the output. For example:

    $ kubectl get resourcequota mem-cpu-demo --namespace=quota-example -o jsonpath='{ .status.used }' | jq .
    
# Attempt to create a second Pod
Here is a manifest for a second Pod:

    apiVersion: v1
    kind: Pod
    metadata:
      name: quota-mem-cpu-demo-2
    spec:
      containers:
      - name: quota-mem-cpu-demo-2-ctr
        image: redis
        resources:
          limits:
            memory: "1Gi"
            cpu: "800m"
          requests:
            memory: "700Mi"
            cpu: "400m"
    
In the manifest, you can see that the Pod has a memory request of 700 MiB. Notice that the sum of the used memory request and this new memory request exceeds the memory request quota: 600 MiB + 700 MiB > 1 GiB.

Attempt to create the Pod:

    $ kubectl apply -f quota-mem-cpu-pod-2.yaml --namespace=quota-example

The second Pod does not get created. The output shows that creating the second Pod would cause the memory request total to exceed the memory request quota.

Error from server (Forbidden): error when creating "examples/admin/resource/quota-mem-cpu-pod-2.yaml":
pods "quota-mem-cpu-demo-2" is forbidden: exceeded quota: mem-cpu-demo,
requested: requests.memory=700Mi,used: requests.memory=600Mi, limited: requests.memory=1Gi

