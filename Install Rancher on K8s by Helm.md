# Install Rancher on K8s by Helm (Helm 으로 쿠버네티스에 Rancher 설치하기)
https://rancher.com/docs/rancher/v2.x/en/installation/install-rancher-on-k8s/

## 설치전 요구사항
- docker
- kubernetes (2021.02.25 현재 Rancher가 설치되는 버전은 v1.19.8 임)
- helm
```bash
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```
or
```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

## 설치

### Add the Helm Chart Repository
```bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
```

### Create a Namespace for Rancher
```bash
kubectl create namespace cattle-system
```

### Choose your SSL Configuration
- 아래 3가지 방법이 있으나 일단 건너 뜀.
  - Rancher-generated TLS certificate
  - Let’s Encrypt
  - Bring your own certificate

### Install cert-manager
```bash
# Install the CustomResourceDefinition resources separately
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.4/cert-manager.crds.yaml

# **Important:**
# If you are running Kubernetes v1.15 or below, you
# will need to add the `--validate=false` flag to your
# kubectl apply command, or else you will receive a
# validation error relating to the
# x-kubernetes-preserve-unknown-fields field in
# cert-manager’s CustomResourceDefinition resources.
# This is a benign error and occurs due to the way kubectl
# performs resource validation.

# Create the namespace for cert-manager
kubectl create namespace cert-manager

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.0.4
```

잘 되나 확인
```bash
kubectl get pods --namespace cert-manager

NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-5c6866597-zw7kh               1/1     Running   0          2m
cert-manager-cainjector-577f6d9fd7-tr77l   1/1     Running   0          2m
cert-manager-webhook-787858fcdb-nlzsq      1/1     Running   0          2m
```

### Install Rancher with Helm and Your Chosen Certificate Option
#### Rancher-generated certificates
```bash
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=rancher.ivycomtech.cloud
```
Rancher가 롤아웃 될때가지 대기
```bash
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```
#### Let's encrypt
```bash
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=letsEncrypt \
  --set letsEncrypt.email=me@example.org
```
Rancher가 롤아웃 될때가지 대기
```bash
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```
#### Certificates from files
If you are installing an alpha version, Helm requires adding the --devel option to the command.
```bash
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=secret
```
If you are using a Private CA signed certificate , add --set privateCA=true to the command:
```bash
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set ingress.tls.source=secret \
  --set privateCA=true
```

### Verify that the Rancher Server is Successfully Deployed
- 설치완료 대기
```bash
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```
- 설치 결과 확인
```bash
kubectl -n cattle-system get deploy rancher
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rancher   3         3         3            3           3m
```
- 로그확인
```bash
kubectl -n cattle-system logs -f -lapp=rancher --all-containers=true
```
- 오류
  - fatal: unable to access 'https://git.rancher.io/charts/': Could not resolve host: git.rancher.io
    - rancher가 호환되는 docker 버전이 아니면 docker 재설치
    - https://rancher.com/support-maintenance-terms/
    ```bash
    curl -fsSL https://get.docker.com | sh
    ```
    또는 docker 버전 수정
    ```bash
    systemctl stop docker && systemctl disable docker
    
    yum list installed | grep docker
    
    yum update docker-ce-19.03.15 docker-ce-cli-19.03.15
    or
    yum erase -y docker-ce docker-ce-cli
    yum install -y docker-ce-19.03.15 docker-ce-cli-19.03.15
    
    systemctl enable docker && systemctl start docker
    ```
    또는 k8s 버전 수정
    ```bash
    yum list installed | grep kubelet
    
    yum update -y --disableexcludes=kubernetes kubeadm-1.19.6-0 kubectl-1.19.6-0 kubelet-1.19.6-0 kubernetes-cni
    or
    yum erase -y kubeadm kubectl kubelet kubernetes-cni
    yum install -y --disableexcludes=kubernetes kubeadm-1.19.6-0 kubectl-1.19.6-0 kubelet-1.19.6-0 kubernetes-cni
    
    kubeadm join 192.168.1.157:26443 --token xor8z4.fe1vf24slczx1p4i \
    --discovery-token-ca-cert-hash sha256:47d1682c0553aedb46863997a981572fdd25a4069038be1827c0ab482dce7aac
    ```
    
- rancher uninstall
```bash
helm uninstall rancher --namespace cattle-system

kubectl -n cattle-system delete pods,daemonset,service,replicaset,deployment --all
```

### Nodeport 서비스 추가
ingress 없는 경우 nodeport 를 등록하여 수동을 접속하도록 한다. ingress 가 있는 경우 ingress 를 통하여 접속됨.

```bash
vi rancher-svc-nodeport.yaml

apiVersion: v1
kind: Service
metadata:
  name: rancher-nodeport
spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30080
  - name: https
    port: 443
    protocol: TCP
    targetPort: 443
    nodePort: 30443
  selector:
    app: rancher
```
적용
```bash
kubectl apply -f rancher-svc-nodeport.yaml -n cattle-system
```
확인
```bash
kubectl get svc -n cattle-system

NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
rancher            ClusterIP   10.101.228.230   <none>        80/TCP,443/TCP               169m
rancher-nodeport   NodePort    10.109.27.125    <none>        80:30080/TCP,443:30443/TCP   52m
rancher-webhook    ClusterIP   10.98.82.5       <none>        443/TCP                      167m
```
