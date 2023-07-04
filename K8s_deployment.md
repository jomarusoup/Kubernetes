# k8s 기본 명령어

## Run conatiner > pod > replicaset > deployment
- pod : 컨테이너들을 포함
- replicaset : 일정 개수의 pod 유지
- deployment : 각 worker node로의 업데이트와 배포 

```shell
$ kubectl apply -f {.yaml file 절대 경로}
```
- pod, replicaset, deployment가 없으면 생성후 컨테이너를 실행
- pod, replicaset, deployment가이미 존재하면 변경된 .yaml파일을 적용

```shell
# 각 pod, replicaset, deployment 삭제 시
# pod = po
# replica set = rs
# deployemnt 
$ kubectl delete po {pod_name} -n {namespaces_name}
$ kubectl delete rs {replicaset_name} -n {namespaces_name}
$ kubectl delete deployment {deployment_name} -n {namespaces_name}
```

## List
```shell
$ kubectl get pods {pod_name} -n {namespaces_name}
$ kubectl get replicaset {replicaset_name} -n {namespaces_name}
$ kubectl get deployment {deployment_name} -n {namespaces_name}
```

## kubectl describe(상세정보 확인)
```shell
$ kubectl describe pods {pod_name}
$ kubectl describe replicaset {replicaset_name}
$ kubectl describe deployment {deployment_name}
```

## Namespace

- 쿠버네티스에서 리소스를 구분하고 격리하는 가상적인 공간입니다. 이를 통해 동일한 클러스터 내에서 작업 시 각각의 리소스를 분리하여 사용하고, 리소스 간 충돌을 방지하며, 리소스에 대한 접근 권한을 제어하고, 리소스 사용량을 제한할 수 있습니다.

- 쿠버네티스 클러스터 내에서 모든 리소스에 대한 접근 권한을 가지는 default 네임스페이스가 존재합니다. 이 외에도 사용자가 직접 생성할 수 있는 다른 네임스페이스를 사용할 수 있습니다.

- Namespace를 사용하면, 동일한 이름의 리소스를 다른 네임스페이스에서 사용할 수 있습니다. 예를 들어, default 네임스페이스에서 my-pod라는 이름의 Pod를 생성하고, test 네임스페이스에서도 my-pod라는 이름의 Pod를 생성할 수 있습니다.

- 리소스 간의 이름 충돌을 방지할 수 있습니다. 예를 들어, default 네임스페이스에서 my-pod라는 이름의 Pod를 생성하고, test 네임스페이스에서도 my-pod라는 이름의 Pod를 생성하더라도, 각각의 네임스페이스에서는 이름 충돌이 발생하지 않습니다.

- 사용하면, 리소스에 대한 접근 권한을 제어할 수 있습니다. 예를 들어, default 네임스페이스에서 생성한 리소스에 대해서는 모든 사용자가 접근할 수 있지만, test 네임스페이스에서 생성한 리소스에 대해서는 특정 사용자만 접근할 수 있도록 설정할 수 있습니다.

- 사용하면, 리소스 사용량을 제한할 수 있습니다. 예를 들어, default 네임스페이스에서 사용할 수 있는 CPU, 메모리 등의 자원을 제한하거나, test 네임스페이스에서 사용할 수 있는 CPU, 메모리 등의 자원을 늘릴 수 있습니다.

```shell
# namespamces를 생성하는 방법
$ kubectl create namespace {namespace}

# namespaces를 삭제하는 방법
$ kubectl delete namespace {namespace}

# namespaces를 변경 불가
```

```yaml
#--- apiVersion : Kubernetes API 버전 지정
apiVersion: apps/v1
#--- deployment
#- kind : 오브젝트의 종류 지정
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: ivs
  name: ivs
#--- replica set
spec:
  #- replicas : Deployment에서 생성할 Pod의 개수 지정
  replicas: 3
  #- selector : Pod를 선택하기 위한 라벨 셀렉터 지정
  selector:
    matchLabels:
      app: ivs
  strategy: {}
  #--- pod
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: ivs
    spec:
      containers:
      - image: 192.168.0.104:5000/ivs_ap_test
        imagePullPolicy: Never
        name: ivs
        command: ["/bin/bash"]
        args: ["-c", "tail -f /dev/null"]
        volumeMounts:
        - name: etc
          mountPath: /etc
        - name: mnt
          mountPath: /mnt
        - name: var
          mountPath: /var
        - name: usr
          mountPath: /usr
        - name: lib64
          mountPath: /lib64
        - name: bin
          mountPath: /bin
        resources:
          limits:
            memory: 2G
      restartPolicy: Always
      hostNetwork: true
      hostIPC: true
      hostPID: true
      dnsPolicy: ClusterFirstWithHostNet
      volumes:
      - name: etc
        hostPath:
          path: /etc
      - name: mnt
        hostPath:
          path: /mnt
      - name: var
        hostPath:
          path: /var
      - name: usr
        hostPath:
          path: /usr
      - name: lib64
        hostPath:
          path: /lib64
      - name: bin
        hostPath:
          path: /bin

```

