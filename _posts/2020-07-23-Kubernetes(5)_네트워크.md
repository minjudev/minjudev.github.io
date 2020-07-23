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
```yaml

```

### 3) 서비스 및 엔드포인트 확인


![Screenshot from 2020-07-23 14-45-13](https://user-images.githubusercontent.com/53208493/88255529-8ba22500-ccf3-11ea-84a5-6969f8dc1101.png)

![Screenshot from 2020-07-23 14-45-27](https://user-images.githubusercontent.com/53208493/88255530-8c3abb80-ccf3-11ea-98e0-eeb05db6f8e2.png)

### 4) 파드 생성 및 엔드포인트 연결
![Screenshot from 2020-07-23 14-45-53](https://user-images.githubusercontent.com/53208493/88255531-8c3abb80-ccf3-11ea-9aa7-41485a1a31bc.png)

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









