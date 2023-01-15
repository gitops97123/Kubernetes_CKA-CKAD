

## Creating Pod example #1 

    root@master1:~/pods# cat pod.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: envs
    spec:
      containers:
      - name: sise
        #    image: quay.io/openshiftlabs/simpleservice:0.5.0
        image: mysql:5.6
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "redhat"
        - name: MYSQL_USER_PASSWORD
          value: "redhat"
        - name: MYSQL_USER
          value: "redhat"
        - name: MYSQL_DATABASES
          value: "redhat"

## Applying pod 
    root@master1:~/pods# kubectl apply -f pod.yaml
    pod/envs created

## listing pods status 

    root@master1:~/pods# kubectl get pods
    NAME   READY   STATUS    RESTARTS   AGE
    envs   1/1     Running   0          4s

## let login pod inside.

    root@master1:~/pods# kubectl exec -it envs -- bash
    root@envs:/# mysql -u root -predhat
    ...
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    +--------------------+
    3 rows in set (0.00 sec)
    mysql> exit
    Bye

## deleting the pod.

    root@master1:~/pods# kubectl delete -f pod.yaml
    pod "envs" deleted







    
