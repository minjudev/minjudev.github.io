## 1. 클러스터 내부 서비스
### 1) 서비스 소개
### 2) 서비스 생성

### 3) 서비스 및 엔드포인트 확인
![Screenshot from 2020-07-23 14-44-16](https://user-images.githubusercontent.com/53208493/88255525-8a70f800-ccf3-11ea-9186-7ebe7207d0c0.png)

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









