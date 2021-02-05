# 하버설치
- https://goharbor.io/docs/1.10/install-config/

## 설치준비

### docker 설치
- https://docs.docker.com/engine/install/centos/

- docker 설치삭제
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

- SET UP THE REPOSITORY
```bash
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

- 도커 엔진 설치
```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

- 도커 시작
```bash
sudo systemctl start docker
```

### docker-compose 설치
- https://docs.docker.com/compose/install/

- 도커 엔진이 설치되어 있어야 함

- 설치버전 목록 조회
https://github.com/docker/compose/releases

- 다운로드
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

- 실행 권한 부여
```bash
sudo chmod +x /usr/local/bin/docker-compose
```

- docker-compose 심볼릭 링크 (docker-compose 명령어가 안먹을 때)
```bash
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

- 버전 확인
```bash
docker-compose --version
```

### openssl 설치
- 설치되어 있어야 함.
```bash
sudo yum install openssl
```

### 방화벽
```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-port=4443/tcp
firewall-cmd --reload
```

## Download the Harbor Installer

- official releases
https://github.com/goharbor/harbor/releases

- download
```bash
sudo curl -L "https://github.com/goharbor/harbor/releases/download/v2.1.3/harbor-offline-installer-v2.1.3.tgz" -o harbor-offline-installer-v2.1.3.tgz
```

- tar 추출
```bash
bash $ tar xvf harbor-offline-installer-v2.1.3.tgz
```

## Https access 구성

## Harbor YML 구성

## 설치 스크립트 실행

## 설치 실패시 문제해결
- https://goharbor.io/docs/1.10/install-config/troubleshoot-installation/


