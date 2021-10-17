---
layout: posts
title:  "Build Tool"
date:   2021-10-14 13:38:27 +0900
categories: 
    - java
---

Maver과 Gradle 모두 빌드 관리 도구라는데에서 공통점이 있다.


# 빌드 관리 도구(Build Tool)

## 정의
- 소스 코드를 컴파일, 테스트, 정적분석 등을 실시하여 싱행 가능한 애플리케이션으로 자동 생성하는 프로그램
- 라이브러리 자동 추가 및 관리
- 프로젝트를 진행하며 시간이 지남에 따라 라이브러리의 버전을 자동으로 동기화

## 종류
- Ant
- Maven
- Gradle


# Maven
- Maven은 Java용 프로젝트 관리도구로 Apache의 Ant 대안으로 만들어졌다.

- 빌드 중인 프로젝트, 빌드 순서, 다양한 외부 라이브러리 종속성 관계를 pom.xml파일에 명시한다.

- Maven은 외부저장소에서 필요한 라이브러리와 플러그인들을 다운로드 한다음, 로컬시스템의 캐시에 모두 저장한다.

​
## 특징
* 빌드 절차 간소화
* 옹일한 빌드 시스템 제공
* 프로젝트 정보 제공

## 구조
![Maven구조](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile30.uf.tistory.com%2Fimage%2F999BF0365A534021248CEE)

---

## Life Cycle - Phase -Plugin - Goal의 구조로 빌드가 실행된다.

---

## 플러그인(Plugin)
메이븐은 플러그인을 구동해주는 프레임워크(plugin execution framework)이다. 모든 작업은 플러그인에서 수행한다.   
플러그인은 다른 산출물(artifacts)와 같이 저장소에서 관리된다.   
메이븐은 여러 플러그인으로 구성되어 있으며, 각각의 플러그인은 하나 이상의 goal(명령,작업)을 포함하고있다. goal은 Maven의 실행단위이다.

![Plugin구조](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile4.uf.tistory.com%2Fimage%2F99989C3E5A533FC31EBE86)

플러그인과 골의 조합으로 실행한다. ex)`mvn <plugin>:<goal>` = `mvn archetype:generate` 

메이븐은 여러 goal을 묶어서 lifecycle phases로 만들고 실행한다. ex)`mvn <phase>` = `mvn install`

---
## 라이프사이클(Life Cycle)
Maven 프로젝트를 빌드하는데 있어 순서가 존재하는데 그 순서를 바로 Maven의 라이프사이클이라고 한다.   

Maven의 라이프사이클 종류는 `Default(기본)`,`Clean`,`Site`가 존재한다.   그리고 각 라이프사이클 안에 phase(compile, test, package...)가 존재한다. 각 phase를 통해 명령을 내릴 수 있고, 나중 단계의 phase를 실행 시켰다면 이전 단계가 모두 실행 되어집니다. 예를 들어 `"mvn install"`라는 명령을 내리면 compile부터 install단계까지 모두 실행되는것이다.

자세한 라이프사이클은 Maven 홈페이지를 참조하도록한다.

![라이프사이클](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FysAxy%2FbtqBTLmONJO%2FDH8KRLC3rkduni6kBY9UnK%2Fimg.jpg)


---

## POM.xml(Project Object Model)

이름 그대로 Project Object Model의 정보를 담고있는 파일이다. 이파일에서 주요하게 다루는 기능들은 다음과 같다.

- 프로젝트 정보 : 프로젝트의 이름, 개발자 목록, 라이센스 등
- 빌드 설정 : 소스, 리소스, 라이프 사이클별 실행한 플러그인(goal)등 빌드와 관련된 설정
- 빌드 환경 : 사용자 환경 별로 달라질 수 있는 프로파일 정보
- POM연관 정보 : 의존 프로젝트(모듈), 상위 프로젝트, 포함하고 있는 하위 모듈 등

:star:POM의 엘리먼트
- `<groupId>`: 프로젝트의 패키지 명칭
- `<artifactId>` : artifact 이름, groupId 내에서 유일해야 한다.
 ```
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
  ```
- `<version>` : artifact 의 현재버전 ex. 1.0-SNAPSHOT
- `<name>` : 어플리케이션 명칭
- `<packaging>` : 패키징 유형(jar, war 등)
- `<distributionManagement>` : artifact가 배포될 저장소 정보와 설정
- `<parent>` : 프로젝트의 계층 정보
- `<dependencyManagement>` : 의존성 처리에 대한 기본 설정 영역
- `<dependencies>` : 의존성 정의 영역
- `<repositories>` : 이거 안쓰면 공식 maven 저장소를 활용하지만, 사용하면 거기 저장소를 사용

- `<build>` : 빌드에 사용할 플러그인 목록을 나열
 
- `<reporting>` : 리포팅에 사용할 플러그인 목록을 나열
 
- `<properties>` : 보기좋게 관리가능, 보통 버전에 많이 쓴다.
 
``` 
   <!-- properties 에 이렇게 추가하면 -->
   <spring-version>4.3.3.RELEASE</spring-version>
   <!-- dependencies 에 이렇게 쓸수 있다. -->
   <version>${spring-version}</version>
   ```


:star: ant와의 차이점
> Ant가 비교적 자유도가 높다. 전처리, 컴파일, 패키징, 테스팅, 배포 가능   
> Maven은 정해진 라이프사이클에 의하여 작업 수행하며, 전반적인 프로젝트 관리 기능까지 포함하고있음. (Build Tool + Project Management)


