---
layout: post
title: 회사 TEST SERVER 도커 이미지로 만들기
tags: [docker, temp, note]
skills: [docker]
author: mainmethod0126
excerpt_separator: <!--more-->
---

# 회사 TEST SERVER를 도커 이미지로 만들기

집에서 회사 TEST SERVER 에 붙고 싶었으나 회사의 경우 내부 네트워크를 사용하지 않을 경우 접근을 할 수 없도록 되어있어 집에서 잠시 회사 업무를 보고자 하거나 공부를 하기위해 회사 TEST SERVER가 필요한 경우에 원격을 붙어서 작업을 해야했습니다.

문제는 이 원격이라는 것이 속도도 느리고 멀티 모니터 사용 및 단축키의 불편함을 느끼게 하기 때문에, 회사 TEST SERVER 와 같은 환경이 개인 PC에도 존재하면 좋겠다 라고 생각하여 도커 이미지로 하나 만들어 보기로 했습니다.

지금부터 한번 만들어 보겠습니다.

## 도커 파일

도커 파일이란 Docker 이미지를 빌드하기 위한 지침이 포함된 텍스트 파일입니다.

사용할 `기본 이미지`와 이미지에 필요한 `종속성` 및 `필수 패키지`를 지정합니다.
 를
Dockerfile은 웹 서버 시작 또는 스크립트 실행과 같이 `이미지가 시작될 때 실행할 명령`도 지정합니다.

간단히 요약하자면 도커 이미지를 생성하기 위한 `설계도` 입니다

만들어야 할 TEST SERVER의 요구사항은 아래와 같습니다.

- CentOS7.1908 버전을 사용해야한다.
- zulu JDK 8, 11, 14, 15, 17 이 설치되어야 한다.
- Nginx 는 1.20.2 가 설치되어야 한다.
- Elasticsearch 7.17.6 버전이 설치되어야 한다.

그럼 TEST SERVER를 만들기 위한 `도커 파일`을 만들어 보겠습니다.

### 도커 파일 만들기

#### 베이스 이미지 선택

베이스 이미지는 다른 이미지를 빌드하기 위한 시작점 입니다.

아무것도 존재하지 않고 단인 OS만 존재하는 이미지로 해당 베이스 이미지를 선택할 수도 있으며,
원하는 OS에 기본적인 셋팅이된 이미 만들어진 이미지를 사용할 수 있습니다.

이 베이스 이미지는 Docker Hub와 같은 공용 레지스트리에서 불러올 수 있습니다.

저는 CentOS7.1908 이라는 OS를 사용할 예정이라 베이스 이미지를 이 CentOS7.1908로 선택하겠습니다.

```dockerfile
FROM centos:7.1908
```

---
## 아래부터는 임시 작성 부분입니다.


### 도커파일

```dockerfile
FROM centos:7.7.1908

# Install MariaDB
RUN yum -y install mariadb-server mariadb

# Start MariaDB and set it to start on boot
RUN systemctl start mariadb && systemctl enable mariadb

# Add user with sudo privileges
RUN adduser --uid 1000 softcamp
RUN echo "softcamp:softcamp123!@#" | chpasswd
RUN echo "softcamp ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```

### dockerfiles 저장 경로

/opt/dockerfiles

왜? opt 디렉토리는 optional 의 약자로 운영체제 시스템과는 별도의 서비스를 설치할때 사용하는 디렉토리입니다.
일반적으 root가 소유권을 갖으면 다른 사용자는 읽기/실행만을 허용합니다.

도커파일 빌드

빌드할 도커파일이 있는 경로로 현재 디렉토리를 이동시킨 후

```bash
sudo docker build -t sk-hynix-pre-release-test .
```


### 도커볼륨 관리하는 디렉토리

nas 를 마운팅하여 볼륨 관리를 진행하려합니다.

nas의 경로는 //10.30.10.177/dxl/docker/volumes/pre-release 입니다.

nas 마운팅을 위해 cifs 를 먼저 설치합니다.

```bash
sudo apt install cifs-utils
```

```bash
sudo mount -t cifs //10.30.10.177/dxl/docker/volumes/pre-release /var/lib/docker/volumes
```

부팅시 자동 마운트를 위하여 /etc/fstab 에 아래와 같은 내용을 추가합니다.

```bash
//10.30.10.177/dxl/docker/volumes/pre-release /var/lib/docker/volumes/pre-release cifs 0 0
```

여기까지 마운트는 성공적으로 이뤄지는걸 확인함.

이제 고객사 이름의 폴더 내부에 있는 각각의 폴더를 정해진 위치에 마운팅하여 사용하도록 컨테이너를 실행해야함.

```bash
sudo docker run -it --log-driver=json-file \
  -v /var/lib/docker/volumes/pre-release/sk-hynix/elasticsearch-7.8.1:/home/softcamp/elasticsearch-7.8.1 \
  -v /var/lib/docker/volumes/pre-release/sk-hynix/gatexcanner:/home/softcamp/GateXcanner \
  -v /var/lib/docker/volumes/pre-release/sk-hynix/GateXcannerApiSvr:/home/softcamp/GateXcannerApiSvr \
  sk-hynix-pre-release-test
```

컨테이너 실행까지는 성공하였으나 마운팅이 이뤄지지 않은 로그 확인 후 분석 필요

### 2023-01-04 알게된 정보

- docker run 은 해당 이미지에다가 무엇을 실행시켜라 라는 명령을 내리는 것 뿐. 해당 명령이 완료성 명령이면 컨테이너가 켜졌다가 명령 완료 후 바로 꺼진다.

- docker 컨테이너 실행 상태를 유지하기 위해서는 지속적으로 실행되는 서비스를 run 명령어의 인자로 실행 시켜야한다.
예를들어

```bash
 docker run -d --name my-ctn my-image /my-back-end-service
```

- docker run 으로 /bin/bash 실행 후 컨테이너를 실행 상태로 유지하고 호스트 터미널로 빠져나오려면 ctrl + p + q 를 이용해야한다.
예를 들어

```bash
  docker run -it --name my-ctn my-image /bin/bash
```

로 실행 시키고 호스트 빠져 나올땐
**ctrl + p + q** 키보드 버튼으로 빠져나오고
차후에 다시 해당 컨테이너의 bash를 사용하려면

```bash
  docker attach my-ctn
```

위 방법으로 다시 bash 에 붙을 수 있다

#### 호스트에 있는 파일을 컨테이너에 옮기는 방법

먼저 컨테이너가 실행중이여야하며, 컨테이너가 실행중이라면

```bash
  docker cp <호스트에 존재하는 파일 경로> <타겟 컨테이너 이름>:<컨테이너 내부에 복사된 파일이 위치할 경로>
```

위 방법으로 파일 복사가 가능하다.

#### 컨테이너에서 systemctl 사용 불가 현상

컨테이너에서 systemctl 을 사용하려하면 **Failed to get D-Bus connection: Operation not permitted** 이라는 에러가 발생합니다.

이를 해결하는 방법들을 찾아보면 **--privileged** 같은 방법을 알려주는 곳이 종종 있는데 이 옵션을 쓸 경우 컨테이너를 통하여 호스트 os에 침범할 수 있는 보안취약점이 생길 수 있어 사용하면 안됩니다.