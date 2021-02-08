# nstall GitLab on Linux (CentOS7)
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
