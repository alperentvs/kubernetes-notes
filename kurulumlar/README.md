## Kubernetes İçin Gerekli Kurulumlar

### Kubectl Kurulumu
* Kubernetes'e komut vermek istediğimiz zaman bunu APIserver aracılığı ile yaparız. APIserver ile iletişim kurmanın üç yöntemi vardır
    - RestAPI çağrıları ile iletişim kurabiliriz.
    - Çeşitli GUI araçları ile işlerimizi halledebiliriz.
    - Kubernetes projesinin resmi CLI aracı olan **kubectl** ile shell üzerinden APIserver ile haberleşebiliriz.
* https://kubernetes.io/docs/tasks/tools/ sitesinden adımları takip ederek kurulumları gerçekleştirebilirsiniz.

### Minikube Kurulumu
* Minikube kendi bilgisayarımızda single node bir kubernetes cluster kurmamızı sağlar. 
* Bir kubernetes cluster kurabilmemiz için temelde linux bir makineye ihtiyacımız var. Minikube bunu sistemde bulunan Docker, Hyperkit, Hyper-V, KVM, Parallels, Podman, VirtualBox, or VMware araçlarından herhangi biri üzerinde bir sanal katman yaratarak kubernetesi burada ayağa kaldırıyor. 
* Belirtmiş olduğum sistemlerden herhangi birisinin kurulu olması gerekmektedir. 
* https://minikube.sigs.k8s.io/docs/ adresinden adımları takip ederek kurulumu gerçekleştirebilirsiniz.

### Kubeadm Kurulumu
* Hızlı şekilde ve gerekli olan minimum konfigürasyonla kubernetes cluster kurulumu sağlayan bir araçtır. Birkaç makine ayağa kaldırıp, bunlar üzerinde koşan kubernetes cluster kurmak istediğimizde kullanacağımız araçtır. Kubeadm ile sanal ya da fiziksel makine kurmayız. Bu makineleri kendiniz kurmanız gerekir. Makinelerin kurulumunu yapıp üstüne Linux işletim sistemi yükledikten sonra Kubeadm ile bu makinelerden oluşan bir cluster kurma işini hallederiz. 

* https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/ adresinden adımları takip ederek kurulumu gerçekleştirebilirsiniz.

* **1-** sanal makine oluşturma
```
$ multipass launch --name master -c 2 -m 2G -d 10G
$ multipass launch --name node1 -c 2 -m 2G -d 10G
```

* **2-** Iptables bridged traffic ayarı

```
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
```

```
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

```
$ sudo sysctl --system
```

* **3-** containerd kurulumu

```
$ cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

```
$ sudo modprobe overlay
$ sudo modprobe br_netfilter
```

```
$ cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

```
$ sudo sysctl --system
```

```
$ sudo apt-get update
$ sudo apt-get upgrade -y
$ sudo apt-get install containerd -y
$ sudo mkdir -p /etc/containerd
$ sudo su -
$ containerd config default | tee /etc/containerd/config.toml
$ exit
$ sudo systemctl restart containerd
```

* **4-** kubeadm kurulumu


```
$ sudo apt-get update
$ sudo apt-get install -y apt-transport-https ca-certificates curl
$ sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
$ echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```

* **5-** kubernetes cluster kurulumu

```
$ sudo kubeadm config images pull

$ sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<ip> --control-plane-endpoint=<ip>
```

```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```
$ kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
$ kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
```