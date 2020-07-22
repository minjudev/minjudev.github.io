## Pod
### 1) Pod 기본
### 2) Pod 정의
파드를 생성하기 위한 yaml 파일을 작성해보자.
- mynapp-pod.yml
```yml
apiVersion: v1
kind: Pod
metadata:
  name: mynapp-pod
spec:
  containers:
  - image: minjulee226/myweb
    name: mynapp
    ports:
      - containerPort: 8080
        protocol: TCP
```

- 파드 리소스의 주요 필드
  - spec.containers: 컨테이너 정의
  - spec.containers.image: 컨테이너에서 사용할 이미지 
  - spec.containers.name: 컨테이너 이름
  - spec.containers.ports: 노출할 포트 정의
  - spec.containers.ports.containerPort: 노출할 컨테이너 포트번호
  - spec.containers.ports.protocol: 노출할 컨테이너 포트의 프로토콜(기본: TCP)
  
### 3) Pod 생성
`kubectl create -f mynapp-pod.yml`
[사진]

### 4) Pod 목록 확인
`kubectl get pods`
[사진]

### 5) 실행 중인 Pod 정의 확인
`kubectl get pods mynapp-pod -o yaml`
[사진]

-o json 옵션을 사용해 JSON 형식으로도 확인할 수 있다.
`kubectl get pods mynapp-pod -o json`
[사진]

### 6) Pod의 자세한 정보 확인
`kubectl describe pods mynapp-pod`
[image]

### 7) Pod 로그 확인
`kubectl logs mynapp-pod`
[image]

### 8) Pod 포트 포워딩
kubectl port-forward 명령을 이용해 호스트 포트를 컨테이너 포트로 포워딩해보자.
`kubectl port-forward mynapp-pod 8080:8080`
[image]

port-forward 명령은 포그라운드 상태에서 동작하므로, 확인 시에는 별도의 터미널을 열어 확인해야 한다.
이제 curl 명령을 이용해 웹 애플리케이션이 잘 동작하는지 확인해보자.
`curl http://localhost:8080`
[image]

## Label / Selector
### 1) Label 소개
Label은 쿠버네티스 클러스터의 모든 오브젝트에 키/값 쌍으로 리소스를 식별하고 속성을 지정하는 데 사용한다.
키는 "접두사(옵션)/이름" 형식으로 사용 가능하며, 최대 63자를 넘지 못한다.

### 2) Label을 사용한 파드 정의
- mynapp-pod-label.yml
env와 tier 레이블을 추가한 yaml 파일을 작성한다.
```yaml

```

### 3) 파드 생성
`kubectl create -f mynapp-pod-label.yml`
[image]

### 4) 파드 레이블 확인
`kubectl get pods --show-labels`
[image]

-L 옵션을 사용해 특정 레이블을 지정해서 필드에 나타낼 수 있다.
`kubectl get pods -L env,tier

### 5) 파드 레이블 수정
파드의 레이블을 추가하거나 새로운 레이블로 수정할 수도 있다.

- 레이블 추가
`kubectl label pods mynapp-pod env=dev`

- 레이블 수정
`kubectl label pods mynapp-pod env=debug --overwrite`

- 레이블 확인
`kubectl get pods -L env,tier`
[image]

### 6) Label Selector
Label Selector는 오브젝트에 부여되어 있는 레이블을 식별하고 검색할 수 있게 해준다.

- 검색 방법
  - 균등 기반 레이블
    - =
    - ==
    - !=
  - 집합성 기반 레이블
    - in
    - notin키와 값이 있는/없는 레이블 포함
    
### 7) 균등 기반 레이블 셀렉터
- tier 키가 포함되어 있는 레이블
`kubectl get pods --show-labels -l tier`

- tier 키를 제외한 레이블
`kubectl get pods --show-labels -l '!tier'`

- env 키에 debug 값이 포함되어 있는 레이블
`kubectl get pods --show-labels -l env=debug`

- env 키를 가지지만 debug 값이 포함되지 않은 레이블
`kubectl get pods --show-labels -l 'env!=debug'`

### 8) 집합성 기반 레이블 셀렉터
- env 키에 debug 또는 dev 값이 포함되어 있는 레이블
`kubectl get pods --show lables -l 'env in (debug,dev)`

- tier 키를 가지지만 frontend 값이 포함되어 있지 않은 레이블
`kubectl get pods --show-labels -l 'tier notin (frontend)`

## Annotation
### 1) 어노테이션 소개
어노테이션은 주석이라고도 하며 오브젝트에 메타데이터를 할당할 수 있다.
레이블과 마찬가지로 키/값을 가지고 있지민, 어노테이션은 어노테이션 셀렉터를 가지고 있지 않다.
즉, 메타데이터를 추가해 추가적인 정보를 제공할 수 있는 것이다. 

