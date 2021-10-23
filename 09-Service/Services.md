# Services

A service is an abstraction for pods, providing a stable, so called virtual IP (VIP) address. While pods may come and go and with it their IP addresses, a service allows clients to reliably connect to the containers running in the pod using the VIP. The "virtual" in VIP means it is not an actual IP address connected to a network interface, but its purpose is purely to forward traffic to one or more pods. Keeping the mapping between the VIP and the pods up-to-date is the job of kube-proxy, a process that runs on every node, which queries the API server to learn about new services in the cluster.


A service can be defined as a logical set of pods. It can be defined as an abstraction on the top of the pod which provides a single IP address and DNS name by which pods can be accessed. With Service, it is very easy to manage load balancing configuration. It helps pods to scale very easily.

A service is a REST object in Kubernetes whose definition can be posted to Kubernetes apiServer on the Kubernetes master to create a new instance.


## Service without Selector

    apiVersion: v1
    kind: Service
    metadata:
      name: black_magic_linux
    spec:
      ports:
      - port: 8080
      targetPort: 31999

The above configuration will create a service with the name black_magic_linux.


## Service Config File with Selector

    apiVersion: v1
    kind: Service
    metadata:
      name: black_magic_linux
    spec:
      selector:
        application: "My Application" 
      ports:
      - port: 8080
        targetPort: 31999

# Types of Services

**ClusterIP** − This helps in restricting the service within the cluster. It exposes the service within the defined Kubernetes cluster.

    spec:
    type: NodePort
    ports:
    - port: 8080
        nodePort: 31999
        name: NodeportService

**NodePort** − It will expose the service on a static port on the deployed node. A ClusterIP service, to which NodePort service will route, is automatically created. The service can be accessed from outside the cluster using the NodeIP:nodePort.

    spec:
    ports:
    - port: 8080
        nodePort: 31999
        name: NodeportService
        clusterIP: 10.20.30.40

**Load Balancer** − It uses cloud providers’ load balancer. NodePort and ClusterIP services are created automatically to which the external load balancer will route.

A full service **yaml** file with service type as Node Port. Try to create one yourself.

    apiVersion: v1
    kind: Service
    metadata:
    name: appname
    labels:
        k8s-app: appname
    spec:
    type: NodePort
    ports:
    - port: 8080
        nodePort: 31999
        name: hellonginx
    selector:
        k8s-app: appname
        component: nginx
        env: env_name

# Port Forwarding

In the context of developing applications on Kubernetes, it is often useful to quickly access a service from your local environment without exposing it using, for example, a load balancer or an ingress resource. In these situations, you can use port forwarding.

Let's create an application consisting of a deployment and a service named simpleservice, serving on port 80:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: sise-deploy
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: sise
      template:
        metadata:
          labels:
            app: sise
        spec:
          containers:
          - name: sise
            image: quay.io/openshiftlabs/simpleservice:0.5.0
            ports:
            - containerPort: 9876
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: simpleservice
    spec:
      ports:
        - port: 80
          targetPort: 9876
      selector:
        app: sise

Let's say we want to access the simpleservice service from the local environment on port 8080. Traffic can be forwarded to your local system using the port-forward subcommand:

    kubectl port-forward service/simpleservice 8080:80

Now we can invoke the service locally like so (using a separate terminal session):

    curl localhost:8080/info

The output should resemble the following:

    {"host": "localhost:8080", "version": "0.5.0", "from": "127.0.0.1"}