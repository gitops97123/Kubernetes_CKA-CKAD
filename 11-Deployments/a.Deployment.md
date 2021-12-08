# Kubernetes Deployment 

A deployment is a supervisor for pods, giving you fine-grained control over how and when a new pod version is rolled out as well as rolled back to a previous state.  

Deployments are upgraded and higher version of replication controller. They manage the deployment of replica sets which is also an upgraded version of the replication controller. They have the capability to update the replica set and are also capable of rolling back to the previous version.

They provide many updated features of matchLabels and selectors. We have got a new controller in the Kubernetes master called the deployment controller which makes it happen. It has the capability to change the deployment midway.

## Changing the Deployment

- **Updating** − The user can update the ongoing deployment before it is completed. In this, the existing deployment will be settled and new deployment will be created.

- **Deleting** − The user can pause/cancel the deployment by deleting it before it is completed. Recreating the same deployment will resume it.

- **Rollback** − We can roll back the deployment or the deployment in progress. The user can create or update the deployment by using DeploymentSpec.PodTemplateSpec = oldRC.PodTemplateSpec.

## Deployment Strategies

Deployment strategies help in defining how the new RC should replace the existing RC.

- **Recreate** − This feature will kill all the existing RC and then bring up the new ones. This results in quick deployment however it will result in downtime when the old pods are down and the new pods have not come up.

- **Rolling Update** − This feature gradually brings down the old RC and brings up the new one. This results in slow deployment, however there is no deployment. At all times, few old pods and few new pods are available in this process.

Let's create a deployment called Deployment.yaml that produces two replicas of a pod as well as a replica set:

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

    $ kubectl create –f Deployment.yaml -–record
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
