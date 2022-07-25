# cenacle-devops
DevOps Summary Note during Intern in cenacle

## 주차별 진행 현황

**22/07/18**
- 클라이언트 서버 개념 정리
- onPremises VS Cloud 개념 정리
- 모노리딕 VS MSA
- k8s
- docker

**22/07/19**
- k8s 공부
- docker-compose / docker file 구축
- jenkins 설치 및 빌드잡 만들기
  - Gradle 설정
  - jdk 설정
  - hook post-receive 설정
  - nodejs 설정
 
**22/07/20**
- post-receive 오류 해결
- k8s 개념 복습
- docker registry 컨테이너 띄우기

**22/07/21**
- git server -> jenkins -> deploy '.jar' on docker container
- git server -> jenkins -> docker 이미지 자동 빌드
- 쿠버네티스 환경 구축
 
**22/07/22**
- 쿠버네티스 스프링 앱 배포
- 트러블 슈팅

## devops 실습 진행사항
- 1일차 
  - 도커 쿠버네티스 개념 정리
- 2일차
  - Local 서버에서 git push 시 -> Jenkins job 자동 build(hooks)
  - Jenkins에 소스 올리고 로컬 레포지토리에 docker 이미지를 띄우는 실습
- 3일차
  - Local 서버에서 git push 시
  - 만들어진 서비스 서버(우분투)
  - build된 jar 파일 ssh로 전송
  - jar 백그라운드 실행 (mysql bind-address, public-key 이슈 등)
- 4일차
  - local 서버 push 시
  - build 된 jar 파일 도커 이미지로 빌드
  - 이후 registry 서버에 이미지 등록
  - minikube로 이미지 pull 후 deploy

## 목표
- deploy java service to msa in k8s cluster

- target
    - docker
    - k8s
    - ci/cd

- step
    - brief
    - install tools
        - git, docker, docker-compose, docker-hub, kubectl, helm, minikube
    - setup git repository
    - setup ci/cd (jenkins)
    - setup k8s cluster (minikube)
    - deploy service to k8s w/jenkins
    - maintain k8s w/kubectl

## 개념 공부

### 클라이언트 - 서버

