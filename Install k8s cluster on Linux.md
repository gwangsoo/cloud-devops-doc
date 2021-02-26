# Install K8s cluster on Linux(CentOS7)
https://kubetm.github.io/practice/appendix/installation_case1/

## 설치 준비

- root 로그인
- SELinux 비활성화
  - SELinux 는 시스템자원 접근 통제 서비스 임.
    ```bash
    setenforce 0
    ```
  - 리부팅시 다시 원복되기 때문에 아래 명령을 통해서 영구적으로 변경
    ```bash
    sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
    ```
  - 적용 내용 내용 확인
  ```bash
  sestatus | grep Current
  Current mode: permissive
  ```
- 방화벽 비활성화
  ```bash
  systemctl disable firewalld && systemctl stop firewalld
  ```
- NetworkManager 비활성화
  ```bash
  systemctl stop NetworkManager && systemctl disable NetworkManager
  ```

- Iptables 커널 옵션 활성화
```bash
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

- swap 비활성화
  - swap이 활성화 되어 있으면 Kubelet 이 실행되지 않음
```bash
swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab
```

- Hosts 등록
```bash
cat << EOF >> /etc/hosts
192.168.1.155 runner.ivycomtech.cloud
192.168.1.156 harbor.ivycomtech.cloud
192.168.1.157 rancher.ivycomtech.cloud
192.168.1.158 gitlab.ivycomtech.cloud
192.168.1.160 nexus.ivycomtech.cloud
192.168.1.161 nodea.ivycomtech.cloud
192.168.1.162 nodeb.ivycomtech.cloud
192.168.1.163 nodec.ivycomtech.cloud
192.168.1.164 noded.ivycomtech.cloud
192.168.1.165 rancher2.ivycomtech.cloud
EOF
```

- Host name 변경
```bash
hostnamectl set-hostname noded.ivycomtech.cloud
```

## K8s 
- K8s yum repo 추가
```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://yum.kubernetes.io/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

- Centos Update
```bash
yum update -y
```

### docker 설치
- 도커 설치 전에 필요한 패키지 설치
```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

- 도커 설치를 위한 저장소 를 설정
```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

- 도커 패키지 설치
```bash
yum update -y && yum install -y docker-ce
```
```bash
mkdir /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

mkdir -p /etc/systemd/system/docker.service.d
```

### K8s 설치
- K8s 설치
```bash
yum install -y --disableexcludes=kubernetes kubeadm kubectl kubelet kubernetes-cni
or
yum install -y --disableexcludes=kubernetes kubeadm-1.19.6-0 kubectl-1.19.6-0 kubelet-1.19.6-0 kubernetes-cni
```

- 실행
```bash
systemctl daemon-reload
systemctl enable docker && systemctl start docker
systemctl enable kubelet && systemctl start kubelet
```

### master node
- K8s 초기화 명령 실행
  - single node 생성
    ```bash
    kubeadm init --pod-network-cidr=20.96.0.0/12 --apiserver-advertise-address=192.168.1.157
    ```
  - multi node 생성
    - control-plane-endpoint: 노드1 DNS/IP 또는 LoadBalancer DNS/IP
    - upload-certs: master node에 인증서가 자동 배포 됨.
    ```bash
    kubeadm init --control-plane-endpoint "192.168.1.157:26443" \
                 --upload-certs \
                 --pod-network-cidr "20.96.0.0/12" \
                 --apiserver-advertise-address=192.168.1.157
    ```

  - 초기화 후 다시 설치해야 하는 경우
  ```bash
  kubeadm reset
  ```

- 실행 후 [Your Kubernetes master has initialized successfully!] 내용 별도로 저장
```bash
cat <<EOF > k8s-install-result.txt
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.1.157:26443 --token xor8z4.fe1vf24slczx1p4i \
    --discovery-token-ca-cert-hash sha256:47d1682c0553aedb46863997a981572fdd25a4069038be1827c0ab482dce7aac \
    --control-plane --certificate-key b78d7fd18125f61590cec56671b48f3af26f955735893d307b331c7b6d1cabb6

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.157:26443 --token xor8z4.fe1vf24slczx1p4i \
    --discovery-token-ca-cert-hash sha256:47d1682c0553aedb46863997a981572fdd25a4069038be1827c0ab482dce7aac
EOF
```

