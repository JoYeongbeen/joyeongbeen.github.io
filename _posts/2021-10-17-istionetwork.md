---
layout: posts
title:  "Istio 설계"
date:   2021-10-17 13:00:00 +0900
categories: 
    - servicemesh
---
## 서문
앞선 글에서 Service Mesh와 Istio의 정의에 대해서 알아봤다.
이제 구체적으로 Istio를 이용한 ServiceMesh 설계에 대해 예제를 코드를 통해 살펴보자.

# Istio 네트워크 구성
* 어플리케이션 IP주소는 변경이 가능하고 기억하기 어렵고 환경에 따라 주소 사이에 변환이 필요한 주소이므로 Istio에서는 이런 취약성을 피하기 위해 이름별로 서비스를 처리한다. 결과적으로 Istio 네트워크 구성은 다음과 같은 이름 중심 모델을 채택했다.

    * **Gateway**는 이름을 외부에 노출한다. (Host, Port)
    * **VirtualService**는 이름을 구성하고 라우팅을 한다. (Path 기반 라우팅)
    * **DestinationRule**은 트래픽을 어떻게 최종 목적지에 전송할지 설정한다. (Traffic Rule 설정)

## Gateway 설계
```
apiVersion: networking.Istio.io/v1beta1
kind: Gateway
metadata:
  name: caller-gateway  #게이트웨이 이름 설정
spec:
  selector:
    Istio: ingressgateway
  servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
        - "caller.example.com" 
```

* Gateway의 이름을 caller-gateway로 설정해주고 있다.
* WAF단에서 SSL Termination하면 Istio Gateway에서 443 Port를 사용할 필요가 없다.
* 만약 443 Port를 사용해야 한다면 인증서를 등록한 후 Gateway에 443 Port를 추가해야 한다. 다음장의 코드를 참고하면 된다.

보안을 위해 별도 443 Port와 인증서를 설정해야한다면 아래와 같이 작성하면 된다.
```
apiVersion: networking.Istio.io/v1alpha3
kind: Gateway
metadata:
  name: foo-com-gateway
spec:
  selector:
    app: gateway-workloads
  servers:
  - hosts:
    - foo.com
    port:
      number: 80
      name: http
      protocol: HTTP
    tls:
      httpsRedirect: true # http 요청에 관해 301 리다이렉트를 보냄
  - hosts:
    - foo.com
    port:
      number: 443
      name: https
      protocol: HTTPS
    tls: #인증에 관련된 정보를 담고있다.
      mode: SIMPLE #이 포트에 HTTPS 사용
      serverCertificate: /etc/certs/foo-com-public.pem
      privateKey: /etc/certs/foo-com-privatekey.pem
```

* tls파라미터에 인증에 관련된 정보를 담고있다.

* Http로 요청이 올 경우 https로 리다이렉트를 할수있도록 httpsRedirect를 true로 설정한다.

## VirtualService 설계
* 들어온 트래픽을 서비스로 라우팅하는 기능이다. 쿠버네티스 안의 Service가 목적지가 되는 것이고, 실제로 VirtualService의 기능은 URI 기반으로 라우팅을 하는 Ingress와 유사하다고 보면 된다.

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
#      timeout: 0.5s # - to enable if using Istio fault on callme-service route
```

* gateways 파라미터에서 앞에 선언한 caller-gateway를 바인딩 해준다.
* destination에는 트래픽이 전달 되어야할 목적지가 있다. 해당 예제는 트래픽을 caller-service의 subset: v1에 전달한다. 
* timeout 파라미터를 통해 timeout조건을 넣을 수 있다. (callme-service에서의 fault injection을 위해서)

이 외에도 VirtualService에는 재시도 정책이나 fault를 주입할수 있다. 이와 관련된 파라미터를 추가한 아래예제를 보자.

```
apiVersion: networking.istio.io/v1beta1
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
      # retries:
      #   attempts: 3
      #   retryOn: gateway-error,connect-failure,refused-stream
      # timeout: 0.5s
      # fault: # --- enable for inject fault into the route
      #   delay:
      #     percentage:
      #       value: 33
      #     fixedDelay: 3s
```

* 해당 VirtualService는 destination이 callme-service의 subset v1, v2 두 곳으로 weigh를 통해서 트래픽을 조정할 수 있다.
* retries를 통해서 재시도 정책에 대한 정의를 할 수 있다. 해당 내용에서 재시도 횟수나 재시도 조건 등을 지정하게 된다.
* fault 의 경우에는 임의로 통신 간에 fault를 발생시킬 수 있게 하는 파라미터로 해당 예제에서는 33%확률로 3초간의 지연을 발생시키고 있다.

## DestinationRule 설계

* VirtualService가 쿠버네티스 Service로 트래픽을 보내도록 라우팅을 했다면, 그 다음 Destination Rule은 그 서비스로 어떻게 트래픽을 보낼것인가에 대한 Rule을 정의한다.

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
  trafficPolicy: # --- circuit breaker 활성화를 위한 구문
    connectionPool:
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
        maxRetries: 0
    outlierDetection:
      consecutive5xxErrors: 3
      interval: 30s
      baseEjectionTime: 1m
      maxEjectionPercent: 100
```

* Circuit breaker를 작동하기 위해서는 아래와 같은 설정이 필요하다. 
* 해당 예제는 30초 간격으로 스캔하고 5xx에러가 3번 발생하게 되면 해당 인스턴스를 1분 동안 100% 방출하는 설정이다.   

:star: coneectionPool
  * http1MaxPendingRequests : Queue에서 connection pool에 연결을 기다리는 request수를 1개로 제한한다.  
  * maxRequestsPerConnection : keep alive 기능을 disable한다. (default = 0)   
  * maxRetrie : 최대 재시도 횟수를 0으로 설정   

:star: outlierDetection
  * interval : Detection을 실행하는 주기   
  * consecutive5xxErrors : 5xx 에러가 발생하는 횟수   
  * baseEjectionTime: 이 시간 만큼 해당 애플리케이션의 인스턴스가 로드밸런싱에서 제외된다   
  * maxEjectionPercent: 실패한 애플리케이션 인스턴스를 로드밸런싱 풀에서 방출하는 퍼센트

