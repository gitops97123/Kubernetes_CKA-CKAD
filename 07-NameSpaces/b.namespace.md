## creating new namespace. 

Creating new namespace named black. 

    root@master1:~# kubectl create ns black
    namespace/black created

checkout the describe namespace. 

    root@master1:~# kubectl describe ns black
    Name:         black
    Labels:       kubernetes.io/metadata.name=black
    Annotations:  <none>
    Status:       Active
    No resource quota.
    No LimitRange resource.

checkout namespace. 

    root@master1:~# kubectl get ns black
    NAME    STATUS   AGE
    black   Active   15s

creating namespace yaml if you dont know about the command enough. 

    root@master1:~# kubectl create ns black --dry-run=client -o yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      creationTimestamp: null
      name: black
    spec: {}
    status: {}

checkout the namespace yaml. 

    root@master1:~# kubectl create ns black --dry-run=client -o yaml > name.yaml
    root@master1:~# ls name.yaml
    name.yaml
    
now applying name.yaml 
    
    root@master1:~# kubectl apply -f name.yaml
    black namespace created.

get namespace. 

    root@master1:~# kubectl get ns black
    NAME    STATUS   AGE
    black   Active   2m38s