- 환경 변수 설정
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- Kubectl 자동완성 기능 설치
  - kubectl 사용시 [tab] 버튼을 이용해서 다음에 올 명령어 리스트를 조회
```bash
yum install bash-completion -y
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

### master node HA 구성
- master node를 복수개 연결하여 HA 구성합니다.
- 외부에 HAProxy 및 Keepalived 설정하거나, master node 에 구성할 수도 있습니다.
- Kubernetes 공시 가이드에도 HAProxy 를 구성을 가이드 하고 있습니다.
- HAProxy 구성 방법
  - node1 IP의 26443포트로 전달받은 데이터를 node1 ~ node3의 6443 포트로 포워드 시켜줍니다. 라운드로빈으로 순차적으로 접근하도록 하겠습니다.
```bash
$ sudo cat <<EOF >> /etc/haproxy/haproxy.cfg
frontend kubernetes-master-lb
bind 0.0.0.0:26443
option tcplog
mode tcp
default_backend kubernetes-master-nodes

backend kubernetes-master-nodes
mode tcp
balance roundrobin
option tcp-check
option tcplog
server rancher 192.168.1.157:6443 check
server rancher2 192.168.1.165:6443 check
EOF

$ sudo systemctl restart haproxy
$ sudo systemctl status haproxy
```

### master node 연결
- Master Init 후 복사 했었던 내용 붙여넣기
```bash
kubeadm join 192.168.1.157:26443 --token xor8z4.fe1vf24slczx1p4i \
    --discovery-token-ca-cert-hash sha256:47d1682c0553aedb46863997a981572fdd25a4069038be1827c0ab482dce7aac \
    --control-plane --certificate-key b78d7fd18125f61590cec56671b48f3af26f955735893d307b331c7b6d1cabb6
```

### worker node
- Master Init 후 복사 했었던 내용 붙여넣기
```bash
kubeadm join 192.168.1.157:26443 --token xor8z4.fe1vf24slczx1p4i \
    --discovery-token-ca-cert-hash sha256:47d1682c0553aedb46863997a981572fdd25a4069038be1827c0ab482dce7aac
```

- Node 연결 확인
  - Master 서버에 접속해서 아래 명령 입력 후 추가된 Node가 보이는지 확인 (Status는 NotReady)
```bash
kubectl get nodes
```

## Networking

### Calico 설치
- Calico는 기본 192.168.0.0/16 대역으로 설치가 되는데, 그럼 실제 VM이 사용하고 있는 대역대와 겹치기 때문에 수정을 해서 설치해야 할 경우
```bash
curl -O https://docs.projectcalico.org/v3.9/manifests/calico.yaml
sed s/192.168.0.0\\/16/20.96.0.0\\/12/g -i calico.yaml
kubectl apply -f calico.yaml
```

- calico와 coredns 관련 Pod의 Status가 Running인지 확인 (2분정도 소요)
```bash
kubectl get pods --all-namespaces
```

## Etc

### master node 에 pod 배포 가능하게 하기
```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

### Componentstatuses Unhealthy (컴포넌트 상태가 unhealty 로 확인되는 경우)
- ComponentStatus 는 v1.19 부터 deprecated 되어서, controller-manager와 scheduler 와 포트가 변경된 사항을 더이상 반영하지 못함.
```bash
kubectl get cs

Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused
etcd-0               Healthy     {"health":"true"}
```
- 해결방법
  - 아래 yaml 파일에서 다음 부분을 주석으로 처리합니다.
  ```bash
      - --port=0
  ```
  ```bash
      #- --port=0
  ```
  ```bash
  sudo vi /etc/kubernetes/manifests/kube-controller-manager.yaml
  sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml
  sudo systemctl restart kubelet.service
  ```
