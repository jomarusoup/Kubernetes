# 1. 쿠버네티스 : RHEL 기반

- root 관리자로 쿠버네티스 설정하기

## 1-1. 각 서버 확인 항목

1. 모든 서버의 시간이 ntp를 통해 동기화돼 있는지 확인

```shell
$ timedatectl set-timezone Asia/Seoul
```

2. 모든 서버의 MAC주소가 다른지 확인

### 1-1-1. 용어 정리

- 쿠버네티스 > 클러스터 > 노드 > 파드 > 컨테이너
- 쿠버네티스 : 다수의 컨테이너를 자동으로 배포, 스케일링, 관리할 수 있도록 도와주는 오픈소스 소프트웨어
- 클러스터 : 여러 대의 노드로 구성된 컴퓨팅 시스템. 쿠버네티스에서 어플리케이션을 실행하는데 필요한 모든 리소스를 제공
  - 클러스터 = 하나 이상의 Master Node + 여러 개의 Woker Node
- 노드 : 클러스터 내에서 어플리케이션 컨테이너를 실행하는 호스트, 클러스터의 일부, App 컨테이너를 실행하기 위한 환경을 제공
  - 환경을 제공하기 위해 노드는 컨테이너 런타임, 스토리지, 네트워크, kubelet 등의 구성 요소를 실행
- 파드 : 쿠버네티스에서 파드(Pod)는 하나 이상의 컨테이너를 포함하는 가장 작은 배포 단위
  - 파드는 노드에서 실행, 컨테이너를 실행하기 위한 환경을 제공
  - 파드는 컨테이너를 논리적으로 그룹화 -> 컨테이너 간의 통신과 공유 리소스에 대한 접근

### 1-1-2. 각 서버 IP 주소 및 Control Plane / Worker Node

| host name |      Node      |  ID Address   | Port                             |
| :-------: | :------------: | :-----------: | :------------------------------- |
|  Dev6-4   | Control Plane  | 192.168.0.104 | 6443 2379-2380 10250 10259 10257 |
|  Dev6-1   | Worker node #1 | 192.168.0.101 | 10250, 30000-32767               |
|  Dev6-2   | Worker Node #2 | 192.168.0.102 | 10250, 30000-32767               |
|  Dev6-3   | Worker Node #3 | 192.168.0.103 | 10250, 30000-32767               |

| Kubernets Port(M) | Explain                                                                |
| :---------------- | :--------------------------------------------------------------------- |
| 6443/tcp          | Kubernetes API 서버와 통신하는 데 사용됩니다.                          |
| 2379-2380/tcp     | etcd 데이터베이스에 대한 클러스터 멤버십 및 데이터 전송에 사용됩니다.  |
| 10250/tcp         | Kubernetes API 서버와 kubelet 간의 통신에 사용됩니다.                  |
| 10251/tcp         | Kubernetes API 서버와 kube-scheduler 간의 통신에 사용됩니다.           |
| 10252/tcp         | Kubernetes API 서버와 kube-controller-manager 간의 통신에 사용됩니다.  |
| 10257/tcp         | Kubernetes API 서버와 kube-proxy 간의 통신에 사용됩니다.               |
| 10248/tcp         | Kubernetes API 서버와 kubelet 간의 read-only Kubelet API에 사용됩니다. |
| 16443/tcp         | Kubernetes API 서버와 kubelet 간의 TLS 통신에 사용됩니다.              |

| Kubernets Port(W) | Explain                                                                |
| :---------------- | :--------------------------------------------------------------------- |
| 30000-32767/tcp   | Kubernetes 서비스 유형 LoadBalancer에 대한 노출에 사용됩니다.          |
| 10250/tcp         | Kubernetes API 서버와 kubelet 간의 통신에 사용됩니다.                  |

| HTTP Port | Explain                        |
| :-------- | :----------------------------- |
| 80/tcp    | HTTP 트래픽을 위한 포트입니다. |