|                   |                                                                               |
| :---------------- | :---------------------------------------------------------------------------- |
| `apiVersion`      | Kubernetes API 버전 지정                                                      |
| `kind`            | 오브젝트의 종류 지정                                                          |
| `replicas`        | Deployment에서 생성할 Pod의 개수 지정                                         |
| `selector`        | Pod를 선택하기 위한 라벨 셀렉터 지정                                          |
| `matchLabels`     | Pod를 선택하기 위한 라벨을 지정                                               |
| `strategy`        | Deployment의 업데이트 전략을 지정                                             |
| `template`        | Pod의 템플릿을 지정                                                           |
| `metadata`        | Pod의 메타데이터를 지정                                                       |
| `labels`          | Pod에 대한 라벨을 지정                                                        |
| `spec`            | Pod의 스펙을 지정                                                             |
| `containers`      | Pod 안에서 실행할 컨테이너를 지정                                             |
| `image`           | 컨테이너에서 사용할 이미지를 지정                                             |
| `imagePullPolicy` | 이미지를 가져올 때 사용할 정책을 지정                                         |
| `name`            | 컨테이너의 이름을 지정                                                        |
| `resources`       | 컨테이너에서 사용할 자원을 지정                                               |
| `restartPolicy`   | 컨테이너의 재시작 정책을 지정                                                 |
| `hostNetwork`     | 컨테이너에서 호스트의 네트워크를 사용할지 여부 지정                           |
| `hostIPC`         | 컨테이너에서 호스트의 IPC(Inter-Process Communication)를 사용할지 여부를 지정 |
| `hostPID`         | 컨테이너에서 호스트의 PID(Process ID)를 사용할지 여부 지정                    |
| `dnsPolicy`       | 컨테이너에서 사용할 DNS 정책을 지정                                           |
| `volumes`         | 컨테이너에서 사용할 볼륨 지정                                                 |
| `hostPath`        | 호스트의 경로를 지정                                                          |


# k8s에서 도커 이미지 구동하기

## 1. 이미지가 노드에 존재하는지 확인
```shell
$ docker images {images_name_1}
```
## 2. deployment create
```shell
$ kubectl create deployment {deployment_name_1} --image={image_name_1}
```

## 3. pod의 상태가 ErrImagePull/ImagePullBackOff 확인
```shell
$ kubectl get pod -w
```
## 4. 내부에 존재하는 이미지를 사용하도록 디플로이먼트 생성
- `-o yaml` : 사용자가 관리하기 쉽게 yaml형태로 추출
- `--dry-run=client` : 해당 내용을 실제로 적용하지 않고 명령 수행
```shell
$ kubectl create deployment {deployment_name_2} --dry-run=client -o yaml --image={images_name_1} > {images_name_2}.yaml
```

## 5. {images_name_2}.yaml 파일에 imagePullPolicy 추가

```shell
imagePullPolicy: Never 
```
- 외부에서 이미지를 가져오지 않고 호스트에 존재하는 이미지를 사용

```shell
$ vi {image_name_2}.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test2
  name: test2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test2
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test2
    spec:
      containers:
      - image: xpm:1.6.1
        imagePullPolicy: Never    # 추가
        name: xpm
        resources: {}
status: {}

```

## 6. kubectl apply -f {image name 2}
이렇게 수정한 yaml파일을 적용해도 ErrImagePull/ImagePullBackOff 발생

## 7. 오류가 발생하는 이미지 모두 삭제

```shell
$ kubectl delete deployment {deployment_name_1}
$ kubectl delete deployment {deployment_name_2}
```
## 8. Worknode로 이동
- Worker node에서 Master node에 존재했던 같은 이미지 생성
- Master Node로 돌아와 {image name 2}.yaml를 {image name 3}.yaml로 변경 및 복사

## {image name 3}.yaml 수정
- replicase를 1에서 원하는 개수로  수정
- 이름도 변경

```shell
$ sed -i 's/replicas: 1/replicas: 3/' xpm.yaml
$ sed -i 's/{image_name_2}/{image_name_3}/' xpm.yaml
```
## 9. Master Node에서 kubectl apply
- Worker Node에서 같은 이미지 빌드 확인
- {image name 3}.yaml 적용

## 6, 생성할 디플로이먼트를 yaml로 추출

```shell
$ kubectl create deployment ivs --dry-run=client -o yaml \
--image=192.168.0.104:5000/ivs_ap --replicas=3 --node-name=dev6-2 > ivs_ap.yaml
```