---
title: Dockerize a website & Pull it privately in k8s
date: 2022-09-08 01:47:58
tags: kubernetes
cover: https://res.cloudinary.com/practicaldev/image/fetch/s--ne7pa20x--/c_imagga_scale,f_auto,fl_progressive,h_900,q_auto,w_1600/https://thepracticaldev.s3.amazonaws.com/i/7xyqmfvmadyqu2girucz.jpg
categories:
- [Kubernetes]
---

In this article you will learn how to containerize a static website using nginx. Then we will push a private docker image in dockerhub. Finally we will use this private image to be pulled in our Kubernetes cluster ! I will split this article to 2 small and easy steps, you can skip any one you want !

# Containerize a static website and push it on dockerhub
In this Section we will choose a template from random websites that provides free css templates, then we will dockerize it !

## Dockerize the website

I'll choose this template from this [link](https://www.free-css.com/free-css-templates/page282/royal-cars)
![](https://imgur.com/ngHGyw0.png)

Download it and let's create our Dockerfile ! 
We will use the `nginx:alpine` image and copy all the assets of the website to the `/usr/share/nginx/html` to be hosted by the nginx webserver. 

```Dockerfile
FROM nginx:alpine
WORKDIR /usr/share/nginx/html
RUN rm -rf ./*
COPY ./ ./
RUN chmod +r -R . 
ENTRYPOINT ["nginx","-g","daemon off;"]
```
Now it's time to build the container image 

```bash command line prompt
raf@4n6nk8s:~$ sudo docker build . -t <user_name>/cars-app
Sending build context to Docker daemon  2.454MB
Step 1/5 : FROM nginx:alpine
alpine: Pulling from library/nginx
213ec9aee27d: Downloading [===================================>               ]  2.018MB/2.806MB
2546ae67167b: Downloading [=========>                                         ]  1.461MB/7.403MB
23b845224e13: Download complete
9bd5732789a3: Download complete
328309e59ded: Waiting
b231d02e5150: Waiting
```
Now let's create a container to test it before make push it! 

```bash command line prompt
raf@4n6nk8s:~$ sudo docker run --name car-demo -p 8686:80 <user_name>/cars-app
2022/09/08 22:53:47 [notice] 1#1: using the "epoll" event method
2022/09/08 22:53:47 [notice] 1#1: nginx/1.23.1
2022/09/08 22:53:47 [notice] 1#1: built by gcc 11.2.1 20220219 (Alpine 11.2.1_git20220219)
2022/09/08 22:53:47 [notice] 1#1: OS: Linux 5.15.0-41-generic
2022/09/08 22:53:47 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2022/09/08 22:53:47 [notice] 1#1: start worker processes
2022/09/08 22:53:47 [notice] 1#1: start worker process 7
2022/09/08 22:53:47 [notice] 1#1: start worker process 8
2022/09/08 22:53:47 [notice] 1#1: start worker process 9
2022/09/08 22:53:47 [notice] 1#1: start worker process 10
```
Sanity Check please ! Oh everything is OK !

![](https://imgur.com/nQ6NcxG.png)

## Push the container image to private repo

Now go and create a private repository in your dockerhub to push it ! 
```bash command line prompt
raf@4n6nk8s:~$ sudo docker image push <user_name>/cars-app
Using default tag: latest
The push refers to repository [docker.io/<user_name>/cars-app]
41c30355eff8: Pushed
f2ba5e032e84: Pushed
43e1f37b87cb: Pushed
bf4e176a4d9b: Mounted from library/nginx
a1d571e4e83d: Mounted from library/nginx
6d97b4d00719: Mounted from library/nginx
2a7647ca3937: Mounted from library/nginx
549c42eea4a6: Mounted from library/nginx
994393dc58e7: Mounted from library/nginx
```
Now it's time to deal with our kubernetes cluster ! 

# Use the private docker image in Kubernetes

In this demo I will use a production kubernetes cluster with 1 master node and 1 worker node 

```bash command line prompt 
raf@4n6nk8s:~$ kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
master    Ready    control-plane,master   77d   v1.23.1
worker1   Ready    <none>                 77d   v1.23.1
```
Let's create a namespace for this demo ! 

```bash command line prompt 
raf@4n6nk8s:~$ kubectl create ns private-docker
namespace/private-docker created
```
To make our cluster pull private images we need to create a special secret object with specific type called `docker-registry` secret. To make this secret work correctly you must specify the docker registry, username , password and docker email!

I prefer to put this params in variable environment to work with it easly 
```bash command line prompt 
export EMAIL=<email>
export USERNAME=<user_name>
export PASSWORD=<password>
export SERVER=<docker_registry>
```
In case you will use dockerhub as your registry you don't have to specify the server !

```
raf@4n6nk8s:~$ kubectl create -n private-docker secret docker-registry docker-secret --docker-username=$USERNAME --docker-password=$PASSWORD --docker-email=$EMAIL
secret/docker-secret created
```

Now let's create a pod with the image that we made it ! 

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: cars-app-pod
  name: cars-app-pod
  namespace: private-docker
spec:
  containers:
  - image: <user_name>/cars-app
    name: cars-app-pod
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
If you try create this pod with this YAML definition you'll get a `ErrImagePull` because we don't specify the docker secret that we created 
```bash command line prompt
raf@4n6nk8s:~$ kubectl get pods -n private-docker
NAME           READY   STATUS         RESTARTS   AGE
cars-app-pod   0/1     ErrImagePull   0          5s
```
Let's figure out the problem with `kubectl describe` command. Take a look at the events that occur when the pod try to pull the container image!
```
 Events:
  Type     Reason          Age                From               Message
  ----     ------          ----               ----               -------
  Normal   Scheduled       44s                default-scheduler  Successfully assigned private-docker/cars-app-pod to worker1
  Normal   SandboxChanged  40s                kubelet            Pod sandbox changed, it will be killed and re-created.
  Normal   Pulling         26s (x2 over 43s)  kubelet            Pulling image "<user_name>/cars-app"
  Warning  Failed          23s (x2 over 40s)  kubelet            Failed to pull image "<user_name>/cars-app": rpc error: code = Unknown desc = Error response from daemon: pull access denied for <user_name>/cars-app, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
  Warning  Failed          23s (x2 over 40s)  kubelet            Error: ErrImagePull
  Normal   BackOff         12s (x4 over 39s)  kubelet            Back-off pulling image "<user_name>/cars-app"
  Warning  Failed          12s (x4 over 39s)  kubelet            Error: ImagePullBackOff

```
Specifying the docker secret in the pod YAML definition will solve this problem. The pod will pull the container image without any problem
Adding `imagePullSecrets` attributes in the `spec` of the pod allow the pod to pull this image. The Final YAML definition will be similar to the following one ! 

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: cars-app-pod
  name: cars-app-pod
  namespace: private-docker
spec:
  containers:
  - image: <user_name>/cars-app
    name: cars-app-pod
    resources: {}
  imagePullSecrets:
  - name: docker-secret
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
Create the pod again and check it with `kubectl get pods` and don't forget to specify the namespace!

``` bash command line prompt
raf@4n6nk8s:~$ kubectl get pods -n private-docker
NAME           READY   STATUS    RESTARTS   AGE
cars-app-pod   1/1     Running   0          4s
```
I want to make sure that everything is ok, so i will expose this pod using the NodePort service. The following service YAML Definition will expose the pod correctly :
```yaml yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    run: cars-app-pod
  name: cars-app-pod
  namespace: private-docker
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: cars-app-pod
  type: NodePort
```
Let's now check the service and get the node port that we will use it to test the pod
```bash command line prompt 
raf@4n6nk8s:~$ kubectl get svc -n private-docker
NAME           TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
cars-app-pod   NodePort   10.111.98.22   <none>        80:31651/TCP   6s
```
Now you can visit either http://master:31651 or http://worker1:31651 and you will find the static website work without any problem!

![](https://imgur.com/3TVOl36.png)
