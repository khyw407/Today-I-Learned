# Ingress

## 1. 개념
1) 쿠버네티스 인그레스란?

- 네트워크 트래픽은 Ingress와 egress로 구분(Ingress는 외부로부터 서버 내부로 유입되는 네트워크 트래픽을, egress는 서버 내부에서 외부로 나가는 트래픽)

- 쿠버네티스의 Ingress는 외부에서 쿠버네티스 클러스터 내부로 들어오는 네트워크 요청(Ingress 트래픽을 어떻게 처리할지 정의)

- Ingress는 외부에서 쿠버네티스에서 실행 중인 Deployment와 Service에 접근하기 위한, 일종의 관문 (Gateway) 같은 역할을 담당한다. 

- Ingress를 사용하지 않았다고 가정했을 때, 외부 요청을 처리할 수 있는 선택지는 NodePort, ExternalIP 등이 있을 수 있음(이러한 방법들은 일반적으로 Layer 4 (TCP, UDP) 에서의 요청을 처리하며, 네트워크 요청에 대한 세부적인 처리 로직을 구현하기는 아무래도 한계)

- 쿠버네티스의 Ingress는 Layer 7에서의 요청을 처리할 수 있음(외부로부터 들어오는 요청에 대한 로드 밸런싱, TLS/SSL 인증서 처리, 특정 HTTP 경로의 라우팅 등을 Ingress를 통해 자세하게 정의)

- 외부 요청을 어떻게 처리할 것인지를 정의하는 집합인 Ingress를 정의한 뒤, 이를 Ingress Controller라고 부르는 특별한 웹 서버에 적용함으로써 추상화된 단계에서 서비스 처리 로직을 정의할 수 있음 

2) Ingress와 Ingress Controller

- 쿠버네티스에서 Ingress를 사용하기 위해서는 두 가지가 필요(첫 번째는 YAML 파일에서 [kind: Ingress] 로 정의되는 Ingress 오브젝트이고, 두 번째는 Ingress 규칙이 적용될 Ingress Controller)

- 그러나 Ingress는 단지 Ingress 규칙을 정의하는 선언적인 오브젝트일 뿐, 외부 요청을 받아들이는 실제 서버가 아님

- Ingress는 Ingress Controller라고 하는 특수한 서버 컨테이너에 적용되어야만 Ingress에 적용된 규칙이 활성화된다. 즉, Ingress Controller가 외부로부터 네트워크 요청을 수신했을 때, Ingress 규칙에 기반해 이 요청을 어떻게 처리할지를 결정가능

- 그렇다면, Ingress Controller를 어떻게 설치해 실행할 수 있을까? 이 경우는 Ingress Controller를 직접 운영할지, 클라우드 플랫폼에 위임할지에 따라서 조금 고민해 볼 필요가 있다. 직접 Ingress Controller를 구동하려면 Nginx 웹 서버 기반의 Nginx Ingress Controller를 사용할 수 있겠고, 클라우드 플랫폼에 위임하려면 GKE (Google Kubernetes Engine) 의 기능을 사용할 수도 있다.
만약 AWS에서 EKS 또는 EC2 기반의 Kubespray를 사용하고 있다면, Nginx Ingress Controller를 직접 생성해 사용하되, 외부에서 Nginx에 접근하기 위한 쿠버네티스 Service를 Load Balancer 타입으로 설정해 로드 밸런서를 생성하는 방법을 생각해 볼 수 있다. 이 때, ELB / NLB / ALB 중 어느 로드 밸런서를 선택해야 하는지는 쿠버네티스 애플리케이션의 특성을 고려해 결정해야 한다. (ex. gRPC 지원 여부, 대규모 트래픽 필요 여부 등)

## 2. 실습
1) Ingress Controller 설치

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
```

2) Ingress 테스트
2-1) Deployment & Service 예제
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello1
  template:
    metadata:
      labels:
        app: hello1
    spec:
      containers:
      - name: hello-container
        image: {이미지경로}
        ports:
        - containerPort: 8000
      imagePullSecrets:
      - name: {private 저장소의 secret}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello2
  template:
    metadata:
      labels:
        app: hello2
    spec:
      containers:
      - name: hello-container
        image: {이미지경로}
        ports:
        - containerPort: 8000
      imagePullSecrets:
      - name: {private 저장소의 secret}
---
apiVersion: v1
kind: Service
metadata:
  name: test-service1
spec:
  selector:
    app: hello1
  type: NodePort
  ports:
    - port: 80
      targetPort: 8000
---
apiVersion: v1
kind: Service
metadata:
  name: test-service2
spec:
  selector:
    app: hello2
  type: NodePort
  ports:
    - port: 80
      targetPort: 8000
```

2-2) Ingress 예제
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: {도메인만 가능(IP나 포트가 들어가면 x)}
    http:
      paths:
      - path: /test1/*
        backend:
          serviceName: test-service1
          servicePort: 80
      - path: /test2/*
        backend:
          serviceName: test-service2
          servicePort: 80
```

## 3. 참조문서

```
https://blog.naver.com/PostView.nhn?blogId=alice_k106&logNo=221502890249

https://arisu1000.tistory.com/27840
```