## Creating new labels for the pod. 
Creating new pods.

    root@master1:~# kubectl run pod1 --image=nginx
    pod/pod1 created

checkout the pod status. 

    root@master1:~# kubectl get pods
    NAME   READY   STATUS              RESTARTS   AGE
    pod1   0/1     ContainerCreating   0          4s

checkout the label.

    root@master1:~# kubectl get pods --show-labels
    NAME   READY   STATUS    RESTARTS   AGE   LABELS
    pod1   1/1     Running   0          10s   run=pod1

creating new label for pod1. 

    root@master1:~# kubectl label pod pod1 app=developer
    pod/pod1 labeled

checkout the labels.

    root@master1:~# kubectl get pods --show-labels
    NAME   READY   STATUS    RESTARTS   AGE   LABELS
    pod1   1/1     Running   0          31s   app=developer,run=pod1

remove the previous label. 

    root@master1:~# kubectl label  pod pod1 run-

listing labels.

    root@master1:~# kubectl get pods --show-labels
    NAME   READY   STATUS    RESTARTS   AGE   LABELS
    pod1   1/1     Running   0          99s   app=developer

overwrite the pod label. 

    root@master1:~# kubectl label --overwrite pod pod1 app=database
    pod/pod1 labeled


listing labels.

    root@master1:~# kubectl get pods --show-labels
    NAME   READY   STATUS    RESTARTS   AGE    LABELS
    pod1   1/1     Running   0          2m7s   app=database



