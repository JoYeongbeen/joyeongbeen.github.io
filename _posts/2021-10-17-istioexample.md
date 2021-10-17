---
layout: posts
title:  "Istio 예제 (1/2) - 개발환경 구축 및 어플리케이션 기동"
date:   2021-10-17 15:00:00 +0900
categories: 
    - servicemesh
---
## 서문
이제 실제 코드와 함께 istio 예제를 실습해보자.   
해당 글은 다음 블로그의 포스팅을 실습하며 작성한 글이다. [piotrminkowski.com](https://piotrminkowski.com/2020/06/01/service-mesh-on-kubernetes-with-istio-and-spring-boot/)   
해당 글과의 차이점은 minikube를 이용하지 않고 GCP환경의 GKE로 쿠버네티스 환경을 구축하였고, VSCode의 CloudCode를 사용해서 서비스 배포를 진행했다.   
예제 내에서 사용하는 명령어는 구글 클라우드 콘솔에서 Cloud Shell내에서 진행한다.   
본글에서는 개발환경 구축과 어플리케이션 기동까지 진행하고 다음글에서 실제 테스트를 진행하도록 하겠다.


# GKE 구축 및 istio 설치
1. GKE 클러스터 생성   
   - GCP에 Kubernetes 클러스터를 생성한다.
    ![](https://user-images.githubusercontent.com/82242095/137614315-e7308442-9c91-469d-bb3d-0aa12c1fc051.png)
2. istio 설치
   - 명령어를 통해 최신 istio를 설치한다.   
        ```
        curl -L https://git.io/getLatestIstio | sh -
        ```
    - PATH를 설정한다.   
        ```
        cd istio-1.11.3

        export PATH=$PWD/bin:$PATH
        ```
   - 다음 명령어를 통해 쿠버네티스에 실제로 istio를 설치한다.   
        ```
        istioctl install --set profile=demo -y
        ```
   - 실제로 설치가 정상적으로 진행되었는지 kubectl 명령어를 통해 확인한다.
        ```
        kubectl get pod -n istio-system
        ```
        ![](https://user-images.githubusercontent.com/82242095/137614322-344485d7-90ef-4295-b9ee-89346464d40c.png)
3. sidecar injection 기능 활성화를 위한 라벨 추가
   - 라벨 추가 명령어
        ```
        kubectl label namespace default istio-injection=enabled
        ```
   - 라벨 확인 명령어
        ```
        kubectl get ns --show-labels
        ```



## 예제 코드 설치 및 Cloud Code설치
1. git clone을 이용한 예제 어플리케이션 설치 (로컬환경의 위치에서 설치)
   ```
   git clone https://github.com/JoYeongbeen/istio.git
   ```
2. VSCode 설치
3. 확장프로그램 CloudCode 설치
   ![](https://user-images.githubusercontent.com/82242095/137614335-4a2cad37-08ea-4507-9155-fc927a587587.png)




## Cloud Code 클러스터 연결 및 어플리케이션 실행
1. `Cloud Code-Kubernetes`의  `Clusters`창의 헤더에서 + 버튼(Add a Cluster to the KubeConfig)을 누른다.
2. Google Kubernetes Engine을 선택하여 클러스터를 선택한후 GCP 계정에 로그인하여 클러스터를 선택한다.
    ![](https://user-images.githubusercontent.com/82242095/137614356-0919a8f7-2850-48a1-b74e-c6484e6ce06c.png)
3. Cloud Code의 메뉴에서 Run on Kubernetes를 선택해 caller-service, callme-service의 scaffold.yaml파일을 각각 실행시킨다. 
4. 아래는 정상 실행된 어플리케이션 모습이다. (각각 caller-service, callme-service)
    
    |caller-service|calleme-service|
    |---|---|
    |![](https://user-images.githubusercontent.com/82242095/137614363-eac465fc-62f5-4ad3-a45d-e6325b50e850.png) | ![](https://user-images.githubusercontent.com/82242095/137614371-7e13663f-52cb-44cb-8082-88d13f886c9a.png)|
5. GKE내에 정상적으로 caller-service, callme-service가 실행중임을 확인할 수 있다.
    ![](https://user-images.githubusercontent.com/82242095/137614830-0c9e09f3-6e10-46d0-8393-691bd917f1f9.png)
 
    