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

![Screenshot from 2020-07-15 11-08-26](https://user-images.githubusercontent.com/53208493/87496132-f2896380-c68d-11ea-8515-2f5f0977800f.png)

해당 도커 파일로 이미지를 제작해보자.  
`docker built -t minju/docker:fromtest ~`  
- -t 옵션: 생설될 이미지의 이름을 정할 수 있다.  
- ~: Dockerfile의 위치  

![Screenshot from 2020-07-15 11-28-03](https://user-images.githubusercontent.com/53208493/87503302-d17d3e80-c69e-11ea-9d3d-d4259de3a947.png)

`docker images` 명령어로 생성된 이미지를 확인해보자.  
![Screenshot from 2020-07-15 11-28-14](https://user-images.githubusercontent.com/53208493/87503305-d2ae6b80-c69e-11ea-9e88-9ffe80e0d292.png)
minju/docker:fromtest 이미지는 centos 이미지를 기반으로 생성되었기 때문에 IMAGE ID가 동일한 것을 확인할 수 있다.  

#### (2) RUN
RUN 커맨드는 bash 쉘에서 입력하는 것과 동일한 기능을 한다고 생각하면 된다.    
![Screenshot from 2020-07-15 13-29-12](https://user-images.githubusercontent.com/53208493/87503474-43ee1e80-c69f-11ea-8b68-504260e76e70.png)

해당 도커 파일로 이미지를 제작해보자.  
`docker built -t minju/docker:runtest ~`  

![Screenshot from 2020-07-15 13-33-33](https://user-images.githubusercontent.com/53208493/87505107-055a6300-c6a3-11ea-9dcf-d2d503f07d6a.png)

생성한 이미지로 컨테이너를 만들어보자.  
`docker run -it --name runtest minju/docker:runtest`  

![Screenshot from 2020-07-15 13-34-27](https://user-images.githubusercontent.com/53208493/87505108-05f2f980-c6a3-11ea-8421-0d78f8f4881a.png)
minju 폴더가 생성되어있음을 확인할 수 있다.  

#### (3) ADD
ADD 명령어는 build 명령 중간에 호스트의 파일 시스템으로부터 파일을 가져오는 것이다. 이미지에 파일을 더하는 것이라고 생각하면 된다.      

먼저, 다음과 같이 입력해 테스트 파일을 생성해보자.  
`echo "hello minju" >> test.txt`  
![Screenshot from 2020-07-15 14-02-25](https://user-images.githubusercontent.com/53208493/87505700-7c442b80-c6a4-11ea-8b0b-b56b4aaf5a8a.png)

Dockerfile은 다음과 같이 작성한다.  
`ADD (호스트 파일) (추가될 이미지 디렉토리)`의 형식으로 작성하자.  

![Screenshot from 2020-07-15 14-03-34](https://user-images.githubusercontent.com/53208493/87505702-7cdcc200-c6a4-11ea-8b94-03e251318492.png)

해당 도커 파일로 이미지를 제작해보자.  
`docker built -t minju/docker:addtest ~`  

![Screenshot from 2020-07-15 14-06-37](https://user-images.githubusercontent.com/53208493/87505703-7d755880-c6a4-11ea-9466-af5e2dc86084.png)

생성한 이미지로 컨테이너를 만들어보자.  
`docker run -it --name addtest minju/docker:addtest`  

![Screenshot from 2020-07-15 14-06-48](https://user-images.githubusercontent.com/53208493/87505704-7e0def00-c6a4-11ea-9ca5-a4ab822d032b.png)   
test.txt 파일이 잘 만들어진 것을 확인할 수 있다. 

#### (4) CMD
Dockerfile에 CMD 수행명령을 주는 것과 더불어 컨테이너 실행 시 인자값도 주게 되면 Dockerfile에 지정된 CMD 값을 대신하여 지정한 인자값으로 컨테이너가 실행된다.  

먼저, 다음과 같이 Dockerfile을 만들어보자.  
![Screenshot from 2020-07-15 18-47-32](https://user-images.githubusercontent.com/53208493/87531402-a3165800-c6cc-11ea-9d0d-8c2d1f1ba158.png)

위에서 정의된 Dockerfile을 통해 df -h 명령어를 한 번 수행하고 종료하는 이미지를 만들 수 있다.  
위의 Dockerfile로 이미지를 만들고 컨테이너를 실행해보았다.  
![Screenshot from 2020-07-15 18-52-08](https://user-images.githubusercontent.com/53208493/87531405-a3aeee80-c6cc-11ea-8d1a-5802d684b804.png)  

![Screenshot from 2020-07-15 18-52-29](https://user-images.githubusercontent.com/53208493/87531408-a4478500-c6cc-11ea-98e1-175216ed189d.png)

빌드된 이미지로 컨테이너를 동작시켜보면, cmd에 작성된 명령어를 실행하는 것을 확인할 수 있다.

이번에는 같은 이미지로 컨테이너 실행 시 추가 인자값도 준다면 컨테이너 실행 시 어떤 변화가 나타나는지 살펴보자.  
docker run으로 컨테이너를 실행할 때, 추가로 ps -aef 인자값도 함께 입력하여 실행해보았다.  
![Screenshot from 2020-07-15 19-08-48](https://user-images.githubusercontent.com/53208493/87532925-d3f78c80-c6ce-11ea-8d65-15910ffc9ab4.png)   
위 명령어를 실행하면 CMD로 받은 명령 대신 추가로 부여한 인자값에 대한 명령을 실행하는 것을 확인할 수 있다. 

docker inspect 명령을 통해 컨테이너의 설정 내용을 자세히 살펴보자.   
![Screenshot from 2020-07-15 19-09-00](https://user-images.githubusercontent.com/53208493/87532929-d4902300-c6ce-11ea-8618-0a49ec8f7fc5.png)   

![Screenshot from 2020-07-15 19-09-41](https://user-images.githubusercontent.com/53208493/87532930-d4902300-c6ce-11ea-9417-1963dd1b78f6.png)

이처럼 Cmd 값이 추가로 부여한 인자값으로 바뀐 것을 알 수 있다. 

#### (5) ENTRYPOINT
ENTRYPOINT를 이용해 새로운 Dockerfile을 만들고 이미지를 빌드해보자.  
![Screenshot from 2020-07-15 19-31-56](https://user-images.githubusercontent.com/53208493/87535102-e626fa00-c6d1-11ea-8858-c57fe00511f0.png)    

![Screenshot from 2020-07-15 19-34-15](https://user-images.githubusercontent.com/53208493/87535297-38681b00-c6d2-11ea-9236-777335fff355.png)

빌드한 이미지를 이용해 새로운 컨테이너를 실행해보았다.  
![Screenshot from 2020-07-15 19-34-21](https://user-images.githubusercontent.com/53208493/87535299-3900b180-c6d2-11ea-91d7-5524591207d7.png)

이렇게 해서 실행된 결과는 cmd를 이용해 Dockerfile을 만들어줬을 때와 다른 점이 없다.  
하지만 docker inspect를 이용해 비교해보면 차이점을 확인할 수 있다.   
![Screenshot from 2020-07-15 19-40-36](https://user-images.githubusercontent.com/53208493/87535901-2aff6080-c6d3-11ea-9ff3-b4e38eafcfa7.png)    

![Screenshot from 2020-07-15 19-40-49](https://user-images.githubusercontent.com/53208493/87535903-2b97f700-c6d3-11ea-8e44-a602cf865301.png)  

Cmd는 null로 비워져있고, Entrypoint 항목에 실행된 명령 정보가 있다.

이제는 위의 cmd에서 했던 것과 같이 docker run 수행 시 인자를 추가로 넣어서 컨테이너를 실행해보겠다.  
![Screenshot from 2020-07-15 19-48-04](https://user-images.githubusercontent.com/53208493/87536474-24bdb400-c6d4-11ea-8c6c-a9b50fad489d.png)  
결과는 이처럼 에러를 출력한다.  
그 이유를 확인하기 위해 docker inspect를 통해 설정 값들을 살펴보자.   
![Screenshot from 2020-07-15 19-48-59](https://user-images.githubusercontent.com/53208493/87536578-44ed7300-c6d4-11ea-964b-43b9d3b2ea55.png)  
컨테이너 실행 시 /bin/df 명령은 유지하고, 추가 인자를 CMD로 받아 처리한 것을 확인할 수 있다.  
/bin/df 명령을 유지한 채 추가 인자도 함께 받아서 실행했기 때문에 오류가 발생해 제대로 동작하지 않은 것이다.

## 참고 사이트
- [alice_k106님의 블로그](https://blog.naver.com/PostView.nhn?blogId=alice_k106&logNo=220646382977&parentCategoryNo=7&categoryNo=&viewDate=&isShowPopularPosts=true&from=search)  
- [ㅍㅍㅋㄷ님의 블로그](https://bluese05.tistory.com/77)


