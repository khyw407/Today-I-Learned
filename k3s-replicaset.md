# Replicaset

## 1. 개념
![replicaset](https://user-images.githubusercontent.com/37721713/71571480-cfe29380-2b1d-11ea-9e57-5e42ed2f73cd.PNG)

- Pod를 여러개(한개 이상) 복제하여 관리하는 오브젝트
    - Pod를 생성하고 개수를 유지
    - 복제할 개수, label selector, template 등을 설정으로 가지고 있음
- 기능이 더 많은 Deployment를 더 자주 사용하나 Replicaset이 기반임

## 2. 실습
1) YAML 파일
```
apiVersion: apps/v1beta2
kind: ReplicaSet
metadata:
  name: hello-rs
spec:
  replicas: 2
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

2) 확인
```
$ kubectl get pod --show-labels

$ kubectl label pod/hello-rs-<{hash값}> type-

$ kubectl label pod/hello-rs-<{hash값}> type=world

$ kubectl scale --replicas=4 -f {YAML파일}

$ kubectl get pod -o wide
```

## 3. 출처
- https://kubernetes.io/ko/docs/concepts/workloads/controllers/replicaset/