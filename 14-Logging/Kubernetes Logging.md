## Logging

Logging is one option to understand what is going on inside your applications and the cluster at large. Basic logging in Kubernetes makes the output a container produces available through the kubectl tool. More advanced setups consider logs across nodes and store them in a central place, either within the cluster or via a dedicated (cloud-based) service.

Let's create a pod called logme that runs a container writing to stdout and stderr:

    apiVersion: v1
    kind: Pod
    metadata:
    name: logme
    spec:
    containers:
    - name: gen
        image: centos:7
        command:
        - "bin/bash"
        - "-c"
        - "while true; do echo $(date) | tee /dev/stderr; sleep 1; done"


To view the five most recent log lines of the gen container in the logme pod, execute:

    kubectl logs --tail=5 logme -c gen
Streaming functionality, similar to running tail -f, is available as well:

    kubectl logs -f --since=10s logme -c gen

ou can also view logs of pods that have already completed their lifecycle. To demonstrate this, create a pod called oneshot that counts down from 9 to 1 and then exits:

    apiVersion: v1
    kind: Pod
    metadata:
    name: oneshot
    spec:
    containers:
    - name: gen
        image: centos:7
        command:
        - "bin/bash"
        - "-c"
        - "for i in 9 8 7 6 5 4 3 2 1 ; do echo $i ; done"
  
By using the -p option, you can print the logs for previous instances of the container in a pod:

    kubectl logs -p oneshot -c gen