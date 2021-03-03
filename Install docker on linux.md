# 최신버전 docker 설치
https://docs.docker.com/engine/install/centos/

## 설치된 docker version 확인
```bash
docker version
or
docker --version
or
yum list installed | grep docker
```

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

### 최신버전 설치
```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

### 버전 선택 설치
- 버전 확인
```bash
sudo yum list docker-ce --showduplicates
```
- 특정버전 설치
```bash
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
sudo yum install docker-ce-19.03.15-3.el7 docker-ce-cli-19.03.15-3.el7
```

## 도커엔진 설치 확인
```bash
yum list docker-ce --showduplicates | sort -r
```

## 도커시작
```bash
sudo systemctl start docker
```

## 엄청 쉬운 또 다른 방법
- 19.03.15
```bash
curl https://releases.rancher.com/install-docker/19.03.15.sh | sh
```
- 18.09.2
```bash
curl https://releases.rancher.com/install-docker/18.09.2.sh | sh
```
- 18.06.2
```bash
curl https://releases.rancher.com/install-docker/18.06.2.sh | sh
```
- 17.03.2
```bash
curl https://releases.rancher.com/install-docker/17.03.2.sh | sh
```
