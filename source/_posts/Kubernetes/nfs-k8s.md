---
title: NFS as Remote Storage for Kubernetes
date: 2022-09-10 23:26:58
tags:
cover: https://www.axigen.com/usr/files/articles/x118.png.pagespeed.ic.B1J-BES_Qy.png
categories:
- [Kubernetes]
---

We will setting up a NFS server to use it as remote storage for our cluster to create a lot of persistent volumes in our local infrastructure !

We Assume that we have 4 Ubuntu 20.04 LTS, The Kubernetes is installed and the `4n6nk8s-nfs` host in the same network with the cluster :
| Role       | Hostname         | IP address      | 
| ---------- | ---------------- | --------------- |
| Master     | 4n6nk8s-master   | 192.168.1.18/24 | 
| Worker     | 4n6nk8s-worker1  | 192.168.1.19/24 | 
| Worker     | 4n6nk8s-worker2  | 192.168.1.20/24 | 
| NFS Server | 4n6nk8s-nfs      | 192.168.1.80/24 |

## What is a NFS (Network File System) Server:
Network File System (NFS) is a networking protocol for distributed file sharing. A file system defines the way data in the form of files is stored and retrieved from storage devices, such as hard disk drives, solid-state drives and tape drives. NFS is a network file sharing protocol that defines the way files are stored and retrieved from storage devices across networks.

This distributed file system protocol allows a user on a client computer to access files over a network in the same way they would access a local storage file.
## Setting up the NFS server 

We need to install the `nfs-kernel-server` package on the NFS server. This package will store additional packages such as `nfs-common` and `4n6nk8s@rpcbind`
``` bash Command Line Prompt
4n6nk8s-nfs:~$ sudo apt install nfs-kernel-server
```
Now let's create an NFS Export Directory 
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-nfs:~$ sudo mkdir /mnt/nfs-data 
```
Now let's give it a read,write and execute privileges to all the contents inside the directory
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-nfs:~$ sudo chmod 777 /mnt/nfs-data
```

Now Lets add a new line to the `/etc/exports` configuration file.
> The `/etc/exports` file indicates all directories that a nfs server exports to its clients. Each line in the file specifies a single directory.
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-nfs:~$ sudo vim /etc/exports
```
You can provide access to a single client, multiple clients, or specify an entire subnet. In this guide, we have allowed an entire subnet to have access to the NFS share.
``` vim
/mnt/nfs-data 192.168.1.0/24(rw,sync,no_subtree_check)
```
After granting access to the subnet, let's export the NFS share directory and restart the NFS

``` bash Command Line Prompt
4n6nk8s@4n6nk8s-nfs:~$ sudo exportfs -a
4n6nk8s@4n6nk8s-nfs:~$ sudo systemctl restart nfs-kernel-server
```
Let's allow NFS access through the firewall
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-nfs:~$ sudo ufw allow from 192.168.43.0/24 to any port nfs
```
## Install the NFS Client on the Kubernetes Nodes
We must install the `nfs-common` packages to access to the NFS share so let's install it by running the following command on each node:
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-worker1:~$ sudo apt install nfs-common
```
This command mount the NFS Share on one node for testing and sanity check only
> The mount command is not a mandatory step. We mount for testing purposes. you can skip to the next section 
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-worker1:~$ sudo mount 4n6nk8s-nfs:/mnt/nfs-data  /mnt
```
Let's Create a file for testing 
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-worker1:~$ cd /mnt
4n6nk8s@4n6nk8s-worker1:/mnt $ touch file 
```
Check the `/mnt/nfs-data` on the NFS server
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-nfs:~$ cd /mnt/nfs-data
4n6nk8s@4n6nk8s-nfs:/mnt/nfs-data$ ls
file 
```
## Kubernetes with NFS remote Storage demo
After Setting up the NFS server and install the NFS client on the kubernetes nodes. Now it's time to do some practice with `Persistent Volume` and `Persistent Volume Claim` with NFS storage.
### Create a Persistent Volume with NFS
Example of Persistent Volume manifest using nfs: 
``` yaml PersistentVolume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  nfs:
    server: 192.168.1.80 # the IP address of 4n6nk8s-nfs host
    path: "/mnt/nfs-data"

```
Make sure to put the correct IP address of the NFS server and the correct NFS Share point!
Create the persistent volume using kubectl 
``` bash Command Line Prompt 
4n6nk8s@4n6nk8s-master:~$ kubectl apply -f nfs-pv.yaml
persistentvolume/pv-nfs created
```
List the Persistent Volumes to make sure for the creation
``` bash Command Line Prompt 
4n6nk8s@4n6nk8s-master:~$ kubectl get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv-nfs   100Mi      RWX            Retain           Available           nfs                     4s

```
### Create a Persistent Volume Claim with NFS
Example of Persistent Volume Claim manifest using nfs: 

