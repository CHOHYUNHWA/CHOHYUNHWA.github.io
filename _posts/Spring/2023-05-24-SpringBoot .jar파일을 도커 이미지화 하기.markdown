---
layout: post
title:  "SpringBoot .jar파일을 도커 이미지화 하기"
date:   2023-05-24 14:42:00 +0900
categories: spring
tags: spring
---

### 서버파일을 도커 이미지화 하기

### Docker란 ?

도커는 배포환경에 구애받지않고 각각의 컨테이너(실행환경)를 생성하여 독립적인 환경에서 애플리케이션을 실행시켜주는 파일 이라고 생각 하면 된다.

가령 A 프로그램은 java8 환경에서 정상작동하고 B프로그램은 java7 환경에서 정상작동한다고 예를들어보자 이경우 하나의 컴퓨터에 두프로그램을 동시에 사용할 경우 java버젼에 따라서 하나의 프로그램은 정상작동이 되지 않을 것이다.
물론 컴퓨터를 두대를 사용해서 하나의 컴퓨터에는 A프로그램 + java8 다른 컴퓨터에는 B프로그램 + java7을 설치하여 사용하면 되겠지만 컴퓨터의 하드웨어리소스가 부족하지 않는 이상 이런경우는 매우 손해라고 볼수 있다.
그래서 Docker라는 프로그램구동시 격리된 환경을 제공하여 하나의 컴퓨터에서 여러개의 프로그램을 동시에 작동하게 해주는 것이 Docker이다. 

우선 Docker 설치법
https://docs.docker.com/desktop/install/mac-install/ 를 참조하여 각자의 상황에 맞게 도커를 설치하면된다 (본인은 Mac)
<br>

다음의 명령어는 도커의 버전을 확인 할 수 있게 해준다.

```
docker --version 
```

자 이제 스프링부트 파일을 도커로 이미지화 시켜보자

![](https://velog.velcdn.com/images/gusk115/post/e46ea31b-6a11-4380-affb-7bf80c83636d/image.png)

우선 gradle 빌드를 통하여 jar파일을 생성 해주어야한다. 
 
빌드를 진행하면 

![](https://velog.velcdn.com/images/gusk115/post/613c6c1b-b3d8-44ec-9129-4466a89dc1da/image.png)

다음과 같이 jar파일이 생성된다. 
여기서 이미지화 시킬 jar파일은 roobits-0.0.1-SNAPSHOT.jar 이다.
<br>


다음은 루트 디렉토리에 Dockerfile을 생성해주어야한다.

```
FROM openjdk:11-jre
COPY build/libs/roobits-*.jar docker-roobits.jar
//ARG JAR_FILE=build/libs/roobits-0.0.1-SNAPSHOT.jar
// ADD ${JAR_FILE} docker-roobits.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

* FROM : 어떤 버젼의 java로 파일을 실행시킬지 이다.
* ARG : 도커에서 필요한 변수를 설정
* ADD : 파일 을 도커 이미지에 추가 
* COPY : roobits-*.jar 파일을 docker-roobits.jar로 복사 하여 저장
* ENTRYPOINT : 도커 컨테이너가 시작될때 실행되는 명령

여기서 나는 COPY만 사용 했다 일반적으로 COPY가 명시적이라 더 많이 사용 한다고 함 (ADD와 큰 차이점은 URL지정 , 압축자동 풀기 를 지원하지 않음)

그후 루트 디렉토리로 가서 빌드해주면 이미지가 생성된다.
```
docker buil -t {지정이미지명} .
```
<br>

생성된 이미지는 다음 명령어로 확인가능하다.

```
docker images
```

<br>

그후에 다음과 같이 이미지를 실행할 수 있다.
```
docker run {이미지명} 
```
하지만 이경우에는 컨테이너 포트가 사용되어서 배포환경의 포트와 매핑해주어야 한다.

```
docker run -p 8080:8080 {이미지명}
```
스프링부트의 default 포트는 8080이기때문에 컨테이너에서 실행된 8080포트와 컨테이너가 구동된 포트와 매핑해주어서 접근이 가능하게 해주는 것이다.
<br>
<br>
추가적으로 백그라운드로 실행하기 위해서는 아래와 같이 -d를 입력해주면 된다.
```
docker run -d -p 8080:8080 {이미지명}
```
