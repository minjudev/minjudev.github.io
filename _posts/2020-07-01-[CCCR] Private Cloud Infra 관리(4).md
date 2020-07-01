# 클라우드 구축을 위한 오픈스택 구성
## 1. 오픈스택 환경 구성
## 2. 오픈스택 설치
## 3. 대시보드를 이용한 오픈스택 인스턴스 운영
### 1) 로그인
![Screenshot from 2020-07-01 09-54-43](https://user-images.githubusercontent.com/53208493/86191024-f5aa2d00-bb80-11ea-8310-1e3d1ca2fdc2.png)
- 사용자 이름: admin
- 암호: 응답파일에서 지정한 패스워드 입력
  - 패스워드를 지정하지 않았다면 /root/answer.txt 파일 혹은 /root/keystonerc_admin 파일에서 확인

### 2) 대시보드 메뉴
![Screenshot from 2020-07-01 09-57-36](https://user-images.githubusercontent.com/53208493/86191181-5afe1e00-bb81-11ea-82f5-32da59d4d779.png)
- project: instance, network, object storage 생성 및 관리
- admin: 해당 메뉴는 관리자 역할을 가진 사용자에게만 표시됨, 오픈스택 전체 모니터링 및 리소스 관리를 위해 사용
- identity: 해당 메뉴는 모든 사용자에게 표시되지만 관리자 역할을 가진 사용자만 관리 가능, project, user, group, role을 관리 

### 3) 대시보드 사용법
### === admin ===
### (1) Project
- identity -> project
- 프로젝트: 사용자가 리소스를 사용할 수 있는 범위, 필요 시 프로젝트에서 사용할 리소스 쿼터 지정 가능
![Screenshot from 2020-07-01 09-58-53](https://user-images.githubusercontent.com/53208493/86191230-7f59fa80-bb81-11ea-819f-5bc8c024089a.png)

### (2) User
- identity -> user
- admin 사용자를 제외한 사용자들은 오픈스택 운영에 필요한 service 계정(service 프로젝트의 멤버로 속해 있음)
![Screenshot from 2020-07-01 12-11-27](https://user-images.githubusercontent.com/53208493/86199059-0fa13b00-bb94-11ea-9179-7d75d2a319e5.png)

### (3) Flavor
- admin -> flavor
- 인스턴스 생성 시 인스턴스에서 사용할 하드웨어 사이즈를 정의해 놓은 프로파일
![Screenshot from 2020-07-01 11-30-36](https://user-images.githubusercontent.com/53208493/86202506-53e50900-bb9d-11ea-99e7-5183ea99b9e9.png)
- VCPUS: 인스턴스에 할당할 VCPU 개수
- RAM: 인스턴스에 할당할 메모리
- Disk
  - Root Disk: 운영체제가 설치될 디스크
  - Ephemeral Disk: 비어있는 디스크
  - Swap Disk: 인스턴스에서 사용할 Swap 디스크

### (4) Image(Public)
- admin -> image
- 인스턴스 생성 시 사용할 디스크 템플릿 이미지
![Screenshot from 2020-07-01 11-26-11](https://user-images.githubusercontent.com/53208493/86202637-c524bc00-bb9d-11ea-9099-ee654b47251f.png)
- Image Source: 로컬에 파일 또는 이미지가 있는 url 주소 선택
- Format: qcow2
- Image Sharing Visibility: Public
  - public: 다른 프로젝트에서도 해당 이미지를 확인하고 사용 가능, 관리자 역할을 가진 사용자만 설정 가능
  - private: 해당 사용자의 프로젝트 내에서만 사용 가능

### (5) External Network
- admin -> network
- 외부 네트워크는 관리자만 설정 가능 
- 실제 외부로 연결되는 네트워크 대역으로 설정해야 하며, 이 외부 네트워크는 후에 유동 IP 생성에 사용될 네트워크   
![Screenshot_from_2020-06-30_13-44-12](https://user-images.githubusercontent.com/53208493/86183329-bbcf2b80-bb6c-11ea-9ff9-1e631a09f80a.png)
- Name
- Project
- Provider Network Type: 인스턴스가 사용할 가상 네트워크를 구현하는 물리적 매커니즘
- Physical Network: Packstack으로 설치 시 응답파일에서 지정한 "extnet" 이름 지정
  - 해당 이름은 grep -r br-ex /etc/neutron 명령어를 통해 확인 가능   
    <img width="386" alt="캡처" src="https://user-images.githubusercontent.com/53208493/86183658-795a1e80-bb6d-11ea-970c-3090c0967c6c.PNG">  
![Screenshot from 2020-07-01 14-29-37](https://user-images.githubusercontent.com/53208493/86206440-59475100-bba7-11ea-9d84-2014b99302af.png)
- Subnet Name: 서브넷 이름
- Network Address: 실제 외부 네트워크 대역과 같은 네트워크 대역
- Gateway IP: 실제 외부 게이트웨이 주소
![Screenshot from 2020-07-01 11-45-19](https://user-images.githubusercontent.com/53208493/86206620-ca870400-bba7-11ea-8957-7b5d6709d001.png)
- Enable DHCP: No
- Allocation Pools: 유동 IP가 할당될 범위

### === member ===
### (1) Internal Network
- 메뉴 다시 적기
- 내부 네트워크: 인스턴스가 사용할 네트워크
[사진]
- Name
[사진]
- Subnet Name: 서브넷 이름
- Network Address: 내부 네트워크에서 사용할 네트워크 대역
- Gateway IP: 내부 네트워크에서 외부로 나갈 때 사용할 라우터의 게이트웨이 주소
[사진]
- Enable DHCP: Yes
- DNS Name Servers: DNS 서버 주소 

### (2) Router
- 메뉴 다시 적기

### (3) Security Group
- 메뉴 다시 적기

### (4) Floating IP
- 메뉴 다시 적기

### (5) Image(Private)
- 메뉴 다시 적기

### (6) Key Pair
- 메뉴 다시 적기

### (7) Instance
- 메뉴 다시 적기

### (8) Volume
- 메뉴 다시 적기

### (9) Object Storage
- 메뉴 다시 적기

### (10) Load Balancer
- 메뉴 다시 적기


## 4. CLI를 이용한 오픈스택 인스턴스 운영
