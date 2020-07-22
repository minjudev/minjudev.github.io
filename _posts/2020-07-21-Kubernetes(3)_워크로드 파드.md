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

### 4) Pod 목록 확인
`kubectl get pods`

### 5) 실행 중인 Pod 정의 확인
`kubectl get pods mynapp-p`

### 6) Pod의 자세한 정보 확인
### 7) Pod 로그 확인
### 8) Pod 포트 포워딩









## Lable / Selector
## Annotation
## Namespace