| Private registry Port | Explain                                            |
| :-------------------- | :------------------------------------------------- |
| 5000/tcp              | Docker 레지스트리에 대한 액세스를 위한 포트입니다. |

| Calico Port | Explain                                                     |
| :---------- | :---------------------------------------------------------- |
| 179/tcp(M, W)     | BGP 프로토콜을 사용하는 Calico 노드 간의 통신에 사용됩니다. |
| 4789/tcp(M)    | Calico 노드 간의 VXLAN 터널링에 사용됩니다.                 |
| 5473/tcp(M)    | Calico 노드 간의 통신에 사용됩니다.                         |
| 443/tcp(M)     | Kubernetes API 서버와 통신하는 데 사용됩니다.               |
| 4291/tcp(M)    | Calico 노드 간의 통신에 사용됩니다.                         |

| Flanneld Port | Explain                               |
| :------------ | :------------------------------------ |
| 8285/udp      | Flannel 노드 간의 통신에 사용         |
| 8472/udp      | Flannel 노드 간의 VXLAN 터널링에 사용 |


마스터 노드에서 필요한 포트:

4291/tcp : Calico 노드 간의 통신에 사용됩니다.
6443/tcp : Kubernetes API 서버와 통신하는 데 사용됩니다.
2379-2380/tcp : etcd 데이터베이스에 대한 클러스터 멤버십 및 데이터 전송에 사용됩니다.
10250/tcp : Kubernetes API 서버와 kubelet 간의 통신에 사용됩니다.
10259/tcp : kube-controller-manager와 kube-scheduler 간의 통신에 사용됩니다.
10257/tcp : Kubernetes API 서버와 kube-proxy 간의 통신에 사용됩니다.
10251/tcp : Kubernetes API 서버와 kube-scheduler 간의 통신에 사용됩니다.
10252/tcp : Kubernetes API 서버와 kube-controller-manager 간의 통신에 사용됩니다.
443/tcp : Kubernetes API 서버와 통신하는 데 사용됩니다.
16443/tcp : Kubernetes API 서버와 kubelet 간의 TLS 통신에 사용됩니다.
179/tcp : BGP 프로토콜을 사용하는 Calico 노드 간의 통신에 사용됩니다.
5473/tcp : Calico 노드 간의 통신에 사용됩니다.
워커 노드에서 필요한 포트:

10250/tcp : Kubernetes API 서버와 kubelet 간의 통신에 사용됩니다.
30000-32767/tcp : Kubernetes 서비스 유형 LoadBalancer에 대한 노출에 사용됩니다.
179/tcp : BGP 프로토콜을 사용하는 Calico 노드 간의 통신에 사용됩니다.
4789/udp : Calico 노드 간의 VXLAN 터널링에 사용됩니다.
5473/tcp : Calico 노드 간의 통신에 사용됩니다.

## 1-2. (오류 발생 시) 클러스터 삭제

쿠버네티스 설치 중 오류가 발행하면 클러스터를 삭제하고 `kubeadm init`부터 다시 클러스터 초기화

```shell
# Master Node & Worker node
$ kubeadm reset

# Master Node & Worker node
$ rm -r /etc/cni/net.d
rm: descend into directory '/etc/cni/net.d'? y
rm: remove 일반 파일 '/etc/cni/net.d/10-flannel.conflist'? y
rm: remove 디렉토리 '/etc/cni/net.d'? y

# Master Node & Worker node
$ rm -r /etc/kubernetes
rm: descend into directory '/etc/kubernetes'? y
rm: remove 디렉토리 '/etc/kubernetes/pki'? y
rm: remove 디렉토리 '/etc/kubernetes/manifests'? y
rm: remove 디렉토리 '/etc/kubernetes'? y

# Master Node
$ rm -r $HOME/.kube/config
rm: remove 일반 파일 '/root/.kube/config'? y
```
- 컨트롤 플레인을 포함한 각 노드에서 클러스터를 삭제하는 명령어
- 명령어 실행 후 재부팅을 해야하며 메모리 스왑 확인과 `kubelet`를 실행시켜줘야한다.


