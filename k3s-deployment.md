# Deployment

## 1. 개념

- Pod와 ReplicaSet에 대한 선언적 업데이트 제공
    - Deployment에서는 상태를 선언하고 현재 상태에서 의도하는 상태로 비율을 조정하며 변경
    - 새 ReplicaSet을 생성하는 Deployment를 정의하거나 새로운 것을 정의 가능

- Deployment가 소유하는 ReplicaSet은 별도로 관리하지 않도록 함

## 2. 실습

1) YAML 파일
```
apiVersion: apps/v1
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
      containers:
      - name: hello-container
        image: {레파지토리/이미지명}
        ports:
        - containerPort: 8000
```

2) deployment 생성

![create-deployment](https://user-images.githubusercontent.com/37721713/71658637-3539bd00-2d88-11ea-8b73-9c01b9ea70a2.png)

```
$ kubectl apply -f {yaml파일명}
```

3) 버전2로 업데이트

![set-image](https://user-images.githubusercontent.com/37721713/71658640-35d25380-2d88-11ea-97b4-2cce3734da3e.png)

![describe](https://user-images.githubusercontent.com/37721713/71658638-3539bd00-2d88-11ea-9924-f1dad40a15f7.png)

```
# 버전2로 업데이트
$ kubectl set image deployment/hello-deployment hello-container=khyw407/hello:v2 --record

$ kubectl get replicaset --watch

$ kubectl describe deployment/hello-deployment
```

4) 버전1로 업데이트

```
# 버전1로 업데이트
$ kubectl set image deployment/hello-deployment hello-container=khyw407/hello:v1 --record
```

5) revision 확인

![rollout](https://user-images.githubusercontent.com/37721713/71658639-3539bd00-2d88-11ea-8465-d9451f0909e6.png)

![undo](https://user-images.githubusercontent.com/37721713/71658636-34a12680-2d88-11ea-88d4-386a47874fac.png)

```
# revision 확인
$ kubectl rollout history -f {YAML파일}
 
# 특정 revision으로 업데이트
$ kubectl rollout history -f {YAML파일} --revision=2
 
# revision 상태확인
$ kubectl rollout status deployment/hello-deployment
 
# rollback
$ kubectl rollout undo deployment/hello-deployment
$ kubectl rollout undo deployment/hello-deployment --to-revision=3
```

6) 출처
- https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/