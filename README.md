# Install-Kubernetes-with-adm-on-ubuntu-18-04-linix-machine

### Requirements 
2 GB or more of RAM per machine (any less will leave little room for your apps).  
2 CPUs or more.

### Please do these configurations on your master and workors machines

##### Letting iptables see bridged traffic
As a requirement for your Linux Node's iptables to correctly see bridged traffic, you should ensure net.bridge.bridge-nf-call-iptables is set to 1 in your sysctl config, e.g.  
Do this

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

```
##### Installing runtime
Install Docker runtime 
follow docker docs for installation

https://docs.docker.com/engine/install/ubuntu/

##### Installing kubeadm, kubelet and kubectl 

* Update the apt package index and install packages needed to use the Kubernetes apt repository:  

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

* Download the Google Cloud public signing key:

```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```
* Add the Kubernetes apt repository:
```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
* Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
##### Configuring a cgroup driver
Both the container runtime and the kubelet have a property called "cgroup driver", which is important for the management of cgroups on Linux machines.  


* On each of your nodes, install the Docker for your Linux distribution as per Install Docker Engine. You can find the latest validated version of Docker in this dependencies file.  
  
* Configure the Docker daemon, in particular to use systemd for the management of the container’s cgroups.  

```
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
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
* Restart Docker and enable on boot:
```
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### Please do these configurations on your master machines

To initialize the control-plane node run:  

```
kubeadm init  kubeadm init --apiserver-cert-extra-sans=<virtual machine public ip>  <args>

```
To make kubectl work for your non-root user, run these commands, which are also part of the kubeadm init output:  
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
##### Installing a Pod network add-on

Deploy Calico network  
```
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
```

##### Get Kubeadm join command 
```
kubeadm token create --print-join-command
```


### Please do these configurations on your master machines
On your workor node please run the command that is the output of this command

```
kubeadm token create --print-join-command
```
### Verifying the cluster on master

##### Get Nodes status

```
kubectl get nodes
```



### Access your cluster from your local machine
##### Download Kubernetes Credentials From Remote Cluster

```
scp -i <ssh key> -r ubuntu@<virtual machine pulbic ip>:/home/ubuntu/.kube .
```
##### Copy Kubernetes Credentials To Your Home
```
cp -r .kube $HOME/
```

##### get nodes

```
kubectl get nodes
```

Have Fun !!

