## 4. 인그레스
### 실습
- 다중 경로, 다중 호스트를 가지는 인그레스 컨트롤러 구현하기
  1. 다중 경로
    - path: mynapp.example.com/web1
        - 백엔드 서비스: mynapp-svc-myweb1
        - 레플리카셋: mynapp-rs-myweb1
    - path: mynapp.example.com/web2
      - 백엔드 서비스: mynapp-svc-myweb2
      - 레플리카셋: mynapp-rs-myweb2
      
  2. 다중 호스트
    - host: web1.example.com
      - 백엔드 서비스: mynapp-svc-web1
      - 레플리카셋: mynapp-rs-web1
    - host: web2.example.com
      - 백엔드 서비스: mynapp-svc-web2
      - 레플리카셋: mynapp-rs-web2

### 내용
1. 인그레스 파일 작성
- mynapp-ing-multi-paths-hosts.yml   
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
        - path: /web1  # 다중 경로
          backend:
            serviceName: mynapp-svc-myweb1  # 백엔드 서비스 지정
            servicePort: 80  # 백엔드 서비스 포트 지정
        - path: /web2  # 다중 경로
          backend:
            serviceName: mynapp-svc-myweb2
            servicePort: 80
  - host: web1.example.com  # 다중 호스트
    http:
      paths:
      - path: /
        backend:
          serviceName: mynapp-svc-web1
          servicePort: 80
  - host: web2.example.com  # 다중 호스트
    http:
      paths:
      - path: /
        backend:
          serviceName: mynapp-svc-web2
          servicePort: 80
```

2. 백엔드 서비스 파일 작성
- 4개의 백엔드 서비스를 하나의 파일에 지정
- mynapp-svc-multi.yml   
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
  selector:
    app: mynapp-rs-myweb1
    
---

apiVersion: v1
kind: Service
metadata:
  name: mynapp-svc-ext-np
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: mynapp-rs-myweb2
    
---

apiVersion: v1
kind: Service
metadata:
  name: mynapp-svc-ext-np
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: mynapp-rs-web1
    
---

apiVersion: v1
kind: Service
metadata:
  name: mynapp-svc-ext-np
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: mynapp-rs-web2
```

3. 레플리카셋 파일 작성
- 4개의 레플리카셋을 하나의 파일에 지정
- mynapp-rs-multi.yml   
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mynapp-rs-myweb1
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: mynapp-rs-myweb1
  template:
    metadata:
      labels:
        app: mynapp-rs-myweb1
    spec:
      containers:
      - name: mynapp
        image: minjulee226/myweb
        ports:
        - containerPort: 8080
        
---

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mynapp-rs-myweb2
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: mynapp-rs-myweb2
  template:
    metadata:
      labels:
        app: mynapp-rs-myweb2
    spec:
      containers:
      - name: mynapp
        image: minjulee226/myweb
        ports:
        - containerPort: 8080
        
---

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mynapp-rs-web1
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: mynapp-rs-web1
  template:
    metadata:
      labels:
        app: mynapp-rs-we1
    spec:
      containers:
      - name: mynapp
        image: minjulee226/myweb
        ports:
        - containerPort: 8080
        
---

apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mynapp-rs-web2
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: mynapp-rs-web2
  template:
    metadata:
      labels:
        app: mynapp-rs-web2
    spec:
      containers:
      - name: mynapp
        image: minjulee226/myweb
        ports:
        - containerPort: 8080
```

4. 레플리카셋 컨트롤러, 백엔드 서비스, 인그레스 컨트롤러 생성  
`kubectl create -f mynapp-rs-multi.yml -f mynapp-svc-multi.yml -f mynapp-ing-multi-paths-hosts.yml`  

현재는 외부 DNS가 없기 때문에 mynapp.example.com FQDN의 A 레코드를 반환하지 못하게 된다.  
그러므로 curl --resolve 옵션을 사용하여 FQDN의 주소를 IP로 변환하는 옵션을 사용해야 한다.   
`curl --resolve mynapp.example.com:80:192.168.122.21 http://mynapp.example.com/myweb1`  
[image]  

`curl --resolve mynapp.example.com:80:192.168.122.21 http://mynapp.example.com/myweb2`   
[image]  

`curl --resolve mynapp.example.com:80:192.168.122.21 http://web1.example.com/`  
[image]  

`curl --resolve mynapp.example.com:80:192.168.122.21 http://web2.example.com/`  
[image]  


이처럼 지정해준 다중 경로 혹은 다중 호스트를 입력하면, 백엔드 서비스에 연결된 파드들에 접근 가능한 것을 확인할 수 있다!  
