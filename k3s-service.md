# Service

## 1. 개념
![service h:250 w:500](https://user-images.githubusercontent.com/37721713/70207653-c8bea480-176e-11ea-9cdd-c937f8001200.PNG)

- 네트워크와 관련된 Kubernetes 오브젝트
    - Pod를 외부 네트워크와 연결해주고 여러개의 Pod를 바라보는 내부 로드밸런서 역할
    - 내부 DNS에 서비스 이름을 도메인으로 등록하기 때문에 서비스 디스커버리 역할도 함
    - 멀티 포트 지원
    - 서비스가 로드밸런싱을 할 때 기본은 Round Robbin이나 지정도 가능
    - type은 크게 3가지(ClusterIP, NodePort, Load Balancer)

## 2. 실습
1) YAML 파일(디폴트로 ClusterIP를 사용)
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
externalIPs를 설정하는 경우 명시된 IP로 접근하는 경우에만 서비스 요청을 처리 가능

2) 확인
```
http://localhost:8200 에 접속하여 확인
```

3) 출처
- https://bcho.tistory.com/1262
- https://kubernetes.io/docs/concepts/services-networking/service/