# Ivy cloud 구성

## 개요
- 고가용성 K8s 구성을 목표로 합니다.
- master node 의 3중화

## 설치순서
1. os 설정
2. docker 설치
3. k8s 설치
4. HAProxy 설치
5. 클러스터 생성

## 설치
### os 설정
- Install K8s cluster on Linux 참조

### docker 설치
- Install K8s cluster on Linux 참조

### k8s 설치
- Install K8s cluster on Linux 참조

### HAProxy 설치
```bash
yum install haproxy -y
```
- 설정
  - node1 IP의 26443포트로 전달받은 데이터를 node1 ~ node3의 6443 포트로 포워드
  - 라운드로빈으로 순차적으로 접근하도록 함.
```bash
# vi /etc/haproxy/haproxy.cfg
server nodea 192.168.1.161:6443 check
server nodeb 192.168.1.162:6443 check
server nodec 192.168.1.163:6443 check

# sudo systemctl restart haproxy
# systemctl status haproxy
```

### 클러스터 생성
클러스터 생성할때 옵션
- control-plane-endpoint: 노드1 DNS/IP 또는 LoadBalancer DNS/IP
- upload-certs: 인증서가 자동 배포

```bash
kubeadm init --control-plane-endpoint "192.168.1.161:26443" \
             --upload-certs \
             --pod-network-cidr "10.244.0.0/16"
```

결과
```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.1.161:26443 --token d4kakd.cf84pi81d59f3mte \
    --discovery-token-ca-cert-hash sha256:cfc99131ed47f1f20ad3a07f41ba8384a2b613f67b6800caf50070bdc7340e08 \
    --control-plane --certificate-key 0ec47f7b085216e520531e754ac1a5fe4669017e4cd7ab61204ffc00c615e762

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.161:26443 --token d4kakd.cf84pi81d59f3mte \
    --discovery-token-ca-cert-hash sha256:cfc99131ed47f1f20ad3a07f41ba8384a2b613f67b6800caf50070bdc7340e08
```

결과의 join 문자열은 저장해 둔다.
```bash
cat <<EOF > k8x-install-result.txt

master 노드용
kubeadm join 192.168.1.161:26443 --token d4kakd.cf84pi81d59f3mte \
    --discovery-token-ca-cert-hash sha256:cfc99131ed47f1f20ad3a07f41ba8384a2b613f67b6800caf50070bdc7340e08 \
    --control-plane --certificate-key 0ec47f7b085216e520531e754ac1a5fe4669017e4cd7ab61204ffc00c615e762

worker 노드용
kubeadm join 192.168.1.161:26443 --token d4kakd.cf84pi81d59f3mte \
    --discovery-token-ca-cert-hash sha256:cfc99131ed47f1f20ad3a07f41ba8384a2b613f67b6800caf50070bdc7340e08
EOF
```

### 클러스터 연결
노드1에서 클러스터 생성시 kubeadm join 명령어가 출력되는데 –control-plane –certificate-key 플래그 포함하여 명령어를 실행하면 마스터 노드로 연결이 되고, 플래그는 추가하지 않고 사용하게 되면 워커노드로 연결됩니다.

- nodeb 및 nodec 를 nodea에 연결
```bash
kubeadm join 192.168.1.161:26443 --token d4kakd.cf84pi81d59f3mte \
    --discovery-token-ca-cert-hash sha256:cfc99131ed47f1f20ad3a07f41ba8384a2b613f67b6800caf50070bdc7340e08 \
    --control-plane --certificate-key 0ec47f7b085216e520531e754ac1a5fe4669017e4cd7ab61204ffc00c615e762
```

- 클러스터 구성 확인

- 마스터에 pod 배포 설정
마스터 노드에는 기본설정으로 pod 배포가 안됩니다. taint명령어를 통해 pod 배포가 가능하도록 설정.
```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

### 가상 네트워크 설치
Calico, Canal, Clilum, Flannel, weave 등 다양한 가상네트워크.

-weave 설치
```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

- Calico 설치
  - Calico는 기본 192.168.0.0/16 대역으로 설치가 되는데, 그럼 실제 VM이 사용하고 있는 대역대와 겹치기 때문에 수정을 해서 설치해야 할 경우
```bash
curl -O https://docs.projectcalico.org/v3.9/manifests/calico.yaml
sed s/192.168.0.0\\/16/20.96.0.0\\/12/g -i calico.yaml
kubectl apply -f calico.yaml
```
