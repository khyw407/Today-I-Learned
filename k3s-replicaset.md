# Replicaset

## 1. 개념
![replicaset](https://user-images.githubusercontent.com/37721713/71571480-cfe29380-2b1d-11ea-9e57-5e42ed2f73cd.PNG)

- Pod를 여러개(한개 이상) 복제하여 관리하는 오브젝트
    - Pod를 생성하고 개수를 유지
    - 복제할 개수, label selector, template 등을 설정으로 가지고 있음
- 기능이 더 많은 Deployment를 더 자주 사용하나 Replicaset이 기반임
    - Deployment는 ReplicaSet을 생성하고 ReplicaSet은 Pod을 생성하고 Pod은 k8s 스케줄러가 관리

## 2. 실습
1) YAML 파일
```
apiVersion: apps/v1
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

![check1](https://user-images.githubusercontent.com/37721713/71655274-62cb3a00-2d79-11ea-8143-c0a7e1dde2bc.PNG)

```
$ kubectl get pod --show-labels
```

3) label 제거 테스트

![check2](https://user-images.githubusercontent.com/37721713/71655275-62cb3a00-2d79-11ea-8024-0189b1fbdd81.PNG)

```
#type라는 label을 제거
$ kubectl label pod/hello-rs-<{hash값}> type-
```

- label 제거시 새로운 pod가 생성됩니다. 

![check3](https://user-images.githubusercontent.com/37721713/71655277-6363d080-2d79-11ea-8c63-dc3c1e489648.PNG)

```
#type라는 label을 다시 추가
$ kubectl label pod/hello-rs-<{hash값}> type=world
```

- label 원복시 한개의 pod를 제거하고 yaml 파일에 선언한 replicas의 수를 유지합니다.

4) scale 변경

![scale](https://user-images.githubusercontent.com/37721713/71655278-6363d080-2d79-11ea-9a11-f0cca8d1f09d.PNG)

```
$ kubectl scale --replicas=4 -f {YAML파일}

$ kubectl get pod -o wide
```

## 3. 출처
- https://kubernetes.io/ko/docs/concepts/workloads/controllers/replicaset/
