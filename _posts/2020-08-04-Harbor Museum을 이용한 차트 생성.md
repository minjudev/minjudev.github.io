## 프로젝트의 목적
본 프로젝트는 Harbor Museum을 이용해서 만든 차트를 Harbor에 올려보는 것을 목적으로 진행되었다.

## 프로젝트 진행 과정
### 1. Harbor Chart Repository를 추가한다.
`helm repo add` 명령을 사용해 harbor 저장소를 추가한다.  
```bash
helm repo add harbor https://helm.goharbor.io
```

harbor 저장소를 추가한 뒤 `helm repo list`를 통해 확인한다.  
![Screenshot_from_2020-08-04_17-53-07](https://user-images.githubusercontent.com/53208493/89299683-fb27f500-d6a1-11ea-802b-046d16a65943.png)

### 2. values.yaml 파일을 수정한다.
```yaml
replicaCount: 1

image:
  repository: minjulee226/myweb
  pullPolicy: Always
  tag: "customport"

service:
  type: NodePort
  port: 80
```

1) image   
image 필드를 docker hub에 올려둔 minjulee226/myweb:customport로 수정해주었다.

2) service  
type을 NodePort로 수정해주었다.

### 3. Namespace를 생성한다.
실습 진행 과정 중 Namespace를 생성하는 과정이 필요하지만 이 부분은 미처 진행하지 못하고 넘어간 부분이다.  
Namespace는 `kubectl create nampespace <namespace 이름>` 명령을 통해 생성 가능하다.  

### 4. Harbor Chart를 설치한다.
```bash
helm install myweb harbor/harbor
```

helm install 명령으로 helm 차트를 설치한다.  
이 커맨드는 myweb이라는 릴리즈 이름으로 harbor 저장소의 harbor 차트를 설치하는 명령어를 나타낸다.  
harbor 차트를 설치한 후 생성되는 리소스들은 다음과 같다.  
![Screenshot_from_2020-08-04_17-54-44](https://user-images.githubusercontent.com/53208493/89300677-8786e780-d6a3-11ea-9fea-0b519bb8be80.png)

### 5. /etc/hosts를 수정한다.
웹 브라우저에서 도메인 주소로 harbor에 접속하기 위해서는 다음과 같이 /etc/hosts 파일을 수정하는 작업이 필요하다.  

1) Master Node  
![Screenshot_from_2020-08-04_17-42-47](https://user-images.githubusercontent.com/53208493/89300810-bd2bd080-d6a3-11ea-8246-89c19bdc2bbd.png)  
Master Node는 위와 같이 수정해주었다.   

2) Local  
![Screenshot_from_2020-08-04_17-43-11](https://user-images.githubusercontent.com/53208493/89300837-c61ca200-d6a3-11ea-96b5-839bebf4f9c6.png)  
Local에서도 마찬가지로 위와 같이 수정해주었다.   

### 6. 웹 브라우저에서 Harbor가 잘 올라오는지 확인한다.  
https://core.harbor.domain 으로 접속해 Harbor 화면이 잘 출력되는지 확인한다.   
![Screenshot_from_2020-08-04_17-58-40](https://user-images.githubusercontent.com/53208493/89301074-227fc180-d6a4-11ea-9711-778320e0a2f6.png)  


### 7. 인증서와 함께 library chart 저장소를 추가한다.
1) harbor portal에서 인증서를 다운받는다.  
![Screenshot_from_2020-08-04_18-15-55](https://user-images.githubusercontent.com/53208493/89301083-24498500-d6a4-11ea-8166-9dd8bb4a5b3e.png)  
위 그림에서 보이는 REGISTRY CERTIFICATE 메뉴를 누르면 인증서를 다운받을 수 있다.  

2) 로컬에 다운받은 인증서를 master node로 옮긴다.  
```bash
scp /home/student/Downloads/ca.crt student@192.168.122.11:/home/student
```

3) helm repo add 명령을 사용해 library chart 저장소를 추가한다.  
```bash
helm repo add library https://core.harbor.domain/chartrepo/library --ca-file ~/Downloads/ca.crt --username admin --password Harbor12345
```

### 8. 모든 노드에서 Harbor Registry를 허용해준다.
1) 먼저 master node에 있는 ca.crt를 모든 워커 노드에 복사한다.  
```bash
scp student@192.168.122.11:/home/student/ca.crt student@192.168.122.21:/home/student/ca.crt
scp student@192.168.122.11:/home/student/ca.crt student@192.168.122.22:/home/student/ca.crt
scp student@192.168.122.11:/home/student/ca.crt student@192.168.122.23:/home/student/ca.crt
```

2) 모든 워커 노드에 복사해준 ca.crt 파일을 모든 Docker host의 /etc/docker/certs.d/<domian>/ca.crt 으로 복사한다.  
```bash
cp /home/student/ca.crt /etc/docker/certs.d/core.harbor.domain/ca.crt
```

3) docker에 harbor 권한으로 로그인 한다.  
```bash
docker login core.harbor.domain
```

4) harbor에 docker image를 올려본다.  
```bash
docker tag minjulee226/myweb:customport core.harbor.domain/library/minjulee226/myweb:new
docker push core.harbor.domain/library/minjulee226/myweb:new
```

### 9. Harbor Registry의 인증을 위해 시크릿을 생성한다.
직접 harbor portal에 들어가 로그인을 하지 않고도 자동으로 kubernetes에서 harbor로 이미지를 올리기 위해 이러한 과정이 필요하다.
```bash
kubectl create secret docker-registry harbor --docker-username="admin" --docker-password="Harbor12345" --docker-server="https://core.harbor.domain/library"
```

## 보완할 점
실습 과정 중 harbor registry의 인증을 위한 시크릿 생성 단계까지는 마쳤으나 harbor에 image를 올리려고 시도했을 때 계속 에러가 발생해 image를 직접 올려보지는 못했다.    
따라서 이러한 점을 보완하여 인터넷이 되지 않는 환경에서도 harbor에 image를 올릴 수 있도록 다시 구성해보아야겠다.
