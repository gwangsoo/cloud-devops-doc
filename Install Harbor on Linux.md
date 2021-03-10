# Install Harbor on Linux(CentOS7) 
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
- https://goharbor.io/docs/1.10/install-config/configure-https/

### 자체인증서방법
#### 인증기관 인증서 생성
프로덕션 환경에서는 CA로부터 인증서를 받아야합니다. 테스트 또는 개발 환경에서 자체 CA를 생성 할 수 있습니다. CA 인증서를 생성하려면 다음 명령을 실행하십시오.
1. CA 인증서 개인 키를 생성
   ```bash
   openssl genrsa -out ca.key 4096
   
   Generating RSA private key, 4096 bit long modulus
   .......................................................................++
   ............................................................++
   e is 65537 (0x10001)
   ```
2. CA 인증서를 생성
   ```bash
   openssl req -x509 -new -nodes -sha512 -days 3650 \
   -subj "/C=CN/ST=Seoul/L=Seoul/O=example/OU=Personal/CN=harbor.ivycomtech.cloud" \
   -key ca.key \
   -out ca.crt
   ```
   
#### 서버인증서 생성
인증서에는 일반적으로 .crt 파일과 .key 파일이 포함됩니다. (harbor.ivycomtech.cloud.crt, harbor.ivycomtech.cloud.key)

1. 개인키 생성
   ```bash
   openssl genrsa -out harbor.ivycomtech.cloud.key 4096
   
   Generating RSA private key, 4096 bit long modulus
   .................................................................++
   ..................................................................++
   e is 65537 (0x10001)
   ```
2. CSR (인증서 서명 요청)을 생성
   ```bash
   openssl req -sha512 -new \
    -subj "/C=CN/ST=Seoul/L=Seoul/O=example/OU=Personal/CN=harbor.ivycomtech.cloud" \
    -key harbor.ivycomtech.cloud.key \
    -out harbor.ivycomtech.cloud.csr
   ```
3. x509 v3 확장 파일을 생성
   Harbor 호스트에 연결하기 위해 FQDN 또는 IP 주소를 사용하는지 여부에 관계없이 SAN (주체 대체 이름) 및 x509 v3을 준수하는 Harbor 호스트에 대한 인증서를 생성 할 수 있도록이 파일을 만들어야합니다. 확장 요구 사항. DNS도메인을 반영 하도록 항목을 바꿉니다 .
   ```bash
   cat > v3.ext <<-EOF
   authorityKeyIdentifier=keyid,issuer
   basicConstraints=CA:FALSE
   keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
   extendedKeyUsage = serverAuth
   subjectAltName = @alt_names
   
   [alt_names]
   DNS.1=harbor.ivycomtech.cloud
   DNS.2=ivycomtech
   DNS.3=harbor
   EOF
   ```
4. v3.ext파일을 사용 하여 Harbor 호스트에 대한 인증서를 생성
   harbor.ivycomtech.cloud CRS 파일을 CRT 파일 이름으로 바꿉니다.
   ```bash
   openssl x509 -req -sha512 -days 3650 \
       -extfile v3.ext \
       -CA ca.crt -CAkey ca.key -CAcreateserial \
       -in harbor.ivycomtech.cloud.csr \
       -out harbor.ivycomtech.cloud.crt
   ```

### Harbor 및 Docker에 인증서 적용
1. 서버 인증서와 키를 Harbor 호스트의 certficates 폴더에 복사합니다.
   ```bash
   sudo mkdir /data/cert
   sudo cp harbor.ivycomtech.cloud.crt /data/cert/
   sudo cp harbor.ivycomtech.cloud.key /data/cert/
   ```
2. Docker에서 사용하기 위해 crt 파일을 cert 파일로 변환합니다.
   Docker 데몬은 .crt파일을 CA 인증서로 해석 하고 .cert파일을 클라이언트 인증서로 해석하기 때문입니다.
   ```bash
   openssl x509 -inform PEM -in harbor.ivycomtech.cloud.crt -out harbor.ivycomtech.cloud.cert
   ```
3. 먼저 적절한 폴더를 만들고 서버 인증서, 키 및 CA 파일을 Harbor 호스트의 Docker 인증서 폴더에 복사합니다.
   ```bash
   sudo mkdir -p /etc/docker/certs.d/harbor.ivycomtech.cloud
   
   sudo cp harbor.ivycomtech.cloud.cert /etc/docker/certs.d/harbor.ivycomtech.cloud/
   sudo cp harbor.ivycomtech.cloud.key /etc/docker/certs.d/harbor.ivycomtech.cloud/
   sudo cp ca.crt /etc/docker/certs.d/harbor.ivycomtech.cloud/
   ```
   기본 nginx 443 포트를 다른 포트에 매핑 한 경우
   ```bash
   /etc/docker/certs.d/harbor.ivycomtech.cloud:port
   or
   /etc/docker/certs.d/harbor_IP:port
   ```
