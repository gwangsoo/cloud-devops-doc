# Nexus maven repository 설치

## 3.x nexus 설치
- https://gsk121.tistory.com/429

### jdk8 설치
```bash
sudo yum update -y
sudo yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel
```

### download
```bash
cd /opt
sudo wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
```

### 압축해제
```bash
sudo tar -xvzf latest-unix.tar.gz
```

### 실행
```bash
./nexus-3.29.2-02/bin/nexus start
./nexus-3.29.2-02/bin/nexus status
./nexus-3.29.2-02/bin/nexus stop
```

### 로그확인
```bash
tail -f ./sonatype-work/nexus3/log/*.log
```

### 방화벽 오픈
```bash
sudo firewall-cmd --permanent --add-port=8081/tcp
sudo firewall-cmd --reload
```

## file descriptor 제한수 수정
```bash
ulimit -a | grep 'open files'

sudo vim /etc/security/limits.conf

admin - nofile 65536
```

### 홈페이지 접속 확인
```bash
http://192.168.1.160:8081/nexus
```

### 로그인 비밀번호
- 계정
admin
- 초기비밀번호
cat ./sonatype-work/nexus3/admin.password

## 2.x nexus 설치
### download
```bash
cd /opt
wget http://www.sonatype.org/downloads/nexus-latest-bundle.tar.gz --no-check-certificate
```

### 압축해제
```bash
tar xvfz nexus-latest-bundle.tar.gz
```

### 설정
- RUN_AS_USER 에 실행시킬 계정을 넣어준다.

```bash
vi nexus-2.14.20-02/bin/nexus
```

### 실행
```bash
./nexus-2.14.20-02/bin/nexus start
```

### 로그확인
```bash
tail -f ./nexus-2.14.20-02/logs/wrapper.log
```

### 방화벽 오픈
```bash
sudo firewall-cmd --permanent --add-port=8081/tcp
sudo firewall-cmd --reload
```

### 홈페이지 접속 확인
```bash
http://192.168.1.160:8081/nexus
```

### 로그인 비밀번호
초기비밀번호 admin / admin1234
