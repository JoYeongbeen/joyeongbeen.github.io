---
layout: posts
title:  "프로토콜 TCP/IP"
date:   2021-10-14 13:38:27 +0900
categories: 
    - am
    - network
---
# 프로토콜(Protocol)

복수의 컴퓨터 사이나 중앙 컴퓨터와 단말기 사이에서 데이터 통신을 원활하게 하기 위해 필요한 통신 규약. 신호 송신의 순서, 데이터의 표현법, 오류(誤謬) 검출법 등을 정함. 통신 규약(通信規約).


보내는 쪽에서는 데이터를 안전하고, 정확하고, 신속하게 `규격화` 즉 포장하는 방법이 필요하고, 받는 쪽에서는 그 데이터를 안전하고 정확하고 신속하게 `해석`하는 방법이 필요한 것이다. 그런 기술적 약속을 `프로토콜` 이라고 한다. 네트워크를 공부한다는 것은 많은 프로토콜을 학습한다는 것과 마찬가지다.

## TCP/IP 프로토콜의 차이

:star:TCP 프로토콜(Trasmission Control Protocl)
> 연결형(connection-oriented) 프로토콜: 연결 설정 후 통신 가능   
> 신뢰성 있는 데이터 전송: 데이터를 재전송   
> 일대일 통신(unicast)  
> 데이터 경계 구분 안함: 바이트 스트림(byte-stream) 서비스
> 패킷 손실, 중복 순서바뀜등이 없도록 보장   


:star:UDP 프로토콜 (User Datagram Protocol)
> 비연결형(connectionless) 프로토콜: 연결 설정 없이 통신 가능   
> 신뢰성 없는 데이터 전송: 데이터를 재전송하지 않음   
> 일대일 통신(unicast), 일대다 통신(broadcast, multicast)   
> 데이터 경계 구분: 데이터그램(datagram) 서비스
> 순서화되지 않음(datagram),헤더가 단순함, 실시간 응용

## 데이터 전송 원리

송신 측 호스트의 응용 프로그램 -> 수신측 호스트의 응용 프로그램 데이터 전송을 위해서는

각 프로토콜에서 정의한 제어정보(IP, 주소, 포트 번호, 오류 체크 코드 등)가 필요하다.   
제어정보는 위치에따라 앞쪽에 붙는 `헤더(Header)`와 뒤쪽에 붙는 `트레일러(Trailer)`로 나뉜다. 데이터는 이러한 제어 정보가 결합된 형태로 전송되며, 이를 `패킷(Packet)`이라고 부릅니다. 즉, 패킷은 `[제어정보 + 데이터]`로 정의할 수 있다.

송신 측 응용프로그램에서 보낸 데이터는 TCP/IP/이더넷 계층을 지나며 __헤더+데이터+트레일러__ 형식의 패킷이 생성되고,

수신측에 도착하면 이더넷/IP/TCP 계층을 지나면서 헤더, 트레일러의 제어정보가 제거되고 최종적으로 수신측 데이터를 받게된다.

이러한 과정중에서 응용 프로그래머는 데이터에만 집중하여 구현하고, 나머지는 운영체제가 제공하는 프로토콜이 처리하도록 맡기면 된다.

![데이터캡슐화](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F993FCD4D5B7A45AB07)

---

# TCP/IP 모델

OSI 모델은 참조 모델일 뿐 실제 사용되는 인터넷 프로토콜은 OSI 7계층 구조를 완전히 따르지는 않는다.   
인터넷 프로토콜 스택은 현재 대부분 TCP/IP를 따른다.

TCP/IP는 인터넷 프로토콜 중 가장 중요한 역할을 하는 `TCP`와 `IP`의 합성어로 데이터의 흐름 관리, 정확성 확인, 패킷의 목적지 보장을 담당한다. __데이터의 정확성 확인은 TCP가, 패킷을 목적지까지 전송하는 일은 IP가 담당한다.__

## 모형

> TCP/IP는 OSI 참조 모델과 달리 표현계층, 세션계층을 응용계층에 다 포함시키고 있지만, 사실상 TCP/IP Model의 Application 계층에서 응용,표현,세션계층의 구현을 다하고 있다고보면된다.

