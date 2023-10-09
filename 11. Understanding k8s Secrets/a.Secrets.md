
## Environment Variables

You can set environment variables for containers running in a pod. Additionally, Kubernetes automatically exposes certain runtime information via environment variables.

Let's launch a pod that we pass an environment variable SIMPLE_SERVICE_VERSION with the value 1.0:
 
    apiVersion: v1
    kind: Pod
    metadata:
    name: envs
    spec:
    containers:
    - name: sise
        image: quay.io/openshiftlabs/simpleservice:0.5.0
        ports:
        - containerPort: 9876
        env:
        - name: SIMPLE_SERVICE_VERSION
          value: "1.0"
 
Now, let's verify from within the cluster if the application running in the pod has picked up the environment variable:
 
    kubectl exec envs -t -- curl -s 127.0.0.1:9876/info

The output reflects the value that was set for the environment variable (the default, unless overridden by the variable, is 0.5.0):

    {"host": "127.0.0.1:9876", "version": "1.0", "from": "127.0.0.1"}

## Kubernetes secrets

Secrets can be defined as Kubernetes objects used to store sensitive data such as user name and passwords with encryption.

There are multiple ways of creating secrets in Kubernetes.

- Creating from text files.
- Creating from yaml file.

## Creating From Text File

In order to create secrets from a text file such as user name and password, we first need to store them in a txt file and use the following command.

    $ kubectl create secret generic tomcat-passwd –-from-file = ./username.txt –fromfile = ./.password.txt

## Creating From Yaml File

    apiVersion: v1
    kind: Secret
    metadata:
    name: tomcat-pass
    type: Opaque
    data:
    password: abcd001
    username: redhat

## Creating the Secret

    $ kubectl create –f Secret.yaml
    secrets/tomcat-pass

## Using Secrets

Once we have created the secrets, it can be consumed in a pod or the replication controller as −

- Environment Variable
- Volume

## As Environment Variable

In order to use the secret as environment variable, we will use env under the spec section of pod yaml file.

    env:
    - name: SECRET_USERNAME
    valueFrom:
        secretKeyRef:
            name: mysecret
            key: tomcat-pass

As Volume

    spec:
    volumes:
        - name: "secretstest"
            secret:
                secretName: tomcat-pass
    containers:
        - image: tomcat:7.0
            name: awebserver
            volumeMounts:
                - mountPath: "/tmp/mysec"
                name: "secretstest"

Secret Configuration As Environment Variable

    apiVersion: v1
    kind: ReplicationController
    metadata:
    name: appname
    spec:
    replicas: 2
    template:
    metadata:
        name: appname
    spec:
        nodeSelector:
            resource-group:
        containers:
            - name: appname
                image:
                imagePullPolicy: Always
                ports:
                - containerPort: 3000
                env: 
                - name: ENV
                    valueFrom:
                      configMapKeyRef:
                        name: appname
                        key: tomcat-secrets

In the above code, under the env definition, we are using secrets as environment variable in the replication controller.

Secrets As Volume Mount

    apiVersion: v1
    kind: pod
    metadata:
    name: appname
    spec:
    metadata:
        name: appname
    spec:
    volumes:
        - name: "secretstest"
            secret:
                secretName: tomcat-pass
    containers:
        - image: tomcat: 8.0
            name: awebserver
            volumeMounts:
                - mountPath: "/tmp/mysec"
                name: "secretstest"

## Secrets

You don't want sensitive information such as a database password or an API key stored in clear text. Secrets provide you with a mechanism to store such information in a safe and reliable way with the following properties:

- Secrets are namespaced objects, that is, exist in the context of a specific namespace
- You can access them via a volume or an environment variable from a container running in a pod
- The secret data on nodes is stored in tmpfs volumes
- A per-secret size limit of 1MB exists
- The API server stores secrets as plaintext in etcd

Let's create a secret named apikey that holds an example API key. The first step is to create a file that contains the secret data:

    echo -n "A19fh68B001j" > ./apikey.txt

That file is passed to the command that creates the secret:

    kubectl create secret generic apikey --from-file=./apikey.txt

Information about the secret is retrieved using the describe subcommand:

    kubectl describe secrets/apikey

Inject in pod 

    apiVersion: v1
    kind: Pod
    metadata:
    name: consumesec
    spec:
    containers:
    - name: shell
        image: centos:7
        command:
        - "bin/bash"
        - "-c"
        - "sleep 10000"
        volumeMounts:
        - name: apikeyvol
            mountPath: "/tmp/apikey"
            readOnly: true
    volumes:
    - name: apikeyvol
        secret:
        secretName: apikey