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
    - docker login -u ivymanager -p $HARBOR_PASSWORD harbor.192-168-1-156.nip.io
    - docker build --cache-from harbor.192-168-1-156.nip.io/library/demo:latest --tag harbor.192-168-1-156.nip.io/library/demo:$CI_COMMIT_SHA --tag harbor.192-168-1-156.nip.io/library/demo:latest .
    - docker push harbor.192-168-1-156.nip.io/library/demo:$CI_COMMIT_SHA
    - docker push harbor.192-168-1-156.nip.io/library/demo:latest
  cache:
    key: "$CI_COMMIT_REF_NAME"
    policy: pull
    paths:
      - build/libs
```
