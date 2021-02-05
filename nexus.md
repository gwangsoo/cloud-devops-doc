# Nexus maven repository 설치

## download
```bash
wget http://www.sonatype.org/downloads/nexus-latest-bundle.tar.gz --no-check-certificate
```

## 압축해제
```bash
tar xvfz nexus-latest-bundle.tar.gz
```

## 설정
- RUN_AS_USER 에 실행시킬 계정을 넣어준다.

```bash
vi nexus-2.14.20-02/bin/nexus
```

## 실행
```bash
./nexus-2.14.20-02/bin/nexus start
```

## 로그확인
```bash
tail -f ./nexus-2.14.20-02/logs/wrapper.log
```

## 방화벽 오픈
```bash
sudo firewall-cmd --permanent --add-port=8081/tcp
sudo firewall-cmd --reload
```

## 홈페이지 접속 확인
```bash
http://192.168.1.160:8081/nexus
```
