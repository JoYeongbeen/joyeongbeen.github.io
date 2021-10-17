---
layout: posts
title:  "ServiceMesh와 Istio"
date:   2021-10-17 12:00:00 +0900
categories: 
    - servicemesh
toc: true
toc_sticky: true
---

# ServiceMesh란?

:star: MSA에서 Service Mesh는 서비스를 구현한 어플리케이션 간의 통신이 Mesh 네트워크 형태를 띄는 것을 빗대어 부르는 용어이다. MSA는 컨테이너화 된 어플리케이션이 느슨하게 결합된 마이크로 서비스로 구성되는데, 서비스 간의 원활한 통신이 필요하여 Service Mesh라는 아키텍처 컨셉이 등장하였다.   
:star: Service Mesh는 서비스가 아니라 서비스의 네트워크 추상화한 프록시를 그물처럼 엮은 것이다. 서비스 옆에 경량화 프록시를 배치하여 서비스 간의 통신을 위해 복잡하게 구현해야 했던 부분을 추상화해서 서비스 간의 네트워크 처리에 대한 개발자의 고민을 덜어주는 것이다.   
:star: 즉, Service Mesh는 서비스를 구현한 어플리케이션에 구축된 전용 인프라 계층(Instructure Layer)이며, 각 서비스를 식별(Discovery)하고 경로를 설정(Routing)하며 로드밸런싱(Load Balancing)을 하고 서비스의 장애 전파를 차단하는 Circuit Breaker, Telemetry와 통합되어 로깅, 모니터링, 트레이싱 기능을 처리한다.


# Istio란?
:star: Istio는 Google, IBM, Redhat 등이 참여해 개발한 Service Mesh를 구현할 수 있는 오픈소스 솔루션이다.
Istio를 사용하면 서비스 코드 변경 없이 로드밸런싱, 서비스 간 인증, 모니터링 등을 적용하며 마이크로 서비스를 쉽게 관리할 수 있다.   
:star: 마이크로 서비스 간의 모든 네트워크 통신을 담당할 수 있는 프록시인 Envoy를 사이드카 패턴으로 마이크로 서비스들에 배포한 다음, 프록시들의 설정 값 저장 및 관리/감독을 수행하고, 프록시들에 설정 값을 전달하는 컨트롤러 역할을 수행한다.   
:star: 각각의 마이크로 서비스에 사이드카 패턴으로 배포한 Envoy 프록시를 Data Plane(데이터 플레인)이라고 하고, Data Plane을 컨트롤 하는 부분을 Control Plane(컨트롤 플레인)이라고 한다.

# Istio 아키텍쳐
Istio는 Data Plane과 Control Plane으로 구성되어 있으며 아키텍처는 아래 그림과 같다.

![istio 아키텍처](https://istio.io/latest/docs/ops/deployment/architecture/arch.svg)

## Data Plane
Data Plane은 마이크로 서비스와 사이드카로 배포된 프록시(Envoy)의 세트로 구성된다. 이러한 프록시는 마이크로 서비스간의 모든 트래픽을 조정하고 제어한다.

## Envoy Proxy
* Envoy Proxy는 C++로 개발된 고성능 프록시 사이드 카로서 Service Mesh의 모든 서비스에 대한 inbound,outbound 트래픽을 처리한다. 

* Envoy는 많은 기능으로 서비스를 보강한다. 그 기능은 아래와 같다.
    * HTTP, TCP, gRPC 프로토콜을 지원
    * TLS client certification 지원
    * HTTP L7 라우팅 지원을 통한 URL 기반 라우팅, 버퍼링, 서버간 부하 분산량 조절 등
    * HTTP2 지원
    * Auto retry, circuit breaker, 부하량 제한, 결함 주입(Fault Injection)등 다양한 로드 밸런싱 기능 제공
    * 다양한 통계 추적 기능 제공 및 Zpikin 통합을 통한 MSA 서비스간의 분산 트랜잭션 성능 측정을 제공
    * Dynamic configuration 지원을 통해서, 중앙 레파지토리에 설정 정보를 동적으로 읽어와서 서버 재시작 없이 라우팅 설정 변경이 가능함
    * MongoDB 및 AWS Dynamo에 대한 L7 라우팅 기능 제공

## Control Plane
Control Plane은 트래픽을 라우팅하기 위해 프록시를 구성하고 관리한다.

## Istiod
* Istiod는 서비스 디스커버리(Service Discovery), 설정 관리(Configuration Management), 인증 관리(Certificate Management) 등을 수행한다.

* Istiod는 아래와 같은 기능을 제공한다.
    * 트래픽 동작을 제어하는 라우팅 규칙을 Envoy 전용 설정으로 변환하고, 마이크로 서비스에 사이드카 방식으로 Envoy를 배포한다.
    * Envoy 설정 변경(Istio의 Traffic Management API 활용)을 통한 서비스 메시 트래픽 세부 제어한다.
    * 내장된 identity나 자격증명관리(Credential Management)를 통해 강력한 서비스 간 인증 및 사용자 인증 기능을 지원한다.
    * 인증기관(Certificate Authority. CA)의 역할 수행. 데이터 플레인에서 안전한 mTLS통신을 허용하는 인증서를 생성한다.

# Istio의 특징
|특징|내용|
|------|---|
|트래픽 관리(Traffic Management)|Istio의 간편한 규칙(Rule) 설정과 트래픽 라우팅 기능을 통해 서비스 간의 트래픽 흐름과 API호출을 제어할 수 있다. 또한 서킷 브레이커(Circuit Breaker), 타임아웃(Timeout), 재시도(Retry) 기능과 같은 서비스 레벨의 속성 구성을 단순화하고, 백분율 기반으로 트래픽을 분할하여 A/B Test, 카나리아 배포, 단계증(Staged) Rollout과 같은 작업을 쉽게 설정할 수 있다.|
|보안(Security)|Istio의 보안기능을 통해 개발자는 어플리케이션 레벨의 보안에 보다 더 집중할 수 있다. 기본적인 보안 통신 채널을 제공하며, 대규모 서비스 통신의 인증(Authentication), 권한부여(Authorization),암호화(Encryption) 등을 관리한다. Istio를 사용하면 기본적으로 서비스 통신은 보호되기 때문에, 다양한 프로토콜이나 런타임에서 어플리케이션 변경을 거의 하지않고 일관된 정책을 시행할 수 있다. Istio는 플랫폼에 독립적이지만 쿠버네티스 네트워크 정책과 함께 사용하면 파드(pod)간 혹은 서비스(Service)간 통신을 보호하는 기능 등의 다양한 이점이 있다.|
|관찰 가능성(Observability)|Istio의 강력한 트레이싱(Tracing), 모니터링(Monitoring), 로깅(Logging) 기능으로 서비스 메시여 배포된 서비스들에 대해 더욱 자세히 파악할 수 있다. Istio의 모니터링 기능을 통해 서비스 성능이 업스트림/다운스트림에 어떤 영향을 끼치는지 파악할 수 있다. 또한 맞춤형 대시보드를 통해 모든 서비스 성능에 대한 가시성 확인 및 성능이 다른 프로세스들에 미치는 영향 등을 확인 할 수 있다.|
|플랫폼 지원(Platform support)|Istio는 플랫폼에 독립적이며 클라우드, On-Premise, Kubernetes, Mesos 등을 포함한 다양한 환경에서 실행되도록 설계되었다.|
|통합과 사용자 정의(Integration and Customization)|Istio의 구성 요소들은 확장 및 커스터마이징을 통해서 ACL, 로깅, 모니터링, 할당량, 감사 등을 위한 기존 솔루션들과 통합될 수 있다.|