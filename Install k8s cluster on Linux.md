# Install K8s cluster on Linux(CentOS7)

## 설치 준비

- root 로그인
- SELinux 비활성화
  - SELinux 는 시스템자원 접근 통제 서비스 임.
  ```bash
  setenforce 0
  ```
- 방화벽 비활성화
```bash
systemctl disable firewalld && systemctl stop firewalld
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

- K8s 설치
```bash
yum install -y docker kubelet kubeadm kubectl kubernetes-cni
```

- 실행
```bash
systemctl enable docker && systemctl start docker
systemctl enable kubelet && systemctl start kubelet
```

- iptable 커널옵션 활성화
```bash
sysctl -w net.bridge.bridge-nf-call-iptables=1
echo "net.bridge.bridge-nf-call-iptables=1" > /etc/sysctl.d/k8s.conf
```

- swap 비활성화
- swap이 활성화 되어 있으면 Kubelet 이 실행되지 않음
```bash
swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab
```
