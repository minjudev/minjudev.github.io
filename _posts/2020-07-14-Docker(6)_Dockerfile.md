## 2. Dockerfile
### 1) Dockerfile 개요
도커는 Dockerfile을 사용해 이미지를 제작할 수 있다.  
Dockerfile은 코드 형태의 텍스트 문서이며, 여러 지시어를 사용해 이미지를 제작할 수 있다.  

### 2) Dockerfile 지시어

|지시어|설명|
|:---:|:---:|
|FROM|베이스 이미지 지정|
|RUN|이미지를 생성하면서 실행할 명령 지정|
|ENTRYPOINT|컨테이너의 애플리케이션 지정|
|EXPOSE|컨테이너의 포트 지정|
|ADD|이미지 생성 시 파일 추가|
|VOLUME|컨테이너의 볼륨 지정|
|WORKDIR|컨테이너의 작업 디렉토리 지지정|
|MAINTAINER|이미지 작성자 명시|
|CMD|컨테이너의 애플리케이션 지정|
|LABEL|이미지의 라벨 지정|
|ENV|컨테이너의 환경 변수 지정|
|COPY|이미지 생성 시 파일 복사|
|USER|컨테이너의 사용자 지정|

- RUN, CMD, ENTRYPOINT 지시어  
이 지시어들은 exec와 shell 형식으로 사용 가능하다.   
exec 형식은 명령을 쉘을 통해 실행하지 않으며, shell 형식은 /bin/sh와 같은 쉘을 통해 지정된 명령을 실행한다.  
  - exec, shell 형식의 표현방법  
  
  |형식|표현방법|
  |:---:|:---:|
  |exec|["yum", "-y", "install", "httpd"]|
  |shell|yum -y install httpd|

### 3) Dockerfile 이미지 제작  
#### (1) FROM
가장 기본적인 커맨드로, 어떤 이미지를 기반으로 새로운 이미지를 생성할 것인지 결정할 수 있다.  

[도커 파일 캡처 사진]

해당 도커 파일로 이미지를 제작해보자.  
`docker built -t minju/docker:fromtest .`  
- -t 옵션: 생설될 이미지의 이름을 정할 수 있다.  
- .(마지막 구두점): Dockerfile의 위치  

[이미지 생성 사진]   

`docker images` 명령어로 생성된 이미지를 확인해보자.  
[이미지 확인 사진]  
minju/docker:fromtest 이미지는 centos 이미지를 기반으로 생성되었기 때문에 IMAGE ID가 동일한 것을 확인할 수 있다.  

#### (2) RUN
RUN 커맨드는 bash 쉘에서 입력하는 것과 동일한 기능을 한다고 생각하면 된다.    

[도커 파일 캡처 사진]  

해당 도커 파일로 이미지를 제작해보자.  
`docker built -t minju/docker:runtest .`  

[이미지 생성 사진]   

생성한 이미지로 컨테이너를 만들어보자.  
`docker run -it --name runtest minju/docker:runtest`  

[컨테이너 캡처 사진]
minju 폴더가 생성되어있음을 확인할 수 있다.  

#### (3) ADD
ADD 명령어는 build 명령 중간에 호스트의 파일 시스템으로부터 파일을 가져오는 것이다. 이미지에 파일을 더하는 것이라고 생각하면 된다.      

먼저, 다음과 같이 입력해 테스트 파일을 생성해보자.  
`echo "hello minju" >> test.txt`  

Dockerfile은 다음과 같이 작성한다.  
`ADD (호스트 파일) (추가될 이미지 디렉토리)`의 형식으로 작성하자.  
[캡처 사진]

해당 도커 파일로 이미지를 제작해보자.  
`docker built -t minju/docker:addtest .`  

[이미지 생성 사진]   

생성한 이미지로 컨테이너를 만들어보자.  
`docker run -it --name addtest minju/docker:addtest`  

[컨테이너 캡처 사진]  
test.txt 파일이 잘 만들어진 것을 확인할 수 있다. 

#### (4) CMD

#### (5) ENTRYPOINT

## 참고 사이트
- [alice_k106님의 블로그](https://blog.naver.com/PostView.nhn?blogId=alice_k106&logNo=220646382977&parentCategoryNo=7&categoryNo=&viewDate=&isShowPopularPosts=true&from=search)



