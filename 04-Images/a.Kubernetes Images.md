# Kubernetes Images

Kubernetes images are the key building blocks of Containerized Infrastructure. As of now, we are only supporting Kubernetes to support Docker images. Each container in a pod has its Docker image running inside it.

When we are configuring a pod, the image property in the configuration file has the same syntax as the Docker command does. The configuration file has a field to define the image name, which we are planning to pull from the registry.

Following is the common configuration structure which will pull image from Docker registry and deploy in to Kubernetes container.

In the above code, we have defined −


    apiVersion: v1
    kind: pod
    metadata:
    name: Tesing_kubernetes_images 
    spec:
      containers:
      - name: nginx
        image: <Name of the Docker image>
        imagePullPolicy: Always
        command: ["echo", "SUCCESS"] 


- **name: Tesing_kubernetes_images** − This name is given to identify and check what is the name of the container that would get created after pulling the images from Docker registry.

- **name: nginx** − This is the name given to the container that we are trying to create. Like we have given nginx.

- **image: <Name of the Docker image>** − This is the name of the image which we are trying to pull from the Docker or internal registry of images. We need to define a complete registry path along with the image name that we are trying to pull.

- **imagePullPolicy − Always** - This image pull policy defines that whenever we run this file to create the container, it will pull the same name again.

- **command: [“echo”, “SUCCESS”]** − With this, when we create the container and if everything goes fine, it will display a message when we will access the container.
