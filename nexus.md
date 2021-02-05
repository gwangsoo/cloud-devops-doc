# Nexus maven repository 설치

## 3.x nexus 설치
- https://sysops.tistory.com/149
- https://m.blog.naver.com/jooda99/221045957905

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

### 폴더이름 변경
```bash
sudo mv nexus-3.29.2-02 nexus
sudo mv sonatype-work nexusdata
```

### 실행계정 생성 및 권한 부여
```bash
sudo useradd --system --no-create-home nexus
sudo chown -R nexus:nexus /opt/nexus
sudo chown -R nexus:nexus /opt/nexusdata
```

### runtime vm 옵션 확인
```bash
vim /opt/nexus/bin/nexus.vmoptions

-Xms2703m
-Xmx2703m
-XX:MaxDirectMemorySize=2703m
-XX:+UnlockDiagnosticVMOptions
-XX:+LogVMOutput
-XX:LogFile=../nexusdata/nexus3/log/jvm.log
-XX:-OmitStackTraceInFastThrow
-Djava.net.preferIPv4Stack=true
-Dkaraf.home=.
-Dkaraf.base=.
-Dkaraf.etc=etc/karaf
-Djava.util.logging.config.file=etc/karaf/java.util.logging.properties
-Dkaraf.data=../nexusdata/nexus3
-Dkaraf.log=../nexusdata/nexus3/log
-Djava.io.tmpdir=../nexusdata/nexus3/tmp
-Dkaraf.startLocalConsole=false
```

### 실행 계정 등록
```bash
sudo vim /opt/nexus/bin/nexus.rc
run_as_user="nexus"
```

### 속성변경
```bash
sudo vim /opt/nexus/etc/nexus-default.properties
-> Change application-host=0.0.0.0 to application-host=127.0.0.1.
```

### file descriptors 설정
```bash
sudo vim /etc/security/limits.conf
nexus - nofile 65536
```

### system 서비스 등록
```bash
vim /etc/systemd/system/nexus.service

[Unit]
Description=Nexus Service
After=syslog.target network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/opt/nexus/bin/nexus start
ExecStop=/opt/nexus/bin/nexus stop
User=nexus
Group=nexus
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### 서비스 적용 및 실행
```bash
sudo systemctl daemon-reload
sudo systemctl enable nexus.service
sudo systemctl start nexus.service
sudo systemctl status nexus.service
```

### nginx 설치
```bash
sudo yum install -y epel-release
sudo yum repolist
sudo yum install nginx
```

### nginx 실행
```bash
sudo systemctl enable nginx
sudo systemctl status nginx
sudo systemctl start nginx

### 인증서 설치
```bash
sudo yum install certbot python2-certbot-nginx
certbot --nginx
```

### nginx 속성 변경
```bash
sudo vim /etc/nginx/nginx.conf

location / {
      proxy_pass "http://127.0.0.1:8081";
      proxy_set_header        Host $host;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;
      proxy_set_header        X-Forwarded-Ssl on;
      proxy_read_timeout      300;
      proxy_connect_timeout   300;
 }
```

### nginx 실행
```bash
sudo systemctl restart nginx
```

### admin 비번확인
```bash
cat /opt/nexusdata/nexus3/admin.password
```

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
