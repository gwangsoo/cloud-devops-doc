# Operation Rancher

## pod 상태 및 로그 확인
https://rancher.com/docs/rancher/v2.x/en/troubleshooting/rancherha/#pod-details

- pod 상태 확인
```bash
kubectl -n cattle-system get pods -l app=rancher -o wide
```

- pod 정보 확인
```bash
kubectl -n cattle-system describe pods -l app=rancher
```

- pod 로그 확인
```bash
kubectl -n cattle-system logs -l app=rancher
```

- Namespace event 확인
```bash
kubectl -n cattle-system get events
```

- Ingress 확인
```bash
kubectl -n cattle-system get ingress
```

- Ingress 로그 확인
```bash
kubectl -n ingress-nginx logs -l app=ingress-nginx
```

- 리더 확인
```bash
kubectl -n kube-system get configmap cattle-controllers -o jsonpath='{.metadata.annotations.control-plane\.alpha\.kubernetes\.io/leader}'

{"holderIdentity":"rancher-85c9cf85ff-qt2k5","leaseDurationSeconds":45,"acquireTime":"2021-03-04T06:29:41Z","renewTime":"2021-03-08T00:43:05Z","leaderTransitions":0}
```

## docker 엔진 상태 및 로그 확인
- docker 서비스 상태 확인
```bash
systemctl status docker.service
```

- docker 서비스 로그 확인
```bash
journalctl -u docker.service -f
or
journalctl -u docker.service | less
```

## node 확인
- 노드 상태 확인
```bash
kubectl get nodes -o wide

NAME                     STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
nodea.ivycomtech.cloud   Ready    master   3d19h   v1.19.6   192.168.1.161   <none>        CentOS Linux 7 (Core)   3.10.0-1160.15.2.el7.x86_64   docker://20.10.4
nodeb.ivycomtech.cloud   Ready    master   3d19h   v1.19.6   192.168.1.162   <none>        CentOS Linux 7 (Core)   3.10.0-1160.15.2.el7.x86_64   docker://20.10.4
nodec.ivycomtech.cloud   Ready    master   3d19h   v1.19.6   192.168.1.163   <none>        CentOS Linux 7 (Core)   3.10.0-1160.15.2.el7.x86_64   docker://20.10.4
noded.ivycomtech.cloud   Ready    <none>   3d19h   v1.19.6   192.168.1.164   <none>        CentOS Linux 7 (Core)   3.10.0-1160.15.2.el7.x86_64   docker://20.10.4
```

- node 정보 확인
Conditions 의 NetworkUnavailable / MemoryPressure / DiskPressure / PIDPressure / Ready 정보 확인
```bash
kubectl describe node nodea.ivycomtech.cloud | less

Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Thu, 04 Mar 2021 15:04:00 +0900   Thu, 04 Mar 2021 15:04:00 +0900   WeaveIsUp                    Weave pod has set this
  MemoryPressure       False   Mon, 08 Mar 2021 10:04:16 +0900   Thu, 04 Mar 2021 14:54:11 +0900   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Mon, 08 Mar 2021 10:04:16 +0900   Thu, 04 Mar 2021 14:54:11 +0900   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Mon, 08 Mar 2021 10:04:16 +0900   Thu, 04 Mar 2021 14:54:11 +0900   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Mon, 08 Mar 2021 10:04:16 +0900   Thu, 04 Mar 2021 15:41:57 +0900   KubeletReady                 kubelet is posting ready status
```

- node 의 NetworkUnavailable / MemoryPressure / DiskPressure / PIDPressure / Ready 문제 상태 확인
```bash
kubectl get nodes -o go-template='{{range .items}}{{$node := .}}{{range .status.conditions}}{{if ne .type "Ready"}}{{if eq .status "True"}}{{$node.metadata.name}}{{": "}}{{.type}}{{":"}}{{.status}}{{"\n"}}{{end}}{{else}}{{if ne .status "True"}}{{$node.metadata.name}}{{": "}}{{.type}}{{":"}}{{.status}}{{"\n"}}{{end}}{{end}}{{end}}{{end}}'
```

## etcd 확인
- 
```bash
kubectl get nodes
```
```bash
docker ps -a -fname=etcd
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES
47c4bc9889c8        0369cf4303ff           "etcd --advertise-cl…"   3 days ago          Up 3 days
             k8s_etcd_etcd-nodea.ivycomtech.cloud_kube-system_f9f32943ba4de64447dda689075f9828_0
faac0c064195        k8s.gcr.io/pause:3.2   "/pause"                 3 days ago          Up 3 days
             k8s_POD_etcd-nodea.ivycomtech.cloud_kube-system_f9f32943ba4de64447dda689075f9828_0
```
```bash
docker logs 47c4bc9889c8
```

## control-plane 노드 확인
로그 데이터를 분석하기 전에 먼저 누가 리더인지 확인해야합니다.
로그 데이터를 가져 오기 위해 각 노드를 살펴볼 수 있지만 그중 하나가 문제의 원인 일 수 있으므로 모든 컨테이너에 대한 로그를 살펴 봐야합니다.

- endpoint 조회
```bash
kubectl get endpoints -n kube-system -o wide

NAME                      ENDPOINTS                                            AGE
kube-controller-manager   <none>                                               3d22h
kube-dns                  10.40.0.1:53,10.46.0.1:53,10.40.0.1:53 + 3 more...   3d22h
kube-scheduler            <none>                                               3d22h
```

- leader kube-controller-manager 확인
```bash
kubectl get endpoints -n kube-system kube-controller-manager \
-o jsonpath='{.metadata.annotations.control-plane\.alpha\.kubernetes\.io/leader}{"\n"}'
{"holderIdentity":"nodea.ivycomtech.cloud_90d894f1-3eed-411f-9e6d-807230a65520","leaseDurationSeconds":15,"acquireTime":"2021-03-04T06:40:35Z","renewTime":"2021-03-08T04:16:21Z","leaderTransitions":3}
```

- leader kube-scheduler 확인
```bash
kubectl get endpoints -n kube-system kube-scheduler \
-o jsonpath='{.metadata.annotations.control-plane\.alpha\.kubernetes\.io/leader}{"\n"}'
{"holderIdentity":"nodeb.ivycomtech.cloud_7e0330c7-68f3-4da2-8d99-69c7dfffb17c","leaseDurationSeconds":15,"acquireTime":"2021-03-04T06:41:09Z","renewTime":"2021-03-08T04:20:57Z","leaderTransitions":3}
```

- leader kube-apiserver 확인
각 노드에서 kube-apiserver container 를 찾아서 로그가 발생되는 node 를 보고 확인한다.
```bash
docker ps -a -fname=kube-apiserver

CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS               NAMES
e426dbc1bfd1        9ba91a90b7d1           "kube-apiserver --ad…"   3 days ago          Up 3 days
             k8s_kube-apiserver_kube-apiserver-nodea.ivycomtech.cloud_kube-system_74f52c6f3dee0f867726a9f75165f903_0
a0a8411e3639        k8s.gcr.io/pause:3.2   "/pause"                 3 days ago          Up 3 days
             k8s_POD_kube-apiserver-nodea.ivycomtech.cloud_kube-system_74f52c6f3dee0f867726a9f75165f903_0
```
```bash
docker logs -f e426dbc1bfd1
```