### Flannel 완전 삭제
- 순서대로 진행 중에 오류가 뜨거나 파일 혹은 디렉토리가 없다고 하면 무시하고 계속 진행

1. Flannel DaemonSet을 삭제 (클러스터에서 Flannel DaemonSet이 삭제)
```shell
$ kubectl delete -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

2. 모든 노드에서 Flannel 인터페이스를 삭제 (모든 노드에서 Flannel 인터페이스가 삭제)
```shell
$ ip link delete flannel.1
```

3. 모든 노드에서 Flannel 관련 파일을 삭제 (모든 노드에서 Flannel 관련 파일이 삭제)
```shell
$ rm -rf /var/run/flannel/
$ rm -rf /etc/flannel/
```

4. 모든 노드에서 Flannel 서비스를 중지하고 삭제
```shell
$ systemctl stop flanneld
$ systemctl disable flanneld
$ rm -rf /etc/systemd/system/flanneld.servic행행
```
5. flanneld 확인
```shell
$ systmectl status flanneld
```

## 1-3. /etc/hosts 파일 설정 - Control Plane과 모든 Node

모든 서버에서 시스템 호스트 이름과 /etc/host 파일 설정

### 1-3-1. /etc/hosts 파일 수정

```shell
$ vi /etc/hosts
```

```
# /etc/hosts에 작성
192.168.0.104 Dev6-4
192.168.0.101 Dev6-1
192.168.0.102 Dev6-2
192.168.0.103 Dev6-3
```

### 1-3-2. 각 호스트 이름에 대해 ping을 실행해 /etc/hosts에 정의된 올바른 IP주소로 지정

```shell
$ ping Dev6-4 -c3
$ ping Dev6-1 -c3
$ ping Dev6-2 -c3
$ ping Dev6-3 -c3
```

## 1-4. 방화벽 규칙 구성(Control plane / Worker node)

쿠버네티스는 일부 포드를 열어야 한다.

```shell
$ kubectl get pods --all-namespaces
$ kubectl get nodes -o wide
$ netstat -tupn | grep -E "6443|2379-2380|10250|10259|10257"
$ netstat -tulpn | grep -E "6443|2379-2380|10250|10259|10257"
$ netstat -an | grep 6443
```

### 1-4-1. [ Kubernetes 컨트롤 플레인의 경우 열어야 하는 포트 및 기동되는 프로세스 ]

| Protocol | Direction | Port Range | Process Name in Server  | Purpose Used By                              |
| :------: | :-------: | :--------- | :---------------------- | :------------------------------------------- |
|   TCP    |  Inbound  | 6443       | kube-apiserver          | Kubernetes API server All                    |
|   TCP    |  Inbound  | 2379-2380  |                         | etcd server client API  kube-apiserver, etcd |
|   TCP    |  Inbound  | 10250      | kubelet                 | Kubelet API Self, Control plane              |
|   TCP    |  Inbound  | 10259      | kube-scheduler          | kube-scheduler  Self                         |
|   TCP    |  Inbound  | 10257      | kube-controller-manager | kube-controller-manager Self                 |
|          |           |            | etcd                    |                                              |
|          |           |            | kube-proxy              |                                              |

| Port            | Explain                                                         |
| :-------------- | :-------------------------------------------------------------- |
| 443/tcp         | Kubernetes API 서버와 통신                                      |
| 2379-2380/tcp   | etcd 데이터베이스에 대한 클러스터 멤버십 및 데이터 전송         |
| 10250/tcp       | Kubernetes API 서버와 kubelet 간의 통신                         |
| 10251/tcp       | Kubernetes API 서버와 kube-scheduler 간의 통신                  |
| 10252/tcp       | Kubernetes API 서버와 kube-controller-manager 간의 통신         |
| 10257/tcp       | Kubernetes API 서버와 kube-proxy 간의 통신                      |
| 10248/tcp       | Kubernetes API 서버와 kubelet 간의 read-only Kubelet API에 사용 |
| 30000-32767/tcp | Kubernetes 서비스 유형 LoadBalancer에 대한 노출                 |
| 16443/tcp       | Kubernetes API 서버와 kubelet 간의 TLS 통신                     |
| 80/tcp          | HTTP 트래픽을 위한 포트                                         |
| 6443/tcp        | Kubernetes API 서버와 통신                                      |

### 1-4-2. 포트 삭제

```shell
# $ firewall-cmd --permanent --zone=public --remove-port=포트번호/프로토콜
$ firewall-cmd --permanent --zone=public --remove-port=<portnumber>/tcp
```

### 1-4-3. < Control Plane이 될 서버의 방화벽에 포트 추가 >

```shell
$ firewall-cmd --add-port=6443/tcp --permanent
$ firewall-cmd --add-port=2379-2380/tcp --permanent
$ firewall-cmd --add-port=10250/tcp --permanent
$ firewall-cmd --add-port=10259/tcp --permanent
$ firewall-cmd --add-port=10257/tcp --permanent
```

### 1-4-4. < 방화벽 reload 및 추가된 포트 확인 >

```shell
$ firewall-cmd --reload
$ firewall-cmd --list-all
```

### 1-4-5. [ Kubernetes 작업자 노드의 경우 열어야 하는 포트 ]

| Protocol | Direction | Port Range  | Process Name in Server |         Purpose Used By         |
| :------: | :-------: | :---------: | :--------------------: | :-----------------------------: |
|   TCP    |  Inbound  |    10250    |        kubelet         | Kubelet API Self, Control plane |
|   TCP    |  Inbound  | 30000-32767 |                        |     NodePort Services, All      |

### 1-4-6. < Worker Node가 될 서버의 방화벽에 포트 추가 >

```shell
$ firewall-cmd --add-port=10250/tcp --permanent
$ firewall-cmd --add-port=30000-32767/tcp --permanent
```

### 1-4-7. < 방화벽 reload 및 추가된 포트 확인 >

```shell
$ firewall-cmd --reload
$ firewall-cmd --list-all
```

## 1-4-8. SELinux "permissive" 변경(all server)

- Kubernetes 서비스 kubelet이 제대로 작동하도록 하려면 기본 SELinux를 "permissive"로 변경하거나 SELinux를 완전히 비활성화
- SELinux가 "permissive" 모드로 설정되어, 경고는 출력하지만 권한 검사는 수행하지 않는다.

### < 기본 SELinux 정책을 "permissive"로 변경 >

```shell
$ setenforce 0
```

- setenforce 0 명령어는 SELinux를 "permissive" 모드로 **일시적으로 변경**하는 명령어
- 현재 쉘 세션에서만 유효
- 새로운 쉘 세션에서는 다시 SELinux가 "enforcing" 모드로 설정

```shell
$ sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