:star: gradle과의 차이점
>XML 대신 groovy 스크립트를 사용하여 동적인 빌드 가능.   
>maven은 멀티프로젝트에서 상속구조인데, gradle은 주입 방식이다. 멀티프로젝트에서 gradle이 더 적합하다.


# Gradle
- Apacahe Maven과 Apache Ant에서 볼수 있는 개념들을 사용하는 대안으로써 나온 프로젝트 빌드 관리 툴이다. (완전한 오픈소스)

- Groovy 언어를 사용한 Domain-specific-language를 사용한다. (설정파일을 xml파일을 사용하는 Maven보다 코드가 훨씬 간결하다.)

- 2007년에 처음 개발되었고, 2013년에 구글에 의해 안드로이드 프로젝트의 빌드 시스템으로 채택되었다.

- 꽤 큰규모로 예상되는 multi-project 빌드를 도울 수 있도록 디자인되었다.

- Gradle은 프로젝트의 어느부분이 업데이트되었는지 체크하기 때문에, 빌드에 점진적으로 추가할 수 있다.

- 업데이트가 이미 반영된 빌드의 부분은 즉 더이상 재실행되지 않는다. (__따라서 빌드 시간이 훨씬 단축될 수 있다! 이 차이는 빌드 설정 규모가 커지면 커질수록 Maven과의 격차는 더욱 커진다.__)

---

## Gradle이 Maven보다 좋은점
- Build라는 동적인 요소를 XML로 정의하기에는 어려운 부분이 많다.
  - 설정 내용이 길어지고 가독성 떨어짐
  - 의존관계가 복잡한 프로젝트 설정하기에는 부적절
  - 상속구조를 이용한 멀티 모듈 구현
  - 특정 설정을 소수의 모듈에서 공유하기 위해서는 부모 프로젝트를 생성하여 상속하게 해야함 (상속의 단점 생김)
- Gradle은 그루비를 사용하기 때문에, 동적인 빌드는 Groovy 스크립트로 플러그인을 호출하거나 직접 코드를 짜면 된다.
  - Configuration Injection 방식을 사용해서 공통 모듈을 상속해서 사용하는 단점을 커버했다.
  - 설정 주입시 프로젝트의 조건을 체크할 수 있어서 프로젝트별로 주입되는 설정을 다르게 할 수 있다.
- Maven에는 비교문서가 없지만, Gradle에는 비교문서가 존재한다. 

---

## Maven과 Gradle의 코드 비교

JUnit의 라이브러리를 의존성에 추가하고 플러그인을 추가한 코드이다.   
XML이 복잡해지고 내용이 추가됨에따라 Grdle의 장점이 더 돋보인다.

## Maven
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>2.12.1</version>
    <executions>
        <execution>
            <configuration>
                <configLocation>config/checkstyle/checkstyle.xml</configLocation>
                <consoleOutput>true</consoleOutput>
                <failsOnError>true</failsOnError>
            </configuration>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>findbugs-maven-plugin</artifactId>
    <version>2.5.4</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-pmd-plugin</artifactId>
    <version>3.1</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
## Gradle
```
apply plugin:'java'
apply plugin:'checkstyle'
apply plugin:'findbugs'
apply plugin:'pmd'
version ='1.0'
repositories {
    mavenCentral()
}
dependencies {
    testCompile group:'junit', name:'junit', version:'4.11'
}
```

---

## 질문거리

Maven과 Gradle의 작동원리 자세하게 찾아보기   
(POM.xml과 bulid.gradle중심으로)

---

## 용어정리
1. Build란?   
일반적으로 빌드란 소스코드 파일을 컴퓨터에서 실행할 수 있는 독립적인 형태로 변환하는 과정과 그 결과   `.java`와 여러가지 정적파일등에 해당하는 `resource`가 존재하는데 빌드를 진행하면 소스코드를 compile하여 `.class`로 변환하고 `resource`를 `.class`에서 참조할수 있는 적절한 위치로 옮기고, 추가적으로 `META-INF`와 `MANIFEST.MF`등을 하나로 묶는 과정을 의미한다.

2. Build tool에서 Build  
   빌드툴은 위에 Build과정을 `task`에 정의된 대로 실행하게 된다. 컴파일이 필요한 언어에만 국한된 의미가 아니고 resource파일과 같이 어플리케이션의 실행에 필요한 파일들을 지정된 위치로 복사를 하거나, 지정한 어플리케이션을 실행하는것 등도 빌드 프로세스에 정의할 수 있다.

   __따라서 빌드 대상에 어떠한 행위를 하는것이 Build Tool에서의 Build의 의미이다. Gradle에서는 이러한 행위의 단위를 `Task`라고 하고 이런 Task를 싱행하는것을 빌드라고한다.__

---
## 출처
https://jisooo.tistory.com/entry/Spring-%EB%B9%8C%EB%93%9C-%EA%B4%80%EB%A6%AC-%EB%8F%84%EA%B5%AC-Maven%EA%B3%BC-Gradle-%EB%B9%84%EA%B5%90%ED%95%98%EA%B8%B0   
https://sjh836.tistory.com/131   
https://hyojun123.github.io/2019/04/18/gradleAndMaven/   
https://galid1.tistory.com/644?category=757944
