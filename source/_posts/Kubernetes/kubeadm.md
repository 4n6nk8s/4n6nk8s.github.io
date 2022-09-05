---
title: Setting Up Kubernetes with Kubeadm
date: 2022-08-05 11:54:12
tags:
cover: https://i.imgur.com/bhUlMFS.png
categories:
- [Kubernetes]

---
[Kubeadm]{.red} is a tool used to build Kubernetes (K8s) clusters. Kubeadm performs the actions necessary to get a minimum viable cluster up and running quickly. By design, it cares only about bootstrapping, not about provisioning machines (underlying worker and master nodes).

:::success
Knowing how to use kubeadm is required for CKA and CKS exams
:::

We configure a 3 [Ubuntu 20.04 LTS]{.pinky} machines in the same network with the following proprietes:
| Role   | Hostname         | IP address      | 
| -------| ---------------- | --------------- |
| Master | 4n6nk8s-master   | 192.168.1.18/24 | 
| Worker | 4n6nk8s-worker1  | 192.168.1.19/24 | 
| Worker | 4n6nk8s-worker2  | 192.168.1.20/24 | 

> Note: Make sure to setup a unique hostname for each host

## Prepare the environments
The following Steps must be applied to each node (both master nodes and worker nodes)
### Disable the Swap Memory
The Kubernetes requires that you disable the swap memory in the host system because the kubernetes scheduler determines the best available node on which to deploy newly created pods. If memory swapping is allowed to occur on a host system, this can lead to performance and stability issues within Kubernetes

You can disable the swap memory by deleting or commenting the swap entry in `/etc/fstab` manually or using the `sed` command

```bash command line prompt
4n6nk8s@4n6nk8s-master$ sudo swapoff -a && sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

This command disbales the swap memory and comments out the swap entry in `/etc/fstab` 

### Configure or Disable the firewall
When running Kubernetes in an environment with strict network boundaries, such as on-premises datacenter with physical network firewalls or Virtual Networks in Public Cloud, it is useful to be aware of the ports and protocols used by Kubernetes components.

The ports used by Master Node:

| Protocol  | Direction     | Port Range    |  Purpose 
| -------   | ------------- | ------------- | -------
| TCP       | Inbound       | 6443          | Kubernetes API server
| TCP       | Inbound       | 2379-2380     | etcd server client API
| TCP       | Inbound       | 10250         | Kubelet API
| TCP       | Inbound       | 10259         | kube-scheduler
| TCP       | Inbound       | 10257         | kube-controller-manager

The ports used by Worker Nodes: 

| Protocol  | Direction     | Port Range    |  Purpose 
| -------   | ------------- | ------------- | -------
| TCP       | Inbound       | 10250         | Kubelet API
| TCP       | Inbound       | 30000-32767   | NodePort Services


You can either disable the firewall or allow the ports on each node.
#### Method 1: Add firewall rules to allow the ports used by the Kubernetes nodes
Allow the ports used by the master node:
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ sudo ufw allow 6443/tcp
4n6nk8s@4n6nk8s-master:~$ sudo ufw allow 2379:2380/tcp
4n6nk8s@4n6nk8s-master:~$ sudo ufw allow 10250/tcp
4n6nk8s@4n6nk8s-master:~$ sudo ufw allow 10259/tcp
4n6nk8s@4n6nk8s-master:~$ sudo ufw allow 10257/tcp
```
Allow the ports used by the worker nodes:
```bash command line prompt
4n6nk8s@4n6nk8s-worker1:~$ sudo ufw allow 10250/tcp
4n6nk8s@4n6nk8s-worker1:~$ sudo ufw allow 30000:32767/tcp
```
#### Method 2: Disable the firewall
```bash command line prompt command
4n6nk8s@4n6nk8s-master:~$ sudo ufw status
Status: active

4n6nk8s@4n6nk8s-master:~$ sudo ufw disable
Firewall stopped and disabled on system startup

4n6nk8s@4n6nk8s-master:~$ sudo ufw status
Status: inactive
```

### Installing Docker Engine
Kubernetes requires you to install a container runtime to work correctly.There are many available options like containerd, CRI-O, Docker etc

