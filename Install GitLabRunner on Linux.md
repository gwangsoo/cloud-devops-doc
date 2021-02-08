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

[admin@runner ~]$ sudo gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=56398 revision=132560ae version=13.9.0~beta.142.g132560ae
Running in system-mode.

Enter the GitLab instance URL (for example, https://gitlab.com/):
http://192.168.1.158/
Enter the registration token:
p4vx_oqpa7hptDtqQYt6
Enter a description for the runner:
[runner.ivycomtech.cloud]:
Enter tags for the runner (comma-separated):
ivy-ci-runner
Registering runner... succeeded                     runner=p4vx_oqp
Enter an executor: docker-ssh+machine, custom, docker, docker-ssh, virtualbox, docker+machine, parallels, shell, ssh, kubernetes:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

https://runner.ivycomtech.cloud/
