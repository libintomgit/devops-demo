# _Kubernetes installation on Ubuntu_

# On master Machine and All worker nodes
# Install Docker
## Debian distros
*As Root*
``` bash line nums="1"
apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
## Red-hat distros
``` bash
 sudo yum install -y yum-utils
 sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
 sudo yum install docker-ce docker-ce-cli containerd.io
 sudo systemctl start docker
 sudo docker run hello-world
 ```

# Install Kubernetes
## Debian distros
*As Root*

``` bash linenums="2"
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
## Red-hat distros
```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

# Set SELinux in permissive mode (effectively disabling it)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```

# On Master machine
## Initiate cluster
*As Root*
``` bash
kubeadm init --apiserver-advertise-address=172.31.15.121 --pod-network-cidr=192.168.0.0/16 --ignore-preflight-errors=Mem,NumCPU
```
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Preserve the  `Kubeadm Join` command. example
```
kubeadm join <masterIP>:6443 --token <some-token> \
    --discovery-token-ca-cert-hash <some-generated token>
```

## Enable AutoCompletion (Optional)
``` bash
echo 'source <(kubectl completion bash)' >>~/.bashrc
source ~/.bashrc
```

## Install network
``` bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

# On All Worker Nodes
## Join cluster
*As Root* run the join command generated during kubeadm init
``` bash
kubeadm join <masterIP>:6443 --token <some-token> \
    --discovery-token-ca-cert-hash <some-generated token>
```

Note: if you forget/misplace the join command, retrieve it from the master machine
```
kubeadm token create --print-join-command
```

## Verify the nodes on the Master
```
kubectl get nodes
```
