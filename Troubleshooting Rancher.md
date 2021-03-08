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

