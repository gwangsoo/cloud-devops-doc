# 하버설치
## 참고 사이즈
- official
https://goharbor.io/docs/1.10/install-config/
- blog
https://waspro.tistory.com/630

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
tar xvf harbor-offline-installer-v2.1.3.tgz
```

## Https access 구성
https://goharbor.io/docs/1.10/install-config/configure-https/

## Harbor YML 구성
https://goharbor.io/docs/1.10/install-config/configure-yml-file/

- hostname ip주소 설정 (127.0.0.1) 로 설정하면 안됨
- https 사용하지 않을 경우 주석(#) 처리

```bash
cp harbor.yml.tmpl harbor.yml

vi harbor.yml

hostname: 192.168.1.156

# https related config
#https:
#  # https port for harbor, default is 443
#  port: 443
#  # The path of cert and key files for nginx
#  certificate: /your/certificate/path
#  private_key: /your/private/key/path
```

## 설치 스크립트 실행
https://goharbor.io/docs/1.10/install-config/run-installer-script/

### install
```bash
sudo ./install.sh --with-clair --with-chartmuseum
```

### 실행확인
```bash
sudo docker ps -a
```

### 로그확인
```bash
sudo tail -f /var/log/harbor/*.log
```

## 변경사항 적용

- stop
```bash
docker-compose down -v
```

- harbor.yml 수정
```bash
vi harbor.yml
```

- prepare 로 적용
```bash
sudo ./prepare --with-clair --with-chartmuseum
```

- start
```bash
sudo docker-compose up -d
```

## 서비스접속
- 초기비밀번호 (harbor.yml 에 정의되어 있음)
admin / Harbor12345

## 설치 실패시 문제해결
- https://goharbor.io/docs/1.10/install-config/troubleshoot-installation/
