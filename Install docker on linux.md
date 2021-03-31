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

## docker push 가이드

### 사설 certificate 주입
docker 는 push 할때 https 만 지원하기 때문에 사설도메인의 경우 사설certificate 를 주입해 주어야 함.
- 주입위치
  /etc/docker/certs.d
- sample
```bash
[root@nodea certs.d]# ll -R
.:
total 0
drwxr-xr-x. 2 root root 138 Mar 30 18:12 harbor.192-168-1-156.nip.io

./harbor.192-168-1-156.nip.io:
total 16
-rw-r--r--. 1 root root 2057 Mar 30 18:12 ca.crt
-rw-r--r--. 1 root root 2143 Mar 30 18:12 harbor.192-168-1-156.nip.io.cert
-rw-r--r--. 1 root root 3243 Mar 30 18:12 harbor.192-168-1-156.nip.io.key
```

### docker login
```bash
docker login -u ivymanager -p Ivymanager1 harbor.192-168-1-156.nip.io

WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

### docker push
- docker image 에 tag 을 달고
  ```bash
  docker tag fbb576e9f482 harbor.ivycomtech.cloud/library/demo:latest
  ```
- 해당 tag 을 push 함
  ```bash
  docker push harbor.ivycomtech.cloud/library/demo:latest
  ```