By default, Kubernetes uses the Container Runtime Interface (CRI) to interface with your chosen container runtime.If you don't specify a runtime, kubeadm automatically tries to detect an installed container runtime by scanning through a list of known endpoints.

You must install the Docker Engine on each node! 

#### 1- Set up the repository 
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ sudo apt update
4n6nk8s@4n6nk8s-master:~$ sudo apt install ca-certificates curl gnupg lsb-release
```
#### 2- Add Docker's official GPG key
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ sudo mkdir -p /etc/apt/keyrings
4n6nk8s@4n6nk8s-master:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
#### 3- Add the stable repository using the following command:
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
#### 4- Install the docker container
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ sudo apt update && sudo apt install docker-ce docker-ce-cli containerd.io -y
``` 

#### 5- Make sure that the docker will work on system startup
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ sudo systemctl enable --now docker 
```
#### 6- Configuring Cgroup Driver:  
The Cgroup Driver must be configured to let the kubelet process work correctly
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```
#### 7- Restart the docker service to make sure the new configuration is applied
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ sudo systemctl daemon-reload && sudo systemctl restart docker
```
### Installing kubernetes (kubeadm, kubelet, and kubectl):

Install the following dependency required by Kubernetes on each node
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ sudo apt install apt-transport-https
```
#### Download the Google Cloud public signing key:
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
``` 
#### Add the Kubernetes apt repository:
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
#### Update the apt package index and install kubeadm, kubelet, and kubeclt
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ sudo apt update && sudo apt install -y kubelet=1.23.1-00 kubectl=1.23.1-00 kubeadm=1.23.1-00
```

## Initializing the control-plane node
At this point, we have 3 nodes with docker, `kubeadm`, `kubelet`, and `kubectl` installed. Now we must initialize the Kubernetes master, which will manage the whole cluster and the pods running within the cluster `kubeadm init` by specifiy the address of the master node and the ipv4 address pool of the pods 

```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ sudo kubeadm init --apiserver-advertise-address=192.168.1.18 --pod-network-cidr=10.1.0.0/16
```
You should wait a few minutes until the initialization is completed. The first initialization will take a lot of time if your connexion speed is slow (pull the images of the cluster components)

### Configuring kubectl 
As known, the `kubectl` is a command line tool for performing actions on your cluster. So we must to configure `kubectl`. Run the following command from your master node:
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ mkdir -p $HOME/.kube
4n6nk8s@4n6nk8s-master:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
4n6nk8s@4n6nk8s-master:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### Installing Calico CNI 
Calico provides network and network security solutions for containers. Calico is best known for its performance, flexibility and power. Use-cases: Calico can be used within a lot of Kubernetes platforms (kops, Kubespray, docker enterprise, etc.) to block or allow traffic between pods, namespaces

#### 1- Install Tigera Calico operator
```bash command line prompt 
4n6nk8s@4n6nk8s-master:~$ kubectl create -f "https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml"
```
The Tigera Operator is a Kubernetes operator which manages the lifecycle of a Calico or Calico Enterprise installation on Kubernetes. Its goal is to make installation, upgrades, and ongoing lifecycle management of Calico and Calico Enterprise as simple and reliable as possible.

#### 2- Download the custom-resources.yaml manifest and change it 
The Calico has a default pod's CIDR value. But in our example, we set the  `--pod-netwokr-cidr=10.1.0.0/16`. So we must change the value of pod network CIDR in `custom-resources.yaml`

```bash command line prompt 
4n6nk8s@4n6nk8s-master:~$ wget  "https://projectcalico.docs.tigera.io/manifests/custom-resources.yaml"
```
Now we edit this file before create the Calico pods

