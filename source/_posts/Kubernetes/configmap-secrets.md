---
title: Introduction To Configmap & Secrets
date: 2022-09-06 11:29:37
tags:
cover: https://kruschecompany.com/wp-content/uploads/2019/08/01-1280x595.png
categories:
- [Kubernetes]
#https://miro.medium.com/max/1400/0*E6s0y6uIDi9X68XB.png
---
## An introduction to Kubernetes Secrets and ConfigMaps
Kubernetes has two types of objects that can inject configuration data into a container when it starts up: Secrets and ConfigMaps. Secrets and ConfigMaps behave similarly in Kubernetes, both in how they are created and because they can be exposed inside a container as mounted files or volumes or environment variables.

### To understand more how configmap and secrets are important let's consider the following scenario:
You have to run a postgres docker image on your host, you explore the documentation of this docker image provided in *[DockerHub]("https://hub.docker.com/_/postgres")* and you find that you the PostgreSQL image uses several environment variables and there is a mandatory variable called `POSTGRES_PASSWORD` must be defined by running this following command:
``` bash
$  docker run --name my-postgres -e POSTGRES_PASSWORD=mypassword -d postgres
```
### But how we can use this environment variables and how we can manage them in kubernetes ?
We can centralize the variables environment in two types of objects and import these variables in the manifest of a pod, replicasets, or deployments

### What is a ConfigMaps
In Kubernetes, a ConfigMap is nothing more than a key/value pair. A ConfigMap store’s non-confidential data, meaning no passwords or API keys. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

A ConfigMap allows you to decouple environment-specific configuration from your container images, so that your applications are easily portable.

#### Example of confimap manifest that store the database name & username: 

``` yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-map
data:
  postgres-db: "database"
  postgres-user: "mohamed"
```
#### Create the ConfigMaps Object:
``` bash
justk8s@justk8s-master:~$ kubectl apply -f first-configmap.yaml
configmap/config-map created
```
#### List the ConfigMap Objects:
``` bash
justk8s@justk8s-master:~$ kubectl get configmap
NAME               DATA   AGE
config-map         2      26s
kube-root-ca.crt   1      13h
```

### What is a Secrets:
Secrets are a Kubernetes object intended for storing a small amount of sensitive data. It is worth noting that Secrets are stored base64-encoded within Kubernetes, so they are not wildly secure.
Secrets are similar to ConfigMaps but are specifically intended to hold confidential data.

#### Example of confimap manifest that store the database password:
we must encode the value that we will stored in the Secrets

``` bash 
justk8s@justk8s-master:~$ echo -n "mohamed" | base64
bW9oYW1lZA==
```
Now we can use the base64 cipher in the Secret manifest
``` yaml
apiVersion: v1
kind: Secret
metadata:
  name: database-secret
type: Opaque
data:
  postgres-pass: "bW9oYW1lZA=="
```
#### Create the ConfigMaps Object:
``` bash
justk8s@justk8s-master:~$ kubectl apply -f first-secret.yaml
secret/database-secret created
```
#### List the ConfigMap Objects:
``` bash
justk8s@justk8s-master:~$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
database-secret       Opaque                                1      8s
default-token-xl8sl   kubernetes.io/service-account-token   3      13h
```
### How to use ConfigMaps and Secrets values in a Pod
we can use the values from `ConfigMaps` and `Secrets` in the pod manifests in the `env` propriety of the container by using the `valueFrom` field that can import values from configMap and Secrets
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: <pod-name>
spec:
  containers:
    - name: <name>
      image: <image>
      env:
        # Define the environment variable
        - name: <variable-name>
          valueFrom:
            configMapKeyRef:
              name: <config-map-object>     
              key: <key-name>
        - name: <variable-name>
          valueFrom:
            secretKeyRef:
              name: <secret-object>     
              key: <key-name>
```

### Create a PostgreSQL Pod that uses values from ConfigMaps and Secrets
#### 1- Create the Pod manifest:
``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: postgresql
spec:
  containers:
    - name: postgres
      image: postgres
      ports:
        - containerPort: 5432
      env:
        # Define the environment variable
        - name: POSTGRES_USER
          valueFrom:
            configMapKeyRef:
              name: config-map   
              key: postgres-user

        - name: POSTGRES_DBNAME
          valueFrom:
            configMapKeyRef:
              name: config-map   
              key: postgres-db
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database-secret   
              key: postgres-pass

```
#### 2- Create the Pod:
``` bash 
justk8s@justk8s-master:~$ kubectl apply -f postgres.yaml
pod/postgresql created
```
#### 3- List The Created Pod:
``` bash 
justk8s@justk8s-master:~$ kubectl get pods

NAME         READY   STATUS    RESTARTS   AGE
postgresql   1/1     Running   0          8s
```

#### 4- Test the Database created with variables of ConfigMap and Secrets:
We can open a bash session on the pod and open the database `mohamed` with the `psql` command provided by the postgreSQL
``` bash
justk8s@justk8s-master:~$ kubectl exec --stdin --tty postgresql  -- /bin/bash

root@postgresql:/# psql -U "mohamed"
psql (14.4 (Debian 14.4-1.pgdg110+1))
Type "help" for help.

mohamed=#
```


#### References:
 *[Get a Shell to a Running Container](https://kubernetes.io/docs/tasks/debug/debug-application/get-shell-running-container/)*

*[ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)*

*[Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)*

*[PostgreSQL Docker](https://hub.docker.com/_/postgres)*