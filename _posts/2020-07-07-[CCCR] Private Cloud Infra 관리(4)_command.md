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
      - env | grep '^OS_' : 오픈스택과 관련된 환경변수

## 실습 1
- 자격증명 파일을 통해 user1에서 생성했던 리소스 확인하기

### 내용
1. user1의 OpenStack RC File v3 다운받기



    - 오픈스택 쓸 때마다 필요한 변수들을 환경변수로 만들어 파일로 만들 거임
    - 환경변수는 시스템 재부팅 시 사라짐
    - 그래서 파일로 저장해서 사라지지 않게 해주기 → 자격증명 파일(credential file)

    ```jsx
    unset OS_TENANT_ID
    unset - 환경변수 해제
    ```

    - 글로벌 옵션으로 입력할 값들을 자격증명 파일에 저장해놨으므로 openstack server list 명령어 사용했을 때 다른 값들 입력 안해줘도 됨
    - 인증받을 주소, 아이디, 패스워드, 프로젝트
    - env 커맨드: 시스템에 등록한 환경변수 볼 수 있음
        - env | grep '^OS_' (오픈스택 관련한 환경변수)
    - 대시보드 사용 시 자격증명 파일 읽어오는 게 가장 먼저 할 일
    - 자격증명 파일 수정 후, 'source 자격증명 파일 주소' 커맨드 입력해주기

    - 우분투에서 다운 받을 것을 scp 커맨드 통해 controller로 전달 가능

        scp 파일 주소 root@서버 주소:~/

## 자격증명 파일 통해 user1에서 진행했던 리소스들 확인하기

- OpenStack RC File v3 다운 받기

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a96eca90-f42d-4d2e-b4a3-329cfa00fb8e/Screenshot_from_2020-07-07_15-36-04.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a96eca90-f42d-4d2e-b4a3-329cfa00fb8e/Screenshot_from_2020-07-07_15-36-04.png)

- 다운받은 파일 열어서 암호 입력해주기

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/da24709c-8f30-4412-9e27-8eac2ef690c6/Screenshot_from_2020-07-07_15-54-49.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/da24709c-8f30-4412-9e27-8eac2ef690c6/Screenshot_from_2020-07-07_15-54-49.png)

- 우분투

    ```jsx
    scp 'project1-openrc(2).sh' root@192.168.122.10:~/
    ```

- controller

    ```jsx
    source 'project1-openrc(2).sh' 
    ```

- controller 노드에서 확인

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4e7ab4a2-e6e1-45f7-bd99-f17eb6c7b76c/Screenshot_from_2020-07-07_15-56-12.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4e7ab4a2-e6e1-45f7-bd99-f17eb6c7b76c/Screenshot_from_2020-07-07_15-56-12.png)

- 우분투

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/50884688-d92f-4436-8a63-8a074ff874b7/Screenshot_from_2020-07-07_15-57-03.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/50884688-d92f-4436-8a63-8a074ff874b7/Screenshot_from_2020-07-07_15-57-03.png)

## Keystone

- 인증: 아이디, 패스워드 입력하는 순간 인증이 됨
- 권한 부여: 프로젝트의 역할 확인해서 권한 부여
- 토큰 관리: 일정 시간동안 사용할 수 있는 토큰(일정 기간, 일정 범위에서 사용 가능)
    - SSO(Single Sign On): 한 번의 인증으로 지속적으로 사용하는 것
    - 만료되면 재발행 필요: identity라는 서비스가 재발행해줌
