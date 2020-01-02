# Docker 이미지를 pull 동작(Docker Hub)

```
$ docker pull nginx:latest
latest: Pulling from library/nginx
c4f8e1e649d2: Pull complete 
7d21d64ad6f4: Pull complete 
8458a7f987b1: Pull complete 
Digest: sha256:b2d89d0a210398b4d1120b3e3a7672c16a4ba09c2c4a0395f18b9f7999b768f2
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

이미지를 pull 받을 때 nginx:latest라는 이미지를 요청했지만, 실제 받아지는 이미지의 이름은 docker.io/library/nginx:latest이다.

도커 이미지의 이름은 단순한 문자열이지만 도커 레지스트리에서 내부적으로 파싱되며 일반적으로 <Namespace>/<Image Name>:<Tag>형식으로 구성됩니다.

따라서 ningx:latest의 정확한 이름은 library/nginx:latest이며 library는 도커 허브의 공식 이미지가 저장되어 있는 특별한 네임스페이스입니다.(docker.io는 이미지 저장소의 실제주소임)

도커 클라이언트의 기본 도커 레지스트리는 Docker Hub(docker.io)이며 이는 docker info 명령으로 확인 가능하다.(항목중 Registry를 확인해보면 됨)

```
$ docker info

Client:
 Debug Mode: false

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 9
 Server Version: 19.03.5
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: b34a5c8af56e510852c35414db4c1f4fa6172339
 runc version: 3e425f80a8c931f88e6d94a8c831b9d5aa481657
 init version: fec3683
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 4.19.75-v7l+
 Operating System: Raspbian GNU/Linux 10 (buster)
 OSType: linux
 Architecture: armv7l
 CPUs: 4
 Total Memory: 3.64GiB
 Name: raspberrypi
 ID: R3KC:VVE6:LD2J:7OEW:OMKJ:LJVT:NDHA:ZGHF:EMNZ:SYE4:7RPG:GVU5
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Username: khyw407
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```

이미지를 pull 받을 때 아래쪽에 sha256 digest 값이 존재하는데 Docker Hub에서 이 값을 가지고 pull 받는 것도 가능합니다.

```
Digest: sha256:b2d89d0a210398b4d1120b3e3a7672c16a4ba09c2c4a0395f18b9f7999b768f2
```

따라서 Docker Hub에서는 다음의 주소가 모두 같은 값을 가리키게 됩니다.

- nginx:latest
- nginx@sha256:b2d89d0a210398b4d1120b3e3a7672c16a4ba09c2c4a0395f18b9f7999b768f2
- library/nginx:latest
- docker.io/library/nginx:latest
- index.docker.io/library/nginx:latest

# pull받은 도커 이미지는 어디에 저장?

Docker pull시 로그메시지 중 다음을 보면 무언가를 나누어 받는 것을 확인할 수 있습니다. 이는 도커 이미지가 layer로 구성되기 때문입니다.
```
c4f8e1e649d2: Pull complete 
7d21d64ad6f4: Pull complete 
8458a7f987b1: Pull complete 
```

이미지를 pull 받으면 레이어들이 독립적으로 저장되고, 컨테이너는 실행시 레이어들을 차례로 쌓아올려 특정 위치에 마운트합니다. 이미지에 속하는 레이어들은 읽기 전용이기 때문에 변하기 않지만 마지막으로 컨테이너 전용 쓰기 가능한 레이어를 쌓고, 컨테이너에서 일어나는 모든 변경 사항을 이 레이어에 저장합니다.

# 저장된 레이어 확인

현재 사용중인 스토리지 드라이버 확인을 진행합니다.

```
$ docker info | grep Storage

Stroage Driver : overlay2
```

도커의 데이터는 기본적으로 시스템 상의 /var/lib/docker/에 저장되며 overlay2 드라이버로 저장된 레이어 데이터는 다시 image/overlay2/layerdb/sha256 아래에 저장됩니다.

```
----중략----
bc00a9f1b9448ed1828d1fb9e8633f03682edae7d727350582427fb3e2a3d4fb 5f33990226e216f08e6f3fce1e5f5c591ab5b3f7039a225f816b11cc980d733f
9a2e46fbc121877f878d9de154bad9744365d45c2ed6a08cdc8d1dffbfbc6f98
----중략----
```

다이제스트 값으로 된 디렉토리 목록을 확인할 수 있는데 문제는 도커 이미지를 다운 받으면서 출력된 값과 일치되는 것이 없습니다. 

이미지 ID가 한 가지가 아니기 때문에 발생하는 문제이며 이미지를 pull할 때 출력되는 다이제스트 값은 원격 도커 레지스트리에서 관리하는 고유한 ID입니다. 그리고 레이어의 ID는 또 별도로 존재합니다.

```
$ docker image inspect nginx 

----중략-----
"RootFS": {
    "Type": "layers",
    "Layers": [
        "sha256:bc00a9f1b9448ed1828d1fb9e8633f03682edae7d727350582427fb3e2a3d4fb",
        "sha256:5f33990226e216f08e6f3fce1e5f5c591ab5b3f7039a225f816b11cc980d733f",
        "sha256:9a2e46fbc121877f878d9de154bad9744365d45c2ed6a08cdc8d1dffbfbc6f98"
    ]
}
----중략----
```

여기서 레이어 목록을 확인해보면 위에서 layerdb/sha256 아래에 있는 디렉토리 이름과 일치하는 것을 확인할 수 있습니다.

layerdb/sha256에서 직접 확인해보면
```
$ cd /var/lib/docker/image/overlay2/layerdb/sha256