[클라이언트-서버 개념](https://coding-factory.tistory.com/329)

#### 클라이언트 서버 시스템
여러 개의 클라이언트가 네트워크 통신을 통해 서버에 접속하고 그 서버와 붙어있는 데이터베이스를 활용할 수 있는 시스템을 말함.

#### 클라이언트-서버 모델 변천사

**MainFrame 모델 : 1960 ~ 1970 HostTerminal**
1. Dummy Terminal(Client)과 Main Frame(Server) 관계
2. 동축/시리얼 케이블
3. COBOL
4. 열악한 UI / 확장성 및 개발 융ㄴ성 낮음.

**Client/Server 모델 : 1980 ~ 1990 Server-PC**
1. 설치 필요한 전용 클라이언트 어플리케이션 - 어플리케이션 서버 미들웨어 2, 3티어
2. LAN / WAN / VPN + 통신프로토콜 TCP/IP
3. 클라이언트 배포/관리 여러 사용 당사자간만의 폐쇄적 구조 클라이언트 운영체제 의존적

**Web 모델 : 1990 ~ 2000**
1. 각종 브라우저 - Web Server
2. LAN / WAN
3. HTTP / HTTPS / SOAP
4. C/S 모델에 비해 취약한 UI 잦은 화면 새로고침 / 서버 과부하 과다한 UI디자인과 이미지

**RIA / Cloud**
1. 각종 브라우저 / APP - Web Server
2. LAN / WAN
3. HTTP / HTTPS / SOAP
4. 서버 사이드 고비용 대용량 데이터


#### RIA(Rich Internet Application)
- 애플리케이션 장점은 유지하면서 기존 웹 브라우저 기반 인터페이스의 단점인 늦은 응답 속도, 조작성 등을 개선하기 위한 기술의 통칭이다.
- 한 페이지로 구현된 웹 응용 프로그램. 페이지 이동과 새로 고침의 깜박임 없이.

#### 네트워킹 - 클라이언트-서버 시스템 구조
- 일반적으로 클라이언트와 서버 시스템 구조는 각각 소켓을 만들어 연결한 뒤 클라이언트와 서버측에서 요청한 메시지를 상호간에 전송하고 받음으로써 요청한 작업을 수행한다.

#### WEB - 클라이언트-서버 통신 구조
- 내부적으로 소켓 통신 사용. 웹 HTTP는 TCP/IP 프로토콜 기반이며, 응용 계층에 위치한 프로토콜이다.
- 그렇기 때문에 인터넷 계층의 IP에 접근할 수 있어야 한다. 그래서 웹은 DNS 서버를 통해 도메인 주소에서 IP 주소를 얻어옵니다.
- 웹 서버 작동과 클라이언트 호출 과정은 C/S 모델에서 소켓 통신과 비슷.
- 단 패킷을 사용하는 대신 HTTP 프로토콜에서 정의한 전문과 헤더를 통해 통신을 처리.


### on premises vs cloud
- [클라우드와 온프레미스](https://velog.io/@dewgang/%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C%EC%99%80-%EC%98%A8%ED%94%84%EB%A0%88%EB%AF%B8%EC%8A%A4)
- [클라우드란 무엇인가?](https://library.gabia.com/contents/infrahosting/9114/)

기존에는 개인 단말기에 설치한 소프트웨어나 저장한 데이터밖에 사용할 수 없었다. 클라우드 환경에서는 인터넷 상에 설치된 소프트웨어나 동영상, 음악 등의 자원을 사용할 수 있고, 로컬 환경에 저장하듯 클라우드에 저장도 가능함.

 
#### 온프레미스
자사가 서버를 구축하는 것. 자사에서 자유롭게 운영 가능 But 서버 구성 어렵고, 기술자 필요.

#### 임대나 공용
- 운영하지 않고 임대하거나 공공장소에 구축된 것을 사용하는 형태.
- 제공 측의 규제를 지켜야 함.
- OS 업데이트나 설치할 수 있는 소프트웨어, 구성 등에 제한이 있는 경우 많고 비용 비쌈.

### 공용 클라우드 VS 사설 클라우드
- 공용 : AWS처럼 임대하는 클라우드
- 사설 : 자사에서 구축하는 클라우드

 

### 클라우드 서비스 유형

#### IaaS
- 사용자가 관리할 수 있는 가장 넓은 범위. 서버 OS 미들웨어 런타임 데이터 어플리케이션 직접 구성 관리. + 클라우드 서비스 제공업체는 데이터 센터를 구축해 다수의 물리 서버를 가상화해 제공하며 네트워크, 스토리지, 전력 등 서버 운영에 필요한 모든 것을 CSP가 책임지고 관리합니다.
- EX) AWS, GCE

#### PaaS
- 가상화된 클라우드 위에 사용자가 원하는 서비스를 개발할 수 있도록 개발 환경을 미리 구축해 이를 서비스 형태로 제공하는 것을 의미.
- EX : 레드헷

#### SaaS
- 클라우드 인프라 위에 소프트웨어를 탑재해 제공하는 형태로 IT 인프라 자원뿐만 아니라 소프트웨어 및 업데이트, 버그 개선 등의 서비스를 업체가 도맡아 제공. 월간/연간 구독 형태로 사용료 지불
- EX : 슬랙 / 마이크로소프트365 / 드롭박스

### 웹호스팅 vs 서버호스팅 vs 클라우드
- 웹호스팅 : 서버 공간 일부만 임대하여 사용하고 웹 서비스(홈페이지)만 운영된다.
- 서버호스팅 : 호스팅 업체의 물리 서버를 단독으로 임대/구매하여 사용. 인프라+기술력 제공 받음.
- 클라우드 : 호스팅업체의 가상 서버를 단독으로 사용. 단 몇 분만에 서버 생성 후 바로 사용.


### monolithic architecture vs MSA(micro service architecture)

[MSA 아키텍처란?](https://steady-coding.tistory.com/595)

 
#### 모노리딕 아키텍처
어플리케이션의 모든 구성 요소가 한 프로젝트에 통합되어 있는 형태를 말함.
 
장점
- 개발 초기에 단순한 아키텍처 구조로 인해 개발 용이
- 어떤 서비스든지 개발되어 있는 환경이 같아서 복잡하지 않음.
- 배포가 간단
- 확장성 쉽다(로드벨런스를 이용해서 로드 부하를 나눠 가지는 방식으로 진행)
- 고가용성 서버 환경을 만들 수 있다.
- End to End 테스트가 용이

단점
- 프로젝트의 규모가 커짐에 따라 애플리케이션 구동 시간이 늘어나고 빌드 및 배포 시간이 길어진다.
- 조그마한 수정 사항이 있어도 전체를 다시 빌드하고 배포해야함.
- 많은 양의 코드가 물려 있어서 개발자가 모든 코드를 이해하기 힘들며, 유지 보수가 어렵다.
- 일부분의 오류가 전체에 영향을 미친다.
- 기술 스택이 한 번 정해지면 바꾸기 어렵다.
- 전체 애플리케이션 확장은 쉽지만, 부하 분산을 위해 각 컴포넌트를 독립적으로 확장하기 어렵다.

#### 마이크로서비스 아키텍처
- MSA 아키텍처는 하나의 큰 애플리케이션을 여러 개의 작은 애플리케이션으로 쪼개어 변경과 조합이 가능하도록 만든 형태를 말한다. 작은 레고 블록 하나하나 붙여 어떠한 큰 결과물을 만드는 레고 놀이라고 생각하면 이해하기 쉽다.
- 마이크로서비스의 단위는 하나의 기능, 하나의 프로젝트가 될 수 있으나, 팀 상황에 맞게 경계를 잘 정해서 설계

장점
- 배포 관점 : 서비스 별 배포 가능 + 독립 배포 가능 + 요구 사항 신속 반영
- 확장 관점 : 특정 서비스에 대한 확장성 용이 + 클라우드 사용에 적합.
- 장애 관점 : 장애가 전체 서비스로 확장될 가능성이 적다 + 부분적 장애에 대한 격리가 수월
- 코드 유지 보수 : 팀 별로 프로젝트가 분리되어 있으므로 코드의 이해도가 증가하고, 유지 보수 쉽다.

단점
- 성능 관점 : 서비스 간 호출 시 API를 사용하여 통신 비용 및 지연 시간 증가
- 데이터 관리 관점 : 데이터가 여러 서비스에 걸쳐 분산되므로 한번에 조화하기 어렵고, 데이터의 정합성 또한 관리하기 어렵다.
- 테스트 / 트랜잭션 관점 : 단위 테스트는 쉽지만, 통합 테스트 E2E 테스트 단위로 들어가면 여러 API를 검증해야 하므로 시간과 비용이 많이 든다. + 각 서비스 별 DB가 있어서 트랜잭션 구현이 까다롭다.
- 아키텍처가 다소 복잡하므로 개발 및 관리가 어렵고 비용이 많이 든다.
 
### 쿠버네티스(k8s) 개요

[쿠버네티스 공식문서](https://kubernetes.io/ko/docs/concepts/overview/what-is-kubernetes/)

쿠버네티스는 컨테이너화된 워크로드와 서비스를 관리하기 위한 확장 가능한 오픈소스 플랫폼이다.

#### 컨테이너 개발 시대
컨테이너는 VM과 유사하지만 격리 속성을 완화하여 애플리케이션 간에 운영체제(OS)를 공유한다. 그러므로 컨테이너는 가볍다고 여겨진다. VM과 마찬가지로 컨테이너에는 자체 파일 시스템, CPU 점유율, 메모리, 프로세스 공간 등이 있다. 기본 인프라와의 종속성을 끊었기 때문에, 클라우드나 OS 배포본에 모두 이식할 수 있다.

#### 쿠버네티스가 필요한 이유?
- 컨테이너는 애플리케이션을 포장하고 실행하는 좋은 방법이다. 프로덕션 환경에서는 애플리케이션 실행하는 컨테이너를 관리하고 가동 중지 시간이 없는지를 확인해야한다.
- 쿠버네티스는 분산 시스템을 탄력적으로 실행하기 위한 프레임워크 제공한다. 애플리케이션 확장과 장애 조치를 처리하고, 배포 패턴 등을 제공한다.

#### 쿠버네티스 특징
- 서비스 디스커버리와 로드벨런싱
- 스토리지 오케스트레이션
- 자동화된 롤아웃과 롤백
- 자동화된 빈 패킹
- 자동화된 복구
- 시크릿과 구성 관리

### Docker 개념 및 사용법 정리

#### Docker
- 컨테이너 기반 가상화 도구. 기존에 사용하던 가상 머신VM은 사용하기 위해 항상 OS를 설치해야 했고, 이미지 안에 OS가 포함되어 있기 떄문에 용량이 매우 크고 속도도 느리다.
- 반면 도커는 반가상화보다 더 경량화된 방식을 사용한다. 도커는 OS 전체를 가상화하지 않고 컨테이너라는 리눅스 커널 레벨에서 제공하는 격리된 가상 공간을 사용한다. 게스트 OS를 설치하지 않기 때문에 호스트와 속도 차이도 거의 없으며 기존의 VM에 비해 월등한 속도로 동작한다.
- 도커는 리눅스 컨테이너 기반으로 이미지를 편리하게 관리하고 배포할 수 있다.


#### 도커 이미지란?
- 도커 이미지는 필요한 프로그램, 라이브러리, 소스 등을 설치한 뒤에 이를 파일로 만든 것.
- 이렇게 만든 이미지를 레포지토리에 올리고, 이것을 받아 도커에서 실행시킨 뒤 사용하면 됨.
- 이미지가 실행된 상태를 컨테이너라고 부른다.
- 이미지를 여러 번 실행시키면 하나의 여러 개의 컨테이너가 만들어진다.
- 운영체제로 본다면 이미지는 일종의 실행파일, 컨테이너는 프로세스와 유사한 개념이다.
- 시스템 도구, 라이브러리 및 설정과 같은 모든 것을 포함하는 가벼운 독립형 실행 SW패키지

#### 도커의 장점
- 빠른 설치 : 어플리케이션 설치하고 사용하는 것에 최소의 시간과 용량을 소비
- 어플리케이션 이식성 : 어플리케이션과 설치 파일들이 하나의 컨테이너에 존재하기 때문에 호스트의 커널이나 플랫폼 버전 등에 상관 없이 도커만 실행할 수 있으면 사용이 가능하다.
- 버전제어 : 컨테이너 버전제어가 쉽고 변경 내역을 확인 가능하며 롤백이 간단하다.
- 컴포넌트 재사용 : 이전 레이어에서 컴포넌트들을 재사용 하기 때문에 가볍게 만들 수 있다.
- 쉬운 유지관리 : 어플리케이션 종속성에 관한 문제들에 대해 유지관리가 쉽다.
 
#### 하이퍼바이저
- 가상 머신 모니터라고도 불리며, 가상머신을 생성하고 실행하는 프로세스이다.
- 메모리 및 처리와 같은 단일 호스트 컴퓨터 리소스를 가상으로 공유하여 호스트 컴퓨터가 여러 게스트
- 가상 머신을 지원할 수 있도록 한다. (도커 엔진과 대비되는 개념? VM을 관리 vs 컨테이너 관리)

#### 도커가 격리를 수행하는 방법
Use Cgroup & Namespace of linux

- Cgroup - CPU, Mem, network, I/O 등의 자원을 제한함
- Namespace - 프로세스를 격리하는 기능

#### 이미지로 컨테이너 만들기
1. 이미지는 File Snapshot과 Command List를 가지고 있음.
2. 컨테이너 HDD에 File Snapshot을 적재하고, 프로세스 영역에 Command List를 이동시킨다.
3. Command List를 실행한다.


#### Window/macOS에서도 Cgroup과 namespace를 사용할 수 있는 이유?
- Linux VM이 설치되고 그 위에 Docker 엔진 및 컨테이너가 설치되기 때문.
- 리눅스 커널을 사용하기 때문에 Cgroup / namespace를 사용할 수 있다.
 
#### namespace?
- 리눅스 네임스페이스는 프로세스를 실행할 때 시스템의 리소스를 분리해서 실행할 수 있도록 도와주는 기능
- 리눅스 커널에서 6가지 네임스페이스를 지원
  - mnt (파일시스템 마운트) : 호스트 파일시스템에 구애받지 않고 독립적으로 파일시스템을 마운트 하거나 언마운트
  - pid (프로세스) : 독립적인 프로세스 공간을 할당.
  - net (네트워크) : namespace 간 network 충돌 방지(중복 포트 바인딩 등)
  - pic (systemV IPC) : 프로세스 간의 독립적인 통신 통로 할당
  - uts (hostname) : 독립적인 hostname 할당
  - user (UID) : 독립적인 사용자 할당

#### Cgroups?
- 자원에 대한 제어를 가능하게 해주는 리눅스 커널의 기능.
  - CPU
  - 메모리
  - I/O
  - 네트워크
  - device 노드(/dev/)

 
#### 도커의 레이어 개념
- 컨테이너를 실행하기 위해 모든 정보를 가지고 있다보니 용량이 큰데, 처음 이미지를 다운 받을 때는 크게 부담이 없음.
- 하지만 기존 이미지에 파일 하나를 추가해싿고 수백메가를 다시 다운받으면 비효율적.
- 그렇기에 레이어 개념을 도입하여, 유니온 파일 시스템을 이용해 여러 개의 레이어를 하나의 파일 시스템으로 사용할 수 있게 해줌.
- 예시로 A + B + C + nginx + source 에서 source 수정 시 새로운 source만 받으면 되기 때문에 매우 효율적임. 

#### 도커 개념 및 명령어 총정리
[도커 개념정리 및 사용법](https://cultivo-hy.github.io/docker/image/usage/2019/03/14/Docker%EC%A0%95%EB%A6%AC/#1-version)

 
#### 기본 명령어

- dudo 없이 사용하기
  - sudo usermod -aG docker $USER # 현재 접속중인 사용자에게 권한주기
  - sudo usermod -aG docker your-user # your-user 사용자에게 권한주기

 

- docker run : 컨테이너 실행하기
- docker ps : 컨테이너 목록 확인
- docker stop : 컨테이너 중지하기
- docker rm : 컨테이너 제거하기
- docker images : 이미지 목록 확인하기
- docker pull : 이미지 다운로드 하기
- docker rmi : 이미지 삭제하기
- docker logs : 컨테이너 로그 보기
- docker exec : 컨테이너 명령어 실행하기 <- 실행중인 컨테이너에 들어가거나 컨테이너의 파일을 실행하고 싶을 때
  - docker exec -it 6F5 bash

#### Docker 이미지 만들기
- docker commit  : 현재까지 작업해 놓은 컨테이너 그대로 저장/커밋

#### Docker 이미지 Push 하기
- docker login : 도커 클라우드 로그인 하기
- docker tag : 이미지에 태그 달기
- docker push : tag가 적용되어 있는 이미지를 도커 클라우드에 푸시

#### DockerFile 기반으로 이미지 만들기
- Dockekfile : 컨테이너에 설치해야 하는 패키지, 추가해야하는 소스코드, 실행해야 하는 명령어 등을 기록해두는 파일. 한 줄은 한 이미지 레이어임.
- docker build : 도커 파일(Dockerfile) 기반으로 추가 구성을 포함한 이미지 생성

#### 캐시 이미지 빌드
- 한번 이미지를 빌드하면, 다시 같은 빌드를 진행할 때 이전 빌드에서 사용했던 캐시를 사용한다.
- 원치 않는다면 no cache 옵션을 준다.
- docker-compose 사용 시에도 새로운 업데이트 한 이미지를 기반으로 컨테이너를 띄우고 싶다면
  - docker-compose up --force-recreate --build -d
- dockerignore 파일을 작성해 Dockerfile은 포함시키지 않고 빌드 해야함.

##### docker-compose 
docker-compose : 도커는 복잡한 설정을 쉽게 관리하기 위해 yaml 방식의 설정 파일을 이용한 docker compose라는 툴을 제공함.
- Dockerfile은 이미지를 어셈블 하기 위해 호출할 수 있는 간단한 텍스트 파일
- Docker-compose는 앱을 구성하는 서비스를 docker-compose.yml에 정의하여 docker-compose up을 실행하여 하나의 명령으로 앱을 실행.
- 즉, 앱이 실행되는 동안 컨테이너를 관리하는 역할.
  
- docker build vs docker-compose build: 도커 이미지 만들기(=클래스 선언을 애플리케이션에 로드)
- docker run의 옵션들 vs docker-compose.yml: 이미지에 붙이는 장식들(=인스턴스의 변수들)
- docker run vs docker-compose up: 장식 붙은 이미지를 실제로 실행(=인스턴스 생성)

### K8s 개념 및 사용법 정리

Pod : 컨테이너이 관리 단위로 생성/관리/배포 가능한 가장 작은 단위의 유닛이며, 클러스터 내에서 실제 어플리케이션이 구동되는 오브젝트.
- 개별 IP를 가진다.
- 하나 이상의 컨테이너로 구성되며, 동일한 Pod 내에서 스토리지와 네트워크 공유할 수 있음.
- pod를 통해서 배포함
- pod의 라이프사이클

1. pending : pod 배포 명령을 내린 직후 pod 상태를 조회하면 나옴. Pod 내의 컨테이너 이미지가 다운로드 되기 전 대기 상태임.
2. running : 컨테이너가 노드에 정상적으로 생성되어 그 안의 프로세스가 정상적으로 동작하고 있는 상태.
3. succeeded : 주로 job 같은 pod 리소스에서 확인할 수 있는 형태로 pod 컨테이너가 생성되어 프로세스를 실행하고 정상적으로 종료된 상태.
4. failed : pod 안의 모든 컨테이너가 종료되었으나 한 개 이상의 컨테이너가 정상 종료 되지 않은 상태
5. unknown : pod 상태를 확인할 수 없는 상태, 주로 host 머신과 통신 문제시 발생.

**pod 템플릿 예시**
``` yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: my-nginx
  name: my-nginx
  namespace: default        
spec:
  containers:
  - image: nginx:1.14.1
    imagePullPolicy: Always
    name: my-nginx
    ports:
    - containerPort: 80
      protocol: TCP
```

- [쿠버네티스 명령어 정리](https://velog.io/@hanblueblue/kubernates-kubectl-%EB%AA%85%EB%A0%B9%EC%96%B4-%EC%A0%95%EB%A6%AC)

- [쿠버네티스 설명 및 예제](https://blog.voidmainvoid.net/146)

 
### Jenkins 개념 및 사용법 정리

[docker 이미지로 만드는 jenkins](https://choseongho93.tistory.com/303) <- 볼륨 설정에 오류 있음.
 

### Devops
#### 개념 정리
- Devops는 애플리케이션과 서비스를 빠른 속도로 제공할 수 있도록 조직의 역량을 향상시키는 도구의 조합
- 조직의 제품을 더 빠르게 혁신하고 개선할 수 있다. 고객에게 더 잘 지원하고 시장에서 좀 더 효과적인 경쟁

#### Devops 동작방식
개발&운영팀이 단일팀으로 병합되어 엔지니어가 개발에서 테스트, 배포, 운영에 이르기까지 전체 어플리케이션 수명 주기에 걸쳐 작업함.
- 속도 : 작업 속도가 빨라짐. 시장 변화에 더 잘 적응
- 신속한 제공 : 릴리즈의 빈도와 속도 개선
- 안정성 : 변경 사항이 제대로 동작하며 안전한지 테스트
- 확장 : 규모에 따라 인프라와 개발 프로세스를 운영 및 관리
- 협업 강화 : 긴밀하게 협력, 많은 책임을 공유
- 보안 : 제어를 유지하고 규정을 준수하며 신속하게 진행
 

#### docker volume container

https://joont92.github.io/docker/volume-container-%EC%B6%94%EA%B0%80%ED%95%98%EA%B8%B0/

- docker에는 저장소 기능이 없음.
- 그렇기 때문에 Host 경로에 있는 Volume을 설정해서, 해당 host에 있는 폴더를 심볼릭 링크 개념으로 매핑해서 사용함.
- 이렇게 되면 컨테이너가 날라가도 데이터가 유실되지 않고 호스트 폴더에 남아 있기 때문에 언제든지 다시 사용 가능.


### Jenkins 관련
#### Jenkins jdk 설정

https://royleej9.tistory.com/entry/Jenkins-jdk-%EC%84%A4%EC%A0%95

https://www.jenkins.io/blog/2022/03/21/java17-preview-availability/

**Dockerfile로 jenkins:jdk v17 이미지 레이어 덮어 씌우고 실행.**

#### 트러블슈팅 Jenkins Hook 설정

https://jonny-cho.github.io/devops/2021/05/17/post-receive-jenkins/

https://www.whatwant.com/entry/Redmine-Git-%EC%97%B0%EB%8F%99-Repository-%EC%9E%90%EB%8F%99-%EA%B0%B1%EC%8B%A0-hooks

- chmod ug+x post-receive
- chmod -R a+x hooks/*
- post-update <- 이것도 작동 안하는 것 확인
- git config core.hooksPath setting
- git server docker 내부에 curl 설치 <- 해결

#### Jenkins 리액트 빌드 설정
- https://songjang.tistory.com/m/112
- Execute shell에 npm run build
- Scripts build 부분에 “CI=false && react~~~”로 변경
- Node 18.5


### 쿠버네티스 개념
- 컨테이너 수가 많아지면 어디에 배포해야하는지 결정이 필ㅇ
- 메모리 CPU 사이즈 다양하게 사용하여 적절한 위치에 배포
- APP 특성에 따라서 같은 물리 서버 혹은 다른 물리서버에 배포
- 적절한 서버에 배포해주는 스케줄링 역할

- [쿠버네티스란?](https://coding-start.tistory.com/308)
- [쿠버네티스의 클러스터 개념](https://seongjin.me/kubernetes-cluster-components/)

- Cluster : 하나의 Master와 여러 Node들이 연결되어 하나의 클러스터를 이룸.
- Master : 쿠버네티스의 메인 기능들을 담당하는 역할. 노드들을 관리하는 역할
- 마스터를 구성하는 관리 컴포넌트
  - kube-apiserver : 쿠버네티스 API를 노출하는 컴포넌트
  - etcd : 키-값 스토어
  - kube-scheduler : 노드를 모니터링하고 컨테이너를 배치할 적절한 노드를 선택
  - kube-controller-manager : 리소스를 제어하는 컨트롤러를 실행한다.
  - dns 서버
- Node : 실제 App(Pod)들이 구동되기 위해 자원을 제공하는 역할.
(Master와 Node는 실제 물리적인 서비스)
- Pod : 컨테이너가 모인 집합체로, 적어도 하나 이상의 컨테이너로 이루어진다. 여기서 컨테이너는 도커 컨테이너. Pod는 각각 IP address + volume + containerized app을 갖는다. 하나의 Pod 내부에서 컨테이너들은 localhost를 통해 통신이 가능하다.
- Namespace : 쿠버네티스는 클러스터 안에 가상 클러스트를 또 만들 수 있는데 이것을 네임스페이스라고 한다. 처음 클러스터를 만들면
  - default
  - docker
  - kube-public
  - kube-system
이라는 4개의 네임스페이스가 만들어져있다. Kubectl get namespace를 사용하면 네임스페이스 목록을 확인할 수 있다.
- Volume : 쿠버네티스의 볼륨은 Pod내의 컨테이너간의 공유가 가능하다. 각 컨테이너에 볼륨을 마운트 해주면 공유 가능.
- Service : 여러 개의 Pod를 서비스 하면서, 이를 로드벨런서를 이용해 하나의 IP포트와 묶어서 서비스 제공. Pod의 경우 동적으로 생성되고, 장애가 생기면 자동으로 재생성 되면서 IP가 바뀐다. 서비스를 정의할 때 어떤 Pod를 서비스로 묶을 것인지 정의하는데, Pod간에 로드벨런싱을 통해 외부 서비스를 제공하는 형태.

#### Manifest
- Pod 매니페스트 : 파드를 구성하는 컨테이너 정보를 적는다.
- 레플리카세트 매니페스트 : 여러 개의 파드를 생성하고 관리할 수 있다.
- 디플로이먼트 매니페스트 : 애플리케이션 배포의 기본 단위가 되는 리소스이다.
- 라벨 : 각 리소스는 라벨을 가질 수 있고, 라벨 검색 조건에 따라 특정 라벨을 가지고 있는 리소스만 선택할 수 있음.
- 컨트롤러 : 기본 오브젝트를 생성하고 이를 관리해주는 역할을 해준다.
  - Replication Controller : Pod를 관리해주는 역할을 함.
  - Replication Set : RC의 새버전. Set 기반의 Selector 이용.
  - DaemonSet Job
  - StatefulSet
  - Deployment : RS를 좀 더 추상회 해놓은 개념이다.

### Docker repository
https://snowdeer.github.io/docker/2018/01/24/docker-use-local-repository/
- Docker Hub 등의 Public Registry의 경우 하나의 이미지만 private 등록이 가능하고 Org의 경우 비용 지불해야함.
- 개인 공간에서 보다 더 많은 공간을 사용할 수 있다. 

### Minikube 사용법
minikube
- https://nangman14.tistory.com/77
minikube ingress
- https://kubernetes.io/ko/docs/tasks/access-application-cluster/ingress-minikube/

 
- Brew install minikube
- Minikube start —nodes 1 —cpus 4 —memory 2g -p Minikube 클러스터 생성
- kubectl get pod -n kube-system

 
### 실습 관련
- [jenkins docker 연동 종합](https://dev-overload.tistory.com/40?category=841809)
- [publish over ssh 설정](https://dbjh.tistory.com/68?category=839984)

1. 서버 우분투 이미지 생성
2. Jenkins 설정 파일 있는 볼륨 매핑된 로컬 호스트에서 rsa key 발급
3. 깃서버에 pub key 등록
4. 깃서버에 openssh 설치
5. Service-server에 pub key 등록
6. 젠킨스 publish over ssh 접근 정보 설정
7. 젠킨스 credentials 설정
8. Nohup command 생성
9. Service-server에 프로젝트 target + logs 디렉토리 생성
10. Mysql bind-address 0.0.0.0 으로 변경

`+refs/pull/*:refs/remotes/origin/pr/*`

이거 뺀 거 까지 함. 그러고 다시 빌드함.

`+refs/pull/*:refs/remotes/origin/* 로 변경하니 잘 됨.`


#### 트러블 슈팅
- [git server ssh 설정](https://linuxhint.com/git_server_ssh_ubuntu/)
- [git server 공개키 등록 방법](https://jjkim01.tistory.com/24)
- [도커 registry로 private 저장소](https://novemberde.github.io/post/2017/04/09/Docker_Registry_0/)
- [도커 커밋으로 현재 컨테이너 이미지화](https://eungbean.github.io/2018/12/03/til-docker-commit/)
- [docker-compose 이미지 업데이트시 동기화](https://hakurei.tistory.com/290)

 
—— 젠킨스에서 빌드 후 도커 이미지 build 하기 ——

**[도커->젠킨스 빌드 종합](https://velog.io/@hind_sight/Docker-Jenkins-%EB%8F%84%EC%BB%A4%EC%99%80-%EC%A0%A0%ED%82%A8%EC%8A%A4%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Spring-Boot-CICD)**
 
1. 빌드 끝난 직후
2. 도커 빌드, 커밋, 푸시로 localhost:5000/docker-bsa 이미지 생성
 
### Docker k8s 연동
도커에 새로운 이미지가 업로드 되면 쿠버네티스가 이를 실행
Deployment.yaml 파일 작성

- [k8s가 registry에서 이미지 가져오는 법](https://ikcoo.tistory.com/65)
- [ErrPullImage 해결](https://stackoverflow.com/questions/52698748/connection-refused-on-pushing-a-docker-image)

 
/User/suheonkim/.docker/daemon.json 경로에
``` json
        "insecure-registries" : [

                "localhost:5000",

                "127.0.0.1",

                "192.168.1.217:5000"

        ]
```
설정하기

### 도커 재시작 on Mac
- launchctl list | grep docker
- launchctl stop docker~~~~
- open -a docker
- launchctl start docker~~~~

### 쿠버네티스 실습
1. minikube 클러스터 생성
- minikube start --insecure-registry="192.168.1.217:5000" --ports=5000:5000 --driver=docker --nodes 1 --cpus 4 --memory 2g -p minikube

2. Ingress-nginx 추가 
- minikube addons enable ingress

3. 젠킨스에서 이미지 빌드
- (빌드시 localhost:5000/docker-bsa  / docker-bra 생성)

4. minikube ssh 접속 후 다음 명령어 실행
- docker run --restart=always -d -p 5000:5000 --name registry registry:2

5. Exit으로 호스트로 돌아옴.

6. docker 127.0.0.1:5000/docker-bsa 이미지 만들고 push
- docker tag localhost:5000 127.0.0.1:5000/docker-bsa
- docker push 127.0.0.1:5000/docker-bsa

7. 이후 kubectl apply -f deployment.yaml 로 deployment 실행
``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bsa-deployment
spec:
  selector:
    matchLabels:
      app: bsa-app
  replicas: 1
  template:
    metadata:
      labels:
        app: bsa-app
    spec:
      containers:
        - name: core
          image: 192.168.1.217:5000/docker-bsa:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
              protocol: TCP
          resources:
            requests:
              cpu: 500m
              memory: 1000Mi
```
8. Kubectl apply -f service.yaml로 service 실행.
``` yml
apiVersion: v1
kind: Service
metadata:
  name: bsa-service
spec:
  type: NodePort
  ports:
    - port: 80
      protocol: TCP
      targetPort: 8080
  selector:
    app: bsa-app
```
9. Minikube service bsa-service —url 로 url 접속 가능하게 실행

### 3일차 실습 요약
- 도커 리포지토리 만들 것임. Registry 55000 포트
- 도커 git server tag 

**프로세스**
1. commit
2. tag
3. push
4. images
5. pull
6. rmi

#### minikube 설치
- minikube start --nodes 1 --cpus 4 --memory 2g -p minikube 클러스터 생성
- minikube start --insecure-registry="192.168.1.217:5000" --ports=5000:5000 --driver=docker --nodes 1 --cpus 4 --memory 2g -p minikube
- kubectl get pod -n kube-system
- 
- minikube addons enable ingress : 외부에서 접속하려면 nginx ingress 필요
- minikube profile list
- minikube delete -p localk8s

- kubectl create deployment web —Images~~~~ 
- kubectl expose deployment —type=NodePort —port=8080 노드 포트

- kubectl get server web
- minikube service web -url
- kubectl get pod

### 실습 질문 사항
1. 커밋을 하고 태그를 달아야 하는가? 태그를 달고 커밋을 해야하는가?
- 로컬에 커밋 -> 리모트에 태깅 -> 푸시
2. 이미지를 만들면 volumes 매핑된 데이터도 같이 이미지로 만들어지는지?
- 이미지 path를 따로 설정해줘야 함.
- Volumes 제외하고 나머지 설치된 것은 만들어 짐.
3. minikube를 깔면 k8s도 자동으로 설치되는가?
4. 노드 포트 연결 8080을 사용하셨는데 이 부분 다시 한번만 설명
5. 젠킨스 빌드 후
- build -> Push -> docker
- deploy -> k8s
- kubectl config view -> 쿠버 server 정보 젠킨스

- 빌드 태그해서 올리는 것.
- 리눅스 jvm 자바 플랫폼 설치된 거에 jar 파일 카피하고 docker 실행

**최종 마무리**
1. 도커로 이미지 잘 되는지
2. 쿠버네티스로 배포

## 유용했던 링크 총정리
- [Docker-compose git server + Jenkins 설치](https://linuxhint.com/setup_git_http_server_docker/)
- [Docker local registry 구축방법](https://snowdeer.github.io/docker/2018/01/24/docker-use-local-repository/)
- [Spring Jenkins + 컨테이너에 ssh 전송 총정리](https://dev-overload.tistory.com/40?category=841809)
- [git server ssh openssh 설치 및 rsa 설정](https://linuxhint.com/git_server_ssh_ubuntu/)
- [docker + Jenkins spring 이미지 빌드 및 배포 총정리](https://velog.io/@hind_sight/Docker-Jenkins-%EB%8F%84%EC%BB%A4%EC%99%80-%EC%A0%A0%ED%82%A8%EC%8A%A4%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Spring-Boot-CICD)
- [docker cask 옵션으로 모두 설치](https://velog.io/@jaryeonge/Docker-Mac%EC%97%90-Homebrew%EB%A1%9C-docker-%EC%84%A4%EC%B9%98)
- [minikube 설치](https://nangman14.tistory.com/77)
- [ingress-nginx 추가](https://kubernetes.io/ko/docs/tasks/access-application-cluster/ingress-minikube/)
- [쿠버네티스 스프링 배포 총정리](https://velog.io/@sgwon1996/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%ED%99%98%EA%B2%BD%EC%97%90-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%96%B4%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0)
- [connection refused on pushing 정리](https://stackoverflow.com/questions/52698748/connection-refused-on-pushing-a-docker-image)
- [docker 시작/재시작 launchctl](https://www.codegrepper.com/code-examples/shell/docker+restart+mac)
  - (중간에 open -a docker 필요) 

## 기타 명령어 및 링크
- docker run -d -p 5000:5000 --name local-registry -v /Users/suheonkim/onboarding-devops/repository registry
- docker run --name service-server -itd -p 8080:8080 -p 8022:22 localhost:5000/service-server:1.0
- 내부 서버 IP http://192.168.1.217/
- 도커 고유 id application.com.docker.docker.3156761.3156766
- 포트 정보
  - 8080 bsa
  - 3000 web
  - 8000 깃서버
  - 8001 젠킨스
  - 5000 이미지 레포

**클라이언트 - 서버**
- https://coding-factory.tistory.com/329
**on premis vs cloud**
- https://velog.io/@dewgang/%ED%81%B4%EB%9D%BC%EC%9A%B0%EB%93%9C%EC%99%80-%EC%98%A8%ED%94%84%EB%A0%88%EB%AF%B8%EC%8A%A4
- https://library.gabia.com/contents/infrahosting/9114/

**monolithic architecture vs MSA(micro service architecture)**
- https://steady-coding.tistory.com/595

**k8s 개요**
- https://kubernetes.io/ko/docs/concepts/overview/what-is-kubernetes/

**devOps**
- https://aws.amazon.com/ko/devops/what-is-devops/
- https://azure.microsoft.com/ko-kr/resources/cloud-computing-dictionary/what-is-devops/#devops-overview

**git servier**
- https://linuxhint.com/setup_git_http_server_docker/

**jenkins server**
- https://velog.io/@revelation/%EB%8F%84%EC%BB%A4%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-%EB%A1%9C%EC%BB%AC%EC%97%90-%EC%A0%A0%ED%82%A8%EC%8A%A4jenkins-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0

**docker**
- https://docs.docker.com/get-started/overview/

**tools**
- docker
- docker-compose
 
**docker-desktop**
- https://freestrokes.tistory.com/150

**docker repository**
- https://snowdeer.github.io/docker/2018/01/24/docker-use-local-repository/

**minikube**
- https://nangman14.tistory.com/77

**minikube ingress**
- https://kubernetes.io/ko/docs/tasks/access-application-cluster/ingress-minikube/
