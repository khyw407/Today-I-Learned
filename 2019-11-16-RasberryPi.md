# Rasberry Pi

## 1. Rasberry Pi에 Docker 설치
Docker는 apt 설치가 아닌, 별도의 script를 이용해야 합니다.
아래의 설치 방법으로 설치시 edge버전(일종의 베타버전)이 설치된다.

```
$ curl -fsSL get.docker.com -o get-docker.sh

$ sudo sh get-docker.sh

$ sudo docker info
```

## 2. pi계정에 Docker 사용권한 부여
Rasberry Pi는 항시 pi 계정으로 로그인되고 Docker는 docker라는 그룹에 권한을 자동으로 부여합니다. pi계정을 docker 그룹에 포함시키면 sudo 명령어 없이 사용가능합니다.

```
$ sudo usermod -aG docker pi

$ docker info

이후 재부팅 필수입니다!!
```

## 3. Rasberry Pi에 k3s 설치
