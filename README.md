# kubeadm

Containerd konfigurasyonunu oluşturalım,

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

Gerekli olan modülleri yükleyelim,

sudo modprobe overlay
sudo modprobe br_netfilter

Kubernetes networking için gerkeli olan network konfigurasyonlarını uygulayalım,

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

Uygulanan konfigurasyonları devreye alalım,

sudo sysctl --system

Containerd kurulumunu gerçekleştirelim,

sudo apt-get update && sudo apt-get install -y containerd

Containerd için default konfigurasyon dizinini oluşturalım,

sudo mkdir -p /etc/containerd

Containerd varsayılan konfigurasyonlarını oluşturup, belirtilen konfig dizinine konfigurasyonu kaydedelim,

sudo containerd config default | sudo tee /etc/containerd/config.toml

Mevcut konfig dosyasını oluşturmuş olduğumuz konfigurasyon alanına kayıt ettiğimizden emin olduktan sonra containerd servisini yeniden başlatalım,

sudo systemctl restart containerd

Servisin çalıştığından emin olalım,

sudo systemctl status containerd

Swapı devre dışı bırakalım,

sudo swapoff -a

Reboot sonrası her açılışta swap devreye girmemesi için fstab üzerinden de swapı devre dışı bırakalım,

sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

Gerekli olan paketlerin kurulumlarını gerçekleştirelim,

sudo apt-get update && sudo apt-get install -y apt-transport-https curl

GPG Keyimizi indirelim ve keylerimiz arasına ekleyelim,

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

Kubernetes repository listimizi oluşturalım,

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

Paket listemizi güncelleyelim,

sudo apt-get update

ve Kubernetes paketlerimizin kurulumuna başlayalım,

sudo apt-get install -y kubelet=1.22.0-00 kubeadm=1.22.0-00 kubectl=1.22.0-00

kubelet, kubeadm ve kubctl paketlerimizin auto update almalarını kapatalım,

sudo apt-mark hold kubelet kubeadm kubectl

Cluster Kurulumu

kubeadm’yi kullanarak control-plane(master) Kubernetes cluster başlatın (Not: Bu yalnızca Control Plane üzerinde gerçekleştirilir):

sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.22.0

Kubectl için erişimi yapılandıralım,

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

ve Test girişimizi aşağıdaki komut ile gerçekleştirelim,

kubectl get nodes

Calico Networking eklentisi kurulumu

Bu eklenti sadece control-plane (master) üzerinde gerçekleştirilir,

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

Control plane durumları kontrol edilir,

kubectl get nodes

Worker Node’ların Cluster’a dahil edilmesi

kubeadm token create --print-join-command
İki worker nodumuzda da ilgili çıktıyı sudo komutu ile uygulayalım,

sudo kubeadm join ...
