# Labels
Labels are the mechanism used to organize Kubernetes objects. A label is a key-value pair with certain restrictions concerning length and allowed values but without any pre-defined meaning. You're free to choose labels as you see fit

Labels are key-value pairs which are attached to pods, replication controller and services. They are used as identifying attributes for objects such as pods and replication controller. They can be added to an object at creation time and can be added or modified at the run time.

    apiVersion: v1
    kind: Pod
    metadata:
    name: labelex
    labels:
        env: development
    spec:
    containers:
    - name: sise
        image: quay.io/openshiftlabs/simpleservice:0.5.0
        ports:
        - containerPort: 9876
  
In the above code, to express environments such as "this pod is running in production" or ownership, like "department X owns that pod".

# Selectors 

Labels do not provide uniqueness. In general, we can say many objects can carry the same labels. Labels selector are core grouping primitive in Kubernetes. They are used by the users to select a set of objects.

Kubernetes API currently supports two type of selectors −

- Equality-based selectors
- Set-based selectors

Let's create a pod that initially has one label (env=development):

    apiVersion: v1
    kind: Service
    metadata:
      name: sp-neo4j-standalone
    spec:
      ports:
      - port: 7474
        name: neo4j
      type: NodePort
      selector:
        app: salesplatform 
        component: neo4j 

In the above code, we are using the label selector as **app: salesplatform** and component as **component: neo4j.**

Once we run the file using the **kubectl** command, it will create a service with the name **sp-neo4j-standalone** which will communicate on port 7474. The type is **NodePort** with the new label selector as **app: salesplatform** and **component: neo4j.**