$ cd bc00a9f1b9448ed1828d1fb9e8633f03682edae7d727350582427fb3e2a3d4fb

$ ls -al
합계 264
drwx------  2 root root   4096  1월  2 13:43 .
drwxr-xr-x 42 root root   4096  1월  2 13:43 ..
-rw-r--r--  1 root root     64  1월  2 13:43 cache-id
-rw-r--r--  1 root root     71  1월  2 13:43 diff
-rw-r--r--  1 root root      8  1월  2 13:43 size
-rw-r--r--  1 root root 248077  1월  2 13:43 tar-split.json.gz
```

실제 데이터는 다른곳에 있습니다. 목록 중 cache-id라는 파일이 있는데 이 값을 출력해보면 실제 데이터가 있는 디렉토리의 다이제스트 값이 출력됩니다.

```
$ cat catche-id
256733196e9d73e846c92dbe38a12be0f39a725bb61dcbe3dc19944c9bf08531
```

레이어 ID는 고유한 값이지만, cache-id는 고유한 값이 아니므로 이미지를 풀 받은 시스템마다 달라집니다. 이 디렉토리는 /var/lib/docker 바로 아래의 overlay2에 있습니다. 

```
$ cd /var/lib/docker/overlay2/256733196e9d73e846c92dbe38a12be0f39a725bb61dcbe3dc19944c9bf08531/diff

$ ls
bin  boot  dev	etc  home  lib	media  mnt  opt  proc  root  run  sbin	srv  sys  tmp  usr  var
```

해당 레이어는 nginx:latest 이미지의 베이스 이미지에 해당하는 debian:buster-slim 이미지입니다. 리눅스의 기본 디렉토리 구성을 확인할 수 있습니다. 
주의할 사항은 해당 디렉토리에서 파일 생성, 수정, 삭제 등의 작업이 가능하므로 변경할 경우 변경사항들이 다른 이미지에 영향을 줄 수도 있습니다. 따라서 실제로는 이런 변경은 하면 안됩니다.

- 이는 로컬을 변경한 것이지 원격을 바꾼것이 아니므로 컨테이너 실행시에 영향을 받을 수 있으나, 삭제후 다시 pull을 받는 경우는 영향받지 않습니다.

# 컨테이너의 layer 계층 이해하기

도커 1.10 버전 이전에는 이미지와 레이어에 거의 차이가 없었습니다. 하지만 최근 버전에서는 '레이어=이미지'라는 공식이 성립되지 않습니다. 

최신 버전에서는 도커 이미지를 pull 받을 때 layer는 layer로 남겨두고 최종 layer를 기반으로 한 이미지만을 사용할 수 있습니다. 중간 layer로 컨테이너를 실행시키는 것이 가능은 하나 권장되지는 않습니다.

docker history 명령을 통해 이미지가 만들어지기 까지의 과정을 확인합니다.

```
$ docker history nginx:latest

IMAGE               CREATED             CREATED BY                                      SIZE                
980e14c219b2        4 days ago          /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
<missing>           4 days ago          /bin/sh -c #(nop)  STOPSIGNAL SIGTERM           0B 
<missing>           4 days ago          /bin/sh -c #(nop)  EXPOSE 80                    0B  
<missing>           4 days ago          /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B 
<missing>           4 days ago          /bin/sh -c set -x     && addgroup --system -…   49.1MB
<missing>           4 days ago          /bin/sh -c #(nop)  ENV PKG_RELEASE=1~buster     0B  
<missing>           4 days ago          /bin/sh -c #(nop)  ENV NJS_VERSION=0.3.7        0B  
<missing>           4 days ago          /bin/sh -c #(nop)  ENV NGINX_VERSION=1.17.6     0B
<missing>           4 days ago          /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B
<missing>           5 days ago          /bin/sh -c #(nop)  CMD ["bash"]                 0B 
<missing>           5 days ago          /bin/sh -c #(nop) ADD file:d252ae1c97d5c80e7…   48.8MB
```

해당 이미지는 베이스 이미지를 포함해 11단계의 명령으로 구성되어 있습니다. 도커 최신버전에서는 이미지에 메타데이터로 저장되는 부분은 레이어로 다루어 지지 않습니다. 따라서 CMD, LABEL, ENV, EXPOSE 등 메타데이터를 다루는 부분은 레이어로 저장되지 않고, ADD나 RUN이 일어나는 곳만이 레이어로 저장됩니다. 이는 pull 받을 때 출력되는 레이어 개수와 일치합니다. 

실제로 레이어로 사용되는 3줄만 남기고 설명하겠습니다.
```
layer3 /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B
layer2 /bin/sh -c set -x     && addgroup --system -…   49.1MB
layer1 /bin/sh -c #(nop) ADD file:d252ae1c97d5c80e7…   48.8MB
```

레이어는 아래부터 위로 쌓입니다. 따라서 layer1이 바닥이고 마지막이 layer3가 올라가면서 이미지가 됩니다. 해당 레이어들은 모두 읽기 전용이고, 컨테이너를 실행할 때 레이어 하나를 이미지 최상위에 올려 쓰기전용으로 사용합니다.

# 출처
- https://www.44bits.io/ko/post/how-docker-image-work
- http://pyrasis.com/docker.html