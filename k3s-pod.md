# Pod

## 1. 개념
![pod](https://user-images.githubusercontent.com/37721713/71571169-644bf680-2b1c-11ea-84db-56a724f81703.PNG)

- 쿠버네티스의 가장 작은 배포단위
    - 하나 이상의 컨테이너가 포함된 집합(1개의 Pod에는 1개의 컨테이너가 권장됨)
    - 종속도가 높은 여러 컨테이너가 있을 경우 컨테이너들은 단일 Pod로 패키징

- Pod에 속한 스토리지와 네트워크는 공유되고 localhost로 접근 가능
- 컨테이너가 1개인 경우도 반드시 Pod로 감싸서 관리해야 함
- Pod는 언제든지 죽을 수 있음

## 2. 실습
1) YAML 파일
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

2) kubectl 명령어
```
$ kubectl apply -f <filename>

#삭제시
$ kubectl delete -f <filename>
```

3) 확인

![get](https://user-images.githubusercontent.com/37721713/71654533-9310d980-2d75-11ea-8d6b-962738f32784.png)

![describe](https://user-images.githubusercontent.com/37721713/71654535-93a97000-2d75-11ea-833a-2a82d35e7ffe.png)

```
$ kubectl get pods
$ kubectl get pods -o wide
$ kubectl get pods -o yaml
$ kubectl get pods -o json

$ kubectl describe pods {Pod Name}
```

4) Pod 로그 확인
```
$ kubectl logs {Pod Name}
```

5) Pod 접속

![exec](https://user-images.githubusercontent.com/37721713/71654534-93a97000-2d75-11ea-841e-32030befc61c.png)

```
$ kubectl exec -it {Pod Name} sh
```

## 3. 출처
- https://kubernetes.io/ko/docs/concepts/workloads/pods/pod-overview/
