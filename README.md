# cloud-devops-doc

cloud 관련 설치 가이드 및 CI/CD 환경 구성 가이드

## host
- 192.168.1.155 gitlab-runner
- 192.168.1.156 harbor
- 192.168.1.158 gitlab
- 192.168.1.160 nexus
- 192.168.1.161 kubernetes node a (master)
- 192.168.1.162 kubernetes node b (master)
- 192.168.1.163 kubernetes node c (master)
- 192.168.1.164 kubernetes node d (worker)
- 192.168.1.181 kubernetes node e (worker)
- 192.168.1.182 elasticsearch + kibana

## host 등록
- xip.io 또는 nip.io 를 사용해서 host 등록하는게 편함 굳이 hosts 를 등록하지 않아도 되니까.
  - https://nip.io/ 참고
```bash
sudo cat << EOF >> /etc/hosts
192.168.1.155 runner.ivycomtech.cloud
192.168.1.156 harbor.ivycomtech.cloud
192.168.1.158 gitlab.ivycomtech.cloud
192.168.1.160 nexus.ivycomtech.cloud
192.168.1.161 nodea.ivycomtech.cloud
192.168.1.162 nodeb.ivycomtech.cloud
192.168.1.163 nodec.ivycomtech.cloud
192.168.1.164 noded.ivycomtech.cloud
192.168.1.181 nodee.ivycomtech.cloud
192.168.1.182 nodef.ivycomtech.cloud
EOF
```

## hostname 변경
```bash
sudo hostnamectl set-hostname rancher.ivycomtech.cloud
```

## 사설 CA root
- ca root 를 하나 만들고 공유해서 사용하려 했으니 귀찮아서 그냥 함. 필요시 ca root 를 그때 그때 만들어서 사용했음.
- ivycomtech.cloud CA Root Certificate

## Rancher 설치 순서
- 여러가지 방법이 있으나 K8s cluster 구성 후 Helm 으로 설치하세요. 
1. centos 설치
2. docker 설치 : Install docker on linux.md 참고
3. K8s 설치 : Install k8s cluster on Linux.md 참고
4. Rancher 설치 : Install Rancher on K8s by Helm.md 참고
