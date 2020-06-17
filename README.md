# 3 Virtual Machines Kubernetes cluster

## Dependencies
Install VitualBox and Vagrant on your host system

## Creating the cluster

You should create a `Vagrantfile` in an empty directory with the following content:

```ruby

Vagrant.configure("2") do |config|
  config.vm.provider :virtualbox do |v|
    v.memory = 1024
    v.cpus = 2
  end

  config.vm.provision :shell, privileged: true, inline: $install_common_tools

  config.vm.define :master do |master|
    master.vm.box = "ubuntu/xenial64"
    master.vm.hostname = "master"
    master.vm.network :private_network, ip: "10.0.0.10"
    master.vm.provision :shell, privileged: false, inline: $provision_master_node
  end

 # If more that 2 nodes need to be created, below line need to be updated
  %w{worker1 worker2}.each_with_index do |name, i|
    config.vm.define name do |worker|
      worker.vm.box = "ubuntu/xenial64"
      worker.vm.hostname = name
      worker.vm.network :private_network, ip: "10.0.0.#{i + 11}"
      worker.vm.provision :shell, privileged: false, inline: <<-SHELL
sudo /vagrant/join.sh
echo 'Environment="KUBELET_EXTRA_ARGS=--node-ip=10.0.0.#{i + 11}"' | sudo tee -a /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
sudo systemctl daemon-reload
sudo systemctl restart kubelet
SHELL
    end
  end

  config.vm.provision "shell", inline: $install_multicast
end


$install_common_tools = <<-SCRIPT
# bridged traffic to iptables is enabled for kube-router.
cat >> /etc/ufw/sysctl.conf <<EOF
net/bridge/bridge-nf-call-ip6tables = 1
net/bridge/bridge-nf-call-iptables = 1
net/bridge/bridge-nf-call-arptables = 1
EOF

# disable swap
swapoff -a
sed -i '/swap/d' /etc/fstab

# Install kubeadm, kubectl and kubelet
export DEBIAN_FRONTEND=noninteractive
apt-get -qq install ebtables ethtool
apt-get -qq update
apt-get -qq install -y docker.io apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

KUBEADM_VERSION='1.18.0-00'
export DEBIAN_FRONTEND=noninteractive

apt-get -qq update
apt-get -qq install -y \
  kubelet=$KUBEADM_VERSION \
  kubeadm=$KUBEADM_VERSION \
  kubectl=$KUBEADM_VERSION
SCRIPT

$provision_master_node = <<-SHELL
JOIN_OUTPUT_FILE=/vagrant/join.sh
KUBECONFIG_OUTPUT_FILE=/vagrant/kubeconfig.yaml

rm -rf ${JOIN_OUTPUT_FILE} ${KUBECONFIG_OUTPUT_FILE}

# Start cluster
sudo kubeadm init --apiserver-advertise-address=10.0.0.10 --pod-network-cidr=10.244.0.0/16
sudo kubeadm token create --print-join-command | tee join.sh ${JOIN_OUTPUT_FILE}
chmod +x ${JOIN_OUTPUT_FILE}

KUBECONFIG="${HOME}/.kube/config"
# Configure kubectl
mkdir -p $(dirname ${KUBECONFIG})
sudo cp -i /etc/kubernetes/admin.conf ${KUBECONFIG}
sudo chown $(id -u):$(id -g) ${KUBECONFIG}
cp ${KUBECONFIG} ${KUBECONFIG_OUTPUT_FILE}

# Fix kubelet IP
echo 'Environment="KUBELET_EXTRA_ARGS=--node-ip=10.0.0.10"' | sudo tee -a /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

sudo systemctl daemon-reload
sudo systemctl restart kubelet

# install weave-net
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

SHELL

$install_multicast = <<-SHELL
apt-get -qq install -y avahi-daemon libnss-mdns
SHELL
```

## Starting the cluster

You can create the cluster with:

```bash
$ vagrant up
```

## Clean up

You can delete the cluster with:

```bash
$ vagrant destroy -f
```
```
CREDIT:
https://gist.github.com/mogaika/49749617a5372907c22ddbf68d1447b7
https://gist.github.com/danielepolencic/3009ea0cbc067818358c715f8da227c8
```
