# Kubernetes Volumes

A Kubernetes volume is essentially a directory accessible to all containers running in a pod. In contrast to the container-local filesystem, the data in volumes is preserved across container restarts. The medium backing a volume and its contents are determined by the volume type: 

- node-local types such as emptyDir or hostPath
- file-sharing types such as nfs
- cloud provider-specific types like awsElasticBlockStore, azureDisk, or gcePersistentDisk
- distributed file system types, for example glusterfs or cephfs
- special-purpose types like secret, gitRepo

## Types of Kubernetes Volume

Here is a list of some popular Kubernetes Volumes −

- **emptyDir** − It is a type of volume that is created when a Pod is first assigned to a Node. It remains active as long as the Pod is running on that node. The volume is initially empty and the containers in the pod can read and write the files in the emptyDir volume. Once the Pod is removed from the node, the data in the emptyDir is erased.

- **hostPath** − This type of volume mounts a file or directory from the host node’s filesystem into your pod.

- **gcePersistentDisk** − This type of volume mounts a Google Compute Engine (GCE) Persistent Disk into your Pod. The data in a gcePersistentDisk remains intact when the Pod is removed from the node.

- **awsElasticBlockStore** − This type of volume mounts an Amazon Web Services (AWS) Elastic Block Store into your Pod. Just like gcePersistentDisk, the data in an awsElasticBlockStore remains intact when the Pod is removed from the node.

- **nfs** − An nfs volume allows an existing NFS (Network File System) to be mounted into your pod. The data in an nfs volume is not erased when the Pod is removed from the node. The volume is only unmounted.

- **iscsi** − An iscsi volume allows an existing iSCSI (SCSI over IP) volume to be mounted into your pod.

- **flocker** − It is an open-source clustered container data volume manager. It is used for managing data volumes. A flocker volume allows a Flocker dataset to be mounted into a pod. If the dataset does not exist in Flocker, then you first need to create it by using the Flocker API.

- **glusterfs** − Glusterfs is an open-source networked filesystem. A glusterfs volume allows a glusterfs volume to be mounted into your pod.

- **rbd** − RBD stands for Rados Block Device. An rbd volume allows a Rados Block Device volume to be mounted into your pod. Data remains preserved after the Pod is removed from the node.

- **cephfs** − A cephfs volume allows an existing CephFS volume to be mounted into your pod. Data remains intact after the Pod is removed from the node.

- **gitRepo** − A gitRepo volume mounts an empty directory and clones a git repository into it for your pod to use.

- **secret** − A secret volume is used to pass sensitive information, such as passwords, to pods.

- **persistentVolumeClaim** − A persistentVolumeClaim volume is used to mount a PersistentVolume into a pod. PersistentVolumes are a way for users to “claim” durable storage (such as a GCE PersistentDisk or an iSCSI volume) without knowing the details of the particular cloud environment.

- **downwardAPI** − A downwardAPI volume is used to make downward API data available to applications. It mounts a directory and writes the requested data in plain text files.

- **azureDiskVolume** − An AzureDiskVolume is used to mount a Microsoft Azure Data Disk into a Pod.

Persistent Volume and Persistent Volume Claim

- **Persistent Volume (PV)** − It’s a piece of network storage that has been provisioned by the administrator. It’s a resource in the cluster which is independent of any individual pod that uses the PV.

- **Persistent Volume Claim (PVC)** − The storage requested by Kubernetes for its pods is known as PVC. The user does not need to know the underlying provisioning. The claims must be created in the same namespace where the pod is created.

Creating PV.yaml

    ---
    apiVersion: v1
    kind: PersistentVolume
    metadata:
    annotations:
        pv.kubernetes.io/bound-by-controller: "yes"
    finalizers:
    - kubernetes.io/pv-protection
    labels:
        volume: pv0001
    name: pv0001
    resourceVersion: "227035"
    selfLink: /api/v1/persistentvolumes/pv0001
    spec:
    accessModes:
    - ReadWriteOnce
    capacity:
        storage: 5Gi
    claimRef:
        apiVersion: v1
        kind: PersistentVolumeClaim
        name: myclaim
        namespace: default
        resourceVersion: "227033"
    hostPath:
        path: /mnt/pv-data/pv0001
        type: ""
    persistentVolumeReclaimPolicy: Recycle
    volumeMode: Filesystem
    status:
    phase: Bound

In order to use a PV, you need to claim it first, using a persistent volume claim (PVC). The PVC requests a PV with your desired specification (size, speed, etc.) from Kubernetes and binds it to a pod where you it is mounted as a volume. Let's create a PVC, asking Kubernetes for 1 GB of storage using the default storage class:

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
    name: myclaim
    spec:
    accessModes:
    - ReadWriteOnce
    resources:
        requests:
        storage: 1Gi

In the above code, we have defined −

- **kind: PersistentVolumeClaim** → It instructs the underlying infrastructure that we are trying to claim a specified amount of space.
- **name: myclaim** → Name of the claim that we are trying to create.
- **ReadWriteOnce** → This specifies the mode of the claim that we are trying to create.
- **storage: 1Gi** → This will tell kubernetes about the amount of space we are trying to claim.

let's create a deployment that uses above PVC and mounts it as a volume into /tmp/persistent:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: pv-deploy
    spec:
    replicas: 1
    selector:
        matchLabels:
        app: mypv
    template:
        metadata:
        labels:
            app: mypv
        spec:
        containers:
        - name: shell
            image: centos:7
            command:
            - "bin/bash"
            - "-c"
            - "sleep 10000"
            volumeMounts:
            - name: mypd
            mountPath: "/tmp/persistent"
        volumes:
        - name: mypd
            persistentVolumeClaim:
            claimName: myclaim

In the above code, we have defined −

- **volumeMounts:** → This is the path in the container on which the mounting will take place.
- **Volume:** → This definition defines the volume definition that we are going to claim.
- **persistentVolumeClaim:** → Under this, we define the volume name which we are going to use in the defined pod.
