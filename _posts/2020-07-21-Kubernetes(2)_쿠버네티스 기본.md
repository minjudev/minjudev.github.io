## 1. 구성요소 및 API

## 2. YAML 및 오브젝트 기본

## 3. 오브젝트 관리

## 4. Node.js 애플리케이션 컨테이너 이미지 만들기
### 1) Node.js 애플리케이션
간단한 Node.js 코드로 작성된 웹 애플리케이션 이미지를 만들어보자.
```js
const http = require('http');
const os = require('os');
const ip = require('ip');
const dns = require('dns');
console.log(Date());
console.log("...Start My Node.js Application...");
var handler = function(request, response) {
	console.log(Date());
	console.log("Received Request From " + request.connection.remoteAddress);
	response.writeHead(200);
	response.write("Message: " + process.argv[2] + "\n");
	response.write("Hostname: " + os.hostname() + "\n");
	response.write("Platform: " + os.platform() + "\n");
	response.write("Uptime: " + os.uptime() + "\n");
	response.write("IP: " + ip.address() + "\n");
	response.write("DNS: " + dns.getServers() + "\n");
	response.end();
};
var www =  http.createServer(handler);
www.listen(8080);
```
위 코드는 8080포트로 접속 시 HTTP 바디에 메시지, 호스트 이름, 플랫폼 종류, 업타임, IP, DNS 서버 주소를 보여주는 긴딘힌 애플리케이션이다.

### 2) 사용자 지정 이미지 생성용 Dockerfile
위에서 작성한 index.js 파일을 포함하는 사용자 지정 이미지를 생성하기 위해 Dockerfile을 작성한다.
```
FROM node:slim
WORKDIR /usr/src/app
COPY index.js .
RUN npm install ip
ENTRYPOINT ["node", "index.js"]
CMD ["Hello World!"]
EXPOSE 8080/tcp
```
- FROM
  - 이미지 생성 사용할 베이스 이미지 지정
  - Node.js 기반의 애플리케이션 실행을 위해 Node 런타임이 기본으로 포함되어 있는 베이스 이미지 선택
- WORKDIR
  - 컨테이너 내에서 사용할 기본 작업 디렉토리를 지정
  - Node.js 이미지에서 웹 애플리케이션의 기본 디렉토리는 /usr/src/app
- COPY
  - 이미지 내에 추가로 복사할 파일의 소스와 타겟을 지정하는 부분
- RUN
  - 사용자 지정 이미지 빌드 시에 실행할 명령
  - ip라는 Node.js 패키지 설치
- ENTRYPOINT
  - 이미지 실행 시 실행할 애플리케이션 지정
- CMD
  - ENTRYPOINT의 인수
- EXPOSE
  - 애플리케이션을 노출하기 위한 포트와 프로토콜 지정
  
이러한 이미지를 빌드한 뒤 컨테이너를 생성하면 기본 실행되는 애플리케이션 명령은 `node index.js "Hello World!" 이다.

### 3) 사용자 지정 이미지 빌드
위에서 준비한 두 개의 파일을 이용해 이미지를 빌드해보자.
`docker image build --network host -t myweb .`
![Screenshot from 2020-07-21 17-59-02](https://user-images.githubusercontent.com/53208493/88034750-16f8aa80-cb7c-11ea-9c4c-7162310345bf.png)

이미지가 잘 생성되었는지 확인해보자.
`docker image ls`
![Screenshot from 2020-07-21 18-01-41](https://user-images.githubusercontent.com/53208493/88036527-a3a46800-cb7e-11ea-887a-5272d23c6d6d.png)

### 4) 사용자 지정 이미지 테스트
#### (1) 컨테이너 실행
위에서 빌드한 myweb:latest 이미지가 잘 동작하는지 확인해보자.
`docker container run -d --name test -p 8080:8080 myweb`
![Screenshot from 2020-07-21 18-08-48](https://user-images.githubusercontent.com/53208493/88036530-a43cfe80-cb7e-11ea-94b6-9530f6e9820f.png)

#### (2) 컨테이너 확인
`docker container ls -l`
![Screenshot from 2020-07-21 18-08-48](https://user-images.githubusercontent.com/53208493/88036530-a43cfe80-cb7e-11ea-94b6-9530f6e9820f.png)

#### (3) 애플리케이션 동작 확인
curl 명령을 이용해 http://localhost:8080 주소로 접속해보자.
`curl http://localhost:8080`
![Screenshot from 2020-07-21 18-10-22](https://user-images.githubusercontent.com/53208493/88036531-a43cfe80-cb7e-11ea-9e55-9ed75d025aac.png)

### 5) 사용자 지정 이미지 업로드
Node.js 파일과 Dockerfile로 만들어준 사용자 지정 이미지를 도커 허브에 업로드 해보자. 

#### (1) 도커 허브 인증
도커 이미지를 저장소에 업로드하기 위해서는 로그인이 필요하다.
`docker login`
![Screenshot from 2020-07-21 18-16-59](https://user-images.githubusercontent.com/53208493/88036538-a56e2b80-cb7e-11ea-91cd-bd72b658cb5f.png)

#### (2) 사용자 지정 이미지 이름 변경
도커 허브에 이미지를 업로드 하기 위해선 이미지의 이름에 본인의 도커 허브 계정 이름이 포함되어있어야 한다.
따라서 이미지의 이름을 변경해주자.
`docker image tag myweb:latest minjulee226/myweb:latest`

#### (3) 사용자 지정 이미지 업로드
myweb 이미지를 도커 허브에 업로드하자.
`docker push minjulee226/myweb:latest`
![Screenshot from 2020-07-21 18-17-40](https://user-images.githubusercontent.com/53208493/88036539-a56e2b80-cb7e-11ea-9898-aaa890c85cb6.png)

도커 허브에서 새롭게 업로드된 이미지를 확인할 수 있다.
![Screenshot from 2020-07-21 18-15-25](https://user-images.githubusercontent.com/53208493/88036532-a4d59500-cb7e-11ea-9de7-d954b48db9ab.png)

## 5. 명령형 명령어를 사용한 애플리케이션 실행
