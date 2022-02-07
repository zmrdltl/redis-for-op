## Service Binding Guide
tomcat에 Redis의 연결정보(password) 주입을 위한 service binding객체 생성
## Prerequisite
- k8s cluster(v1.11+)
- [redis-operator(v0.10.0)](https://ot-container-kit.github.io/redis-operator/)
- [service-bidning-operator(v1.0.0)](https://redhat-developer.github.io/service-binding-operator/userguide/getting-started/installing-service-binding.html)
- redis-standalone custom resource(kind: Redis)
- tomcat(v8.5) deployment, svc

## Tomcat
- 생성
    ```bash
    kubectl apply -f tomcat-deploy.yaml
    ```
- 확인
    - 다음 명령어 입력 후 tomcatsvc명의 service객체(type: NodePort) 생성여부 확인
    ```bash
    kubectl get service
    ```
    - 다음 명령어 입력 후 NAME(tomcat-deploy) 포함여부 확인
    ```bash
    kubectl get pods
    ```
## Servicebinding
- 생성
    ```bash
    kubectl apply -f servicebinding-standalone.yaml
    ```
- 확인
    - 다음 명령어 입력 후 NAME: demo-sb-standalone, READY: True, REASON: ApplicationsBound 확인
    ```bash
    kubectl get servicebinding
    ```
- 삭제
    ```bash
    kubectl delete -f servicebinding-standalone.yaml
    ```