- `sed` : 파일 내용을 변경하는 명령어
- `-i` : 파일 내용을 직접 변경
- 명령어는 /etc/selinux/config 파일에서 SELINUX=enforcing이라는 문자열을 찾아 SELINUX=permissive로 변경
- **영구적으로 변경**

### < SELinux가 여전히 활성화되어 있지만 "permissive" 정책이 적용 확인 >

```shell
$ sestatus
...                                      
Current mode:                      permissive 
Mode from config file:             permissive
...                                
```

## 2-6. Kubernetes는 커널 모듈 "overlay" 및 "br_netfilter"가 모든 서버에서 활성화되도록 요구

- iptbales가 브리지된 트래픽을 볼 수 있다

### < 커널 모듈 "overlay" 및 "br_netfilter"를 활성화 >

```shell
$ modprobe overlay
$ modprobe br_netfilter
```

### < 영구적으로 활성화 > ... \/etc/modules-load.d/k8s.conf\에 구성 파일 생성

```shell
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
> overlay          
> br_netfilter     
> EOF              
```

### < 필요한 systemctl 매개변수를 생성 >

```shell
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
> net.bridge.bridge-nf-call-iptables = 1   
> net.bridge.bridge-nf-call-ip6tables = 1  
> net.ipv4.ip_forward = 1  
> EOF   
```