- 카탈로그: 오픈스택 앤드포인트의 목록

    ```jsx
    openstack catalog list
    ```

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b89130cb-1c87-4488-be26-dcb1fab9e7c9/Screenshot_from_2020-07-07_16-43-17.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b89130cb-1c87-4488-be26-dcb1fab9e7c9/Screenshot_from_2020-07-07_16-43-17.png)

    - 이 기능 사용 위해서 admin 역할 가진 사용자로 자격증명 필요
    - 앤드포인트 각각의 주소: API 주소
    - endpoint = API 주소
    - endpoint의 목록 = catalog
        - public(외부에서 접근: 외부네트워크 대역과 같아야 함, 오픈스택 사용하면서 외부로 접근 가능한 대역 넣기)
        - internal(내부용, admin과 같은 네트워크 대역 사용)
        - admin(관리용, internal과 같은 네트워크 대역 사용)
        - 적어도 admin과 public은 분리되어야 함
        - 이걸 관리하는 거 **Keystone**
    - endpoint(API 주소)가 하는 역할
        - 통신하는 것

    - 사용자가 노바 서비스에서 가상머신 만들어달라 요청
    - 그 전에 사용자는 IDENTITY 서비스에 의해 인증 받고, 권한 부여, 토큰까지 받아야 함

    ```jsx
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

    - 노바라는 서비스에게 내가 이런 토큰 가지고 있고 인스턴스 만들고 싶다고 말함(플레이버 1 쓰고 센트오에스 이미지 써서 인스턴스 만들거라고 함)
    - 노바가 처리가능한 건 플레이버 밖에 없음
    - 유저가 어디로 요청해야 노바에게 요청하는 건가
    - 사용자는 노바의 api 주소로 이 값들을 전달해야 함
    - 사용자는 노바의 에이피아이주소를 모름 - 아이덴티티 서비스가 관리
    - 아이덴티티 서비스가 노바의 주소를 알고 있음 그래서 알려줌 사용자에게
    - 사용자는 노바의 에이파이주소로 전달
    - 노바는 플레이버 처리
    - 이미지, 네트워크 처리 못함
    - 이미지: 글랜스에게 요청
    - 노바는 사용자와 마찬가지로 토큰 가짐
    - 사용자가 요청한 이미지를 노바가 글랜스에게 요청해야함
    - 노바는 글랜스 서비스가 어딨는지 모름
    - 노바와 글랜스는 외부에서 접근할 핑요가 없음
    - 노바가 글랜스 관리하기 위해 요청하는 게 아니니까 그냥 internal 쓰면 됨
    - INTERNAL: 서비스와 서비스가 통신
    - 네트워크를 세가지로 분리한 이유
        - 작업량 분산시키려고
        - 트래픽의 유형별로 분산
            - 외부 트래픽은 PUBLIC
            - INTERNAL
            - ADMIN
    - api 주소를 알려면 아이덴티티 서비스가 동작하고 있어야함
    - 컨트롤러 노드에서 이 두가지 서비스는 살아남아야 함
        - keystone
        - Message Queue

    - **Message Queue ⇒ AMQP(Advanced Message Queuing Protocol)**
        - 프로토콜: 규칙

        ⇒ RabbitMQ(비동기 전달 방식)

        - 사용자가 노바 에이피아이에게 글랜스 만들어달라고 전달하는데 전달해주는 매개체가 RabbitMQ
        - 운송수단: 메시지 큐
        - 설정파일 안에 래빗엠큐에 대한 설정파일 들어있음
        - 이 서비스의 역할 꼭 기억해두기!

## OpenStack Command로 프로젝트, 사용자 생성, 역할 추가

1. 프로젝트 생성
    - admin으로 로그인
    - 아래 파일 다운로드

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/89a1f444-c34b-4fbf-b579-575a7ed64b8d/Screenshot_from_2020-07-07_18-03-49.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/89a1f444-c34b-4fbf-b579-575a7ed64b8d/Screenshot_from_2020-07-07_18-03-49.png)

    - 우분투

        ```jsx
        scp admin-openrc.sh root@192.168.122.10:~/
        ```

    - controller

        ```jsx
        source admin-openrc.sh 
        openstack project create project2
        openstack project list
        ```

        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d2a983c0-2208-4648-8349-158b0058c64d/Screenshot_from_2020-07-07_18-15-41.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d2a983c0-2208-4648-8349-158b0058c64d/Screenshot_from_2020-07-07_18-15-41.png)

2. 사용자 생성
    - controller

    ```jsx
    openstack user create --project project2 --password 'dkagh1.' user2
    openstack user list
    ```

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/59ca7470-443e-40e9-bbb3-a76e40357345/Screenshot_from_2020-07-07_18-15-56.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/59ca7470-443e-40e9-bbb3-a76e40357345/Screenshot_from_2020-07-07_18-15-56.png)

    - OpenStack에서 로그인

        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3295d806-8ec1-46c0-8ffe-5459494c2396/Screenshot_from_2020-07-07_17-58-55.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3295d806-8ec1-46c0-8ffe-5459494c2396/Screenshot_from_2020-07-07_17-58-55.png)

        - 오류 발생
        - 토큰을 못 받았다는 얘기
        - role이 없어서 권한부여를 못 받음(역할에 따른 권한)

            → 역할 추가해줘야 정상적으로 로그인 됨

3. 역할 추가
    - controller

    ```jsx
    openstack role add --project project2 --user user2 _member_
    ```

    - OpenStack에서 로그인(user2로 로그인 완료)

        ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5586e419-bf94-46c2-a119-3ac09917431d/Screenshot_from_2020-07-07_18-02-44.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5586e419-bf94-46c2-a119-3ac09917431d/Screenshot_from_2020-07-07_18-02-44.png)

- user1이라는 사용자를 다른 프로젝트의 멤버에도 추가하기

    프로젝트 아이디, 도메인 

    export OS_PROJECT_NAME=project2

    이렇게 변경해서 프로젝트 바꿔주면 됨

    아니면 자격증명 파일을 해당되는 값으로 바꿔주면 됨

    source project.sh

    openstack server list