``` yaml PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  resources:
    requests:
        storage: 100Mi

```
Create the persistent volume using kubectl 
``` bash Command Line Prompt 
4n6nk8s@4n6nk8s-master:~$ kubectl apply -f nfs-pvc.yaml
persistentvolumeclaim/pvc-nfs created
```
List the `Persistent Volumes Claims` to make sure for the creation
``` bash Command Line Prompt 
4n6nk8s@4n6nk8s-master:~$ kubectl get pvc
NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-nfs   Bound    pv-nfs   100Mi      RWX            nfs            3s
```
### Create Nginx Deployment 
We use the `volumeMounts` and `volumes `attributes in this manifest to use the persistent volume we created:
``` yaml Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
    name: nginx-deployment
    labels:
        role: webserver
spec:
    replicas: 3
    selector:
        matchLabels:
            role: webserver
    template:
        metadata:
            labels:
                role: webserver
        spec:
            containers:
            - name: frontend
              image: nginx  # we use wrong image for the demo !
              ports:
                - name: nginx-port
                  containerPort: 80
              volumeMounts:
                - name: nfs
                  mountPath: /usr/share/nginx/html
            volumes:
            - name: nfs
              persistentVolumeClaim:
                claimName: pvc-nfs
``` 
Deploy the `nginx-deployment.yaml` using the `kubectl apply -f`.
``` bash Command Line Prompt 
4n6nk8s@4n6nk8s-master:~$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
``` 
Make sure that the deployment was created without any problems!
``` bash Command Line Prompt 
4n6nk8s@4n6nk8s-master:~$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7976956b49-fgbb4   1/1     Running   0          16s
nginx-deployment-7976956b49-hzrmm   1/1     Running   0          16s
nginx-deployment-7976956b49-kg5tx   1/1     Running   0          16s
```
### Sanity Check (Testing the NFS volumes):
Let's get shell on one of the running containers and go to the mount point then create a file! 
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-master:~$ kubectl exec --stdin --tty nginx-deployment-7976956b49-fgbb4 -- /bin/bash Command Line Prompt
root@nginx-deployment-7976956b49-fgbb4:/# ls
bin  boot  dev  docker-entrypoint.d  docker-entrypoint.sh  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
Now the shell is opened. Let's create a file in `/usr/share/nginx/html`:
``` bash Command Line Prompt
root@nginx-deployment-7976956b49-fgbb4:/# cd /usr/share/nginx/html/
```
 We find the file created in the client test xD
``` bash Command Line Prompt
root@nginx-deployment-7976956b49-fgbb4:/usr/share/nginx/html# ls
file
```
Create a file named "hi from the other side!"
``` bash Command Line Prompt
root@nginx-deployment-7976956b49-fgbb4:/usr/share/nginx/html# touch "hi from the other side!"
root@nginx-deployment-7976956b49-fgbb4:/usr/share/nginx/html# ls
file  'hi from the other side!'
```
Let's open another shell on another running container: 
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-master:~$ kubectl exec --stdin --tty nginx-deployment-7976956b49-kg5tx -- /bin/bash Command Line Prompt
root@nginx-deployment-7976956b49-kg5tx:/# cd /usr/share/nginx/html/
```
Bingoo! we find the same content on the same share point!
``` bash Command Line Prompt
root@nginx-deployment-7976956b49-kg5tx:/usr/share/nginx/html# ls
 file  'hi from the other side!'
root@nginx-deployment-7976956b49-kg5tx:/usr/share/nginx/html# exit
```
Now we will try to delete the deployment and recreate another to check the data in the  persistent volume
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-master:~$ kubectl delete -f nginx-deployment.yaml
deployment.apps "nginx-deployment" deleted

4n6nk8s@4n6nk8s-master:~$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
```
Check the deployment created or not !
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-master:~ kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
nginx-deployment-7976956b49-7d5vw   1/1     Running             0          10s
nginx-deployment-7976956b49-9r5gx   0/1     ContainerCreating   0          10s
nginx-deployment-7976956b49-fdq7w   1/1     Running             0          10s
```
Open another shell on running container from the new deployment to check the content of the persistent volume:
``` bash Command Line Prompt
4n6nk8s@4n6nk8s-master:~$ kubectl exec --stdin --tty nginx-deployment-7976956b49-7d5vw -- /bin/bash Command Line Prompt
root@nginx-deployment-7976956b49-7d5vw:/# cd /usr/share/nginx/html/
```
Display the content of the mount point `/usr/share/nginx/html/`
``` bash Command Line Prompt
root@nginx-deployment-7976956b49-7d5vw:/usr/share/nginx/html# ls
 file  'hi from the other side!'
root@nginx-deployment-7976956b49-7d5vw:/usr/share/nginx/html# exit
exit
```

Bingoo! The content still in the persistent volume without any problem !