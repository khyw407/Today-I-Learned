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

### 1) k3s?

![캡처](https://user-images.githubusercontent.com/37721713/69027486-76615200-0a12-11ea-8b0f-eeeb0b95f83d.PNG)

    - k3s는 Rancher Lab에서 최소 자원을 사용하는 k8s 클러스터 구성을 위한 솔루션으로 시작되었고, 바이너리 전체가 약 40mb로 가볍다는 특징이 있다.

    - 현재 최신버전은 2019-11-18을 기준으로 v0.10.2이다.

    - 주로 Edge, IoT 등 저전력, 저사양 기반 ARM계열 컴퓨팅에 최적화 되어 있다.

    - k8s에서 Cloud Provider, Storage Plugin을 제거하였고, default 저장소가 etcd가 아닌 sqlite3로 되어 있다.

### 2) k3s 설치

```
$ sudo su

$ curl -sfL https://get.k3s.io | sh -
```

### 3) k3s 설치확인

k3s가 정상적으로 설치되었는지는 다음의 명령어를 통해 확인하면 된다.
```
$ kubectl get nodes

$ kubectl get deployments --all-namespaces

$ kubectl get pods --all-namespaces

$ kubectl get service --all-namespaces
```

아래의 명령어는 node를 추가할 때 사용하는 조인키이다.
```
$ sudo cat /var/lib/rancher/k3s/server/node-token
```

### 4) k3s에 어플리케이션 배포 실습

hello.js

```
var http = require('http');
var content = function(req, res) {
 res.end("Hello world!");
 res.writeHead(200);
}

http.createServer(content).listen(8000);
```

Dockerfile
```
FROM node:slim
EXPOSE 8000
COPY hello.js .
CMD node hello.js
```

Docker Hub Site
- [Docker Hub] https://hub.docker.com/

Docker Container Run
```
docker build -t {레파지토리/이미지명} .{Dockerfile경로}
-t : 레파지토리/이미지명:버전(태그옵션)

docker images
docker run -d -p 8100:8000 {docker 이미지명}
-d : 백그라운드 모드
-p : 포트변경

docker ps
docker exec -it {컨테이너id} /bin/bash
```

Docker Image Push
```
docker login
docker push {docker 이미지명}
```

Kubernetes
- Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  labels:
    app: hello
spec:
  containers:
  - name: hello-container
    image: {레파지토리/이미지명}
    ports:
    - containerPort: 8000
```

- Service(디폴트로 ClusterIP를 사용)
```
apiVersion: v1
kind: Service
metadata:
  name: hello-svc
spec:
  selector:
    app: hello
  ports:
    - port: 8200
      targetPort: 8000
  externalIPs:
  - {외부접속에서 접속하는IP}
```

yaml 파일을 생성하고 kubectl create -f {파일명.yaml} 형태로 생성하는 방법도 있음.

### 5) Kubernetes Dashboard 설치

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml
```

#### 5-1) Kubernetes Login 인증키 생성

serviceaccount 생성
```
$ cat << EOF | kubectl create -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
EOF
```

ClusterRoleBinding 생성
```
$ cat << EOF | kubectl create -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
EOF
```

토큰 확인
```
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}') 
```

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login 접속 후 토큰 인증
```

#### 5-2) Proxy를 사용하는 방법

```
$ kubectl proxy 명령어로 실행

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/ 접속하여 확인
```

```
롤백시 아래의 명령어를 사용

$ kubectl delete -f dashboard-admin.yaml
```

#### 5-2) NodePort를 사용하는 방법

Kubernetes Service edit
```
$ kubectl edit service kubernetes-dashboard -n kubernetes-dashboard
```

yaml 파일 수정
```
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"k8s-app":"kubernetes-dashboard"}
  creationTimestamp: "2019-11-19T08:59:26Z"
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
  resourceVersion: "26067"
  selfLink: /api/v1/namespaces/kubernetes-dashboard/services/kubernetes-dashboard
  uid: 9d8c9397-0a70-4539-9878-dc61f0dbafff
spec:
  clusterIP: 10.43.136.241
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 31377
    port: 443
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: ClusterIP   # 이 부분을 ClusterIP -> NodePort로 수정
status:
  loadBalancer: {}
```

바인딩 포트 확인
```
$ kubectl get service kubernetes-dashboard -n kubernetes-dashboard
```

https://<master-ip>:바인딩포트로 접속
```
토큰 확인은 아래의 명령어 사용

$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}') 
```

### 6. 출처

```
https://k3s.io/

https://kubernetes.io/ko/

https://docs.docker.com/get-started/
```
