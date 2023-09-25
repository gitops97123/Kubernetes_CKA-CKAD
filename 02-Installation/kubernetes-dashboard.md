# Kubernetes Dashboard

#### Kubernetes Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage applications running in the cluster and troubleshoot them, as well as manage the cluster itself.


## Install

To deploy Dashboard, execute following command:

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

## Access

To access Dashboard from your local workstation you must create a secure channel to your Kubernetes cluster. Run the following command:

    kubectl proxy

Now access Dashboard at:

    http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/.

## Create An Authentication Token (RBAC)

To find out how to create sample user and log in follow Creating sample user guide.

**NOTE:**

- Kubeconfig Authentication method does not support external identity providers or certificate-based authentication.
- Metrics-Server has to be running in the cluster for the metrics and graphs to be available. Read more about it in Integrations guide.

## Creating a Service Account
We are creating Service Account with the name admin-user in namespace kubernetes-dashboard first.

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: kubernetes-dashboard
      namespace: kubernetes-dashboard

## Creating a ClusterRoleBinding

In most cases after provisioning the cluster using kops, kubeadm or any other popular tool, the ClusterRole cluster-admin already exists in the cluster. We can use it and create only a ClusterRoleBinding for our ServiceAccount. If it does not exist then you need to create this role first and grant required privileges manually.

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: admin-user
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: kubernetes-dashboard
      namespace: kubernetes-dashboard

## Getting a Bearer Token

Now we need to find the token we can use to log in. Execute the following command:

    kubectl -n kubernetes-dashboard create token kubernetes-dashboard

It should print something like:
    
    eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXY1N253Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwMzAzMjQzYy00MDQwLTRhNTgtOGE0Ny04NDllZTliYTc5YzEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Z2JrQlitASVwWbc-s6deLRFVk5DWD3P_vjUFXsqVSY10pbjFLG4njoZwh8p3tLxnX_VBsr7_6bwxhWSYChp9hwxznemD5x5HLtjb16kI9Z7yFWLtohzkTwuFbqmQaMoget_nYcQBUC5fDmBHRfFvNKePh_vSSb2h_aYXa8GV5AcfPQpY7r461itme1EXHQJqv-SN-zUnguDguCTjD80pFZ_CmnSE1z9QdMHPB8hoB4V68gtswR1VLa6mSYdgPwCHauuOobojALSaMc3RH7MmFUumAgguhqAkX3Omqd3rJbYOMRuMjhANqd08piDC3aIabINX6gP5-Tuuw2svnV6NYQ
    
Now copy the token and paste it into the Enter token field on the login screen.

![image](https://user-images.githubusercontent.com/92106894/173192806-653515b6-bca9-4548-a97d-4c240494f519.png)

![image](https://github.com/gitops97123/Kubernetes_CKA-CKAD/blob/main/02-Installation/login.png)
