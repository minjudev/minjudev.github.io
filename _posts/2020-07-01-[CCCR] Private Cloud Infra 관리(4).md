# 클라우드 구축을 위한 오픈스택 구성
## 1. 오픈스택 환경 구성
## 2. 오픈스택 설치
## 3. 대시보드를 이용한 오픈스택 인스턴스 운영
### 1) 로그인
[사진] 
- 사용자 이름: admin
- 암호: 응답파일에서 지정한 패스워드 입력
  - 패스워드를 지정하지 않았다면 /root/answer.txt 파일 혹은 /root/keystonerc_admin 파일에서 확인

## 2) 대시보드 메뉴
[사진]
- project: instance, network, object storage 생성 및 관리
- admin: 해당 메뉴는 관리자 역할을 가진 사용자에게만 표시됨, 오픈스택 전체 모니터링 및 리소스 관리를 위해 사용
- identity: 해당 메뉴는 모든 사용자에게 표시되지만 관리자 역할을 가진 사용자만 관리 가능, project, user, group, role을 관리 

## 3) 프로젝트 생성
- identity -> project
- 프로젝트: 사용자가 리소스를 사용할 수 있는 범위, 필요 시 프로젝트에서 사용할 리소스 쿼터 지정 가능
- 프로젝트 생성
[사진]

## 4) 사용자 생성
- identity -> user
- admin 사용자를 제외한 사용자들은 오픈스택 운영에 필요한 service 계정(service 프로젝트의 멤버로 속해 있음)
- 사용자 생성
[사진]

## 5) Flavor 관리
- admin -> flavor
- 인스턴스 생성 시 인스턴스에서 사용할 하드웨어 사이즈를 정의해 놓은 프로파일
- VCPUS: 인스턴스에 할당할 VCPU 개수
- RAM: 인스턴스에 할당할 메모리
- Disk
  - Root Disk: 운영체제가 설치될 디스크
  - Ephemeral Disk: 비어있는 디스크
  - Swap Disk: 인스턴스에서 사용할 Swap 디스크
- 인스턴스 생성
[사진]

## 6) 이미지 생성
- admin -> image
- 인스턴스 생성 시 사용할 디스크 템플릿 이미지
[사진]
- create image 버튼 클릭
  - 이미지 포맷: qcow2
  - 이미지 소스: 로컬에 파일 또는 이미지가 있는 url 주소 선택
  - 이미지 공유의 가시성: 공용
    - public: 다른 프로젝트에서도 해당 이미지를 확인하고 사용 가능, 관리자 역할을 가진 사용자만 설정 가능
    - private: 해당 사용자의 프로젝트 내에서만 사용 가능

## 7) 외부 네트워크 생성
- admin -> network
- 외부 네트워크는 관리자만 설정 가능 
- 실제 외부로 연결되는 네트워크 대역으로 설정해야 하며, 이 외부 네트워크는 후에 유동 IP 생성에 사용될 네트워크
![Screenshot_from_2020-06-30_13-44-12](https://user-images.githubusercontent.com/53208493/86183329-bbcf2b80-bb6c-11ea-9ff9-1e631a09f80a.png)
- Create Network
  - Name
  - Project
  - Provider Network Type: 인스턴스가 사용할 가상 네트워크를 구현하는 물리적 매커니즘
  - Physical Network: Packstack으로 설치 시 응답파일에서 지정한 "extnet" 이름 지정
    - grep -r br-ex /etc/neutron 명령어를 통해 확인 가능   
      <img width="386" alt="캡처" src="https://user-images.githubusercontent.com/53208493/86183658-795a1e80-bb6d-11ea-970c-3090c0967c6c.PNG">  

## 4. CLI를 이용한 오픈스택 인스턴스 운영
