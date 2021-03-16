# cloud-devops-doc

cloud 관련 설치 가이드 및 CI/CD 환경 구성 가이드

## host
- 192.168.1.155 runner
- 192.168.1.156 harbor
- 192.168.1.158 gitlab
- 192.168.1.160 nexus
- 192.168.1.161 kubernetes node a (master)
- 192.168.1.162 kubernetes node b (master)
- 192.168.1.163 kubernetes node c (master)
- 192.168.1.164 kubernetes node d (worker)
- 192.168.1.181 kubernetes node e (worker)
- 192.168.1.182 kubernetes node f (worker)

## host 등록
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
ivycomtech.cloud CA Root Certificate
