---
title: Securinets Quals CTF 23 Forensics & Misc Challenges Writeup
date: 2023-08-06 19:00:00
tags:
cover: https://i.imgur.com/a72bPDt.png
categories:
- [Writeups]
---

# The CTF

Securinets Quals CTF was this Securinets generation's first CTF event on CTFTime where all the big teams have demonstrated therir skills to get to the finals in Tunisia this late-September.

We present four challenging tasks, two forensics involving Containerization and Kubernetes, Hiberfil.sys, Restore Point, and Crypto-currency Wallet, and two Misc involving Steganography and Reverse Engineering a set-top-box firmware and analyzing its memory dump.

Congratulations to SKSD, r3kapig, Hexagon for earning the top 3 spots in the CTF, and we deliver a special apperciation to the Securinets masterminds for their hard work and dedication to make this event a huge success, and we hope to see you all in the finals. 

Scoreboard:

![](https://i.imgur.com/eYBMvzh.jpg)

# The Betrayers

![](https://i.imgur.com/lAWQkqM.png)

### Description:

An IT company provides a critical service to customers. One day, the service is attacked by DDoS. The Attack is originated from within the company's internal network, and that each compromised host was sending a large number of requests to the server.

The attack occurred shortly after the company installed a new patch to one of its software programs. Every new version should be hosted by a special web server with DNS record in the private network but this time the new version is hosted by the Technical Director server!!!

The Technical Director is in trouble and he said that is not the attacker!

The strange thing is that the program is deleted directly after the blue team figure out the attack!

We need to get a release from that version and figure out how the attack occur!

> The binary expect to do DDoS attack on specifc IP. In this challenge we changed the IP and the rate of send requests

Author: **Raf²**

File: [Download](https://drive.google.com/file/d/1QNUnd4CEXXrqAC9cnvlUE-AW-aeWDszd/view?usp=sharing)

### Write-up 

Ok Let's figure out what is happen. We get the disk image of the Technical Director system. The system that host the new program patch. (It must not be this one)

Let's start figure out what softwares are installed and check the history of the system. 


Oh as expected there is a mail software. It's thunderbird. We will check it later.
![](https://i.imgur.com/y6i9bfK.png)


Let's Check the browsing history of the system and check what is going on here! We found Edge only here! 

Go to `C:\Users\Kong Office\AppData\Local\Edge\UserData\Default` and check the `History` database

![](https://i.imgur.com/E1oNXEo.png)

After checking every single link and search. we found searchs about whatsapp desktop how to install it and official whatsapp download website is visited.

![](https://i.imgur.com/geAQL42.png)

Thunderbird is installed for sure. But still no clue for whatsapp for now. Let's see what we have on the inbox now! It's time to check the mails! To check the inbox you can go to `C:\Users\Kong Office\AppData\Roaming\Thunderbird\Profiles\0ppw4tlt.default-release\ImapMail\imap.gmail.com` and investigate the `INBOX` file 

![](https://i.imgur.com/1J4A8Bx.png)

You can open the file using any text editor to check the mails but it's the worst way. you'll find the attachements encoded to base 64 and many trashy details 

You'll get an ugly view like this if you open it with vscode.
![](https://i.imgur.com/bH5fHr5.png)

But you can choose any thunderbird inbox viewer available in the internet. I tried a random one and i got a better exeperience to understand what happen. 

![](https://i.imgur.com/6T7zVMC.png)

Let's summerize what we found. One mail telling that the kubernetes cluster is updgraded successfully from the platform engineer, another one from a system adminstration telling the Technical Director about the security updates. Another one from `Securinets INSAT` asking for sponsorship to the National Cyber Security Congress! And the suspicision one is from a dummy mail `4n6nk8s@gmail.com`. In this mail we found a clue! 
A stranger tell the Technical Director about deploying a malicious binary and send a mega link! Ahhh Protected one

![](https://i.imgur.com/zalrPXh.png)

The link lead you to a wordlist that have strong and secure passwords. The mail talks about subsystem!! What a subsystem? Is it a `WSL` ??


![](https://i.imgur.com/1Sfnhxu.png)

and here we finished mail browser history and mail investigation. still the whatsapp thing! Where is whatsapp hide his data?? good question. Checking AppData don't help us. We didn't find any WhatsApp directory! 

After googling we found that there is some whatsapp version can store his data inside `%AppData%\Local\Packages`. Let's check that folder and Bingoo!! We found whatsapp directory! WhatsApp was installed here! Let's see what we can do here! 

![](https://i.imgur.com/4x79Dni.png)

You can find some conversation inside `messages.db` file. It might be encrypted and it might be decrypted. Let's try to open this database and see what we can find here!

![](https://i.imgur.com/mfM1duI.png)

I am chocked!! Two employees were behind all this!!
Two employees hate the director and they want to be in a trouble!

The attackers talks about encryption and subsystem password and they are shocked because the system is windows! and yes the mega decryption key is shared in this discussion. They send the mail to the director to make it in trouble!! 

![](https://i.imgur.com/8ATiHDP.png)

Let's download that wordlist and check the powershell history commands! I am sure there is something there. Because I found Hyber-V directory in random place while investigating. The history is stored in `%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history. txt`

![](https://i.imgur.com/GnecotB.png)

Yeah! Everything is cleared now! subsytem, subsystem! They are talking about WSL. WSL is installed here! and it used to host the malicious program! But what they does mean by encryption and the password. Mmmmm I guess we will face some cracking in the few next minutes! 

Let's check WSL now! Where is windows store the WSL? Just check `%userprofile%\AppData\Local\Packages` 

![](https://i.imgur.com/wIfONXj.png)

Let's mount this vhdx disk now in our Linux/WSL and check the files in this system. You can use `libguestfs-tools` to achieve this.

```bash
raf@4n6nk8s:~$ sudo apt-get install libguestfs-tools
```
Then, you can mount now the wsl disk image:
![](https://i.imgur.com/0QqjpLl.png)

We found a user named `Kong` and have a home directory. Let's check his files and bash history! 

![](https://i.imgur.com/W7p1CnE.png)

Let's start with `.bash_history` first. It's my number one rule! 
![](https://i.imgur.com/7kXO7nj.png)

Wow! The attacker configure wsl.conf then install docker! Mmm He configure it to be boot with systemd!
Then he installed something called kind, After that he download `kubectl`! Kubectl??? Ah It's `Kubernetes` time!

But what is kind? Searching on it. I found that kind is Kubernetes inside Docker. It's a lightweight kubernetes cluster.

There is 3 yaml files. For sure there are worth to check! Let's see what is inside these files!

```yaml
### k-config.yaml content ####
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  podSubnet: "10.244.0.0/16"
  serviceSubnet: "10.96.0.0/12"
  # the default CNI will not be installed
  disableDefaultCNI: false
           
nodes:
- role: control-plane
  # add a mount from /path/to/my/files on the host to /files on the node
  extraMounts:
  - hostPath: /etc/crio/crio.conf
    containerPath: /etc/crio/crio.conf
    # optional: if set, the mount is read-only.
    # default false
    readOnly: true
    # default false
    selinuxRelabel: false

  - hostPath: /etc/kubernetes
    containerPath: /etc/kubernetes
    # optional: if set, the mount is read-only.
    # default false
    readOnly: false
    # default false
    selinuxRelabel: false

  - hostPath: /var/lib/netcd
    containerPath: /var/lib/etcd
    # optional: if set, the mount is read-only.
    # default false
    readOnly: false
    # default false
    selinuxRelabel: false
- role: worker
  kubeadmConfigPatches:
  - |
    kind: JoinConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "app-type=web-app"
  extraPortMappings:
  - containerPort: 30999
    hostPort: 80
    # optional: set the bind address on the host
    # 0.0.0.0 is the current default
    listenAddress: "0.0.0.0"
    # optional: set the protocol to one of TCP, UDP, SCTP.
    # TCP is the default
    protocol: TCP
```
It's a kind cluster configuration file!! This fiel is used to declare the specification of your cluster! And as you see `/etc/kubernetes` and `/var/lib/etcd` are mounted. We have 2-node cluster and a `extraPortMappings` to expose a service. In addition there is node-labels on the worker-node

> This challenge requires some kubernetes knowledge. You must be familiar with the basics of the kubernetes. Like nodes, deployements, pods, services, etc... 

```yaml
### content deploy.yaml ###
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: update-app
  name: update-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: update-app
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: update-app
    spec:
      imagePullSecrets:
        - name: regcred
      containers:
      - image: medrafk8s/kubersex
        name: update-app
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
      affinity:
        nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                - matchExpressions:
                    - key: app-type
                      operator: In
                      values:
                      - web-app
                      - front-end
status: {}
```
This is a deployment manifest that pull a private image `medrafk8s/kubersex` using the imagePullSecrets fields! The nodeAffinity is used for some scheduling specification. In this example the deployment will be deployed in node with labels `app-type=web-app`

Let's summerize now! The attacker installed Kind to deploy the web app that hosts the malicious inside a kubernetes cluster! 

So our objective is to recover the webapp that contains the malicious binary! But how can we restore that container! It's a private container! What we should to do? 

To get the private container we need to get the pull secret. The credential that allow us to pull the container image! The credential are stored inside a `secret` (Kubernetes Object).

According to the bash history we have this commands:
```bash
USER=XXXXXXXXXXXXXXXXXXXX
PASSWORD=XXXXXXXXXXXXXXXXXXX
SERVER=XXXXXXXXXXXXXXXXXX
kubectl create secret docker-registry regcred --docker-server=$SERVER --docker-username=$USER --docker-password=$PASSWORD
kubectl get secrets
```
> The attacker store the credential inside environement and create a pull secret using `kubectl` command 

The only way to get the credential is to extract the secret from the `etcd`. But We have a problem! The etcd is protected! 

Let's Back to the bash history you'll find that the attacker edit a `kube-apiserver.yaml` and `enc.yaml`. 

```yaml
### enc.yaml ###
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
      - identity: {}
      - secretbox:
          keys:
            - name: key1
              secret: <BASE64 ENCODED>
```

Let's talk a little about the `Encryption Configuration` in Kubernetes. 

![](./images/https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

As known. Etcd is a distributed key-value store that is used by Kubernetes to store and retrieve the configuration data for the cluster. The Kubernetes API server interacts with etcd to read and write configuration data, such as the current state of the cluster and the desired state. The etcd data store serves as the single source of truth for the Kubernetes cluster's configuration data, and the API server ensures that this data is always up to date and consistent across the cluster.

The API server accepts an argument `--encryption-provider-config` that controls how API data is encrypted in etcd. In other words, API server can store the data inside etcd but not in plain text! It's encrypted to secure the data inside the database! for more details check this [link](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)

So our mission now is to find the secret value of the `EncryptionConfiguration` to use it after restoring the etcd by setting up a new api server that can read the encrypted data from the `etcd`.

![](https://i.imgur.com/XEB64Su.png)

Getting back to the whatsapp discussion. The attacker use the same password for the encryption key and subsystem password!

Bingo!! Crack time! We know that the encryption key must be 32 bytes. So this information will help us filtering the downloaded wordlist! 

Linux stores user passwords hashes in `/etc/shadow`. Great, we have the hash and the wordlist! Let's do it!

![](https://i.imgur.com/SanopKU.png)

Let's start filter out the passwords with 32 characters from the wordlist using grep and tail to pick from the bottom:
```bash
grep -E '^.{32}$' passwords.txt | tail -n 100000 > pass.txt
```
Now it's time to extract the user information from `/etc/shadow` and `/etc/password`
```bash
grep kong etc/passwd > passwd.txt
grep kong etc/shadow > shadow.txt
```
Before start cracking using `John The Ripper`. We need to combine the passwd and shadow into a format that john can read. `unshadow` can do this work!

```bash
unshadow passwd.txt shadow.txt > crack-me.txt
```
Now our hash is ready for crack. Let's use john with `--format=crypt` to specify that we are trying to crack yescrypt hash

```bash
john --wordlist=pass.txt crack-me.txt --format=crypt
```
After some minutes we got the password!! The password is : `@a*Hd~u32@q1Db/LUiOFxC*W2THm5p*V`

![](https://i.imgur.com/c1mV0ef.png)

Now it's time to resotre the `etcd`. Restoring a Kubernetes cluster in case of a disaster can be accomplished using etcd snapshots. However, the process of restoring an etcd snapshot is not straightforward and requires expertise. Actually in kubernetes etcd can take snapshot automatically. So we will count on that.

In this case i will use Kind to generate a lightweight cluster. In case you will use your own cluster you must to know that restoring etcd can lead you to lost your cluster data. So be careful!

To keep it simple I'll use the same configuration as `k-config.yaml`. 

```bash
kind create cluster --config=k-config.yaml
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
...
...

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```
In Kind each node will be represented as a container

```bash
raf@4n6nk8s:~$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS
  NAMES
eba4afe51855   kindest/node:v1.25.3   "/usr/local/bin/entr…"   2 minutes ago   Up 2 minutes   127.0.0.1:43323->6443/tcp   kind-control-plane
13d7b697ba1f   kindest/node:v1.25.3   "/usr/local/bin/entr…"   2 minutes ago   Up 2 minutes   0.0.0.0:80->30999/tcp       kind-worker
```
After making sure that the api-server works correctly we need to add the `Encryption Configuration` to make sure that the api server can read the encrypted data after restoring etcd. 

Go and put this file on `/etc/kubernetes/enc/enc.yaml`

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
      - identity: {}
      - secretbox:
          keys:
            - name: key1
              secret: QGEqSGR+dTMyQHExRGIvTFVpT0Z4QypXMlRIbTVwKlY=
```
Then go to `/etc/kubernetes/manifests/api-server.yaml` to add these lines.

```yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 172.18.0.2:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    ...
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml  # <-- add this line
    volumeMounts:
    ...
    - name: enc                           # <-- add this line
      mountPath: /etc/kubernetes/enc      # <-- add this line
      readonly: true                      # <-- add this line
    ...
  volumes:
  ...
  - name: enc                             # <-- add this line
    hostPath:                             # <-- add this line
      path: /etc/kubernetes/enc           # <-- add this line
      type: DirectoryOrCreate             # <-- add this line
  ...
```
After changing the manifest the api-server will be stopped and down. If you did it corretly, you need to wait a few seconds until the api-server works again! 


After configuring the api-server correctly let's take the content `/var/lib/netcd` and try to use the `db` as our snapshot that we want to restore it! 

![](https://i.imgur.com/fu0HEsS.png)

I suggest to copy `db` inside the control-plane node (in this case container). I will copy it to one of the extraMounts of kind. 

Now let's open a shell inside the control-plane container:

```bash
docker exec -it kind-control-plane bash
```
Now we need to use an utility called etcdctl that will help us to interact with the etcd server. 
```bash
apt update 
apt install etcd-client
```
After install it succesfully. An important thing it must be done. Don't skip this step! you must export an env variable to make etcdctl behave as we need!
```bash
export ETCDTL_API=3
```
now it's time to use this utility! There is a resotre command in this utility that allow us to restore the backup or the snapshot
```bash

etcdctl --endpoints=https://[etcd-server] \
--cacert=<path-to-ca-certification> \
--cert=<path-to-etcd-server-cert> \
--key=<path-to-etcd-server-key> \
--data-dir <path-to-restored-data> \ 
snapshot restore <path-to-snapshot>
```
You need to fill this arguments with your own. you can get theses information from `/etc/kubernetes/manifests/etcd.yaml`

In my case the command it will be like this:

```bash
ETCDTL_API=3 etcdctl --endpoints=https://[127.0.0.1:2379] \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
--data-dir /var/lib/etcd-backup \
snapshot restore ./db
```

You migth get an error or warning about hash stuff after running this command. Skip it! Let's check if `/var/lib/etcd-backup` directory is created and if it contains `member` directory!

If you got it! Then you are on the right way! Now it's time to do the crazy thing! Let's replace our new data placed in `/var/lib/etcd-backup/member/snap` to the original etcd database!

```bash
cp /var/lib/etcd-backup/member/snap/db /var/lib/etcd/member/snap/
```
Just wait few seconds! and try a kubectl command! 

```bash
kubectl get nodes
NAME                  STATUS     ROLES           AGE   VERSION
kind-control-plane    NotReady   <none>          51m   v1.25.3
kind-worker           NotReady   <none>          51m   v1.25.3
quals-control-plane   Ready      control-plane   26h   v1.25.3
quals-worker          Ready      <none>          26h   v1.25.3
```

Bingo! It works fine! but as you see here we have nodes naming issues cause of the data of the etcd! 

Let's check if we have a secret object or not!

```bash
kubectl get secrets
NAME      TYPE                             DATA   AGE
regcred   kubernetes.io/dockerconfigjson   1      26h
```
And Yes!!! We got our secret! Let's now inspect the data inside it!

```bash
 kubectl get secret regcred -o jsonpath='{.data.\.dockerconfigjson}' | base64 -d
{"auths":{"docker.io":{"username":"medrafk8s","password":"dckr_pat_MLkRjYtjdk7dwT80W_dx3VTuac8","auth":"bWVkcmFmazhzOmRja3JfcGF0X01Ma1JqWXRqZGs3ZHdUODBXX2R4M1ZUdWFjOA=="}}}
```

And finally! The registry is `docker.io`, the username is `medrafk8s` and the password is `dckr_pat_MLkRjYtjdk7dwT80W_dx3VTuac8`

With this creds we can pull the container that host the malicious binary and analyse his traffic! Let's do it!

```bash
raf@4n6nk8s:~$ docker login docker.io -u medrafk8s -p dckr_pat_MLkRjYtjdk7dwT80W_dx3VTuac8
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded
```
Let's pull this image now! 

```bash
raf@4n6nk8s:~$ docker pull medrafk8s/kubersex
Using default tag: latest
latest: Pulling from medrafk8s/kubersex
c158987b0551: Pull complete
1e35f6679fab: Pull complete
cb9626c74200: Pull complete
b6334b6ace34: Pull complete
f1d1c9928c82: Pull complete
9b6f639ec6ea: Pull complete
ee68d3549ec8: Downloading [=======>                                           ]  1.813MB/11.49MB
4caa31e6cbc5: Download complete
a1434f597582: Waiting
```

The 1st i used to do when I pull a container image in forensics challenges is to check the layers of the image! 

```bash
raf@4n6nk8s:~$ docker history medrafk8s/kubersex
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
77a3b3f4c35c   13 minutes ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
0b383b70426b   13 minutes ago   /bin/sh -c rm Serial-Key.txt                    0B
b6ca15406c7d   13 minutes ago   /bin/sh -c #(nop)  EXPOSE 80                    0B
64eaf6945cd1   13 minutes ago   /bin/sh -c #(nop) COPY file:ed75fc1a725ff91b…   20B
748d1b219207   13 minutes ago   /bin/sh -c #(nop) COPY dir:48e5409ee398a470a…   5.4MB
4937520ae206   6 weeks ago      /bin/sh -c set -x     && apkArch="$(cat /etc…   29.6MB
<missing>      6 weeks ago      /bin/sh -c #(nop)  ENV NJS_VERSION=0.7.12       0B
<missing>      6 weeks ago      /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
<missing>      6 weeks ago      /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B
<missing>      6 weeks ago      /bin/sh -c #(nop)  EXPOSE 80                    0B
<missing>      6 weeks ago      /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B
<missing>      6 weeks ago      /bin/sh -c #(nop) COPY file:e57eef017a414ca7…   4.62kB
<missing>      6 weeks ago      /bin/sh -c #(nop) COPY file:36429cfeeb299f99…   3.01kB
<missing>      6 weeks ago      /bin/sh -c #(nop) COPY file:d4375883ed5db364…   276B
<missing>      6 weeks ago      /bin/sh -c #(nop) COPY file:5c18272734349488…   2.12kB
<missing>      6 weeks ago      /bin/sh -c #(nop) COPY file:7b307b62e82255f0…   1.62kB
<missing>      6 weeks ago      /bin/sh -c set -x     && addgroup -g 101 -S …   4.74MB
<missing>      6 weeks ago      /bin/sh -c #(nop)  ENV PKG_RELEASE=1            0B
<missing>      6 weeks ago      /bin/sh -c #(nop)  ENV NGINX_VERSION=1.25.1     0B
<missing>      6 weeks ago      /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B
<missing>      6 weeks ago      /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>      6 weeks ago      /bin/sh -c #(nop) ADD file:828b07e74c184e7f2…   7.05MB
```

This is a webserver! Something important is here. No one can ignore the `Serial-Key.txt` file
We will get it later! Let's run the web app now 
```bash
raf@4n6nk8s:~$ sudo docker run --name kubersex -p 81:80 medrafk8s/kubersex
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/08/02 17:37:52 [notice] 1#1: using the "epoll" event method
2023/08/02 17:37:52 [notice] 1#1: nginx/1.25.1
2023/08/02 17:37:52 [notice] 1#1: built by gcc 12.2.1 20220924 (Alpine 12.2.1_git20220924-r4)
```
Mmm there is a Download button! Press it and you will get a binary! Bingo! This is the malicious release! 


![The web app](https://i.imgur.com/jDkGJef.png)

After checking the binary and run it, we found that the binary requires a serial key! Let's recover that file! 

```bash
raf@4n6nk8s:~$ docker save medrafk8s/medraf > image.tar 
raf@4n6nk8s:~$ tar -xf image.tar 
raf@4n6nk8s:~$ cd aae4861ba6bde0fbe820ba547bbe794c7c7e9c8be14c8972a6691eaa584161bc
raf@4n6nk8s:~$ tar -xf layer.tar && ls
Serial-Key.txt  VERSION  json  layer.tar
```

And this is the content of the file ! 

```
I will put the serial key in the container image in case you forget! YOU FOOOOL YOU USED TO FORGET IT! GO AND DESTROY THE SERVICE WITH THE BOSS HOST!

KEY:
KFTV-RCFL-VC5W-HX3X
```

> At this point you can reverse the binary and get the flag! or patch the binary to change a IP server with your choice! In this step reversing is the main idea so i put a public IP of a reachable server to make the binary send packets when players decice to run it !

Let's continue: 

Run the binary again and insert that key! It seems the binary works! I bet this is the responsible binary for the DDoS attack! So, I will check the behavior of this binary and open Wireshark while this one is running!


![Record Traffic with Wireshark](https://i.imgur.com/Opesxe5.png)

Mmmm this is http flow! Let's check the requests! 

![We got the flag!](https://i.imgur.com/tzWSbjW.png)

Binggo we got the flag! The flag is on the http header

FLAG: `Securinets{c03cefb79791e431011d0f86de9dd83aee67aebcec946bfefad00cd4807fc9c3}`


# Jackpot

![](https://i.imgur.com/GONLlyG.png)

We were able to find a left-behind cranky old cartel laptop in hibernation state. We believe that this device can help us retrieve the cartel's crypto-currency wallet and withdraw their future gains.

We know that the cartel uses an intranet to communicate their news within couple of blocks in the city undergrounds.

Provided this intellisense, can you win us the jackpot? Wrap your finding in Securinets as indicated below.

Flag format: Securinets{drug-enforcement-administration-for-the-win-style}

[adm & mida0ui](https://twitter.com/admida0ui)

## Attachment

Given a disk image, `disk.img`. The disk image is a raw disk image, and can be investigated using FTK Imager.
[https://drive.google.com/file/d/1Ss4QTQi4XbRXbTfy3FU0nDbqePFhuRFI/view?usp=sharing](https://drive.google.com/file/d/1Ss4QTQi4XbRXbTfy3FU0nDbqePFhuRFI/view?usp=sharing)

## Writeup

The disk image is a raw disk image, and can be investigated using FTK Imager. The disk image contains a single partition, which is a Windows XP installation.

Looking at the root of the disk image, we can see a few interesting files:

![](https://i.imgur.com/Qx1z5zy.png)

- `Documents and Settings` - Contains the user profiles of the system
- `Program Files` - Contains the installed programs
- `Windows` - Contains the Windows installation files
- `pagefile.sys` - The page file of the system, aka swap file, which holds the memory of the system when it is not in use or when it is not enough RAM to hold the memory of the system.
- `hiberfil.sys` - The hibernation file of the system, which contains the memory of the system when it is hibernated.
- `System Volume Information` - Contains the shadow copies of the system, which is a backup of the system files. It is used to restore the system in case of a system failure. It is also used by the Windows Backup utility to create a backup of the system. It is also used by the Windows Restore utility to restore the system to a previous state. This folder is present in all disk images, people are usually not interested in it. However, it is very interesting in this case, as it contains a folder named `_restore{xxxxxx}`. This folder contains a file named `RPx`, this cleary a restore point of the system. We will investigate it later as well.

and the rest of the stuff that would really find in any Windows disk...

Let's start with the fact that the laptop was in hibernation mode, we know for a fact that the hiberfil.sys stores the memory content at hibernation, maybe we can treat it as an actual memory and find the intranet chat!!?

Googling a bit, I found this (Superuser question)[https://superuser.com/questions/660649/how-to-read-windows-hibernation-file-hiberfil-sys-to-extract-data] and it says that if we convert the hiberfile.sys file to a raw image, we can use Volatility to analyze it!

and this is the command to use, as indicated:

```bash
vol -f hiberfil.sys --profile WinXPSP3x86 imagecopy -O hiberfil.raw
```

![](https://i.imgur.com/AgJlPlG.png)

Cool, we hope now we can use Volatility to analyze the hiberfil.raw file. Let's start by checking the processes and commands history.

![](https://i.imgur.com/MGjHNpu.png)

Cmdscan now maybe!

![](https://i.imgur.com/8KCPq36.png)

There we go, we can see what they meant by the legacy, old school chat. They were using `msg.exe` to communicate with each other. This is a very old program, and it is used to send messages to other users in the same network. It is very similar to the LAN messenger that we found earlier, but it is a command line program. `msg` can also be used to transmit files as well.

The chat goes with the description and indicates that our guy got some kind of password that was placed in a certain registry key and permanetly deleted it after.

Now, let's further investigate the programs now to build an understanding around this situation

![](https://i.imgur.com/NGRW8VU.png)

We only see Chrome to be the only interesting thing in this disk evidence.

We know the crypto-currency part might need some searching to know that Metamask the most famous crypto wallet is a browser extension!!

And yeah, Metamask is being used in Chrome. Metamask is a browser extension that allows you to use Ethereum, Binance Chain, and more. It is a very popular extension, and it is used by a lot of people. Metamask gives you a wallet address, and you can use it to send and receive cryptocurrencies whether in the Ethereum network, BSC network, or custom networks. But an address is only the public key you use to send and receive, we are looking further than that, we want the private one to be able to control the wallet completely.

Metamask Extension for Chrome

![](https://i.imgur.com/wWzkTjZ.png)

Could there be an active wallet within Metamask? Let's check it out.

Usually the extension data is stored within the 'Local Extension Settings' folder. And as expected, that folder contains some interesting files.

![](https://i.imgur.com/t8T8lYY.png)

The `Local Extension Settings` folder contains a folder named `nkbihfbeogaeaoehlefnkodbefgpgknn`, which is the ID of the Metamask extension. This folder contains a file named 0000xx.ldb, which is a DB file. Let's check it out.

According to this [Ethereum Stackexchange question](https://ethereum.stackexchange.com/questions/52658/where-does-metamask-store-the-wallet-seed-file-path). We could be able to completely control the existing wallet if we have the password. Could the challenge be all about that? and how to get the wallet password?

Yes it is possible, and we will get back to it later, for now, finding password is the priority!! We know it is a registry key, that was deleted.

Let's think how we can RESTORE it xD

Is it the time, to investigate the `_restore{xxxxxx}` folder? right?

We can make use of a VM, however (OP: says the disk must not be converted back to vmdk whether by using a different file format or breaking the boot files). This way we can't use a VM to check the restore point. So, we have to do it manually. My idea is to spin my own Windows XP on VMware, you can get the ISO officially by Microsoft with serial number from the WayBackMachine SP3 and x86 of course. After that we create a dummy restore point and modify its files by copying the RP folders from `System Volume Information` and then we can proceed to perform a restore point to see what is inside. I hope you can understand what I mean here.

(OP Note: for order I guess I will accept any provided order or make the user follow the order as shown in the leaked chat.)

Let's do it.

- Spinning a new Windows XP machine

![](https://i.imgur.com/gAJWn0m.png)

- Creating a dummy restore point or just get the disk UUID, anyways we need Windows to create a restore point folder for us, so let's do it.

![](https://i.imgur.com/hoZj5DU.png)

- Copying the RP folders from the Disk image `System Volume Information` to the new VM's `System Volume Information`

Export the restore folder from System Volume Information to your desktop or some folder.

Adjust these folder options on Windows XP VM

![](https://i.imgur.com/LH8W47T.png)

Then use this command to gain access to the System Volume Information folder

```powershell
cacls "C:\System Volume Information" /E /G Administrator:F
```

It can be administrator, Everyone or your specific VM username.

![](https://i.imgur.com/6JWtrGQ.png)

Then copy the restore folder contents to the System Volume Information folder

This is the current dummy restore point we made

![](https://i.imgur.com/ReEeaE9.png)

We add the exported RP5 and RP6 folders from the disk image with them

In each of the folders there is a `drivetable.txt` file that contains the old disk UUID, we need to replace it with the new one.
  
- Modifying Disk UUID in the restore point

You can also modify the `domain.txt` since it contains the old user-id but I believe the Restore point utility can very much detect the change.

- Restoring the system like if it was a real restore point we made.

![](https://i.imgur.com/NW9lTab.png)

We can see a new entry added called "restore_point", so let's proceed with it!

This is the domain thing, hit 'OK'

![](https://i.imgur.com/VMADEqo.png)

We wait for a quick restart.

![](https://i.imgur.com/X1drpyr.png)

And that's a success!

Let's open the registry now, shall we?

![](https://i.imgur.com/ynNyJqX.png)

And there we go, a key named password containing a string named deleteMe! Bingo!

At this point, there is only one thing left, let's get the flag!

Remember that Ethereum stack exchange question I mentioned earlier? here is the link again, also bring the content of the ldb file's vault which is near the word "keyring", you can CTRL+F that in the ldb files and adjust the format if some letters are escaped or encoded because data is never lost, you can also upload the whole file to the [tool](https://metamask.github.io/vault-decryptor/)

[https://ethereum.stackexchange.com/questions/52658/where-does-metamask-store-the-wallet-seed-file-path](https://ethereum.stackexchange.com/questions/52658/where-does-metamask-store-the-wallet-seed-file-path).

![](https://i.imgur.com/FxNDjWP.png)

![](https://i.imgur.com/Mhm1M63.png)

Let's get our flag now

![](https://i.imgur.com/3tK5T2J.png)

Done, we have some critical information here. The seed phrase! We wrap it in Securinets

`Securinets{female-fire-strong-accuse-spring-update-bird-exchange-home-embark-latin-mom}`

Thanks for reading, we hope you enjoyed it, and we will be happy to hear your feedback and suggestions.

GGs **itunderground** for solving this challenge

## Idea and Final Words

Just wanted to use pagefile.sys, hiberfile.sys, the Metamask thing, and Restore points because nobody used them like this in a CTF. So don't know if that's a great thing or rather will make players frustrated. Yet, tried to make everything as clear as possible for the players to understand and enjoy the challenge as a whole.

# Raf Hide

![](https://i.imgur.com/QRKzGoO.png)

### Description:
> My friend write a tool that can hide data inside an image. He give me this tool with an anime video without any source code and challenge me to get the secret! . Can you help me extract the flag? 
Flag format: Securinets{Secrect}

> In this Challenge you will get a go binary and a video! I will not explain how to reverse the binary. The main Idea is to understand what the binary do and extract the data from the video

Author: **Raf²**

Binary: [Download](https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/rafhide)

This is a python version of what the binary can do : 

```python
from PIL import Image
import argparse

# Create an ArgumentParser object
parser = argparse.ArgumentParser(description='Example program using options and arguments')

# Add options and arguments
parser.add_argument('-i', '--input', help='select cover-image', type=str)
parser.add_argument('-e', '--embedfile', help='select image to be embedded', type=str)
parser.add_argument('-o', '--output', help='The output', type=str)

# Parse the arguments
args = parser.parse_args()

# Check if the options and arguments are correct
if args.input and args.embedfile and args.output:
    print('Starting Hiding ...')
else:
    # Provide an usage message if the input is incorrect
    parser.print_usage()
    exit()

image=Image.open(args.input)
embed=Image.open(args.embedfile).convert('1')
if image.size != embed.size:
    print('The Images are not in the same size!')
    exit()


embed_data=embed.load()
image_data=image.load()
index=0

for j in range(image.height):
    for i in range(image.width):
        flag_pix=int(embed_data[i,j])
        flag_pix &=1
        im_pix=list(image_data[i,j])
        data = ((flag_pix << index%2) | (254-index%2)) 
        if flag_pix ==1 :
            im_pix[index%3]|= data & (1+index%2)
        else:
            im_pix[index%3]&= data
        image_data[i,j]=tuple(im_pix)
        index+=1
image.save(args.output)
```

This is a golang version of what the binary can do :
 
```go
package main

import (
	"fmt"
	"image"
	"image/color"
	"image/png"
	"os"
	"flag"
)

func main() {

	// Create a new FlagSet to parse command line arguments
	flagSet := flag.NewFlagSet("steganography", flag.ExitOnError)

	// Define command line options and arguments
	inputFileName := flagSet.String("i", "", "select cover-image")
	embedFileName := flagSet.String("e", "", "select image to be embedded")
	outputFileName := flagSet.String("o", "", "The output")

	// Parse the command line arguments
	flagSet.Parse(os.Args[1:])

	// Check if the options and arguments are correct
	if *inputFileName == "" || *embedFileName == "" || *outputFileName == "" {
		flagSet.Usage()
		os.Exit(1)
	}



	// Open the input file
	inputFile1, err := os.Open(*embedFileName)
	if err != nil {
		panic(err)
	}
	defer inputFile1.Close()

	// Decode the PNG image
	inputImage1, _, err := image.Decode(inputFile1)
	if err != nil {
		panic(err)
	}

	// Check if the image is grayscale
	grayImage, ok := inputImage1.(*image.Gray)
	if !ok {
		fmt.Println("Error: input image is not grayscale")
		return
	}

	// Get the width and height of the image
	embed_width := grayImage.Bounds().Size().X
	embed_height := grayImage.Bounds().Size().Y

	// Open the input file
	inputFile, err := os.Open(*inputFileName)
	if err != nil {
		panic(err)
	}
	defer inputFile.Close()

	// Decode the image as a generic Image interface value
	inputImage, _, err := image.Decode(inputFile)
	if err != nil {
		panic(err)
	}

	// Assert the image to an *image.RGBA or *image.NRGBA value
	rgba, ok := inputImage.(*image.RGBA)
	if !ok {
		nrgba, ok := inputImage.(*image.NRGBA)
		if !ok {
			panic("Input image format not supported")
		}
		rgba = image.NewRGBA(nrgba.Bounds())
	}

	// Get the width and height of the image
	bounds := rgba.Bounds()
	width := bounds.Size().X
	height := bounds.Size().Y
	index:=0

	if width!=embed_width || height!=embed_height {
		panic("The Images are not in the same size!")

	}
	// Invert the colors of the image by subtracting each RGB component from 255
	for y := 0; y < height; y++ {
		for x := 0; x < width; x++ {
			intensity := grayImage.GrayAt(x, y).Y & 1
			i:=uint8(index%2)
			data:= ((intensity << i) | (254-i))
			r, g, b, a := rgba.At(x, y).RGBA()
			im_pix:= [3]uint8{uint8(r),uint8(g),uint8(b)}
			j:= uint(index%3)
			if intensity == 1 {
				im_pix[j] = im_pix[j] | (data & (1+i))
			} else {
				im_pix[j] = im_pix[j] & data
			}
			newColor := color.RGBA{
				R: uint8(im_pix[0]),
				G: uint8(im_pix[1]),
				B: uint8(im_pix[2]),
				A: uint8(a),
			}
			rgba.SetRGBA(x, y, newColor)
			index++
		}

	}

	// Create the output file
	outputFile, err := os.Create(*outputFileName)
	if err != nil {
		panic(err)
	}
	defer outputFile.Close()

	// Encode the output image as PNG
	if err := png.Encode(outputFile, rgba); err != nil {
		panic(err)
	}

	fmt.Println("Output image saved successfully")
}
```

As we see here this script try to hide image inside another image in a specific way. We need to understand how he hide data. 

The tool take a black-white image that have only 2 values in his pixels and try to inject this pixels in the LSB of RGB channels of the other image! 
But be carefull is not a traditional or simple LSB steganoraphy here! In each iteration it take a single channel. For example the 1st pixel will use the Red value, 2nd will use the Green value, 3rd will use the Blue and so on. 

We need to reverse this algorithm and write a script that can extract from images used by this tool.

Let's use this one! 

```python
from PIL import Image

INPUT_FILFE="input.png"
EXTRACTED_DATA="extracted.png"
image=Image.open(INPUT_FILFE)
image_data = image.load()
extracted = Image.new('1', (image.width,image.height), 1)
index=0
output_data=extracted.load()
for j in range(image.height):
    for i in range(image.width):  
        im_pix=list(image_data[i,j])
        flag_pix = (im_pix[index%3]&(1+index%2)) 
        if flag_pix >= 1:
            flag_pix=255
        output_data[i,j]=flag_pix
        index+=1
extracted.save(EXTRACTED_DATA)
```
To make sure that everything is ok now! We can use the given tool and hide a black-white image and try to extract it with our new script.

We assume that everything is fine. Let's take a look now how can we extract data from the video!

![](https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/video-gif.gif)

Let's extract every frame from this video and try to extract a data from it using our last script. Maybe we will find something here!

### Extract all the frames using ffmpeg
After running this command. we get 383 frames.
```bash
raf@Securinets:~$ mkdir frames
raf@Securinets:~$ ffmpeg -i challenge.mp4 frames/%3d.png
```
### Extract data from each frame using our solver

Let's use our solver to try extracting data from each frame.
```bash 
raf@Securinets:~$ mkdir data
```
We will put our solver inside a function and use a for loop to run it on each frame 
```python

def extract(frame,output):
    image=Image.open(frame)
    image_data = image.load()
    extracted = Image.new('1', (image.width,image.height), 1)
    index=0
    output_data=extracted.load()
    for j in range(image.height):
        for i in range(image.width):  
            im_pix=list(image_data[i,j])
            flag_pix = (im_pix[index%3]&(1+index%2)) 
            if flag_pix >= 1:
                flag_pix=255
            output_data[i,j]=flag_pix
            index+=1
    extracted.save(output)

for i in range(383):
    index="0"*(3-len(str(i+1)))+str(i+1)+".png"
    extract("frames/"+index,"data/"+index)
```

Let's now check the result and what we get!
Of couse we will get 383 images. But is all the images are usefull or not? Let's check.

We find this image extracted from the 1st frame of the image!

![](https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/000.png)

Bingo! We are on the right way! The author told us to check the last images! Yeah we will but let's check what we got in all the images!

We found a big number of image that have a black line in random position of the image!

<p float="left">
  <img src="https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/015.png" width="450" style="margin-right:50px;" />

  <img src="https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/210.png" width="450" /> 
</p>

Ok Let's focus on the last images as the author said. This is will be better for us. Wait what?! We found a 2 pastebin links and 2 images putted as hint for us. And of couse a rabbit hole(Fake Flag)

<p float="center">
  <img src="https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/372.png" width="320"style="margin-right:10px;" />
  <img src="https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/377.png" width="320"style="margin-right:10px;" /> 
  <img src="https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/381.png" width="320"style="margin-right:10px;" /> 
</p>

Let's check the content of the 2 pastebin links and see what we have! 

```txt
Advertising flash, it's time to relax and rest.
This pastebin is the solution to your problem 
Those links will Absolutely help you 
 
https://4n6nk8s.tech
https://anas-cherni.me/
https://ironbyte.me/
https://yassine-belkhadem.tech/ ( Don't waste your time with this link)
https://github.com/M0ngi
https://soter14.tech/
 
 
Thi is the FLAG: GG_Tr0ll1ng_1s_MY_G4mE_!
 
https://github.com/Mohamed-Rafraf
https://github.com/adamlahbib
https://www.linkedin.com/in/mohamed-rafraf/
https://www.linkedin.com/in/adamlahbib/
https://www.linkedin.com/in/anascherni/
https://www.linkedin.com/in/yassine-belkhadem-396266204/
https://www.linkedin.com/in/m0ngi/
https://www.linkedin.com/in/mohamed-ali-ouachani-00a452237/
https://www.linkedin.com/in/rania-midaoui-b0163a1bb/
https://www.linkedin.com/in/mohamed-ayadi-5a10621a4/
https://www.linkedin.com/in/yassine-belarbi-853303208/
```

There is another fake flag & links for the blogs of the author and his team members! Yeah It's SOter14 team. Let's Check the other one. Maybe it will help

```txt
6 <--> 29
15 <--> 25
11 <--> 34
10 <--> 24
1 <--> 23
14 <--> 35
2 <--> 19
17 <--> 33
3 <--> 32
8 <--> 27
13 <--> 31
0 <--> 18
7 <--> 20
5 <--> 28
12 <--> 30
```

Absolutely this thing will help but we still didn't know how this text file will help us! Let's understand the explanation putted on the other frames.

![](https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/377.png)

The author told us that he split the image to many slices and put each slice in an empty unique image!!! We get it! The random lines are tiny pieces of unique image!

But how can we collect this slice and re-order them to get the original? The lines are ordered randomly? Let's think.

### collect the pieces and recover the orginal image

The solution is simple. That's assume that the white color have a `0 value` and the black color have `1 value`. We can sum the pixels of all the images with this concept to recover the image, right? 

> We know that our image size is 720x720 (The resolution of our video) and from our extracted images we know that we have 360 images that splitted to slices. which mean that every line present 2px (width) from the original image.

Take a look at this script then!

```python
from PIL import Image

img=Image.new('1',(720,720),255)
imgpixs=img.load()
for n in range(1,360):
    name="0"*(3-len(str(n+1)))+str(n+1)+".png"
    image=Image.open("data/"+name)
    image_pixs=image.load()
    print("Image Number : ",n)
    for i in range(720):
        for j in range(720):
            if image_pixs[i,j]!=255:
                imgpixs[i,j]=image_pixs[i,j]

img.save("Recovered.png")
```
In this script we collect all the black pixels and ignore the white pixels. Because we generate a white image and we will fill it with the black color. Let's recover the image now!!

Bingo! There is a progress, we got this image!

![](https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/recoverd.png)

Oh god! It's a broken QR code. How can we deal with this thing. Oh yeah! We have another clue! The author left this hint for us! 

![](https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/381.png)

Oh cool! The author split the QRCode to small square and shuffle them in random order! We need to recover them. The squares are numeroted from 0 to n-1 pieces. This order will help us. Now We get the utility of the pastebin link!

```txt
6 <--> 29
15 <--> 25
11 <--> 34
10 <--> 24
1 <--> 23
14 <--> 35
2 <--> 19
17 <--> 33
3 <--> 32
8 <--> 27
13 <--> 31
0 <--> 18
7 <--> 20
5 <--> 28
12 <--> 3
```
This is will help us how re-order the QRCode. Let's take a deep look now the the broken QRCode!.

The QRCode is splitted by 6 in width and 6 in height. So each piece's size is 120x120. And we have 36 pieces. It's impossible to recover the image without the help of pastebin link. Let's take this line for example `0 <--> 18`, this link told us that the author switch the 1st piece with the 19th piece. (We Start counting from 0 due to the explanation). So we need to write a script that can order this QRCode! We need to recover it!

Let's put the content of the pastebin inside a textfile named `help.txt` and start write our solver.
```python
from PIL import Image

W_SPLIT=6
H_SPLIT=6

ASSEMBLED_IMAGE="Assembled.png"
SPLITTED_IMAGE="Recovered.png"
image= Image.open(SPLITTED_IMAGE)
width, height = image.size
pixels = image.load()

def change_chunk(index1,index2):
    pos1=[]
    pos1.append(index1 % W_SPLIT)
    pos1.append(index1 // W_SPLIT)
    pos2=[]
    pos2.append(index2 % W_SPLIT)
    pos2.append(index2 // W_SPLIT)
    height_chunk=height // H_SPLIT
    width_chunk= width // W_SPLIT
    for i in range (width_chunk):
        for j in range (height_chunk):
            aux= pixels[pos1[0]*width_chunk+i,pos1[1]*height_chunk+j]
            pixels[pos1[0]*width_chunk+i,pos1[1]*height_chunk+j]= pixels[pos2[0]*width_chunk+i,pos2[1]*height_chunk+j]
            pixels[pos2[0]*width_chunk+i,pos2[1]*height_chunk+j]= aux

data=open("help.txt","r").read().split("\n")
for i in range(len(data)-1):
    position=data[i].split("<-->")
    print(position)
    change_chunk(int(position[0]),int(position[1]))

image.save(ASSEMBLED_IMAGE)
```
Yeah It works fine!! We recover the QRCode! 

![](https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/Assembled.png)

But something is wrong here! It sills broken and We can't scan it :(. But why? are we have any problem with our solver?

Ah No. I get it! We have 36 pieces and each line from `help.txt` have correspond to 2 pieces! So we have to find 18 lines, right ? 

```bash 
raf@Securinets:~$ wc -l help.txt
15
```
Oh no we have 15 lines, So we have 6 pieces we need to re-order them by ourselves! Now I get why the Author said Help `LITTLE BIIT`. He left the 6 pieces to deal with it.

In this case we need to do a bruteforce to all the combinasions. We have 6 pieces, so we have 6! possibility
and 6! = 720 which is possible to deal with it using a python script. Let's re-order the 6 pieces each time and try to scan the QRCode. Once the QRCode is readable, we will stop our program!

Let's get the flag now !!! Let's DO IT NOW! We're so close!

So the 1st thing that we need to do it is that we need to get the index of missed pieces. The `help.txt` is the key!

Let's generate an array that have the order (index) of the missed pieces. Then we use our `change_chunck()` method and try all the combinaisons using the `itertools`. Finally try to scan the QRCode each time using the `qrtools`

Let's install the qrtools first 

```bash 
raf@Securinets:~$ sudo apt-get install python3-qrtools

```
Now let's write down our last script (I Hope) :
```python

from PIL import Image
import qrtools
from itertools import permutations

qr=qrtools.QR()

def missed_data():
    not_missed=[]
    data=open("help.txt","r").read().split("\n")
    for i in range(len(data)-1):
        position=data[i].split("<-->")
        not_missed.append(int(position[0]))
        not_missed.append(int(position[1]))
    print(not_missed)
    missed=[]
    for i in range(0,36):
        if not (i in not_missed):
            missed.append(i)
    return missed

chuncks=missed_data()
print(chuncks)
H_SPLIT=6
W_SPLIT=6
SPLITTED_IMAGE="Assembled.png"
height,width=720,720

permutations=list(permutations(chuncks,6))
image=Image.open("Assembled.png")
pixels=image.load()

def change_chunk(index1,index2):
    pos1=[]
    pos1.append(index1 % W_SPLIT)
    pos1.append(index1 // W_SPLIT)
    pos2=[]
    pos2.append(index2 % W_SPLIT)
    pos2.append(index2 // W_SPLIT)
    height_chunk=height // H_SPLIT
    width_chunk= width // W_SPLIT
    for i in range (width_chunk):
        for j in range (height_chunk):
            aux= pixels[pos1[0]*width_chunk+i,pos1[1]*height_chunk+j]
            pixels[pos1[0]*width_chunk+i,pos1[1]*height_chunk+j]= pixels[pos2[0]*width_chunk+i,pos2[1]*height_chunk+j]
            pixels[pos2[0]*width_chunk+i,pos2[1]*height_chunk+j]= aux

print('[-] Trying to fix the QRCode and Getting the flag ...')

for i in range(len(permutations)):
    comb=list(permutations[i])
    change_chunk(comb[0],comb[1])
    change_chunk(comb[2],comb[3])
    change_chunk(comb[4],comb[5])
    image.save("Final.png")
    if qr.decode("Final.png"):
        print(qr.data)
        break
```

And we got the flag as an output Here!!! BINGO! We get the flag! 

![](https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/finally.png)

And of course the QRCode is saved! And this is our QRCode!

![](https://github.com/4n6nk8s/quals-2023-write-ups/raw/main/Raf-Hide/Final.png)

> The Final flag is : Securinets{Every_Fr4Me_H4S_HiS_own_S7ory}


# Couch Potato

![](https://i.imgur.com/jHut52F.png)

It's past one, I was probably sleeping in front of my screen, I'm not even sure if I was awake for a bit or not, I'm not even sure if I'm dreaming or not... All I know is that I left the device on my favorite channel and that piece of crap failed me. Agh I need to fix it now. It is no longer responding to the commands I give it, and it is no longer showing my favorite channel! I hate my life, can you help me fix it?

I am looking for three stuff:

- _What nonsense was I watching the night the device failed me?_
- _I might need the Up and Down key codes in NEC 32-bit format?_
- _The expiration date of my IPTV subscription._

Flag formal: **Securinets{show_name_00UP1234_DOWN1234_YYYYMMDD}**

[adm & mida0ui](https://twitter.com/admida0ui)

## Attachment

A file called `dump.bin` was provided in a zip. Weighing 8 MB. Dating back to February the 1st, 2020. 1:07 AM.

Another file is what seems to be an updated firmware for the device, `Firmware.bin` weighing 4 MB. And dating back to August the 25th, 2020.

[Download Link](https://drive.google.com/drive/folders/1TGNAOWKZZbCzZuVImklA18IH_cUvtkNy?usp=sharing)

## Analysis

A Couch potato is a lazy guy that sits in front of the TV screen, indicating that we are dealing with a console, set-top-box STB or Digital Video Recorder DVR and not a PCI card satellite/Tuner card or a computer. The user mentioned that he was watching his favorite channel when he fall asleep which make it more likely an STB, or a receiver for simplicity.
This also means that the show name we are looking for was broadcasted in his favorite channel.

The user also mentioned that the device failed him, which means that the device is not responding to the commands he is sending to it. Kind of talking about a control unit, or that's clearly a remote control that has some keys malfunctioning, specifically the UP and DOWN arrow keys as we know.

On other hand, the user is suspecting that the IPTV subscription was the thing behind the interruption of his channel. This means that the IPTV subscription was a service provided by the device itself or maybe a third party service. This also means that the STB we're dealing with is a hybrid device, meaning that it can receive satellite signals and also IPTV, making it a 2nd generation STB and there are plenty in the market today, for third world countries.

However, the user could be mistaken as the channels that you can shortlist in the favorites are more likely to be satellite channels, hence the IPTV subscription is not the reason behind the interruption of his favorite channel rather than the satellite signal itself or the cardsharing service he is using.

With all that in mind, we can start our information gathering phase.

## Information Gathering

TL;DR.

The thing is devices like this are hard to find in UK, Europe and the US. Unlike Africa and Asia. That's because in European countries and the US, they are mostly banned due to the fact that they are used for cardsharing and IPTV piracy. However, in Africa and Asia, they are still widely used and sold.

So where to look? These devices are not open source nor they have some kind of documentation is to where to look in their ROMs. Satellite forums, subreddits, and Facebook groups are good resources and some deep search would yield some useful tools and details to deal with dump. We kind of need to know how the memory is splitted. The file is indeed a ROM/EEPROM snapshot, so it has a mapping of the memory. We just need to know where the information part is, channel list is, and applications or services are.

The mapping generally includes the following:

- Bootloader
- Kernel or maincode
- User data
- Menu
- ...

Each at specific offsets and specific lengths. The user data is the most important part as it contains the channel list.

Keep in mind that if the IPTV subscription was provided by the device itself, then the expiration date could be stored in the device itself or their renewal website. If the IPTV subscription was provided by a third party service, then the expiration date is stored in the third party service database or user panel. In both cases, we need to find the device serial number or MAC address to be able to identify the device and the user. I believe that how they are likely to be stored in the memory dump rather than a plain expiration date.

## Resources

- [https://www.satellites.co.uk/forums/](https://www.satellites.co.uk/forums/)
- [https://www.reddit.com/r/satellite/](https://www.reddit.com/r/satellite/)
- [https://www.tunisia-sat.com/forums/](https://www.tunisia-sat.com/forums/)
- [https://sat-universe.com/](https://sat-universe.com/)

## Tools

HexWorkshop / HxD Editor is a good tool to start with. It is a hex editor that can be used to view and edit files in hexadecimal and binary formats. It is a good tool to start with as it can be used to view the memory dump and search for strings. It can also be used to search for patterns and hex values.

## Writeup

First, let's investigate the dump file.

`NCRCBootloader` is the start point, which is the bootloader used for these chipsets as indicated in the Hex Editor screen:

- ALI3329
- ALI3606
- ALI3601
- ALI3511
- ALI3510
- ALI3516
- ALI3618
- ALI3821

![](https://i.imgur.com/vSB9IGX.png)

Devices with those chipset have these brands _Starsat, Sunplus, Tiger, StarMax, Geant, Mediastar, QMAX, AzAmerica, Samsat and many more..._

And all of their built-in subscriptions are part of **Gosat**.

Now, to actually find the specific brand and model, we need to inspect the firmware update file.

To clear things up, the memory dump serves for user data mainly the channels and services as I said while the firmware update file will indicate the remote control being used.

The memory dump is a mapping and can be splitted manually and further investigated. However the firmware is encrypted, you can tell by a first glance, no strings or something in there, and can't help us much to delve into the remote control unit in use.

Time to google for a way to decrypt such chipset firmware. Let's use the keyword `ALI3329` or `ALI3511` as they are the most common chipset used in these devices and combine the search with decrypt or unpack tool.

For example:

![](https://i.imgur.com/DfscXB4.png)

And in Arabic:

![](https://i.imgur.com/rdG9k09.png)

Using the tool, we determine that the model is SR-2000HD HYPER and the remote control is SR-2000HD HYPER. Hence the brand is Starsat.

![](https://i.imgur.com/uWfBYaJ.png)

The unpack and repack features are used to decrypt and extract parts of the firmware like the bootloader, maincode, user data if any was supplied by the manufacturer, the menu, the remote, and sometimes a softcam (that's out of our scope today, maybe in another challenge!). And the repack to insert modified parts and RSA encrypt the whole thing again. Like for instance, we can insert the main menu (including themes and applications) or the remote control unit of a different model or brand and repack it to be used with the device we have, as long as they are using the very same chipset. Well this time, rest assured it is the Hyper remote control and when we talk about key code we mean the IR Infrared key code.

So the whole memory dump thing and the firmware kind of contain similar stuff at least, one being encrypted and the other not.

See here, the content of the folder when unpacked.

![](https://i.imgur.com/P8IBsRt.png)

So why not, giving the dump a try and unpack it as well, we are chasing the user data part anyway. And the tool might very much help. Otherwise I will show how to proceed manually knowing the offsets and lengths found on a forum.

### Finding the show

As we said before, we can deal with this part either using the tool or manually.
Well, the tool was able to yield this:

![](https://i.imgur.com/f1fon6N.png)

Very nice, we can see `database.sdx` file there!

Well, to proceed manullay you need to know the offsets and lengths beforehand.
This is what we meant:

```
++++++++++++++++++++++++++++++++++++++
Name : Bootloader.bin
SIZE : 128,00 KB
CRC : 0x00AF7F6C
offset : 0x00000128
++++++++++++++++++++++++++++++++++++++
++++++++++++++++++++++++++++++++++++++
Name : maincode.bin
SIZE : 3,25 MB
CRC : 0x19E8C348
offset : 0x00020128
++++++++++++++++++++++++++++++++++++++
++++++++++++++++++++++++++++++++++++++
Name : menu.bin
SIZE : 1,38 MB
CRC : 0x10934143
offset : 0x00360128
++++++++++++++++++++++++++++++++++++++
++++++++++++++++++++++++++++++++++++++
Name : Database.sdx
SIZE : 9,80 KB
CRC : 0x00146C71
offset : 0x004C0128
++++++++++++++++++++++++++++++++++++++
++++++++++++++++++++++++++++++++++++++
Name : softcam.bin
SIZE : 42,38 KB
CRC : 0x008D5F15
offset : 0x004C2860
++++++++++++++++++++++++++++++++++++++
++++++++++++++++++++++++++++++++++++++
Name : sattp.bin
SIZE : 12,54 KB
CRC : 0x00097FDA
offset : 0x004CD1E8
++++++++++++++++++++++++++++++++++++++
++++++++++++++++++++++++++++++++++++++
Name : Padding.bin
SIZE : 522,06 KB
CRC : 0x040E1F6B
offset : 0x004D040C
++++++++++++++++++++++++++++++++++++++
```

Found here: [https://www.tunisia-sat.com/forums/threads/3242761/page-47](https://www.tunisia-sat.com/forums/threads/3242761/page-47)

We know `database.sdx` holds the user data meaning the channels. sattp is another file that holds the different satellites and their frequencies aka transponders.

When we open the file in a hex editor, We can't read any useful information out of it. This means that there should be a tool in place to view and edit the channels for the specific chipset family we are dealing with as the file seems highly compressed.

Here's a view from hex editor first.

Starts off like this

![](https://i.imgur.com/7FHWF6K.png)

And ends padded with 0xFF

![](https://i.imgur.com/xqvWPxc.png)

Let's google a bit...

![](https://i.imgur.com/P5bXWY4.png)

We got this

![](https://i.imgur.com/hVzUWC9.png)

When trying to first open the database, we got this error message

![](https://i.imgur.com/ASWzJB9.png)

And the issue is that the Userdata has a static size as indicated before. As the receiver's capacity fits a maximum of 6100 channels. So the database file is padded with 0xFF to reach the maximum size. Let's get rid of the padded data and try again..

![](https://i.imgur.com/NUIk82u.png)

Now, it's working like charm, and I already see some familiar TV channels on Astra 1 (19.2 East).

Let's expand the favorites section

![](https://i.imgur.com/AmkpQPE.png)

And there is Sky Sports Main Event, UK-based sports channel that is part of BSkyB or Sky Group, pay-television channel and availble for satellite subsribers via Eurobird 1, Astra 2 (28.2 East). Sky Sports Main Event broadcasts the biggest events of sports in the UK.

Now, great work to reach this point, but we are not done yet. We need to find the right show at that past date, meaning at 1:07 AM on February the 1st, 2020.

Well, Sky TV Guide won't keep such data for more than a week or two, so we need to find another way.

Guess you know what we mean, what else than the way back machine!

Head over to archive.org and submit the link for the Sky TV Guide and try to narrow your search to a date that's equal or close to the date in question, as shown below!

![](https://i.imgur.com/B5scqzJ.png)

And that is the first part of the flag: `live_rugby_7's` or `live_rugby_7s`

### Finding the keycodes

We know it is Starsat SR-2000HD Hyper's remote control, well unless you have the same remote control or probably a remote of the same brand and an Arduino card embedded with an IR receiver, you will need to improvise! Like check online there GitHub repos that keep track of IR key codes for different remotes. Or you can use a forum like [https://www.remotecentral.com/](https://www.remotecentral.com/) to find the key codes for the remote control.

However for this part, we will use an Android app like IRplus or any alternatives and pick the device in question from the list then head over to check the keys, we will provide screenshots for what we've found. Since it is the simplest and fastest way.

This is the LIRC protcol of Starsat https://lirc.sourceforge.net/remotes/starsat/120

We are looking for NEC, so we found an app for that.

[net.binarymode.android.irplus](https://irplus-remote.github.io/)

1. Selecting the remote control from the list:

![](https://i.imgur.com/3hjuMYS.png)

2. Exporting the remote config and checking the Channel Up arrow and Channel Down arrow

![](https://i.imgur.com/lsXXO6K.png)

And that is the second part of the flag: `00ff30cf_00ff8877`

we can make sure that is the correct format using a website like https://www.yamaha.com/ypab/irhex_converter.asp
converting NEC 32-bit to Pronto hex format (RAW HEX) and then to global caché to see the full sendIR command...

### Finding the expiration date

Now for the fun part, if you dig around the web about Starsat 2000 Hyper you will end up with one official built in service, Apollo that offers IPTV as well as VOD Video on Demand.

And there is an online service to renew and check your current subscription by just providing the serial number.

[www.renewbox.net](http://www.renewbox.net/index.php)

The serial number in most cases is a long number like consists of maybe more than 10 digits. And it can't be in the firmware since it is released to the public to update their devices. So it must be within the memory of the device itself.

Let's grep that easily!

![](https://i.imgur.com/XmWhATI.png)

And there it is, the serial number, let's query the IPTV validity.

![](https://i.imgur.com/lTGDuFS.png)

And the last part of the flag is `20141105`

## Final Words

Flag: **Securinets{live_rugby_7s_00ff30cf_00ff8877_20141105}**

GGs **th3_r0n1ns** for solving this challenge