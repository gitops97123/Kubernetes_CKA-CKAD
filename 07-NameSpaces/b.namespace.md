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

