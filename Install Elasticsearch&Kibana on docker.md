# Install Elasticsearch&Kibana on docker
Rancher에서 사용 할 Elasticsearch 와 Kibana 을 설치합니다.
https://rancher.com/docs/rancher/v2.x/en/logging/v2.5/#configuring-the-logging-application

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/docker.html
https://www.elastic.co/guide/en/kibana/current/docker.html

## 전제사항
- Docker
- Docker-compose

## Single node 실행
1. yml 및 data 폴더 생성
   ```bash
   mkdir -p ./eks/data
   chmod 777 ./eks/data
   cd ./eks
   ```

2. docker-compose.yml 파일 생성
   elasticsearch 와 kibana 의 버전 호환성 확인해야함. 왠만하면 같은 버전으로...
   ```bash
   cat <<EOF > docker-compose.yml
   version: '2.2'
   services:
     elasticsearch:
       image: docker.elastic.co/elasticsearch/elasticsearch:7.11.2
       environment:
         - discovery.type=single-node
         - bootstrap.memory_lock=true
         - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
       ulimits:
         memlock:
           soft: -1
           hard: -1
       volumes:
         - ./data:/usr/share/elasticsearch/data
       ports:
         - 9200:9200
         - 9300:9300

     kibana:
       image: docker.elastic.co/kibana/kibana:7.11.2
       ports:
         - 5601:5601
   EOF
   ```

3. run
   ```bash
   docker-compose up -d
   ```

4. stop
   ```bash
   docker-compose down -v
   ```

5. Rancher Cluster Logging 설정
   - Rancher의 Logging 이 배포되어 있어야 함.
     - https://rancher.com/docs/rancher/v2.x/en/logging/v2.5/#configuring-the-logging-application
     - FluentBit, FluentD 가 클러스터 기반으로 준비 됨.
     - ![image](https://user-images.githubusercontent.com/7520111/112265694-3bca9200-8cb6-11eb-8efb-eaeb31f5f433.png)
   - ClusterOutputs 생성
     ```bash
     kubectl get ClusterOutput -n cattle-logging-system -o wide
     
     cat <<EOF | kubectl apply -f -
     apiVersion: logging.banzaicloud.io/v1beta1
     kind: ClusterOutput
     metadata:
       name: default-elasticsearch
       namespace: cattle-logging-system
     spec:
       elasticsearch:
         host: 192.168.1.182
         index_name: ivycloud-syslog
         port: 9200
         scheme: http
     EOF
     ```
   - ClusterFlows 생성
     ```bash
     kubectl get ClusterFlows -n cattle-logging-system
     
     cat <<EOF | kubectl apply -f -
     apiVersion: logging.banzaicloud.io/v1beta1
     kind: ClusterFlow
     metadata:
       name: default-flow
       namespace: cattle-logging-system
     spec:
       globalOutputRefs:
       - default-elasticsearch
     EOF
     ```

6. Elasticsearch 상태확인
   - 상태확인
     ```bash
     http://nodef.ivycomtech.cloud:9200/_cat/health?v
     
     epoch      timestamp cluster        status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
     1616567856 06:37:36  docker-cluster yellow          1         1      8   8    0    0        1             0                  -                 88.9%
     ```
   - node 확인
     ```bash
     http://nodef.ivycomtech.cloud:9200/_cat/nodes?v
     
     ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role  master name
     172.20.0.2           72          73   0    0.00    0.01     0.05 cdhilmrstw *      a4281b67158d
     ```
   - indices 확인
     ```bash
     http://nodef.ivycomtech.cloud:9200/_cat/indices?pretty
     
     yellow open ivycloud-syslog                 acuG8orTQQCKPDLk2vg4DQ 1 1 12937   0 3.8mb 3.8mb
     green  open .apm-custom-link                hBsDZE2HQsqiccu0ETL9rA 1 0     0   0  208b  208b
     green  open .kibana_task_manager_1          Nh4bgjwtQHiq2RLwu6L-QQ 1 0     8 543 112kb 112kb
     green  open .kibana-event-log-7.11.2-000001 TodqC_0QT2uvzC0iQYuOZA 1 0     1   0 5.6kb 5.6kb
     green  open .apm-agent-configuration        A22bQsCGSdOQ4o0RaBosdQ 1 0     0   0  208b  208b
     green  open .async-search                   kFVV_3ZsSY-fD1d3beq28g 1 0     0 204 2.2mb 2.2mb
     green  open .kibana_1                       AjPR5n96Rd-RDZMiRKutOQ 1 0    48 118 2.1mb 2.1mb
     ```

7. Kibana 설정
   - https://epicarts.tistory.com/75

8. 기타
   - 로그가 elasticsearch 로 전송안되는 경우
     - fluentd 로그 확인
       ```bash
       kubectl exec -n cattle-logging-system rancher-logging-fluentd-0 -- cat /fluentd/log/out
       ```
     - elasticsearch log 확인
       ```bash
       docker-compose logs -f elasticsearch
       ```
       - elasticsearch log에서 NoShardAvailableActionException 발생했을 것이다.
         - 싱글노드(마스터노드)로만 운용하면 생기는 문제이므로 멀티노드로 데이터 노드를 운용해야 함.
         - 싱글노드로 계속 운영하는 경우 로그가 안올라 오면
           - elasticsearch 재기동
             ```bash
             docker-compose restart elasticsearch
             ```
           - fluentd 스테이트풀셋 재기동
             ```bash
             kubectl -n cattle-logging-system rollout restart statefulset.apps/rancher-logging-fluentd
             ```
