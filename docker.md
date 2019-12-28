# Docker의 역사

도커는 2013년 3월 산타클라라에서 열린 Pycon Conference에서 dotCloud의 창업자인 Solomon Hykes가 The future of Linux Container라는 세션을 발표하면서 처음 세상에 알려졌습니다.

이 발표 이후 도커가 인기를 얻기 시작했고, 2014년 6월 도커 1.0을 발표합니다. 현재 대부분의 엔터프라이즈에서 개발 및 운영에 사용중입니다.

# Docker란?

도커는 컨테이너 기반의 오픈소스 가상화 플랫폼입니다. 다양한 프로그램 또는 실행환경 등을 컨테이너로 추상화하고 동일한 인터페이스를 제공하여 프로그램의 배포 및 관리를 단순하게 해줍니다.

### 컨테이너(Container)

컨테이너는 격리된 공간에서 프로세스가 동작하는 기술입니다. VMWare나 VirtualBox와 같은 전가상화, CPU의 가상화 기술을 활용한 KVM, 반가상화 방식의 Xen는 추가적인 OS를 설치하게 되는데 이는 성능에 대한 이슈가 있었고, 이를 개선하기 위해 프로세스 격리 방식이 등장합니다.

리눅스에서는 이 방식을 리눅스 컨테이너라고 하고 단순히 프로세스를 격리시키기 때문에 가볍고 빠르게 동작합니다. CPU나 메모리를 프로세스가 필요한 만큼 할당하기 때문에 리소스를 더욱 효율적으로 사용가능 합니다.

하나의 서버에 여러개의 컨테이너를 실행하면 서로 영향을 미치지 않고 독립적으로 실행되어 마치 가벼운 VM을 사용하는 것과 같은 효과를 얻을 수 있습니다. 또한 각 컨테이너에 접속하여 필요한 패키지를 설치할 수 있고, 호스트의 자원을 공유하여 사용하는 기능도 지원합니다.

### 이미지(Image)

이미지는 컨테이너 실행에 필요한 파일과 설정값들을 포함하고 있는 것으로 상태값을 가지지 않고, 변하지 않습니다. 컨테이너는 이미지를 실행한 상탤고 보면 되고, 추가하거나 변경되는 값은 컨테이너에 저장됩니다.

이미지는 컨테이너를 실행하기 위한 모든 정보를 가지고 있기 때문에 의존성이 있는 패키지 또는 파일을 설치할 필요가 없습니다.

도커의 이미지는 Docker hub에 등록하거나 Docker Registry 저장소를 직접 만들어 관리할 수 있습니다.

### 레이어 저장방식



### 기본 명령어

| 명령어  |  설명  |
|---|---|
| run | 컨테이너 실행하기 |
| ps | 컨테이너 목록 확인하기 |
| stop | 컨테이너 중지하기 |
| rm | 컨테이너 제거하기 |
| logs | 컨테이너 로그보기 |
| images | 이미지 목록 확인하기 |
| pull | 이미지 다운로드하기 |
| rmi | 이미지 삭제하기 |

### 컨테이너 생성 실습

**ubuntu 컨테이너 생성**

```
docker run --rm -it ubuntu:18.04 /bin/sh
```

**간단한 웹 애플리케이션 생성**

```
docker run -d -p 4567:4567 subicura/docker-workshop-app:1
```

http://xxxx:4567 접속

**MySQL 생성**

```
docker run -d -p 3306:3306 \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --name mysql \
  mysql:5.7
```

Database 생성

```
docker exec -it mysql mysql
create database wp CHARACTER SET utf8;
grant all privileges on wp.* to wp@'%' identified by 'wp';
flush privileges;
quit
```

**Wordpress 생성**

```
docker run -d -p 8000:80 \
  -e WORDPRESS_DB_HOST=172.17.0.1 \
  -e WORDPRESS_DB_NAME=wp \
  -e WORDPRESS_DB_USER=wp \
  -e WORDPRESS_DB_PASSWORD=wp \
  wordpress
```

http://xxxx:8000 접속

**컨테이너 목록 확인**

```
docker ps -a
```

**컨테이너 로그**

```
docker logs xxx
```

**컨테이너 종료**

```
docker stop xxx
```

**컨테이너 삭제**

```
docker rm xxx
```

**이미지 목록 확인**

```
docker images
```

**네트워크 생성**

```
docker network create app-network
```

**네트워크에 연결된 컨테이너 생성**

```
docker run -d \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --name mysql \
  --network=app-network \
  mysql:5.7
```

```
docker exec -it mysql mysql
create database wp CHARACTER SET utf8;
grant all privileges on wp.* to wp@'%' identified by 'wp';
flush privileges;
quit
```

```
docker run -d -p 8000:80 \
  --network=app-network \
  -e WORDPRESS_DB_HOST=mysql \
  -e WORDPRESS_DB_NAME=wp \
  -e WORDPRESS_DB_USER=wp \
  -e WORDPRESS_DB_PASSWORD=wp \
  wordpress
```

## Exam 1. 방명록 만들기

**frontend**

이미지
- subicura/guestbook-frontend:latest

환경변수
- PORT # ex) 8000
- GUESTBOOK_API_ADDR # ex) backend:8000

**backend**

이미지
- subicura/guestbook-backend:latest

환경변수
- PORT # ex) 8000
- GUESTBOOK_DB_ADDR # ex) mongodb:27017

**mongodb**

이미지
- mongo:4

기본포트
- 27017

## 정리

```
docker stop xxx
docker rm xxx
docker system prune -a
```
