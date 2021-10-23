# Jobs

A job in Kubernetes is a supervisor for pods that run for a certain time to completion, for example a calculation or a backup operation.

    apiVersion: batch/v1
    kind: Job
    metadata:
    name: countdown
    spec:
    template:
        metadata:
        name: countdown
        spec:
        containers:
        - name: counter
            image: centos:7
            command:
            - "bin/bash"
            - "-c"
            - "for i in 9 8 7 6 5 4 3 2 1 ; do echo $i ; done"
        restartPolicy: Never

In the above code, we have defined −

**kind: Job** → We have defined the kind as Job which will tell kubectl that the yaml file being used is to create a job type pod.

**Name: countdown** → This is the name of the template that we are using and the spec defines the template.

**name: countdown** → we have given a name as py under container spec which helps to identify the Pod which is going to be created out of it.

**Image: centos** → the image which we are going to pull to create the container which will run inside the pod.

**restartPolicy: Never** →This condition of image restart is given as never which means that if the container is killed or if it is false, then it will not restart itself.

# Scheduled Job

 Scheduled job in Kubernetes uses Cronetes, which takes Kubernetes job and launches them in Kubernetes cluster.

- Scheduling a job will run a pod at a specified point of time.
- A parodic job is created for it which invokes itself automatically.

**Note** − The feature of a scheduled job is supported by version 1.4 and the betch/v2alpha 1 API is turned on by passing the **--runtime-config=v1** while bringing up the API server.

We will use the same yaml which we used to create the job and make it a scheduled job.

    apiVersion: v1
    kind: Job
    metadata:
    name: py
    spec:
    schedule: h/30 * * * * ? 
    template:
        metadata
            name: countdown
        spec:
          containers:
          - name: countdown
            image: python
            args:
            - "/bin/sh" 
            - "-c"
            - "ps –eaf"  
          restartPocliy: OnFailure

In the above code, we have defined −

- **schedule: h/30 * * * * ?** → To schedule the job to run in every 30 minutes.
- **/bin/sh:** This will enter in the container with /bin/sh
- **ps –eaf** → Will run ps -eaf command on the machine and list all the running process inside a container.