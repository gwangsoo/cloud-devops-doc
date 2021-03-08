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
