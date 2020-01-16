# Helm

* 설치는 helm 3.0.1 버전을 기준(2.x 버전과는 달리 tiller 서버가 제거됨)
* Private Chart Repository로 Chartmuseum을 사용하였으며 Rancher에서 Catalog 형태로 설치
* helm push 명령어 사용을 위해 helm-push 플러그인을 별도로 설치하였음

## 1. 개념
1) helm 특징
- 복잡한 어플리케이션 배포 관리
  - 쿠버네티스 환경에서의 복잡한 애플리케이션의 배포를 코드(helm chart)로 관리하여 자동 배포를 제공
  - 애플리케이션의 빠른 배포를 통해 다양한 테스트 환경 배포 및 운영 환경 배포 시간을 단축

- Hooks
  - 쿠버네티스 환경에서 helm을 통해 chart를 설치, 업그레이드, 삭제, 롤백 등의 생명주기에 개입 가능

- 릴리즈 관리
  - helm으로 배포된 애플리케이션은 하나의 릴리즈로 불림(해당 릴리즈는 배포된 애플리케이션의 버전 관리를 가능하도록 함)

2) 구성
![1](https://user-images.githubusercontent.com/37721713/72503425-0985ff00-387f-11ea-860d-2b94c0405891.PNG)

- Helm Chart : 쿠버네티스에서 리소스를 만들기 위한 템플릿화 된 yaml 형식의 파일

- Helm Repository : 해당 리포지토리에 있는 모든 차트의 메타데이터를 포함하는 저장소(Public, Private 존재)

- Helm Client : 외부의 저장소에서 Chart를 가져오거나 gRPC로 Helm Server와 통신하여 요청

- Helm Server(Tiller) : 클라이언트의 요청을 처리하며 chart를 설치하고 릴리즈를 관리

## 2. 설치
1) helm 설치

```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3

$ chmod 700 get_helm.sh

$ ./get_helm.sh

# 설치확인
$ helm version
```

helm 3.x 버전부터 설치가 더욱 간결해졌고 tiller 서버 등 별도의 설치요소들이 제거되었다.(단순히 스크립트로만 설치가 가능)

2) Chartmuseum 설치
* 가장 많이 사용하는 Private Chart Repository

![2](https://user-images.githubusercontent.com/37721713/72503426-0a1e9580-387f-11ea-8e7b-d43aa7ce7c7f.PNG)

![3](https://user-images.githubusercontent.com/37721713/72503428-0a1e9580-387f-11ea-916c-c4957bda7cbe.PNG)

![4](https://user-images.githubusercontent.com/37721713/72503429-0ab72c00-387f-11ea-8f9f-5416309ece58.PNG)

```
# helm repository 추가
$ helm repo add chartmuseum http://chartmuseum-chartmuseum.chartmuseum.10.52.181.241.xip.io/

# helm repo 업데이트
$ helm repo update

# 확인
$ helm repo list
```

3) helm-push 설치
```
# repository에 chart를 push하기 위한 플러그인
$ helm plugin install https://github.com/chartmuseum/helm-push

# 설치확인
$ helm push --help
```

## 3. 배포 샘플
1) chart 생성
```
# 테스트용 디폴트 chart 생성
$ helm create mychart
```

2) 프로젝트 구조 및 디렉토리와 파일의 역할
```
mychart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

- charts/ : 해당 디렉토리에 종속성을 가지고 있는 helm chart를 저장한다.

- templates/ : 실제 배포에 필요한 yaml 파일이 저장되어 있다. 각 yaml 파일은 템플릿화되어 지정된 변수에 따라서 릴리즈를 생성할 수 있도록 제공된다.
  - deployment.yaml : kubernetes 오브젝트 중 deployment 형태로 배포되기 위해 사용되는 yaml 파일
  - ingress.yaml : kubernetes 오브젝트 중 ingress 형태로 배포되기 위해 사용되는 yaml 파일
  - service.yaml : kubernetes 오브젝트 중 service 형태로 배포되기 위해 사용되는 yaml 파일
  - NOTES.txt : 배포 후 사용자에게 제공되는 사용법이나, 구조 등이 설명되어 있는 txt파일로 대부분의 경우 서비스 접속방법이나 로그인 정보를 추가

- values.yaml : 템플릿화 되어 있는 chart의 변수(기본값)를 정의

- Chart.yaml : Chart에 대한 정보가 포함되어 있는 yaml파일(명시된 version은 helm repository에서 버전별로 관리)

3) chart 파일 예시
3-1) values.yaml
```
...

replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name:

...
```

- values.yaml에는 템플릿화된 template/ 디렉토리 하위의 yaml 파일들에 대하여 변수를 정의한다.

3-2) deployment.yaml
```
...

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "mychart.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "mychart.selectorLabels" . | nindent 8 }}
    spec:
    {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}

...
```

- 실제 deployment로 배포되는 yaml이 정의되어 있으며 위에서 설정한 values.yaml 파일에 정의된 변수들을 호출하여 재사용 가능도록 작성되어 있음.

4) Repository push
```
# mychart라는 새로 생성한 chart를 chartmuseum 리포지토리에 push
$ helm push mychart chartmuseum

# 리포지토리 업데이트
$ helm repo update

# 확인
$ helm search repo mychart
```

5) install과 확인
```
# first-chart라는 이름으로 helm chart를 실행하고 chartmuseum리포지토리의 mychart를 다운받아 설치
$ helm install first-chart chartmuseum/mychart

# 확인
$ helm ls

$ kubectl get all
```

- helm ls명령어로 현재 사용중인 chart를 확인할 수 있음

- kubectl get all 사용시 pod, service, ingress 등의 오브젝트가 새로 생성된걸 확인 가능

6) 버전 변경(업그레이드) 및 접속 테스트
6-1) mychart/Chart.yaml 수정
```
version: 0.2.0 
```
- (0.1.0을 0.2.0으로 변경)

6-2) mychart/values.yaml
```
service:
  type: NodePort
```

- 디폴트로 ClusterIP를 사용하는 것을 NodePort 타입으로 변경하여 외부접속 테스트가 가능하도록 함

6-3) upgrade
```
# 변경사항 적용(변경내용은 chart repository에도 반영되어 관리됨)
$ helm upgrade first-chart mychart/

# 확인시 Revision이 증가한 것을 볼수 있음
$ helm ls

# revision history 확인
$ helm history first-chart
```

6-4) 접속
- 엔드포인트 정보를 확인하고 접속

![5](https://user-images.githubusercontent.com/37721713/72503430-0ab72c00-387f-11ea-82b4-71b53b8a81cc.PNG)