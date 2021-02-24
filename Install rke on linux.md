# Install rke on linux

## rke
### config 생성
### 지원되는 버전 확인
- 지원되는 kubernetes 버전 확인
```bash
rke config --list-version --all
v1.16.8-rancher1-3
v1.17.4-rancher1-3
v1.15.11-rancher1-3
```

## yum 명령 참고
- 설치된 버전 확인
```bash
yum list installed
```
- repo 에서 설치가능 버전 확인
```bash
yum search kubectl --showduplicates 
```
- 설치제거
```bash
yum remove 지우려는패키지
```

## 설치 준비
### CentOS7 준비
### Docker 설치
### Kubernetes 설치
```bash
yum install -y ectl-1.17.4-0 kubelet-1.17.4-0 kubeadm-1.17.4-0
```