< 재부팅 없이 새 sysctl 구성을 적용 >

```shell
sudo sysctl --system
```

## 2-7. 메모리 Swap off(all server)

< 영구적으로 Swap off >

```shell
$ sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

< 현재 세션에서 Swap off >

```shell
$ swapoff -a
$ sed -i -e '/swap/d' /etc/fstab
$ sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

< SWAP OFF 확인 >

```shell
$ free -m
...                                                      
           total           used             free          shared        buff/cache        available  
...                                                   
Swap :       0               0               0 
...  
```

## 2-8. 컨테이너 런타임 설치 : Containerd(all server)

- Kubernetes 클러스터를 설정하기 위해 포드가 실행 될 수 있도록 모든 서버에 컨테이너 런타임 설치
  - 컨테이너 런타임 : 컨테이너 이미지를 가져와 컨테이너를 생성, 실행, 상태를 관리하는 등의 역할
  - containerd, CRI-O, Mirantis Container Runtime, Docker Engine
- 모든 마스터와 워커 노드에 컨테이너 런타임 설치 필요
- 컨테이너 런타임 Conatinerd를 설치하기 위해 Docker 저장소에서 제공하는 바이너리 패키지 설치
  - Docker를 설치하면 컨테이너 런타임을 Docker Engine을 사용 (공식 홈페이지 참고)

### <저장소 추가 전에 "dnf-utils" 추가 도구 설치>

```shell
$ dnf install dnf-utils
```

### <CentOS 기반 시스템용 Docker 리포지토리를 추가>

```shell
$ sudo yum-config-manager \
>    --add-repo \
>    https://download.docker.com/linux/centos/docker-ce.repo
```

### <리포지토리를 확인 및 새 메타데이터 캐시를 생성>

```shell
$ dnf repolist
...
docker-ce-stable                 Docker CE Stable - x86_64
...
$ dnf makecache
...
Docker CE Stable - x86_64
...
```

### <containerd 패키지를 설치>

```shell
$ dnf install containerd.io
```

### <기본 containerd 구성을 백업하고 새 containerd 구성 파일을 생성>

```shell
$ mv /etc/containerd/config.toml /etc/containerd/config.toml.orig
$ containerd config default > /etc/containerd/config.toml
```

### <containerd 구성 파일 "/etc/containerd/config.toml"을 수정>

```shell
$ vi /etc/containerd/config.toml
```

- cgroup 드라이버 "SystemdCgroup = false"의 값을 "SystemdCgroup = true"로 변경
  - containerd 컨테이너 런타임에 대한 systemd cgroup 드라이버가 활성화

```shell
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    ...
    SystemdCgroup = true
  ...
```

### <containerd 서비스를 확인 및 containerd가 영구 활성화>

```shell
$ systemctl is-enabled containerd
$ systemctl status containerd
```

## 2-9. Kubernetes 패키지 설치

- 모든 Linux 시스템에 Kubernetes 패키지를 설치
- kubeadm : 쿠버네티스 클러스터를 부트스트래핑
- kubelet : 쿠버네티스 클러스터의 주요 구성 요소
- kubectl : 쿠버네티스 클러스터를 관리하기 위한 명령줄 유틸리티
  - 클러스터를 초기화하거나 노드를 클러스터에 조인할 때 모든 이벤트를 기다리는 기본 Kubernetes 서비스
- Kubernetes에서 제공하는 리포지토리를 사용하여 Kubernetes 패키지를 설치
- 모든 Rocky Linux 시스템에 Kubernetes 리포지토리를 추가 필요

