## 1. 클러스터 내부 서비스
파드는 클러스터 외부의 요청이나 클러스터 내부의 다른 파드의 요청에 응답해야 한다. 또, 파드가 다른 파드에서 제공하는 애플리케이션에 접근하기 위해서는 다른 파드를 찾을 수 있어야 한다.
이를 가능하게 하려면 파드는 단일 고정 IP를 통해서 서비스를 제공해야 한다.

### 1) 서비스 소개
서비스(Service)는 쿠버네티스 시스템에서 같은 애플리케이션을 실행하는 컨트롤러의 파드 그룹에 단일 네트워크 진입점을 제공하는 리소스이다.
서비스가 종료될 때까지 서비스에 부여된 IP는 변경되지 않는다.
이렇게 서비스에 부여된 IP를 통해 클라이언트는 파드에 접근하게 된다. 

**kubernetes 서비스**
이 default 네임스페이스에 처음부터 존재하던 서비스로 마스터의 API 서버로 접근할 수 있는 서비스이다.
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
   sessionAffinity: ClientIP
   ports:
   - port: 80
     targetPort: 8080
   selector:
     app: mynapp-rs
```

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

### 5) 서비스 접근 테스트
![Screenshot from 2020-07-23 14-47-37](https://user-images.githubusercontent.com/53208493/88255533-8cd35200-ccf3-11ea-8349-f29f7aaf587b.png)

스케일링(scale 명령으로도 가능) 후 엔드포인트 확인
![Screenshot from 2020-07-23 14-53-27](https://user-images.githubusercontent.com/53208493/88255812-67931380-ccf4-11ea-8b00-03ca7d54775b.png)
![Screenshot from 2020-07-23 14-53-40](https://user-images.githubusercontent.com/53208493/88255815-682baa00-ccf4-11ea-9e59-78ae453c7152.png)

### 6) 서비스의 세션 어피니티 구성
앞에서 살펴본 서비스 접근 테스트에서 

- mynapp-svc.yml
```yaml
apiVersion: v1
kind: Service
metadata: 
  name: mynapp-svc
spec:
   sessionAffinity: ClientIP
   ports:
   - port: 80
     targetPort: 8080
   selector:
     app: mynapp-rs
```

- 서비스 접근 테스트
![Screenshot from 2020-07-23 15-15-44](https://user-images.githubusercontent.com/53208493/88256901-6ca59200-ccf7-11ea-9542-5140302f83b1.png)

## 2. 서비스 탐색
### 1) 환경변수를 이용한 서비스 탐색
쿠버네티스 클러스터는 파드가 시작될 때 존재하는 서비스를 파드 내의 쉘 환경 변수로 설정한다.
현재 실행 중인 파드에 `kubectl exec` 명령을 이용해 환경변수를 확인해보도록 하겠다.
![Screenshot from 2020-07-23 18-12-38](https://user-images.githubusercontent.com/53208493/88270312-68d23980-cd10-11ea-9839-7cbd8d0054d9.png)

여러 개의 환경 변수 중 대표적으로 다음과 같은 환경 변수를 참조할 수 있다.
- MYNAPP_SVC_SERVICE_HOST = 10.233.8.233
- MYNAPP_SVC_SERVICE_PORT = 80
- MYNAPP_SVC. KUBERNETES_SVC_ 이름은 실제 서비스의 이름과 매핑된다

애플리케이션은 서비스 이름만 알고 있다면 환경 변수를 참조해서 접근하고자 하는 서비스의 IP와 포트를 확인할 수 있다.

## 3. 클러스터 외부 서비스
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

![Screenshot from 2020-07-24 10-37-42](https://user-images.githubusercontent.com/53208493/88353814-e4c49400-cd99-11ea-9f3e-b5fa6113a7a7.png)
![Screenshot from 2020-07-24 10-38-37](https://user-images.githubusercontent.com/53208493/88353817-e55d2a80-cd99-11ea-9912-5189b0c5fba5.png)

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

로드밸런서 생성
`kubectl create -f mynapp-svc-ext-loadbalancer.yml`

### 5) LoadBalancer 서비스 확인
![Screenshot from 2020-07-24 11-08-58](https://user-images.githubusercontent.com/53208493/88355083-28b99800-cd9e-11ea-8932-622509f22586.png)
![Screenshot from 2020-07-24 11-09-19](https://user-images.githubusercontent.com/53208493/88355084-29522e80-cd9e-11ea-843b-4f1cf6b7cf46.png)

### 6) ExternalName 서비스 생성
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

외부 이름 서비스를 생성해보자.
`kubectl create -f mynapp-svc-ext-extname.yml`

### 7) ExternalName 서비스 확인
생성된 서비스를 확인해보자.
`kubectl get svc`
![Screenshot from 2020-07-24 11-46-12](https://user-images.githubusercontent.com/53208493/88356609-6836b300-cda3-11ea-9bd0-ba73d81618d4.png)

네트워크 도구를 가지고 있는 파드를 실행해서 서비스 이름 입력시 외부의 FQDN의 주소를 출력하는지 확인해보자.
![Screenshot from 2020-07-24 11-46-48](https://user-images.githubusercontent.com/53208493/88356611-6836b300-cda3-11ea-92e0-105aed34e60c.png)

### 8) 컨트롤러 및 서비스 삭제

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: mynapp-ing-mpath-mhost
spec:
  rules:
  - host: mynapp.example.com
    http:
      paths:
        - path: /web1
          backend:
            serviceName: mynapp-svc-myweb1
            servicePort: 80
        - path: /web2
          backend:
            serviceName: mynapp-svc-myweb2
            servicePort: 80
  - host: web1.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: mynapp-svc-web1
          servicePort: 80
  - host: web2.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: mynapp-svc-web2
          servicePort: 80
```

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: mynapp-ing-mpath-mhost
spec:
  rules:
  - host: mynapp.example.com
    http:
      paths:
        - path: /web1
          backend:
            serviceName: mynapp-svc-myweb1
            servicePort: 80
        - path: /web2
          backend:
            serviceName: mynapp-svc-myweb2
            servicePort: 80
  - host: web1.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: mynapp-svc-web1
          servicePort: 80
  - host: web2.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: mynapp-svc-web2
          servicePort: 80
```

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: mynapp-ing-mpath-mhost
spec:
  rules:
  - host: mynapp.example.com
    http:
      paths:
        - path: /web1
          backend:
            serviceName: mynapp-svc-myweb1
            servicePort: 80
        - path: /web2
          backend:
            serviceName: mynapp-svc-myweb2
            servicePort: 80
  - host: web1.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: mynapp-svc-web1
          servicePort: 80
  - host: web2.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: mynapp-svc-web2
          servicePort: 80
```











