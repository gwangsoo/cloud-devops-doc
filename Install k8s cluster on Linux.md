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

- hosts 등록
```bash
cat << EOF >> /etc/hosts
192.168.1.161 nodea
192.168.1.162 nodeb
192.168.1.163 nodec
192.168.1.164 noded
EOF
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