```yml

# This section includes base Calico installation configuration.
# For more information, see: https://projectcalico.docs.tigera.io/v3.23/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    # Note: The ipPools section cannot be modified post-install.
    ipPools:
    - blockSize: 26
      cidr: 10.1.0.0/16 #change this value with yours
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()

---

# This section configures the Calico API server.
# For more information, see: https://projectcalico.docs.tigera.io/v3.23/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer 
metadata: 
  name: default 
spec: {}
```
After Editing the `custom-resources.yaml` file. Run the following command:
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ kubectl create -f "custom-resources.yaml" 
```
Before you can use the cluster, you must wait for the pods required by Calico to be downloaded. You must wait until you find all the pods running and ready! 
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ kubectl get pods --all-namespaces
NAMESPACE          NAME                                       READY   STATUS    RESTARTS       AGE
calico-apiserver   calico-apiserver-5989576d6d-5nw7n          1/1     Running   1 (4min ago)    4min
calico-apiserver   calico-apiserver-5989576d6d-h677h          1/1     Running   1 (4min ago)    4min
calico-system      calico-kube-controllers-69cfd64db4-9hnh5   1/1     Running   1 (4min ago)    4min
calico-system      calico-node-lshdl                          1/1     Running   1 (4min ago)    4min
calico-system      calico-typha-76dd7c96d7-88826              1/1     Running   1 (4min ago)    4min
kube-system        coredns-64897985d-jkpwh                    1/1     Running   1 (4min ago)    4min
kube-system        coredns-64897985d-zk9wx                    1/1     Running   1 (4min ago)    4min
kube-system        etcd-master                                1/1     Running   1 (4min ago)    4min
kube-system        kube-apiserver-master                      1/1     Running   1 (4min ago)    4min
kube-system        kube-controller-manager-master             1/1     Running   1 (4min ago)    4min
kube-system        kube-proxy-4nf4q                           1/1     Running   1 (4min ago)    4min
kube-system        kube-scheduler-master                      1/1     Running   1 (4min ago)    4min
tigera-operator    tigera-operator-7d8c9d4f67-j5b2g           1/1     Running   2 (103s ago)    4min
```

## Join the worker nodes
Now our cluster is ready to work! let's join the worker nodes to this cluster by getting the token from the master node 
```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ sudo kubeadm token create --print-join-command
kubeadm join 192.168.1.18:6443 --token g4mgtb.e8zgs1c0kpkaj9wt --discovery-token-ca-cert-hash sha256:047628de2a0a43127b7c4774093203631d315451874efc6b63421a4da9bee2ec
```
Now let's move to the worker node and run the following command given by `kubeadm token create`

```bash command line prompt 
4n6nk8s@4n6nk8s-worker1:~$ sudo kubeadm join 192.168.1.18:6443 --token g4mgtb.e8zgs1c0kpkaj9wt 
\--discovery-token-ca-cert-hash sha256:047628de2a0a43127b7c4774093203631d315451874efc6b63421a4da9bee2ec
``` 

The output must be similar to the following 
```bash command line prompt
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
W0623 12:45:07.940655   23651 utils.go:69] The recommended value for "resolvConf" in "KubeletConfiguration" is: /run/systemd/resolve/resolv.conf; the provided value is: /run/systemd/resolve/resolv.conf
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

```
Now let's Check the cluster by running `kubectl get nodes` command on the master node.

```bash command line prompt
4n6nk8s@4n6nk8s-master:~$ kubectl get nodes

NAME              STATUS     ROLES                  AGE    VERSION
4n6nk8s-master    Ready      control-plane,master   40m5s  v1.23.1
4n6nk8s-worker1   Ready      <none>                 3m7s   v1.23.1
4n6nk8s-worker2   Ready      <none>                 2m3s   v1.23.1
```




## References:


{% links %}
- site: Kubernetes Documentation
  url: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
  desc: Creating a cluster with kubeadm
  image: https://upload.wikimedia.org/wikipedia/commons/thumb/3/39/Kubernetes_logo_without_workmark.svg/1200px-Kubernetes_logo_without_workmark.svg.png
  color: "#2296fd"
- site: Calico Documentation
  desc: Install Calico Networking for on-premises deployments
  url: https://projectcalico.docs.tigera.io/getting-started/kubernetes/self-managed-onprem/onpremises
  image: https://projectcalico.docs.tigera.io/images/favicon.png
  color: "#FB7031"
- site: Docker Documentation
  url: https://docs.docker.com/engine/install/ubuntu/
  desc: Install Docker Engine on Ubuntu
  image: https://www.docker.com/wp-content/uploads/2022/05/Docker_Temporary_Image_Google_Blue_1080x1080_v1.png
  color: "#31bffb"
{% endlinks %}

