# Install GitLabRunner on Linux(CentOS7)
https://docs.gitlab.com/runner/install/

## Git설치

- 설치된 git 버전 확인
```bash
git --version
```

- git 설치
```bash
sudo yum install git
```

## Docker 설치
https://docs.docker.com/engine/install/centos/

- Uninstall old versions
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

- yum-utils 설치
```bash
sudo yum install -y yum-utils
```

- Docker Repository 등록
```bash
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

- Docker Repository 활성화
```bash
sudo yum-config-manager --enable docker-ce-nightly docker-ce-test
```

- Docker Repository 비활성화 (설치때는 사용하지 않습니다.)
```bash
sudo yum-config-manager --disable docker-ce-nightly
```

- Docker 설치
```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

- Docker 설치 확인
```bash
yum list docker-ce --showduplicates | sort -r
```

- Docker 서비스 시작 및 종료
```bash
sudo systemctl start docker
sudo systemctl stop docker
```

## GitLab Runner 설치

- GitLab 저장소 추가
```bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash
```

- 최신버전 GitLab Runner 설치
```bash
export GITLAB_RUNNER_DISABLE_SKEL=true; sudo -E yum install gitlab-runner
```

- GitLab Runner 등록
```bash
sudo gitlab-runner register

[admin@runner ~]$ sudo gitlab-runner register -n \
--url http://gitlab.ivycomtech.cloud/ \
--registration-token vqxuHk32syKByEDrQQD7 \
--executor docker \
--description "My Docker Runner" \
--docker-image "docker:stable" \
--docker-privileged \
--docker-volumes "/certs/client"

Runtime platform                                    arch=amd64 os=linux pid=34430 revision=132560ae version=13.9.0~beta.142.g132560ae
Running in system-mode.
Registering runner... succeeded                     runner=vqxuHk32
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

## Gitlab runner 사설 인증서 설정
https://docs.gitlab.com/ce/ci/docker/using_docker_build.html

- 사설인증서 적용하려면 아래 "/etc/docker/certs.d" 폴더를 docker.runner 의 volume에 적용
  ```bash
  [root@runner ~]# ll -R /etc/docker/certs.d
  /etc/docker/certs.d:
  total 0
  drwxr-xr-x. 2 root root 138 Mar 30 15:59 harbor.192-168-1-156.nip.io
  drwxr-xr-x. 2 root root  91 Mar 25 17:32 harbor.ivycomtech.cloud
  
  /etc/docker/certs.d/harbor.192-168-1-156.nip.io:
  total 16
  -rw-r--r--. 1 root root 2057 Mar 30 15:53 ca.crt
  -rw-r--r--. 1 root root 2143 Mar 30 15:59 harbor.192-168-1-156.nip.io.cert
  -rw-r--r--. 1 root root 2143 Mar 30 15:56 harbor.192-168-1-156.nip.io.crt
  -rw-r--r--. 1 root root 3243 Mar 30 15:57 harbor.192-168-1-156.nip.io.key
  
  /etc/docker/certs.d/harbor.ivycomtech.cloud:
  total 12
  -rw-r--r--. 1 root root 2049 Mar 25 17:32 ca.crt
  -rw-r--r--. 1 root root 2126 Mar 25 17:32 harbor.ivycomtech.cloud.cert
  -rw-r--r--. 1 root root 3243 Mar 25 17:32 harbor.ivycomtech.cloud.key
  ```

- config.toml 파일 전체
```bash
vi /etc/gitlab-runner/config.toml

concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "My Docker Runner"
  url = "http://gitlab.ivycomtech.cloud/"
  token = "BCxAXNbKnyPVefjhxsXR"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "docker:19.03.12"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/certs/client", "/cache", "/etc/docker/certs.d:/etc/docker/certs.d"]
    shm_size = 0
```

## GitLab runner command
- 등록된 Runner 조회
  ```bash
  gitlab-runner list
  ```
- Runner 등록 취소
  ```bash
  gitlab-runner unregister --all-runners
  ```
