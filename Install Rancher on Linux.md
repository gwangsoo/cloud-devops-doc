# 리눅스에 Rancher 설치하기
https://rancher.com/docs/rancher/v2.x/en/installation/install-rancher-on-linux

## 전제사항
### 시스템요구사항
- systemd를 지원하는 OS
- RHEL or CentOS8은 추가 작업 필요
- cpu : 2
- mem : 8GBye

### root 로 설치
```bash
sudo -s
or
su
```

## 구성 설정
### 고정 된 등록 주소로 인증서 오류를 방지
```bash
mkdir -p /etc/rancher/rke2
vi /etc/rancher/rke2/config.yaml
token: my-shared-secret
tls-san:
  - 192.168.1.157
```

## 첫번째 서버노드 설치
### RancherD 설치 프로그램을 실행
```bash
curl -sfL https://get.rancher.io | sh -
```

### 설치버전 확인
```bash
rancherd --help
NAME:
   rancherd - Rancher Kubernetes Engine 2
...
```

### RancherD를 시작
```bash
systemctl enable rancherd-server.service
systemctl start rancherd-server.service
```

### 설치로그 확인
RancherD가 시작되면 RKE2 Kubernetes 클러스터를 설치됨.
다음 명령어를 사용하여 Kubernetes 클러스터가 표시되면 로그를 확인.
```bash
journalctl -eu rancherd-server -f
```

## kubectl을 사용하여 kubeconfig 파일 설정
Kubernetes 클러스터가 설정되면 RancherD의 kubeconfig 파일을 설정하고 kubectl다음을 수행하십시오.
```bash
vi ~/.bash_profile
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml PATH=$PATH:/var/lib/rancher/rke2/bin
```

## Rancher가 Kubernetes 클러스터에 설치되어 있는지 확인
```bash
kubectl get all -A
kubectl get daemonset rancher -n cattle-system
kubectl get pod -n cattle-system
```

## 초기 Rancher 비밀번호 설정
```bash
rancherd reset-admin
```

## 제거
```bash
/usr/local/bin/rancherd-uninstall.sh
```

## 고가용성
Rancher 서버를 사용하여 다운 스트림 Kubernetes 클러스터를 관리하려는 경우 Rancher는 가용성이 높아야합니다.
이 단계에서는 고 가용성 클러스터를 달성하기 위해 더 많은 노드를 추가합니다.
Rancher는 데몬 셋으로 실행되기 때문에 추가 한 노드에서 자동으로 시작됩니다.
클러스터 데이터를 포함하는 etcd 클러스터에는 쿼럼 손실을 방지하기 위해 대부분의 라이브 노드가 필요하기 때문에 홀수의 노드가 필요합니다.
쿼럼이 손실되면 백업에서 클러스터를 복원해야 할 수 있습니다. 따라서 3 개의 노드를 사용하는 것이 좋습니다.

이 단계를 수행 할 때 여전히 루트로 로그인해야합니다.

### 새 노드에서 고정 등록 주소 구성
server및 token매개 변수를 지정해야한다는 점을 제외하면 추가 서버 노드는 첫 번째와 매우 유사하게 시작
```bash
mkdir -p /etc/rancher/rke2
vi /etc/rancher/rke2/config.yaml

server: https://my-fixed-registration-address.com:9345
token: my-shared-secret
tls-san:
  - my-fixed-registration-address.com
  - another-kubernetes-domain.com
```

### 추가 서버 노드 시작
새 노드에서 설치 프로그램을 실행
```bash
curl -sfL https://get.rancher.io | sh -
```

RancherD를 시작
```bash
systemctl enable rancherd-server.service
systemctl start rancherd-server.service
```

### 반복
다른 Linux 노드에 대해 1 단계와 2 단계를 반복하여 클러스터의 노드 수를 3 개로 만듭니다.
