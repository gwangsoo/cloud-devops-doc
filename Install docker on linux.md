# 최신버전 docker 설치
https://docs.docker.com/engine/install/centos/

## Uninstall old versions
```bash
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine
```

## yum-utils 설치
```bash
sudo yum install -y yum-utils
```

## docker 레포지토리 추가
```bash
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

## 도커레포지토리 활성화
```bash
sudo yum-config-manager --enable docker-ce-nightly docker-ce-test
```

## 도커레포지토리 비활성화 (설치때 사용 하지 않음)
```bash
sudo yum-config-manager --disable docker-ce-nightly
```

## 도커엔진 설치
```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

## 도커엔지 설치 확인
```bash
yum list docker-ce --showduplicates | sort -r
```

## 도커시작
```bash
sudo systemctl start docker
```
