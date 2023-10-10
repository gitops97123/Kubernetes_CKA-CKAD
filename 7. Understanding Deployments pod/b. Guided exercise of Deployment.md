Let's create a deployment called Deployment.yaml that produces two replicas of a pod as well as a replica set:

    $ cat Deployment.yaml 
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: sise-deploy
    spec:
    replicas: 2
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
            env:
            - name: SIMPLE_SERVICE_VERSION
            value: "0.9"

Create Deployment

    $ kubectl create -f Deployment.yaml --record
    deployment "Deployment" created Successfully.

Fetch the Deployment

    $ kubectl get deployments
    NAME           DESIRED     CURRENT     UP-TO-DATE     AVILABLE    AGE
    Deployment        3           3           3              3        20s

Check the Status of Deployment
    
    $ kubectl rollout status deployment/Deployment

Updating the Deployment
    
    $ kubectl set image deployment/Deployment tomcat=tomcat:6.0

Rolling Back to Previous Deployment

    $ kubectl rollout undo deployment/Deployment –to-revision=2


