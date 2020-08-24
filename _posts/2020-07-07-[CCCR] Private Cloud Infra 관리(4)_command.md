## OpenStack Command
1. CLI를 이용해 OpenStack 인스턴스를 운영하기 위한 설치 과정
```
- 우분투

sudo apt install -y python-pip
sudo apt install -y python3-openstackclient
pip3 install oslo_log
**오픈스택 명령어 자동완성 기능 설치
openstack complete | sudo tee /etc/bash_completion.d/openstack
source /etc/bash_completion.d/openstack
```

2. 명령어 형식
- openstack RESOURCE ACTION [global action] [resource option]
  - RESOURCE 
    - 1. project
    - 2. user
    - 3. role
    - 4. flavor
    - 5. image
    - 6. network
    - 7. subnet
    - 8. router
    - 9. security group
    - 10. key pair
    - 11. floating ip
    - 12. volume
    - 13. server
    - 14. container
    - 15. object storage
  - ACTION
    - create
    - delete
    - list
    - show
    - add
    - remove
    - set

3. 환경변수
    - 오픈스택을 사용할 때마다 필요한 변수들을 환경변수로 만들어 파일로 생성
    - 환경변수는 시스템 재부팅 시 사라지므로 파일로 만들어 사라지지 않게 해주기
    - 이렇게 만든 파일을 자격증명 파일(credential file)이라 함
    - env 커맨드: 시스템에 등록한 환경변수를 볼 수 있음

## 실습 1
- 자격증명 파일을 통해 user1에서 생성했던 리소스 확인하기

### 내용
1. user1의 OpenStack RC File v3 다운받기   
![Screenshot_from_2020-07-07_15-36-04](https://user-images.githubusercontent.com/53208493/86925620-115a9800-c16c-11ea-8ab9-5720f16985ef.png)

2. 다운받은 자격증명 파일을 Controller node에 전달   
```
scp project1-openrc.sh root@192.168.122.10:~/
```
![Screenshot_from_2020-07-07_15-56-12](https://user-images.githubusercontent.com/53208493/86926261-cee58b00-c16c-11ea-855c-2a971db78267.png)

3. 우분투에서 로그인   
```
source project1-openrc.sh
```

## Identity(Keystone)  관리
- 인증: 아이디, 패스워드 입력하는 순간 인증이 됨
- 권한 부여: 프로젝트의 역할 확인해서 권한 부여
- 토큰 관리: 일정 시간동안 사용할 수 있는 토큰 관리
    - SSO(Single Sign On): 한 번의 인증으로 지속적으로 사용하는 것
- 카탈로그: 오픈스택 앤드포인트의 목록
  ```
  openstack catalog list
  ```
  ![Screenshot_from_2020-07-07_16-43-17](https://user-images.githubusercontent.com/53208493/86927251-13255b00-c16e-11ea-8500-0218270c0ff2.png)
  - 각 endpoint의 주소: API 주소


```
User
  - Token

+---> Identity
|
User(Token, Instance[flavor1, centos7, network1])
+
|
+ Nova(nova-api) ---> Glance(glance-api)
      (Token, centos7)
```
- 추가 설명   
사용자가 nova 서비스에서 인스턴스 생성을 요청함  
그 전에 사용자는 identity 서비스에 의해 인증을 받고, 권한 부여, 토큰까지 받아야 함  
사용자가 nova라는 서비스에게 자신이 이런 토큰을 가지고 있는데 특정 인스턴스(flavor1, centos 이미지 사용해 생성)를 만들고 싶다고 요청  
사용자는 nova의 API 주소로 위 값들을 전달해야하지만, nova의 API 주소를 모름  
nova의 API 주소는 identitiy 서비스가 관리     
identity 서비스가 nova의 주소를 사용자에게 알려줌  
사용자는 nova의 API 주소로 위 값(flavor1, centos 이미지)들을 전달  
하지만, nova는 flavor만 처리 가능(이미지, 네트워크는 처리 불가능)  
사용자가 요청한 이미지를 nova가 glance에게 요청해야 함  

## Message Queue(RabbitMQ) 서비스 관리
- AMQP(Advanced Message Queuing Protocol) 프로토콜 사용
- RabbitMQ(비동기 전달 방식)
- 사용자가 nova API에게 glance 생성을 요청하는데 전달을 해주는 매개체가 RabbitMQ

## 실습 2
OpenStack Command로 프로젝트, 사용자 생성, 역할 추가

### 내용
1. 프로젝트 생성
  - admin으로 로그인
  - 아래 파일 다운로드   
  ![Screenshot_from_2020-07-07_18-03-49](https://user-images.githubusercontent.com/53208493/86936303-ca26d400-c178-11ea-8756-687149f88eea.png)
  - 우분투
  ```
  source admin-openrc.sh 
  openstack project create project2
  openstack project list
  ```
  ![Screenshot_from_2020-07-07_18-15-41](https://user-images.githubusercontent.com/53208493/86936554-1f62e580-c179-11ea-9c3d-b63894a0ac24.png)

2. 사용자 생성
  - 우분투
  ```
  openstack user create --project project2 --password 'dkagh1.' user2
  openstack user list
  ```
  ![Screenshot_from_2020-07-07_18-15-56](https://user-images.githubusercontent.com/53208493/86936794-710b7000-c179-11ea-8708-4a830d7927e9.png)
  - OpenStack에서 로그인   
  ![Screenshot_from_2020-07-07_17-58-55](https://user-images.githubusercontent.com/53208493/86936996-aca63a00-c179-11ea-9784-b6c941c5f41b.png)
      - 토큰을 아직 받지 못해 오류 발생
      - role이 없어서 권한부여를 못 받음(역할에 따른 권한)
        → 역할 추가해줘야 정상적으로 로그인 가능

3. 역할 추가
```
openstack role add --project project2 --user user2 _member_
```
- OpenStack에서 로그인
![Screenshot_from_2020-07-07_18-02-44](https://user-images.githubusercontent.com/53208493/86937328-145c8500-c17a-11ea-8052-0fd9305007d5.png)
