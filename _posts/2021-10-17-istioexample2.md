---
layout: posts
title:  "Istio 예제 (2/2) - Ping, Fault Injection, Circuit Breaker Test"
date:   2021-10-17 16:00:00 +0900
categories: 
    - servicemesh
---
## 서문
이전 글에 이어서 실제로 서비스를 호출해보고 yaml 파일을 수정하며 istio의 기능을 테스트 해보자.

해당 예제의 yaml 파일의 파라미터에 관한 설명은 앞에 작성한 다음글을 참고한다. [istio 설계](https://joyeongbeen.github.io/servicemesh/istionetwork/)

해당 글은 다음 블로그의 포스팅을 실습하며 작성한 글이다. [piotrminkowski.com](https://piotrminkowski.com/2020/06/01/service-mesh-on-kubernetes-with-istio-and-spring-boot/)   
해당 글과의 차이점은 minikube를 이용하지 않고 GCP환경의 GKE로 쿠버네티스 환경을 구축하였고, VSCode의 CloudCode를 사용해서 서비스 배포를 진행했다. 
예제 내에서 사용하는 명령어는 구글 클라우드 콘솔에서 Cloud Shell내에서 진행한다.   

# 프로그램 구조도
* 해당 Istio 예제에서 트래픽의 흐름은 아래와 같이 진행되게 된다.
* 외부에서 요청된 `GET http://caller.example.com/caller/ping` 요청이 **ingress gateway**를 통해 **VirtualService**로 이동하고 **VirtualService**에서는 어느 서비스로 트래픽을 이동시킬지 라우팅 진행 되고 **DestinationRule**에 의해 라우팅된 서비스에 정의해놓은 Rule을 가지고 트래픽이 이동하는 모습이다.

![](https://user-images.githubusercontent.com/82242095/137615156-a9658e3d-34da-48ed-afcc-e5ab20a72633.png)

# 예제코드 디렉토리
* 예제 코드는 호출을 하는 caller-service와 호출을 받는 callme-service로 이루어져 있다.
* 아래는 caller-service의 디렉토리 구조이다.

|디렉토리|설명|
|----|---|
|![](https://user-images.githubusercontent.com/82242095/137615284-e7f13a73-a984-456d-9381-6c28ebcd9001.png)|K8s </br> - deployment.yaml 쿠버네티스에 배포와 관련된 yaml파일 <br/>- Istio-rule.yaml Istio에서 사용되는 gateway, VirtualService, DestinationRule에 관한 정보를 갖고있는 yaml파일<br/><br/>Controller<br/>- 실제 호출을 수행하는 부분으로 /caller 밑에 /ping, /ping-with-random-error, /ping-with-random-delay로 구성 되어있다.<br/><br/>skaffold.yaml<br/>- 로컬환경에서 개발한 내용을 쿠버네티스에 편하게 배포할 수 있도록 도와주는 skaffold에 관련된 yaml파일|

* 아래는 callme-service의 디렉토리 구조이다.
* 
|디렉토리|설명|
|----|---|
|![](https://user-images.githubusercontent.com/82242095/137615542-1c8dfeda-e49b-415e-a12f-16b7ca25e508.png)|K8s </br> - deployment.yaml 쿠버네티스에 배포와 관련된 yaml파일 <br/>- Istio-rule.yaml Istio에서 사용되는 VirtualService, DestinationRule에 관한 정보를 갖고있는 yaml파일<br/><br/>Controller<br/>- 실제 호출을 수행하는 부분으로 /callme 밑에 /ping, /ping-with-random-error, /ping-with-random-delay로 구성 되어있다.<br/><br/>skaffold.yaml<br/>- 로컬환경에서 개발한 내용을 쿠버네티스에 편하게 배포할 수 있도록 도와주는 skaffold에 관련된 yaml파일|

# Ping Test
1.   controller의 /ping을 호출해본다. 
       ```
       curl -v -H "Host:caller.example.com" http://ingress-gateway 엔드포인트 주소/caller/ping
       ```
       ![](https://user-images.githubusercontent.com/82242095/137615627-f374de73-2d3f-468e-ba56-9ebf468bc073.png))
* 이때 Host는 gateway에 지정된 값이고 IP의 경우에는 Istio-ingressgateway의 endpoint이다.
* 정해진 트래픽 비율로 정상적으로 호출됨을 확인할 수 있다.


# Fault Injection Test
1. caller-service의 Istio-rules.yaml의 VirtualService 변경 (timeout 파라미터 부분 주석해제)
   ```
    apiVersion: networking.Istio.io/v1beta1
    kind: VirtualService
    metadata:
      name: caller-service-route
    spec:
      hosts:
        - "caller.example.com"
      gateways:
        - caller-gateway
      http:
        - route:
            - destination:
                host: caller-service
                subset: v1
          timeout: 0.5s # - to enable if using Istio fault on callme-service route
   ```
2. callme-service의 Istio-rules.yaml의 VirtualService 변경 (fault 파라미터 부분 주석해제)
    ```
    apiVersion: networking.Istio.io/v1beta1
    kind: VirtualService
    metadata:
      name: callme-service-route
    spec:
      hosts:
        - callme-service
      http:
        - route:
          - destination:
              host: callme-service
              subset: v2
            weight: 80
          - destination:
              host: callme-service
              subset: v1
            weight: 20
          fault: # --- enable for inject fault into the route
            delay:
              percentage:
                value: 33
              fixedDelay: 3s
    ```
3. /caller/ping 호출        
    ```
    curl -v -H "Host:caller.example.com" http://ingress-gateway 엔드포인트 주소/caller/ping
    ```
    ![](https://user-images.githubusercontent.com/82242095/137615805-ac469a21-9d97-4ef4-999e-a5645721a120.png)
    * 지정한 비율대로 timeout fault가 발생하는 것을 확인할 수 있다.

# Circuit Break Test
1. callme-service의 Istio-rules.yaml의 DestinationRule 변경 (trafficPolicy) 파라미터 부분 주석해제)
   ```
    apiVersion: networking.Istio.io/v1beta1
    kind: DestinationRule
    metadata:
      name: callme-service-destination
    spec:
      host: callme-service
      subsets:
        - name: v1
          labels:
            version: v1
        - name: v2
          labels:
            version: v2
      trafficPolicy: # --- enable for adding circuit breaker into DestinationRule
        connectionPool:
          http:
            http1MaxPendingRequests: 1
            maxRequestsPerConnection: 1
            maxRetries: 0
        outlierDetection:
          consecutive5xxErrors: 2
          interval: 30s
          baseEjectionTime: 1m
          maxEjectionPercent: 100
   ```
2. callme-service의 복제본을 생성한다. (Circuit breaker를 통해 차단되는 인스턴스를 확인하기위해)
   ```
   kubectl scale --replicas=2 deployment/callme-service-v2
   ```
3. ping-with-random-error를 발생시킨다. 
   ```
   curl -H "Host:caller.example.com" http://ingress-gateway 엔드포인트 주소/caller/ping-with-random-error
   ```

4. 해당 결과는 kiali에서 확인한다.
    ```
    istioctl dashboard kiali
    ```
![](https://user-images.githubusercontent.com/82242095/137615908-a7520e34-7150-4c85-a8b5-2477831cd3b6.png)
* Kiali에서 Circuit Breaker는 번개모양으로 표시되며 현재 5xx에러가 2번 이상 발생하여 callme-service의 인스턴스가 방출되어 열쇠모양으로 잠긴 것을 확인할 수 있다.