### 2) 어노테이션 추가 및 수정
어노테이션은 레이블과 같게 오브젝트 생성 시에 선언할 수도 있고, 오브젝트 생성 후에도 추가하거나 수정할 수 있다.

- 어노테이션 추가
`kubectl annotate pods mynapp-pod devops-team/developer="minju"`

- 어노테이션 확인
`kubectl describe pods mynapp-pod`
[image]

`kubectl get pods mynapp-pod -o yaml`
[image]

## Namespace
### 1) 네임스페이스 소개
레이블을 부여해 오브젝트를 식별하고 검색할 수 있지만, 이러한 방법은 오브젝트를 논리적으로 분리시켜주진 못한다.
많은 오브젝트를 관리할 때는 논리적으로 오브젝트의 분리가 필요하다.
특히 사용자가 여러 명인 환경의 경우, 오브젝트와 리소스를 완전히 분리시킬 필요가 있다.
네임스페이스는 오브젝트를 논리적으로 분리할 수 있는 논리적 파티션으로, 위의 문제점을 해결해준다.

### 2) 네임스페이스 확인
`kubectl get namespaces`
[image]
오브젝트 생성 시 특정한 네임스페이스를 지정하지 않으면 기본 네임스페이스(default)를 사용한다.

- 기본적인 네임스페이스의 용도
  - default
    - 기본 네임스페이스
  - kube-node-lease
    - 쿠버네티스 노드의 가용성 체크를 위한 네임스페이스
    - 해당 네임스페이스에는 하트비트를 위한 lease 오브젝트가 있음
    - 쿠버네티스 1.14 이상
  - kube-public
    - 모든 사용자가 읽기 권한으로 접근할 수 있음
    - 관례적으로 만들어져 있지만 아무 리소스가 없으며, 반드시 사용해야하는 네임스페이스는 아님
  - kube-system
    - 쿠버네티스 클러스터의 리소스가 배치되는 네임스페이스
  - ingress-nginx
    - kubespray로 Nginx 인그레스 컨트롤러를 배포하는 네임스페이스
    
### 3) 네임스페이스의 오브젝트 확인
kube-systemc 네임스페이스에 존재하는 파드의 목록을 확인해보자.
`kubectl get pods -n kube-system`
[image]
kube-system 네임스페이스는 쿠버네티스 시스템이 동작하기 위한 구성요소들이 파드 형태로 동작하고 있다.

### 4) 네임스페이스 생성
- 명령어를 이용한 네임스페이스 생성
`kubectl create namespace development`

네임스페이스를 확인해보자.
`kubectl get namespaces`

- yaml 파일을 이용한 네임스페이스 생성
  - mynapp-ns.yml
  ```yaml
  
  ```

  `kubectl create -f mynapp-ns.yml`
  
네임스페이스를 확인해보자.
`kubectl get namespaces`
[image]

### 5) 특정 네임스페이스에서 파드(및 오브젝트) 생성
오브젝트 생성 시 특정 네임스페이스를 지정하면 해당 네임스페이스에 오브젝트를 생성할 수 있다.
`kubectl create -f mynapp-pod.yml -n development`

네임스페이스를 확인해보자.
`kubectl get pods -n development`

yaml파일의 pods.metadata.namespaces 필드에 직접 네임스페이스를 선언하는 것도 가능하다.
- mynapp-pod-ns.yml
```yaml

```

### 6) 리소스 삭제
- 오브젝트 이름으로 삭제
`kubectl delete pods mynapp-pod`
네임스페이스를 지정하지 않았으므로 default 네임스페이스의 mynapp-pod가 삭제된다.

- 오브젝트 정의 파일(YAML 및 JSON) 삭제
`kubectl delete -f mynapp-pod-ns.yml`
해당 yaml 파일은 네임스페이스를 development로 지정해주었기 때문에 파일을 이용해 파드 삭제 시 development 네임스페이스의 mynapp-pod가 삭제된다.

- 오브젝트 레이블로 삭제
`kubectl delete pods -l tier=frontend`
default 네임스페이스에서 tier=frontend 레이블을 가진 모든 오브젝트를 삭제한다.

- 네임스페이스 삭제
  - 명령어를 이용한 네임스페이스 삭제
    `kubectl delete namespaces development`

  - yaml 파일을 이용한 네임스페이스(quality-assurance) 삭제
    `kubectl delete -f mynapp-ns.yml`
    
- 삭제된 네임스페이스 확인
  `kubectl get ns`
  [image]
