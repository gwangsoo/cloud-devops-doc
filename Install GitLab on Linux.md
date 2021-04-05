# install GitLab on Linux (CentOS7)
https://about.gitlab.com/install/#centos-7?version=ce

## 설치 준비

### Git 설치
- 설치된 git 버전 확인
```bash
git --version
```

- git 설치
```bash
yum install git
```

### git lab 설치전 필요한 의존성 설치
https://about.gitlab.com/installation/#centos
```bash
sudo yum install -y curl policycoreutils-python openssh-server perl
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
```

### Postfix를 설치하여 알림 이메일
```bash
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

## GitLab 설치

### GitLab download
```bash
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```

### GitLab Install
```bashsudo EXTERNAL_URL="https://gitlab.ivycomtech.cloud" yum install -y gitlab-ce
or
sudo yum install -y gitlab-ce
```

## 설치 후

### SSL 인증서 적용 방법

#### 자체인증서방법
정식 인증서가 있는 경우 pass!!
```bash
cd /etc/gitlab/ssl/
```

##### 인증기관 인증서 생성
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
   -subj "/C=CN/ST=Seoul/L=Seoul/O=example/OU=Personal/CN=gitlab.192-168-1-158.nip.io" \
   -key ca.key \
   -out ca.crt
   ```
   
##### 서버인증서 생성
인증서에는 일반적으로 .crt 파일과 .key 파일이 포함됩니다. (gitlab.192-168-1-158.nip.io.crt, gitlab.192-168-1-158.nip.io.key)

1. 개인키 생성
   ```bash
   openssl genrsa -out gitlab.192-168-1-158.nip.io.key 4096
   
   Generating RSA private key, 4096 bit long modulus
   .................................................................++
   ..................................................................++
   e is 65537 (0x10001)
   ```
2. CSR (인증서 서명 요청)을 생성
   ```bash
   openssl req -sha512 -new \
    -subj "/C=CN/ST=Seoul/L=Seoul/O=example/OU=Personal/CN=gitlab.192-168-1-158.nip.io" \
    -key gitlab.192-168-1-158.nip.io.key \
    -out gitlab.192-168-1-158.nip.io.csr
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
   DNS.1=gitlab.192-168-1-158.nip.io
   DNS.2=gitlab.ivycomtech.cloud
   DNS.3=gitlab
   EOF
   ```
4. v3.ext파일을 사용 하여 Harbor 호스트에 대한 인증서를 생성
   gitlab.192-168-1-158.nip.io CRS 파일을 CRT 파일 이름으로 바꿉니다.
   ```bash
   openssl x509 -req -sha512 -days 3650 \
       -extfile v3.ext \
       -CA ca.crt -CAkey ca.key -CAcreateserial \
       -in gitlab.192-168-1-158.nip.io.csr \
       -out gitlab.192-168-1-158.nip.io.crt
   ```

5. 생성된 인증서 파일을 복사한다.
   CA.crt, ca.key, ca.srl,  gitlab.192-168-1-158.nip.io.crt, gitlab.192-168-1-158.nip.io.csr, gitlab.192-168-1-158.nip.io.key
   ```bash
   cp ca.* /etc/gitlab/ssl
   cp gitlab.192-168-1-158.nip.io.* /etc/gitlab/ssl
   ```
   
### GitLab 설정 파일 수정
```bash
sudo vi /etc/gitlab/gitlab.rb

external_url 'https://gitlab.192-168-1-158.nip.io'
nginx['redirect_http_to_https'] = true
nginx['ssl_client_certificate'] = "/etc/gitlab/ssl/ca.crt"
letsencrypt['enable'] = false

sudo gitlab-ctl reconfigure
```

### 설정변경
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
```bash
sudo vi /etc/gitlab/gitlab.rb
sudo gitlab-ctl reconfigure
```

### 서비스 시작 및 종료
```bash
sudo systemctl enable gitlab-runsvdir.service
sudo systemctl status gitlab-runsvdir.service
sudo systemctl start gitlab-runsvdir.service
sudo systemctl stop gitlab-runsvdir.servic
```

### 비밀번호 재설정
- Rails 콘솔 시작
```bash
sudo gitlab-rails console -e production
```

- 사용자 ID 또는 이메일 ID로 사용자를 찾기
```bash
user = User.find(123)
or
user = User.find_by(email: 'user@example.com')
or
user = User.where("username = 'root'")
or
user = User.find(1)
```

- 비밀번호 재설정
```bash
user.password='new_password'
user.password_confirmation='new_password'
```

- GitLab은 사용자가 비밀번호를 변경했음을 알리는 이메일
```bash
user.send_only_admin_changed_your_password_notification!
```

- 변경 사항을 저장
```bash
user.save!
```

## CI/CD Pipline

### 참고
- GitLab CI/CD variables
  - https://docs.gitlab.com/ee/ci/variables/README.html
- 사전 정의된 변수
  - https://docs.gitlab.com/ee/ci/variables/predefined_variables.html
- 자동 DevOps <- 삽질필요
  - https://docs.gitlab.com/ee/topics/autodevops/

### Spring boot 빌드 후 docker push 스크립트
- GitLab 은 .gitlab-ci.yml (기본파일명이며 Gitlab 설정에서 바꿀수 있음)
- $HARBOR_PASSWORD 는 GitLab 프로젝트에 등록된 사용자 환경변수, 그 외에는 사전 정의된 변수.
- build step의 결과물을 package step에 전달하기 위해 cache push/pull 사용

```bash
image: docker:19.03.12

services:
  - docker:19.03.12-dind

variables:
  GRADLE_OPTS: "-Dorg.gradle.daemon=false"
  DOCKER_HOST: tcp://docker:2376
  DOCKER_TLS_CERTDIR: "/certs"

before_script:
  - uname -a

stages:
  - build
  - package

build:
  image: gradle:latest
  stage: build
  script:
    - gradle bootJar
  cache:
    key: "$CI_COMMIT_REF_NAME"
    policy: push
    paths:
      - build/libs

package:
  stage: package
  script:
    - docker login -u ivymanager -p $HARBOR_PASSWORD gitlab.192-168-1-158.nip.io
    - docker build --cache-from gitlab.192-168-1-158.nip.io/library/demo:latest --tag gitlab.192-168-1-158.nip.io/library/demo:$CI_COMMIT_SHA --tag gitlab.192-168-1-158.nip.io/library/demo:latest .
    - docker push gitlab.192-168-1-158.nip.io/library/demo:$CI_COMMIT_SHA
    - docker push gitlab.192-168-1-158.nip.io/library/demo:latest
  cache:
    key: "$CI_COMMIT_REF_NAME"
    policy: pull
    paths:
      - build/libs
```