### <RHEL/CentOS 기반 운영 체제용 Kubernetes 리포지토리를 추가>

```shell
$ cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
> [kubernetes]
> name=Kubernetes
> baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
> enabled=1
> gpgcheck=1
> gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
> exclude=kubelet kubeadm kubectl
> EOF
```

### <새 메타데이터 캐시를 생성 & Kubernetes 리포지토리가 추가 확인 >

```shell
$ yum repolist
...
Kubernetes                         Kubernetes
...
$ yum makecache
...
Kubernetes
...
```

### <yum 명령을 사용하여 Kubernetes 패키지>

```shell
yum install kubelet kubeadm kubectl --disableexcludes=kubernetes
```

### <systemctl 명령을 실행하여 kubelet 서비스를 시작하고 활성화>

- kubelet 서비스를 확인하고 모든 노드에서 활성화되고 실행 중인지 확인

```shell
$ sudo systemctl enable --now kubelet
```

## 2-10. CNI : Flannel & Calico

- CNI는 컨테이너 간의 네트워킹을 제어할 수 있는 플러그인을 만들기 위한 표준
- 다양한 형태의 컨테이너 런타임과 오케스트레이터 사이의 네트워크 계층을 구현하는 방식이 다양하게 분리
- 각자만의 방식으로 발전하게 되는 것을 방지하고 공통된 인터페이스를 제공이 목적
- 쿠버네티스에서는 Pod 간의 통신을 위해서 CNI 를 사용
- 쿠버네티스 뿐만 아니라 Amazon ECS, Cloud Foundry 등 컨테이너 런타임을 포함하고 있는 다양한 플랫폼들은 CNI를 사용
- 쿠버네티스는 기본적으로 'kubenet' 이라는 자체적인 CNI 플러그인을 제공
  - 네트워크 기능이 매우 제한적인 단점이 있습니다.
- Kubenet의 단점을 보완하기 위해, 3rd-party 플러그인을 사용
  - Flannel, Calico, Weavenet, NSX 등 다양한 종류의 3rd-party CNI 플러그인들이 존재

|     Provider      |  Network Model  | Route Distribution | NetworkP olicy | Mesh  | External Datastore | Encryption | Ingress/Egress Policies | Commercial Support |
| :---------------: | :-------------: | :----------------: | :------------: | :---: | :----------------: | :--------: | :---------------------: | :----------------: |
|      Calico       |     Layer 3     |        Yes         |      Yes       |  Yes  |        Etcd        |    Yes     |           Yes           |        Yes         |
|      Flannel      |      vxlan      |         No         |       No       |  No   |        None        |     No     |           No            |         No         |
|       Canal       |      vxlan      |         No         |       No       |  No   |        None        |     No     |           No            |         No         |
| kopeio-networking |       BGP       |         No         |      Etcd      |  No   |        Yes         |    Yes     |           Yes           |         No         |
|    kube-router    |     Layer 3     |        BGP         |      Yes       |  No   |         No         |     No     |           No            |         No         |
|      romana       |     Layer 3     |        OSPF        |      Yes       |  No   |        Etcd        |     No     |           Yes           |        Yes         |
|     Weave Net     | Layer 2 / vxlan |        N/A         |      Yes       |  Yes  |         No         |    Yes     |           Yes           |        Yes         |

## CNI : Flannel
1. Flannel를 위한 포트 open
```shell
$ firewall-cmd --add-port=8285/tcp --permanent
$ firewall-cmd --add-port=8472/tcp --permanent
$ firewall-cmd --reload
```
| Port     | Explain                               |
| :------- | :------------------------------------ |
| 8285/udp | Flannel 노드 간의 통신에 사용         |
| 8472/udp | Flannel 노드 간의 VXLAN 터널링에 사용 |

2. 새 디렉터리 "/opt/bin" 생성

```shell
$ mkdir -p /opt/bin/
```

3. Flannel의 바이너리 파일 다운로드

