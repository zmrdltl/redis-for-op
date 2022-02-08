## Service Binding Guide
tomcat에 Redis의 연결정보(password) 주입을 위한 service binding객체 생성
## Prerequisite
- k8s cluster(v1.11+)
- [redis-operator(v0.10.0)](https://ot-container-kit.github.io/redis-operator/)
- [service-bidning-operator(v1.0.0)](https://redhat-developer.github.io/service-binding-operator/userguide/getting-started/installing-service-binding.html)
- redis-standalone custom resource(kind: Redis)
- tomcat(v8.5) deployment, svc

## Service Binding Operator(SBO)
정의된 binding data key
- type : 고정값(redis)
- host : custom resource(kind: Redis)의 metadata.name
- password : redis secret pod의 password

## Servicebinding 생성시 내부동작
### 구조
![image](https://user-images.githubusercontent.com/22141521/152931074-6393d75a-b45b-4110-8c0e-2b96e3ef0ae1.png)
template instance인 redis와 redis-secret은 생성되었다고 가정(빨강 선)
<br/>
번호는 생성 및 동작 순서
### ① Tomcat
- 간단한 예시 WAS resource
    </br>
    deployment(image : tomcat v8.5)와 service(Type: NodePort 30007)로 구성
- 생성
    ```bash
    kubectl apply -f tomcat-deploy.yaml
    ```
- 확인
    - 다음 명령어 입력, 확인
    ```bash
    kubectl get service
    ```
    - 출력결과
    ```bash
    NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT           AGE
    tomcatsvc  NodePort    10.96.16.71     <none>        8080:30007TCP  18h
    ```

    - 다음 명령어 입력, 확인
    ```bash
    kubectl get pods
    ```
    - 출력결과
    ```bash
    NAME                            READY   STATUS    RESTARTS   AGE
    tomcat-deploy-5d6b6975cc-qv5c9  1/1     Running   0          18h
    ```
    

### ② servicebinding
- ./servicebinding/servicebinding-standalone.yaml에서 주석 설명 참조
- 생성
    ```bash
    kubectl apply -f servicebinding-standalone.yaml
    ```
### ③ : servicebinding객체를 watch하는 SBO가 services(Redis), application(tomcat)인식
### ④ : services(Redis)의 host와 redis-secret의 password값을 가져와 application에 주입
- 확인
    - 다음 명령어 입력, 확인
    ```bash
    kubectl get servicebinding
    ```
    - 출력결과
    ```bash
    NAME                 READY   REASON              AGE
    demo-sb-standalone   True    ApplicationsBound   19h
    ```

    - tomcat container 접속
    ```bash
    kubectl exec -it ${tomcat deployment pod명} -- bash
    ```
    - 접속 성공시 terminal 예
    ```bash
    root@tomcat-deploy-5d6b6975cc-qv5c9:/usr/local/tomcat#
    ```
    - container의 root folder 이동
    ```bash
    cd /
    ```
    - ls명령어로 bindings folder생성 확인
    ![image](https://user-images.githubusercontent.com/22141521/152932031-447c9872-ec61-48d2-b169-35a610891403.png)

    - servicebinding의 metadata.name으로 생성된 folder확인
    ![image](https://user-images.githubusercontent.com/22141521/152932212-0e428a13-fbe9-4ab7-9714-fe3a69be804b.png)

    - cd명령어로 해동 folder내부에서 ls명령어로 생성된 binding data files확인
    ![image](https://user-images.githubusercontent.com/22141521/152932279-04d97bac-cc5e-4c8f-8e37-a0a55924a864.png)

    - cat명령어로 file 내용 출력, 예시
    ```bash
    root@tomcat-deploy-mskim-6cd946fc88-rrbq8:/bindings/demo-sb# cat host
    redis-standaloneroot@tomcat-deploy-mskim-6cd946fc88-rrbq8:/bindings demo-sb# cat password
    abcd==root@tomcat-deploy-mskim-6cd946fc88-rrbq8:/bindings/demo-sb# cat type
    redisroot@tomcat-deploy-mskim-6cd946fc88-rrbq8:/bindings/demo-sb# 
    ```
- 삭제
    ```bash
    kubectl delete -f servicebinding-standalone.yaml
    ```