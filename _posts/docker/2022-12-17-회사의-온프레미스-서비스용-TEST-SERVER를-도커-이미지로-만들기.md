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

기본 요구사항

- CentOS7.1908 버전을 사용해야한다.
- zulu  이 설치되어야 한다.
- Nginx가 설치되어야 한다.
- Elasticsearch 버전이 설치되어야 한다.

기업별 상세 요구사항

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

## 아래부터는 임시 작성 부분입니다

### 도커파일

```dockerfile
FROM centos:7.7.1908

# Create the softcamp user with UID 1000 and add to the sudo group
RUN useradd -m -u 1000 -G wheel softcamp \
    && echo 'softcamp ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Install Git and Elasticsearch dependencies
RUN yum install -y git java-1.8.0-openjdk-headless wget

# Declare build arguments for the user name and password
ARG GIT_USERNAME
ARG GIT_PASSWORD

# Clone the gx-server-config-management repository using the build arguments as credentials
RUN git clone https://$GIT_USERNAME:$GIT_PASSWORD@dev.azure.com/Security365/GateXcanner/_git/gx-se
rver-config-management /opt/gx-server-config-management

# Change ownership of the repository directory to the softcamp user
RUN chown -R softcamp:softcamp /opt/gx-server-config-management

# Set the working directory to the Elasticsearch installation directory
WORKDIR /opt/gx-server-config-management

RUN git pull origin main

# Copy service files
RUN cp -r /opt/gx-server-config-management/etc/systemd/system /etc/systemd/system

# daemon reload
RUN systemctl daemon-reload

# Run Elasticsearch as the softcamp user
USER softcamp

# Expose the Elasticsearch ports
EXPOSE 9200 9300
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

> nas 에 nfs 공유 설정까지 했지만 nfs 마운팅은 실패하였습니다. 이유를 못찾아서 결국 cifs 로 사용했습니다.

### 기본 데이터를 포함한 



### 도커 실행에 필요한 인자 자동으로 넣기


### 프로젝트에 필요한 Config 파일들을 별도의 Git Repo