```shell
$ curl -fsSLo /opt/bin/flanneld https://github.com/flannel-io/flannel/releases/download/v0.19.0/flanneld-amd64
```

4. 파일의 권한을 변경하여 "flanneld" 바이너리 파일을 실행 가능하게 변경

```shell
$ chmod +x /opt/bin/flanneld
```

## CNI : Calico
- root에서 진행
- `kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address={Control_Plane_IP}` 로 초기화 이후 진행
- 기본적으로 192.168.0.0\16 대역을 사용 (변경 가능)

1. Calico를 사용하기 위한 추가 포트 open
```shell
# all node
$ firewall-cmd --add-port=179/tcp --permanent
$ firewall-cmd --add-port=4789/udp --permanent
$ firewall-cmd --add-port=5473/tcp --permanent
$ firewall-cmd --add-port=443/tcp --permanent
$ firewall-cmd --add-port=4291/tcp --permanent
$ firewall-cmd --reload
```
| Port     | Explain                                                     |
| :------- | :---------------------------------------------------------- |
| 179/tcp  | BGP 프로토콜을 사용하는 Calico 노드 간의 통신에 사용됩니다. |
| 4789/udp | Calico 노드 간의 VXLAN 터널링에 사용됩니다.                 |
| 5473/tcp | Calico 노드 간의 통신에 사용됩니다.                         |
| 443/tcp  | Kubernetes API 서버와 통신                                  |
| 4291/tcp | Calico 노드 간의 통신에 사용됩니다.                         |

2. 클러스터에 연산자 설치
```shell
$ kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
```
3. Calico를 구성하는데 필요한 리소스 다운
```shell
$ curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml -O
```
4. Calico를 설치하기 위해 매니페스트를 생성합니다.
```shell
kubectl create -f custom-resources.yaml
```
5. custom-resources.yaml IP 수정
```yaml
# This section includes base Calico installation configuration.
# For more information, see: https://projectcalico.docs.tigera.io/master/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    # Note: The ipPools section cannot be modified post-install.
    ipPools:
    - blockSize: 26
     #cidr: 192.168.0.0/16    
      cidr: 10.244.0.0/16    #수정
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()

---

# This section configures the Calico API server.
# For more information, see: https://projectcalico.docs.tigera.io/master/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}
```

## 2-11. Kubernetes Control Plane 초기화

- 컨트롤 플레인 노드를 처음으로 초기화하여 Kubernetes 클러스터를 시작
- Kubernetes 컨트롤 플레인으로 정한 서버의 IP 주소에 설치

### <"br_netfilter" 커널 모듈이 활성화되었는지 확인>

```shell
$ lsmod | grep br_netfilter
```

### <Kubernetes 클러스터에 필요한 이미지를 다운로드>

- Kubernetes 클러스터 생성에 필요한 모든 컨테이너 이미지를 다운
  - coredns, kube-api 서버, etcd, kube-controller, kube-proxy ,pause 컨테이너 이미지

```shell
$ kubeadm config images pull
```

### <Kubernetes 클러스터를 초기화 Dev6-4:IP:192.168.0.104>

```shell
$ kubeadm init \
--pod-network-cidr=10.244.0.0/16 \
--apiserver-advertise-address=192.168.0.104 \
--cri-socket=unix:///run/containerd/containerd.sock
```

- 클러스터를 처음 초기화하기 때문에 자동으로 Kubernetes 컨트롤 플레인으로 선택
- pod의 네트워크를 어떤 플러그인을 사용하는 지에 따라 기본 네트워크 범위 설정
  - Flannel CNI 플러그인의 기본 네트워크 범위 : 10.244.0.0/16 (변경 가능)
  - Calico CNI 플러그인의 기본 네트워크 범위 : 192.168.0.0/16 (변경 가능)

```shell
$ --apiserver-advertise-address <_Control Plane Sever IP Adress_>
```

- Kubernetes API 서버가 실행될 IP 주소를 결정
- 내부 IP 주소 사용

