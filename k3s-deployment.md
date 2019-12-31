# Deployment

## 1. 개념

- Pod와 ReplicaSet에 대한 선언적 업데이트 제공
    - Deployment에서는 상태를 선언하고 현재 상태에서 의도하는 상태로 비율을 조정하며 변경
    - 새 ReplicaSet을 생성하는 Deployment를 정의하거나 새로운 것을 정의 가능

- Deployment가 소유하는 ReplicaSet은 별도로 관리하지 않도록 함

## 2. 실습
1) YAML 파일
```
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: hello-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
      type: world
  template:
    metadata:
      labels:
        app: hello
        type: world
    spec:
      conatiners:
      - name: hello-container
        image: {레파지토리/이미지명}
        ports:
        - containerPort: 8000
```

2) 확인
```
# 버전2로 업데이트
$ kubectl set image deployment/hello-deployment hello-container=khyw407/hello:v2 --record

$ kubectl apply -f {YAML 파일}

$ kubectl get rs -w

$ kubectl describe deployment/hello-deployment

# 버전1로 업데이트
$ kubectl set image deployment/hello-deployment hello-container=khyw407/hello:v1 --record

$ kubectl rollout history -f {YAML파일}

$ kubectl rollout history -f {YAML파일} --revision=2

$ kubectl rollout status deployment/hello-deployment

$ kubectl rollout undo deployment/hello-deployment

$ kubectl rollout undo deployment/hello-deployment --to-revision=3
```

3) 출처
- https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/