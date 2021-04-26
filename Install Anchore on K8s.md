# Install Sonarqube on Kubernetes
- Docker Image의 취약점을 분석하고 조치하기 위한 open source
- https://engine.anchore.io/docs/quickstart/

## 설치준비
- docker 설치
  - Docker v1.12 이상
  - https://github.com/gwangsoo/cloud-devops-doc/blob/master/Install%20docker%20on%20linux.md
- docker-compose 설치
  - https://docs.docker.com/compose/install/
  ```bash
  curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  chmod +x /usr/local/bin/docker-compose
  docker-compose --version
  ```

## 설치
1. docker-compose.yaml 파일을 다운로드 및 시작
   ```bash
   mkdir anchore
   cd anchore
   curl -O https://engine.anchore.io/docs/quickstart/docker-compose.yaml
   docker-compose up -d
   ```
2. 서비스 가용성 확인
   - 컨테이너가 실행 중인지 확인
     ```bash
     docker-compose ps
     ```
   - Anchore Engine 서비스의 상태를 가져 오는 명령을 실행
     ```bash
     docker-compose exec api anchore-cli system status
     ```
   - 피드 동기화 상태를 확인
     Anchore Engine을 처음 실행하면 취약성 데이터가 엔진에 동기화되는 데 약간의 시간 (네트워크 속도에 따라 10 분 이상)이 걸립니다. 최상의 경험을 위해 계속하기 전에 핵심 취약성 데이터 피드가 완료 될 때까지 기다리십시오. CLI를 사용하여 피드 동기화 상태를 확인할 수 있습니다.
     ```bash
     docker-compose exec api anchore-cli system feeds list
     
     Feed                   Group                  LastSync                          RecordCount        
     vulnerabilities        alpine:3.10            2020-04-27T19:49:45.186409        1725               
     vulnerabilities        alpine:3.11            2020-04-27T19:49:59.993730        1904               
     ...
     ...
     vulnerabilities        ubuntu:20.04           pending                           None
     ```
     모든 취약성 그룹에 대해 RecordCount 값> 0이 표시되면 시스템이 완전히 채워지고 취약성 결과를 표시 할 준비가됩니다. 피드 동기화는 증분이므로 다음에 Anchore Engine을 시작할 때 즉시 준비됩니다. CLI 도구에는 피드가 성공적인 동기화를 완료 할 때까지 차단하는 유용한 유틸리티가 포함되어 있습니다.
     ```bash
     docker-compose exec api anchore-cli system wait
     
     Starting checks to wait for anchore-engine to be available timeout=-1.0 interval=5.0
     API availability: Checking anchore-engine URL (http://localhost:8228)...
     API availability: Success.
     Service availability: Checking for service set (catalog,apiext,policy_engine,simplequeue,analyzer)...
     Service availability: Success.
     Feed sync: Checking sync completion for feed set (vulnerabilities)...
     Feed sync: Checking sync completion for feed set (vulnerabilities)...
     ...
     ...
     Feed sync: Success.
     ```
