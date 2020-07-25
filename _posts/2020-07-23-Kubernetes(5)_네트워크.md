## 1. 클러스터 내부 서비스
파드는 클러스터 외부의 요청이나 클러스터 내부의 다른 파드의 요청에 응답해야 한다. 또, 파드가 다른 파드에서 제공하는 애플리케이션에 접근하기 위해서는 다른 파드를 찾을 수 있어야 한다.
이를 가능하게 하려면 파드는 단일 고정 IP를 통해서 서비스를 제공해야 한다.

### 1) 서비스 소개
서비스(Service)는 쿠버네티스 시스템에서 같은 애플리케이션을 실행하는 컨트롤러의 파드 그룹에 단일 네트워크 진입점을 제공하는 리소스이다.
서비스가 종료될 때까지 서비스에 부여된 IP는 변경되지 않는다.
이렇게 서비스에 부여된 IP를 통해 클라이언트는 파드에 접근하게 된다. 

**kubernetes 서비스**
default 네임스페이스에 처음부터 존재하던 서비스로 마스터의 API 서버(kube-apiserver)로 접근할 수 있는 서비스이다.
![Screenshot from 2020-07-23 14-44-16](https://user-images.githubusercontent.com/53208493/88255525-8a70f800-ccf3-11ea-9186-7ebe7207d0c0.png)

### 2) 서비스 생성
#### (1) 명령을 이용하는 방법
`kubectl expose <CONTROLLER_TYPE> <CONTROLLER_NAME> [--type=<SVC_TYPE>] --name <SVC_NAME>`

#### (2) YAML 파일을 이용하는 방법
- mynapp-svc.yml
```yaml
apiVersion: v1
kind: Service
metadata: 
  name: mynapp-svc
spec:
   ports:
   - port: 80 # 이 서비스의 포트
     targetPort: 8080 # 파드의 포트
   selector:
     app: mynapp-rs # 파드 레이블 셀렉터
```

yaml 파일을 이용해 서비스를 생성해보자.
`kubectl create -f mynapp-svc.yml`

### 3) 서비스 및 엔드포인트 확인
#### (1) 서비스 목록 확인
![Screenshot from 2020-07-23 14-45-13](https://user-images.githubusercontent.com/53208493/88255529-8ba22500-ccf3-11ea-84a5-6969f8dc1101.png)
서비스의 타입은 ClusterIP 이며, 이는 쿠버네티스 클러스터 내부에서만 접근 가능한 타입을 말한다. 

#### (2) 엔드포인트 확인
엔드포인트 목록은 서비스의 레이블 셀렉터에 의해 연결된 파드의 IP 목록을 의미한다. 
![Screenshot from 2020-07-23 14-45-27](https://user-images.githubusercontent.com/53208493/88255530-8c3abb80-ccf3-11ea-98e0-eeb05db6f8e2.png)
현재는 위의 yml 파일에서 정의한 레이블 셀렉터로 연결된 파드가 없어 엔드포인트가 나타나지 않는 것을 볼 수 있다.
즉, 서비스의 클러스터 IP로 접속하더라도 포워딩할 파드가 없는 것이다.

### 4) 파드 생성 및 엔드포인트 연결
서비스 생성을 위해 작성한 yaml에 제시되어 있는 레이블을 가진 파드를 생성하면 서비스는 자동으로 엔드포인트를 추가하게 된다. 

전에 만든 mynapp-rs.yml 파일을 이용해 파드를 생성해보겠다.
해당 파일은 app=mynapp-rs 레이블을 사용하는 컨트롤러와 파드를 생성하는 파일이다.
![Screenshot from 2020-07-23 14-45-53](https://user-images.githubusercontent.com/53208493/88255531-8c3abb80-ccf3-11ea-9aa7-41485a1a31bc.png)

![Screenshot from 2020-07-24 09-38-16](https://user-images.githubusercontent.com/53208493/88352306-497cf000-cd94-11ea-8f18-ad54af9d2631.png)
![Screenshot from 2020-07-24 09-57-04](https://user-images.githubusercontent.com/53208493/88352310-4a158680-cd94-11ea-9a65-7a057b9927de.png)

파드의 개수를 늘린 후 엔드포인트의 목록을 확인해도 엔드포인트 목록에 보이는 파드의 IP의 주소의 개수가 늘어난 것을 볼 수 있다.
![Screenshot from 2020-07-23 14-53-27](https://user-images.githubusercontent.com/53208493/88255812-67931380-ccf4-11ea-8b00-03ca7d54775b.png)
![Screenshot from 2020-07-23 14-53-40](https://user-images.githubusercontent.com/53208493/88255815-682baa00-ccf4-11ea-9e59-78ae453c7152.png)

### 5) 서비스 접근 테스트
![Screenshot from 2020-07-23 14-47-37](https://user-images.githubusercontent.com/53208493/88255533-8cd35200-ccf3-11ea-8349-f29f7aaf587b.png)
curl 명령을 통해 서비스에 할당된 IP로 접근하면, 기본적으로 부하 분산이 이루어지면서 각 파드에 접근하는 것을 볼 수 있다.

### 6) 서비스의 세션 어피니티 구성
클라이언트의 요청을 매번 똑같은 파드로 연결하고 싶은 경우에는 세션 어피니트(Session Affinity)로 서비스를 구성할 수 있다.
- mynapp-svc-session-affinity.yml
```yaml
apiVersion: v1
kind: Service
metadata: 
  name: mynapp-svc-ses-aff
spec:
   sessionAffinity: ClientIP
   ports:
   - port: 80
     targetPort: 8080
   selector:
     app: mynapp-rs
```
세션 어피니티의 설정을 ClientIP로 설정하면, 쿠버네티스 클러스터의 프록시(kube-proxy)는 클라이언트의 IP를 확인해 매번 같은 파드로 연결해준다.

- 서비스 접근 테스트
![Screenshot from 2020-07-23 15-15-44](https://user-images.githubusercontent.com/53208493/88256901-6ca59200-ccf7-11ea-9542-5140302f83b1.png)

### 7) 서비스 다중 포트 구성
파드의 애플리케이션은 항상 하나의 포트로만 서비스 하지 않고, 여러 포트를 사용해 서비스 할 수 있다.
만약 파드의 서비스 포트가 다중 포트인 경우, 클라이언트에게 노출할 서비스 포트 역시 다중 포트로 구성할 수 있다.
- mynapp-svc-multiport.yml
```
apiVersion: v1
kind: Service
metadata: 
  name: mynapp-svc-mp
spec:
   ports:
   - name: mynapp-http
     port: 80
     targetPort: 8080
   - name: mynapp-https
     port: 443
     targetPort: 8443
   selector:
     app: mynapp-rs
```
이처럼 다중 포트를 설정할 때는 반드시 포트의 이름을 부여해야 한다.

## 2. 서비스 탐색
### 1) 환경변수를 이용한 서비스 탐색
쿠버네티스 클러스터는 파드가 시작될 때 존재하는 서비스를 파드 내의 쉘 환경 변수로 설정한다.
현재 실행 중인 파드에 `kubectl exec` 명령을 이용해 환경변수를 확인해보도록 하겠다.
![Screenshot from 2020-07-23 18-12-38](https://user-images.githubusercontent.com/53208493/88270312-68d23980-cd10-11ea-9839-7cbd8d0054d9.png)

여러 개의 환경 변수 중 대표적으로 다음과 같은 환경 변수를 참조할 수 있다.
- MYNAPP_SVC_SERVICE_HOST = 10.233.8.233
- MYNAPP_SVC_SERVICE_PORT = 80
- MYNAPP_SVC. KUBERNETES_SVC_ 이름은 실제 서비스의 이름과 매핑된다

이처럼 애플리케이션은 서비스 이름만 알고 있다면 환경 변수를 참조해서 접근하고자 하는 서비스의 IP와 포트를 확인할 수 있다.

### 2) DNS를 이용한 서비스 탐색
#### (1) NodeLocal DNSCache
NodeLocal DNSCache 기능은 쿠버네티스의 각 노드에 DNS 캐시 기능을 가지고 있는 파드를 데몬셋으로 배치한다.
NodeLocal DNSCache 기능에 대해 간단하게 살펴보자.
kube-system 네임스페이스에 nodelocaldns 데몬셋 컨트롤러가 존재한다.
[image] - 135p

현재 이 시스템의 마스터 1대, 노드 3대의 구성이기 때문에 총 4개의 파드가 배포된다.
nodelocaldns 관련 파드 목록을 확인해보자.
[image] - 136p

nodelocaldns 파드의 아규먼트 설정이다.
이는 각 노드에서 169.254.255.10 IP인 링크로컬 주소 대역을 사용해 DNS 캐시 서비스를 제공한다는 것을 나타낸다.
[image] - 136p

#### (2) FQDN을 이용한 서비스 탐색
그럼 서비스의 FQDN을 이용해 서비스와 연결된 파드에 접근해보자.

[imgae] - 137p

이처럼 서비스의 IP 주소를 직접적으로 알지 못하더라도, NodeLocal DNSCache 기능을 이용해 서비스의 이름으로 각 파드들에 접근할 수 있다. 

## 3. 클러스터 외부 서비스
쿠버네티스 클러스터에서 웹의 프론트엔드 서비스를 실행하는 파드의 경우 쿠버네티스 클러스터의 외부로 노출시켜 접근 가능하도록 구성해야 한다.
클러스터 외부 서비스는 다음과 같다.
- NodePort
  - 쿠버네티스 모든 노드(호스트)에 외부 접근용 포트를 할당함
  - 노드의 포트로 접근하면 서비스에 의해 파드로 리다이렉션함
- LoadBalancer
  - 클러스터 외부의 로드 밸런서를 사용해 외부에서 접근 가능
  - 외부 로드 밸런서로 접근하면 서비스를 통해 파드로 리다이렉션함

### 1) 외부 서비스용 레플리카셋 생성 및 확인
- mynapp-rs.yml
```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: mynapp-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mynapp-rs
  template:
    metadata:
      labels:
        app: mynapp-rs
    spec:
      containers:
      - name: mynapp
        image: minjulee226/myweb
        ports:
        - containerPort: 8080
```

명령어를 이용해 컨트롤러와 파드를 생성한다.
`kubectl create -f mynapp-rs.yml`

컨트롤러 및 파드가 생성되었는지 확인한다.
`kubectl get replicasets`
![Screenshot from 2020-07-24 10-30-40](https://user-images.githubusercontent.com/53208493/88353555-eb064080-cd98-11ea-8816-e71031f4c0e0.png)
![Screenshot from 2020-07-24 10-31-44](https://user-images.githubusercontent.com/53208493/88353557-eb9ed700-cd98-11ea-8baf-405a5dd324f5.png)

### 2) NodePort 서비스 생성
- mynapp-svc-ext-nodeport.yml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mynapp-svc-ext-np
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 31111
  selector:
    app: mynapp-rs
```

- 서비스 타입: NodePort
- 노드에 노출할 포트: 31111(30000-32767)

노드 포트 서비스를 생성하자.
`kubectl create -f mynapp-svc-ext-nodeport.yml`

### 3) NodePort 서비스 확인
서비스 목록 및 상태를 확인하자.
![Screenshot from 2020-07-24 10-36-32](https://user-images.githubusercontent.com/53208493/88353735-98795400-cd99-11ea-89f3-d6fbf4c6edc3.png)

노드의 31111 포트로 접근하면 서비스의 80 포트로 리다이렉션된다.
서비스의 엔드포인트를 확인해보면 다음과 같다.
![Screenshot from 2020-07-24 10-37-42](https://user-images.githubusercontent.com/53208493/88353814-e4c49400-cd99-11ea-9f3e-b5fa6113a7a7.png)
이처럼 서비스의 80 포트는 파드의 8080포트로 리다이렉션된다.

노드의 IP는 다음과 같다.
![Screenshot from 2020-07-24 10-38-37](https://user-images.githubusercontent.com/53208493/88353817-e55d2a80-cd99-11ea-9912-5189b0c5fba5.png)

curl 명령어로 노드 IP 주소:노드 포트번호를 입력하면 최종적으로 파드에 접근할 수 있다.
![Screenshot from 2020-07-24 10-39-49](https://user-images.githubusercontent.com/53208493/88353862-06be1680-cd9a-11ea-9926-5a019bcc5ac2.png)

### 4) LoadBalancer 서비스 생성
- mynapp-svc-ext-loadbalancer.yml
```yml
apiVersion: v1
kind: Service
metadata: 
  name: mynapp-svc-ext-lb
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: mynapp-rs
```

로드 밸런서 서비스를 생성해보자.
`kubectl create -f mynapp-svc-ext-loadbalancer.yml`

### 5) LoadBalancer 서비스 확인
![Screenshot from 2020-07-24 11-08-58](https://user-images.githubusercontent.com/53208493/88355083-28b99800-cd9e-11ea-8932-622509f22586.png)
이처럼 로드 밸런서의 외부 IP(192.168.122.201)가 정상적으로 할당된 것을 볼 수 있다.

curl 명령어로 로드 밸런서의 IP 주소를 입력하면 각 파드에 부하 분산되어 접근하는 것을 확인할 수 있다.
![Screenshot from 2020-07-24 11-09-19](https://user-images.githubusercontent.com/53208493/88355084-29522e80-cd9e-11ea-843b-4f1cf6b7cf46.png)

### 6) ExternalName 서비스 생성
ExternalName 서비스는 외부에서 접근하기 위한 서비스 종류가 아닌 내부 파드가 외부의 특정 FQDN에 접근하기 위한 서비스이다.
쿠버네티스 클러스터의 coredns 서비스가 특정 FQDN에 대한 CNAME을 제공함에 따라 해당 CNAME을 이용해 쉽게 통신할 수 있다.

- mynapp-svc-ext-extname.yml
```yaml
apiVersion: v1
kind: Service
metadata: 
  name: mynapp-svc-extname-gl
spec:
  type: ExternalName
  externalName: www.google.com
```
외부 FQDN은 www.google.com 이며 이에 대한 CNAME은 mynapp-svc-extname-gl 이름을 제공하는 서비스이다.

외부 이름 서비스를 생성해보자.
`kubectl create -f mynapp-svc-ext-extname.yml`

### 7) ExternalName 서비스 확인
생성된 서비스를 확인해보자.
![Screenshot from 2020-07-24 11-46-12](https://user-images.githubusercontent.com/53208493/88356609-6836b300-cda3-11ea-9bd0-ba73d81618d4.png)
서비스 타입은 ExternalName이며, 외부 IP에는 www.google.com 의 FQDN이 할당되어 있다.
즉, 파드는 mynapp-svc-extname-gl CNAME을 통해 www.google.com의 주소를 알 수 있다.

네트워크 도구를 가지고 있는 파드를 실행해서 서비스 이름 입력시 외부의 FQDN의 주소를 출력하는지 확인해보자.
![Screenshot from 2020-07-24 11-46-48](https://user-images.githubusercontent.com/53208493/88356611-6836b300-cda3-11ea-92e0-105aed34e60c.png)
