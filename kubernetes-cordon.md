# Kubetctl cordon, drain

## 1. Kubetctl cordon, drain

### 1) 언제 사용하는가?

쿠버네티스 클러스터를 사용하다 특정 노트에 있는 pod를 모두 다른 곳으로 옮기거나(장애 등의 요인) 쿠버네티스에서 스케줄링이 되지 않도록 제한을 걸 필요가 있습니다. 이런 기능을 제공하는 명령어가 cordon과 drain 입니다.

### 2) cordon 사용법

kubectl cordon은 지정된 노드에 더이상 pod들이 스케쥴링되서 실행되지 않도록 합니다. cordon을 해보기 위해서 노드 이름을 확인한 다음에 cordon을 합니다. 
cordon을 한 다음에 다시 노드를 확인해 보면 노드의 status에 Ready외에 SchedulingDisabled이 추가된걸 확인할 수 있습니다.

![1](https://user-images.githubusercontent.com/37721713/82412505-bcc16780-9aae-11ea-8d09-cc0cd1eb368e.PNG)

![2](https://user-images.githubusercontent.com/37721713/82412511-bdf29480-9aae-11ea-8b6f-b6c942dd6d94.PNG)

![3](https://user-images.githubusercontent.com/37721713/82412512-be8b2b00-9aae-11ea-96f2-527a18cb776a.PNG)

### 3) drain 사용방법
kubectl drain은 노드 관리를 위해서 지정된 노드에 있는 pod들을 다른곳으로 이동시키는 명령입니다. 
우선 새로운 pod가 노드에 스케쥴링 되어서 실행되지 않도록 설정합니다. 그리고 나서 기존에 이 노드에서 실행중이던 pod들을 삭제합니다. (단, daemonset과 컨트롤러에 의해 실행되지 않는 pod 등이 있으면 drain 실패)
daemonset, 단일 pod 등이 있는 경우 옵션을 통해 강제로 drain을 수행하는 것이 가능합니다.

![4](https://user-images.githubusercontent.com/37721713/82412514-be8b2b00-9aae-11ea-8977-70b35778a93c.PNG)

![5](https://user-images.githubusercontent.com/37721713/82412516-bf23c180-9aae-11ea-9474-0b7df5cdbcd7.PNG)

### 4) uncordon

uncordon 명령어로 노드가 다시 스케줄링 될 수 있도록 합니다.

![6](https://user-images.githubusercontent.com/37721713/82412520-bfbc5800-9aae-11ea-8885-e8482d4fe461.PNG)