![TCP/IP](https://madplay.github.io/img/post/2018-02-04-network-tcp-udp-tcpip-1.png)

데이터는 아래 그림과 같이 단계 별로 헤더(Data → Segment → Datagram → Frame)를 붙여 전송하며 이를 `데이터 캡슐화`라고 한다.

![데이터캡슐화](https://t1.daumcdn.net/cfile/tistory/27786B485715047219)

---

### 1계층 - 네트워크 연결 계층(Network Access Layer/Network Interface Layer)

물리적으로 데이타가 네트워크를 통해 어떻게 전송되는지를 정의

논리주소(IP주소 등)이 아닌 물리주소(예. MAC주소(Media Access Control Address))을 참조해 장비간 전송

기본적으로 에러검출/패킷의 프레임화 담당

최종적으로 데이타 전송을 하기 전 패킷헤더에 MAC주소와 오류 검출을 위한 부분을 첨부

* PDU : 프레임(Frame)     
* 전송주소 : MAC
* 프로토콜 : Modem, Cable, Fiber, RS-232Ehternet(이더넷), Token Ring, PPP 
  
---

### 2계층 - 인터넷 계층(Internet Layer)

네트워크상 최종 목적지까지 정확하게 연결되도록 연결성을 제공

단말을 구분하기위해 논리적인 주소(Logical Address) IP를 할당

출발지와 목적지의 논리적 주소가 담겨있는 IP datagram이라는 패킷으로 데이타를 변경
데이터 전송을 위한 주소 지정

라우팅(Routing) 기능을 처리

최종 목적지까지 정확하게 연결되도록 연경성 제공

세그먼트를 목적지까지 전송하기 위해 시작 주소와 목적지의 논리적 주소를 붙인 단위. 데이타 + IP Header

* PDU : 패킷(Packet)
* 전송 주소 : IP
* 프로토콜 :IP, ARP, ICMP, RARP, OSPF

---

### 3계층 - 전송 계층(Transport Layer)


통신 노드 간의 연결 제어 및 자료 송수신을 담당

애플리케이션 계층의 세션과 데이터그램 통신서비스 제공

실질적인 데이터 전송을 위해 데이타를 일정 크기로 나눈 것. 발신, 수신, 포트주소, 오류검출코드가 붙게된다


* PUD : 세그먼트(Segment)
* 전송 주소: Port
* 프로토콜 : TCP, UDP, RTP, RTCP

---

### 4계층 - 응용 계층(Application Layer)


사용자와 가장 가까운 계층으로 사용자가 소프트웨어 application과 소통할 수 있게 해준다

응용프로그램(application)들이 데이터를 교환하기 위해 사용되는 프로토콜

사용자 응용프로그램 인터페이스를 담당

* PDU : 데이터/메시지(Data/Message)
* 프로토콜 : FTP, HTTP, SSH, Telnet, DNS, SMTP

---



## 질문거리


## 용어정리
   >호스트(Host)   
   >최종 사용자(end-user) 응용 프로그램을 수행하는 주체
   
   >라우터(Router)   
   >호스트에서 생성된 데이터를 여러 네트워크를 거쳐 전송함으로써 서로 다른 네트워크에 속한 호스트 간에 데이터를 교환할 수 있게 하는 장비
   
   >프로토콜(Protocol)   
   >호스트-라우터, 라우터-라우터, 호스트- 호스트의 통신을 위해서 정해진 절차와 방법을 따라야 하는데 이를 통신 프로토콜이라고 부른다.
   
   >라우팅(Routing)   
   >전송 경로를 결정하고 그에 따라 데이터를 전달하는 절차가 필요한데 이를 라우팅이라고한다.


출처   
https://velog.io/@hidaehyunlee/%EB%8D%B0%EC%9D%B4%ED%84%B0%EA%B0%80-%EC%A0%84%EB%8B%AC%EB%90%98%EB%8A%94-%EC%9B%90%EB%A6%AC-OSI-7%EA%B3%84%EC%B8%B5-%EB%AA%A8%EB%8D%B8%EA%B3%BC-TCPIP-%EB%AA%A8%EB%8D%B8   

https://12bme.tistory.com/61

https://valuefactory.tistory.com/507
  