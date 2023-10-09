## Single Container Pod

As the name suggests, if there is only a single container running in the Kubernetes pod, then it is called a single container pod and it can easily be created with the help of a YAML file. Here is a sample YAML file used to create a pod with the postgres database. 

    apiVersion: v1 
    kind: Pod 
    metadata: 
      name: Postgres 
    spec: 
      containers: 
      - name: Postgres 
        image: Postgres: 3.1 
        ports: 
        - containerPort: 8000 
        imagePullPolicy: Always 

## Multi Container Pod

In this type of pod, we can run multiple containers within a single pod. It can be created by specifying the containers in the same YAML file. Here is a sample YAML file used to create a multi container Pod with Tomcat and MongoDB images.

    apiVersion: v1 
    kind: Pod 
    metadata: 
    name: Tomcat 
    spec: 
      containers: 
      - name: Tomcat 
        image: tomcat: 8.0 
        ports: 
        - containerPort: 7500 
        imagePullPolicy: Always 
      - name: Database 
        image: mongoDB 
        ports: 
        - containerPort: 7501 
        imagePullPolicy: Always 

## Creating Pod example 

    root@master1:~/pods# cat pod.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: mysql
    spec:
      containers:
      - name: sqlimage
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
    pod/mysql created

## listing pods status 

    root@master1:~/pods# kubectl get pods
    NAME   READY   STATUS    RESTARTS   AGE
    mysql   1/1     Running   0          4s

## let login pod inside.

    root@master1:~/pods# kubectl exec -it mysql -- bash
    root@mysql:/# mysql -u root -predhat
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
    pod "mysql" deleted







    
