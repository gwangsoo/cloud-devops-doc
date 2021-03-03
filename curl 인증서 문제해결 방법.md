# curl 에 신뢰하는 인증기관 인증서(CA Cert) 추가하기
https://www.lesstif.com/gitbook/curl-ca-cert-15892500.html

curl 은 기본적으로 https 사이트의 SSL 인증서를 검증한다. 인증 기관의 인증서 목록이 없거나 모르는 기관에서 발급한 인증서일 경우 다음과 같은 인증서 검증 에러를 발생시키고 동작을 중지하게 된다.
```bash
cURL error 60: SSL certificate problem: unable to get local issuer certificate (see http://curl.haxx.se/libcurl/c/libcurl-errors.html)
```

## 해결방법

### 검증을 하지 않는 옵션인 -k(--insecure) 옵션을 주고 curl 을 구동
```bash
curl -k -L google.com
```
```bash
curl --insecure -L google.com
```

### 인증기관 목록 추가
1. curl 을 실행시 -v 옵션으로 CA 목록을 어디에서 가져오는지 위치를 확인
- RHEL/CentOS 는 아래와 같이 /etc/pki/tls/certs/ca-bundle.crt 에서 CA 목록을 로딩함
```bash
curl -v  https://google.com
 
* About to connect() to google.com port 443 (#0)
*   Trying 74.125.128.139... connected
* Connected to google.com (74.125.128.139) port 443 (#0)
* Initializing NSS with certpath: sql:/etc/pki/nssdb
*   CAfile: /etc/pki/tls/certs/ca-bundle.crt
  CApath: none
```
- Ubuntu 는 /etc/ssl/certs/ca-certificates.crt 또는  /etc/ssl/certs 디렉터리에서 CA 목록 로딩
```bash
* Rebuilt URL to: https://google.com/
* Trying 172.217.25.110...
* Connected to google.com (172.217.25.110) port 443 (#0)
* found 173 certificates in /etc/ssl/certs/ca-certificates.crt
* found 694 certificates in /etc/ssl/certs
* ALPN, offering http/1.1
* SSL connection using TLS1.2 / ECDHE_ECDSA_AES_128_GCM_SHA256
* server certificate verification OK
```
- Windows 에서 curl.exe 를 사용시 다음 순서대로 ca-bundle.crt 를 찾음
```bash
application's directory
current working directory
Windows System directory (e.g. C:\windows\system32)
Windows Directory (e.g. C:\windows)
all directories along %PATH%
```

2. 서버 인증서 및 인증기관 인증서(CA certificate)를  BASE64 로 저장한 내용을 ca-bundle.crt 에 추가
3. 또는 curl 실행시 --cacert  옵션으로 CA certificate 를 지정할 수 있음
```bash
curl -v --cacert myca-bundle.crt https://google.com
```

### CA 인증서 파일 갱신
curl 홈페이지에서 최신 CA 인증서 목록을 다운받아서 기존 파일에 덮어써도 된다.
1. https://curl.haxx.se/ca/cacert.pem 에서 인증서를 다운받는다.
```bash
wget --no-check-certificate https://curl.haxx.se/ca/cacert.pem
```
```bash
curl -k -O https://curl.haxx.se/ca/cacert.pem
```
2. url -v 옵션으로 CA 인증서 목록 파일의 위치를 확인한 후에 예전 파일은 백업하고 다운받은 인증서 파일을 덮어쓴다.
