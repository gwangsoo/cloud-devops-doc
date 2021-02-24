# Installing Rancher (Docker)
- Unit 2.1.1
- https://academy.rancher.com/asset-v1:RANCHER+K101+2019+type@asset+block@Handout-2.1.1.pdf

## Installing Rancher
Rancher는 모든 설치에 대해 단일 Docker 컨테이너를 사용하는 샌드 박스 방법과 RKE에 설치하는 HA 방법의 두 가지 설치 방법을 제공합니다.
Docker 방법은 HA 방법과 호환되지 않습니다. 현재로서는 한 곳에서 다른 곳으로 마이그레이션 할 수있는 방법이 없으므로 프로덕션 클러스터를 배포하고 나중에 HA 버전으로 마이그레이션하려는 경우 모든 다운 스트림 클러스터 및 구성을 다시 만들어야합니다.
따라서 Rancher 설치가 어떻게 될지 고려하는 것이 중요합니다. 테스트 드라이브에 사용하고 있지만 프로덕션에서 사용할 수 있다고 생각하는 경우 HA 방법으로 시작하고 단일 노드 설치를 사용하는 것이 가장 좋습니다.

### Common Parameters
컨테이너를 시작할 때 최소한 다음 옵션을 전달합니다:
- -d to daemonize
- -p 80:80 -p 443:443 to pass through ports 80 and 443
- --restart=unless-stopped - 백업 및 업그레이드 프로세스를 수행하려면 컨테이너를 중지해야합니다. 항상 사용되며 멈추지 않습니다. 강제로 kill 시켜야 재시작하지 않습니다.

### SSL Considerations
Rancher 서버에 대한 액세스를 보호하는 방법에는 네 가지 옵션이 있습니다.

#### Rancher-Generated Self-Signed Certs
기본 옵션은 가장 쉽고 사용자가 필요로하지 않습니다. Rancher가 처음 시작될 때 제공된 SSL 구성 옵션이없는 경우 자체 서명 된 인증서를 생성하고이를 사용하여 액세스를 보호합니다.

#### BYO Self-Signed Certs
자체 인증서를 생성하여 Docker 볼륨으로 컨테이너에 연결할 수도 있습니다. 세 개의 파일이 필요합니다.
- Private Key
- Full Chain Certificate
- CA Certificate

#### BYO Real Certs
다른 사람이 액세스하는 공용 환경에서 Rancher를 실행하는 경우, 알 수없는 기관에서 서명 한 인증서에서 오는 보안 경고를 방지하기 위해 실제 인증서로 보안을 설정할 수 있습니다. 이를 위해서는 두 개의 파일 만 필요합니다.
- Private Key
- Full Chain Certificate

이 단계는 자체 서명 된 인증서를 가져 오는 것과 동일하지만 CA 인증서에 대한 볼륨을 만들 필요가 없다는 점이 다릅니다. Rancher는 컨테이너와 함께 제공되는 인증서 저장소의 CA 인증서를 사용합니다.

#### Auto-Generated Let's Encrypt Certificate
Rancher는 http-01 challenge format을 통해 Let's Encrypt에서 직접 인증서를 요청할 수 있습니다. 이것은 Rancher 서버에 직접, 포트 포워딩을 통해 또는 공용로드 밸런서를 통해 인터넷에서 연결할 수있는 환경에서 작동합니다.
Rancher 서버에는 Rancher 서버 호스트의 IP를 가리키는 DNS 이름이 있어야하며 포트 80에서 도달 할 수 있어야합니다. 문제는 어디에서나 올 수 있으므로 포트 80은 전 세계에 공개되어야합니다.
Let's Encrypt에서 인증서 요청을 시작하려면 Rancher 서버 컨테이너에 대한 시작 옵션으로 --acmedomain을 제공합니다.
이것은 Docker에 대한 플래그가 아니므로 이미지 뒤에 배치하면 플래그가 Rancher 서버 컨테이너 진입 점으로 전달되고 Rancher가 요청을합니다.

### Advanced Options
API 감사 로깅, 사용자 지정 CA 인증서, 수정 된 TLS 설정, 에어 갭 환경 및 영구 데이터와 같은 기능을 활성화하여 컨테이너 시작 프로세스에 전달할 수있는 다른 고급 옵션이 있습니다. 이에 대해서는 아래 경로에서 자세히 다루지 만 여기서는 그중 하나를 다룰 것입니다.
https://rancher.com/docs/rancher/v2.x/en/installation/other-installation-methods/single-node-docker/advanced/

#### Bind-Mounted Volume for Persistent Data
Rancher는 영구 데이터에 대한 Docker 볼륨을 생성하고 컨테이너 내부의 /var/lib/rancher 에 마운트합니다. 원하는 경우 호스트에서이 위치로 디렉토리를 대신 바인딩 할 수 있습니다. 바인드 마운트 된 디렉토리를 사용하면 Docker 볼륨을 사용할 때보 다 백업 및 복원이 더 쉽습니다.

## Sample
```bash
docker run -d --restart=unless-stopped -p 80:80 -p 443:443 -v /opt/rancher:/var/lib/rancher rancher/rancher:v2.4.1
Unable to find image 'rancher/rancher:v2.4.1' locally
v2.4.1: Pulling from rancher/rancher
5bed26d33875: …
Digest: sha256:7796eb2a851ad00509052d012afa5e8f74c9e84da51c798f14850e0255160558
Status: Downloaded newer image for rancher/rancher:v2.4.1 430a461add079f67ed549b83b48a79244e9bce925ee0d9bce5bc8730a413898f
```

## References
- Single-node Docker Installation
  - https://rancher.com/docs/rancher/v2.x/en/installation/otherinstallation-methods/single-node-docker/
- Advanced Installation Options
  - https://rancher.com/docs/rancher/v2.x/en/installation/otherinstallation-methods/single-node-docker/advanced/
