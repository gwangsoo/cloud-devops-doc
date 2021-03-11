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
    kubeadm init --pod-network-cidr=20.96.0.0/16 --apiserver-advertise-address=192.168.1.161
    ```
  - multi node 생성
    - control-plane-endpoint: 노드1 DNS/IP 또는 LoadBalancer DNS/IP
    - upload-certs: control plane 인스턴스에서 공유해야 하는 인증서를 업로드(자동배포), 수동으로 인증서를 복사할 경우는 이 옵션 생략
    ```bash
    kubeadm init --control-plane-endpoint=192.168.1.161:6443 \
                 --upload-certs \
                 --pod-network-cidr=20.96.0.0/16 \
                 --apiserver-advertise-address=192.168.1.161 \
                 --v=5
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

#### 참고
- certificate-key를 지정하여 나중에 조인에서 사용할 수 있음
```bash
shell> sudo kubeadm alpha certs certificate-key
f8902e114ef118304e561c3ecd4d0b543adc226b7a07f675f56564185ffe0c07 
```
- join token 은 다시 확인할 수 있다.
```bash
kubeadm token create --print-join-command
```
- 인증서를 다시 업로드하고 새 암호 해독 키를 생성하려면 이미 클러스터에 연결된 control plane 노드에서 다음 명령을 사용
```bash
shell> sudo kubeadm init phase upload-certs --upload-certs
W0322 16:20:40.631234  101997 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0322 16:20:40.631413  101997 validation.go:28] Cannot validate kubelet config - no validator is available
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
7f97aaa65bfec1f039c4dbdf3a2073de853c708bd4d9ff9d72b776b0f9874c9d
```
- 클러스터를 control plane 조인(join)하는 데 필요한 전체 'kubeadm join' 플래그를 출력
```bash
shell> sudo kubeadm token create --print-join-command --certificate-key \ 7f97aaa65bfec1f039c4dbdf3a2073de853c708bd4d9ff9d72b776b0f9874c9d
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
- CNI란?
  - Container Network Interface (CNI)는 Linux Container의 Network Interface를 설정할때 이용되는 Interface이다. 자세한 내용은 아래 주소 참조.
  - https://ssup2.github.io/theory_analysis/Container_Network_Interface/
- K8s CNI plugin 비교 https://rancher.com/blog/2019/2019-03-21-comparing-kubernetes-cni-providers-flannel-calico-canal-and-weave/
- 네트워크 정책 제공자 https://kubernetes.io/ko/docs/tasks/administer-cluster/network-policy-provider/
- 네트워크 정책 선언 https://kubernetes.io/ko/docs/tasks/administer-cluster/declare-network-policy/

### Weave 설치
```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

### Frannel 설치
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### Calico 설치
- Calico는 기본 192.168.0.0/16 대역으로 설치가 되는데, 그럼 실제 VM이 사용하고 있는 대역대와 겹치기 때문에 수정을 해서 설치해야 할 경우
```bash
curl -O https://docs.projectcalico.org/v3.9/manifests/calico.yaml
sed s/192.168.0.0\\/16/20.96.0.0\\/16/g -i calico.yaml
kubectl apply -f calico.yaml
```

- calico와 coredns 관련 Pod의 Status가 Running인지 확인 (2분정도 소요)
```bash
kubectl get pods --all-namespaces
```

### Canal 설치
- todo

## Ingress controller 설치
인그레스 컨트롤러는 클러스터와 함께 자동으로 실행되지 않는다.
클러스터에 가장 적합한 인그레스 컨트롤러 구현을 선택해서 설치해야 한다.
프로젝트로서 쿠버네티스는 AWS, GCE와 nginx 인그레스 컨트롤러를 지원하고 유지한다.
https://kubernetes.io/ko/docs/concepts/services-networking/ingress-controllers/

### nginx-ingress controll 설치
https://github.com/kubernetes/ingress-nginx/blob/master/README.md

#### 직접 설치 (Bare-metal)

1. 설치
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/baremetal/deploy.yaml
```

2. 설치 확인
```bash
kubectl get svc --namespace=ingress-nginx
```

3. Ingress 생성
Ingress Controller는 Ingress 규칙을 관리하기 위한 서버일 뿐이고, 실제 Ingress 규칙(L7규칙)을 생성 해야함.
- Ingress 를 통해 연결할 서비스 등록
```bash
cat <<EOF > demo-svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: demo-svc
spec:
  selector:
   app: demo
  ports:
     - name: http
       port: 80
       targetPort: 8080
EOF

kubectl apply -f demo-svc.yaml -n ivy-default
```
- Ingress 등록
```bash
cat <<EOF > demo-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: demo-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: nodea.ivycomtech.cloud
    http:
      paths:
      - path: /
        backend:
          serviceName: demo-svc
          servicePort: 80
EOF

kubectl apply -f demo-ingress.yaml -n ivy-default
```

4. 베어메탈설치 시 고려사항 (HA 구성을 위한 network topology 등)
https://kubernetes.github.io/ingress-nginx/deploy/baremetal/

#### helm 으로 설치
1. helm 을 통해 설치
   ```bash
   helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
   helm repo update
   helm install ingress-nginx ingress-nginx/ingress-nginx
   ```
2. 설치된 버전 확인
   ```bash
   POD_NAME=$(kubectl get pods -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}')
   kubectl exec -it $POD_NAME -- /nginx-ingress-controller --version
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

### coredns 상태가 Pending 에서 멈춘경우
- 가상네트워크(calico/weave/flannel/canal) 를 설치해야함.

### coredns 상태가 ContainerCreating 에서 멈춘경우
- 가상네트워크(calico/weave/flannel/canal) 가 설치되었는지 확인
- ifconfig 로 실제 network 및 lo 를 제외하고 가상네트웍은 제거하는게 좋다
- CNI가 꼬였을 수 있으므로 cluster reset 후 cluster 재구성 (kubeadm init)
- reset 후 reboot now
- 재구성 후 에도 cni 의 문제가 지속되면 reboot now

```bash
kubeadm reset

rm -rf /etc/cni/net.d
ipvsadm --clear

systemctl stop kubelet
systemctl stop docker

rm -rf /var/lib/cni/
rm -rf /var/lib/kubelet/*
rm -rf /etc/cni/

ifconfig -s | awk '{print $1 " down"}' | grep -v ens33 | grep -v lo | grep -v Iface | xargs ifconfig
or
ifconfig cni0 down
ifconfig flannel.1 down
ifconfig weave down
ifconfig canal down
ifconfig califcc3247d241 down <- calico 는 cali 로 시작하는 이름으로 여러개 생겼을 수 있음
ifconfig docker0 down

ip link | grep '<' | grep -v lo | grep -v ens33 | grep -v lo | grep -v @ | grep -v datapath | awk -F ':' '{print $2}'

ip link delete cni0
ip link delete flannel.1
ip link delete weave
ip link delete canal
ip link delete califcc3247d241  <- calico 는 cali 로 시작하는 이름으로 여러개 생겼을 수 있음
```
