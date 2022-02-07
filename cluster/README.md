## Redis Cluster Template Guide
Redis Operator를 이용하는 Custom Resource(RedisCluster) 생성 template
## Prerequisite
- k8s cluster(v1.11+)
- [redis-operator(v0.10.0)](https://ot-container-kit.github.io/redis-operator/)


## Cluster Template
- 생성
    ```shell
    kubectl apply -f redis-cluster-template.yaml
    ```
- 확인
    - 다음 명령어 입력 후 출력여부 확인
    ```shell
    kubectl get clustertemplate | grep ot-container-kit-redis-cluster-template
    ```
- 삭제
    ```shell
    kubectl delete -f redis-cluster-template.yaml
    ```

## Template Instance
- 생성
    ```shell
    kubectl apply -f redis-cluster-instance.yaml
    ```
- 확인
    - 다음 명령어로 확인 후 { APP_NAME }-leader-0, { APP_NAME }-follower-0등의 NAME확인
    ```shell
    kubectl get pods
    ```
- 삭제
    ```shell
    kubectl delete -f redis-cluster-instance.yaml
    ```