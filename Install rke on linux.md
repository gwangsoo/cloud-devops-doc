# Install rke on linux

## rke
### config 생성
```bash
[root@rancher2 rke]# ./rke config
[+] Cluster Level SSH Private Key Path [~/.ssh/id_rsa]:
[+] Number of Hosts [1]: 1
[+] SSH Address of host (1) [none]: 192.168.1.165
[+] SSH Port of host (1) [22]:
[+] SSH Private Key Path of host (192.168.1.165) [none]:
[-] You have entered empty SSH key path, trying fetch from SSH key parameter
[+] SSH Private Key of host (192.168.1.165) [none]:
[-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.ssh/id_rsa
[+] SSH User of host (192.168.1.165) [ubuntu]: root
[+] Is host (192.168.1.165) a Control Plane host (y/n)? [y]: y
[+] Is host (192.168.1.165) a Worker host (y/n)? [n]: y
[+] Is host (192.168.1.165) an etcd host (y/n)? [n]: y
[+] Override Hostname of host (192.168.1.165) [none]:
[+] Internal IP of host (192.168.1.165) [none]:
[+] Docker socket path on host (192.168.1.165) [/var/run/docker.sock]:
[+] Network Plugin Type (flannel, calico, weave, canal) [canal]:
[+] Authentication Strategy [x509]:
[+] Authorization Mode (rbac, none) [rbac]:
[+] Kubernetes Docker image [rancher/hyperkube:v1.17.4-rancher1]:
[+] Cluster domain [cluster.local]:
[+] Service Cluster IP Range [10.43.0.0/16]:
[+] Enable PodSecurityPolicy [n]:
[+] Cluster Network CIDR [10.42.0.0/16]:
[+] Cluster DNS Service IP [10.43.0.10]:
[+] Add addon manifest URLs or YAML files [no]:
```
### 지원되는 버전 확인
- 지원되는 kubernetes 버전 확인
```bash
rke config --list-version --all
v1.16.8-rancher1-3
v1.17.4-rancher1-3
v1.15.11-rancher1-3
```

### 설치
- 설치옵션
```base
[root@rancher2 rke]# ./rke up --help
NAME:
   rke up - Bring the cluster up

USAGE:
   rke up [command options] [arguments...]

OPTIONS:
   --config value               Specify an alternate cluster YAML file (default: "cluster.yml") [$RKE_CONFIG]
   --local                      Deploy Kubernetes cluster locally
   --dind                       Deploy Kubernetes cluster in docker containers (experimental)
   --dind-storage-driver value  Storage driver for the docker in docker containers (experimental)
   --dind-dns-server value      DNS resolver to be used by docker in docker container. Useful if host is running systemd-resovld (default: "8.8.8.8")
   --update-only                Skip idempotent deployment of control and etcd plane
   --disable-port-check         Disable port check validation between nodes
   --init                       Initiate RKE cluster
   --cert-dir value             Specify a certificate dir path
   --custom-certs               Use custom certificates from a cert dir
   --ssh-agent-auth             Use SSH Agent Auth defined by SSH_AUTH_SOCK
   --ignore-docker-version      Disable Docker version check
```
- 설치
```bash
./rke up --local
```
- 제거
```bash
./rke remove --force --local
```

#### 설치오류들
- Can't retrieve Docker Info: Error response from daemon: client version 1.41 is too new. Maximum supported API version is 1.40
```bash
export DOCKER_API_VERSION=1.40
```
- 그외에 뭔가 이상 쫑나서 잘 안되면 제거하고 다시 설치

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
- install
```bash
yum install -y kubectl-1.17.4-0 kubelet-1.17.4-0 kubeadm-1.17.4-0
```
- run
```bash
systemctl daemon-reload
systemctl enable kubelet && systemctl start kubelet
```
- init
```bash
kubeadm init --apiserver-advertise-address=192.168.1.165
```