4. Docker Engine을 다시 시작합니다.
   ```bash
   systemctl restart docker
   ```

OS 수준에서 인증서를 신뢰해야 할 수도 있습니다. 자세한 내용은 하버 설치 문제 해결 을 참조하십시오.
https://goharbor.io/docs/1.10/install-config/troubleshoot-installation/#https
다음 예는 사용자 지정 인증서를 사용하는 구성을 보여줍니다.
```bash
/etc/docker/certs.d/
    └── harbor.ivycomtech.cloud:port
       ├── harbor.ivycomtech.cloud.cert  <-- Server certificate signed by CA
       ├── harbor.ivycomtech.cloud.key   <-- Server key signed by CA
       └── ca.crt                        <-- Certificate authority that signed the registry certificate
```

## Harbor YML 구성
https://goharbor.io/docs/1.10/install-config/configure-yml-file/

### https 사용하지 않는 방법
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
  
### https 사용방법
- 서버인증서 crt 파일, key 파일은 미리 준비해 둬야 함.
  ```bash
  vi harbor.yml
  
  hostname: harbor.ivycomtech.cloud
  https:
    port: 443
    certificate: /home/admin/harbor/cert/harbor.ivycomtech.cloud.crt
    private_key: /home/admin/harbor/cert/harbor.ivycomtech.cloud.key
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

## docker push 방법
1. docker daemon.json 에 내부저장소 추가
   .docker 폴더아래에 daemon.json 파일을 열어서 다음과 같이 내부저장소 정보를 넣는다.
   ```bash
   {
     "registry-mirrors": [],
     "insecure-registries": ["harbor.ivycomtech.cloud"],
     "debug": false,
     "experimental": false,
     "features": {
       "buildkit": true
     }
   }
   ```
2. docker 재시작
3. docker login
   ```bash
   docker login harbor.ivycomtech.cloud
   
   Username: gwangsoo
   Password:
   Login Succeeded
   ```
4. docker build
   ```bash
   docker build -t harbor.ivycomtech.cloud/library/demo:20210310-1 .
   or
   docker build .
   ```
5. docker tag
   ```bash
   docker tag gwangsoo72/demo:demoSpringApp-v1 harbor.ivycomtech.cloud/library/demo:20210310
   ```
6. docker push
   ```bash
   docker push harbor.ivycomtech.cloud/library/demo:20210310-1
   ```

## k8s pull
자체인증서는 k8s에서 이미지 pull 할때 신뢰할 수 없는 인증서 이므로 다음의 오류가 발생됩니다.
- imagepullbackoff x509 certificate signed by unknown authority
각 k8s 클러스터 각 노드에 인증서를 넣어주야 합니다.
```bash
# mkdir -p /etc/docker/certs.d/harbor.ivycomtech.cloud

# scp -r root@192.168.1.156:/etc/docker/certs.d/harbor.ivycomtech.cloud /etc/docker/certs.d
The authenticity of host '192.168.1.156 (192.168.1.156)' can't be established.
ECDSA key fingerprint is SHA256:/wyDIAOXeW8GOcKbNZc5JX3MxG1XEgNV3xJwb98NAI4.
ECDSA key fingerprint is MD5:ad:12:c5:6a:c2:da:7b:bf:f7:8b:5b:1e:b4:95:78:81.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.156' (ECDSA) to the list of known hosts.
root@192.168.1.156's password:
harbor.ivycomtech.cloud.cert      100% 2126     2.9MB/s   00:00
harbor.ivycomtech.cloud.key       100% 3243     3.0MB/s   00:00
ca.crt                            100% 2049     2.7MB/s   00:00

# ll /etc/docker/certs.d/harbor.ivycomtech.cloud
total 12
-rw-r--r--. 1 root root 2049 Mar 10 14:14 ca.crt
-rw-r--r--. 1 root root 2126 Mar 10 14:14 harbor.ivycomtech.cloud.cert
-rw-r--r--. 1 root root 3243 Mar 10 14:14 harbor.ivycomtech.cloud.key

# systemctl restart docker
```

## 설치 실패시 문제해결
- https://goharbor.io/docs/1.10/install-config/troubleshoot-installation/