```shell
$ --cri-socket
```

- "/run/containerd/containerd.sock"에서 사용할 수 있는 컨테이너 런타임 소켓에 CRI 소켓을 지정
- 다른 Container Runtime을 사용하는 경우
  - 사용하는 컨테이너 런타임의 소켓 파일의 경로를 변경
  - kubeadm이 Container Runtime 소켓을 자동으로 감지 -> --cri-socket" 옵션을 제거

### <Kubernetes 클러스터 사용을 시작하기 전에 Kubernetes 자격 증명을 설정>

```shell
$ mkdir -p $HOME/.kube
$ cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ chown $(id -u):$(id -g) $HOME/.kube/config
```

### Flannel Pod 네트워크 플러그인을 설치

```shell
$ kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

### coredns가 Pending 상태 & 각 node가 Notready인 상태인 경우

1. Control Plane 초기화 후 `kubectl get pods --all-namespace`로 확인
2. coredns가 pending 상태
3. Network Addon Plugin CNI : Plannal이 설치되어 있는지 먼저 확인해야 한다.
4. resolv.conf에 nameserver추가
5. 각 노드들 메모리 스왑, 방화벽 확인

```shell
# /etc/resolv.conf에 추가 후 재부팅
namesever 10.96.0.10
```

### 재부팅시 resolv.conf가 초기화 되는 경우

1. root사용자로 텍스트 편집기로 /etc/NetworkManager/conf.d/90-dns-none.conf 파일 생성

```shell
[main]
dns=none
```

2. NetworkManager 서비스 다시 로드

```shell
$ systemctl reload NetworkManager
```

3. 서비스를 다시 로드한 후 NetworkManager는 더 이상 /etc/resolv.conf 파일을 업데이트하지 않는다.
4. 파일의 마지막 내용은 보존
5. 선택적으로 /etc/resolv.conf 에서 Generated by NetworkManager 주석을 제거하여 혼동을 방지

## 2-12. Kubernetes에 작업자 노드 추가

- "master" 서버에서 Kubernetes 컨트롤 플레인을 초기화
- 작업자 노드 "worker"를 Kubernetes 클러스터에 추가
- "worker"서버에서 "master"에서 초기화 시 생성된 "kubeadm join" 명령을 실행

  - " worker"을 Kubernetes 클러스터에 추가
  - 다른 토큰과 ca-cert-hash가 있을 수 있다.
- 컨트롤 플레인 노드를 초기화할 때 출력 메시지에서 이 정보의 세부 정보 확인 가능

```shell
# 예시
$ kubeadm join 192.168.5.10:6443 --token wlg23u.r5x2nxw2vdu95dvp \
        --discovery-token-ca-cert-hash sha256:71fd28ac2b8108a3d493648a9c702acd2e39a8a0e7efc07326d7b0384c929066
```

### <모든 네임스페이스에 대해 실행 중인 모든 포드를 확인>

- 모든 Kubernetes 구성 요소에 추가 포드가 있는지 확인해야 합니다.

```shell
kubectl get pods --all-namespaces
```

### <Kubernetes 클러스터에서 사용 가능한 모든 노드를 확인>

- kube-master" 서버가 Kubernetes 컨트롤 플레인으로 실행
- worker서버가 작업자 노드로 실행 중

```shell
kubectl get nodes -o wide
```

## 쿠버네티스 명령어 모음

```shell
# 각 노드들 로그 보기
$ kubectl describe node <Node Name>

# Cnotrol Plane에서 노드 List
$ kubectl get nodes
# 노드 list 자세히 보기
$ kubectl get nodes -o wide

# Control Plane에서 Pod list
$ kubectl get pods
# 모든 네임스페이스(namespace)에 있는 파드(pod) list
$ kubectl get pods --all-namespaces
# 실시간으로 pod 감시
$ watch kubectl get pods --all-namespaces

# Control Plane에서 Pod delete
$ kubectl delete pod <pod name>
```
