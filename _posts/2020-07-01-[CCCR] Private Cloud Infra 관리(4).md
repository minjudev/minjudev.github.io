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
- Identity -> Projects
- 프로젝트: 사용자가 리소스를 사용할 수 있는 범위, 필요 시 프로젝트에서 사용할 리소스 쿼터 지정 가능
![Screenshot from 2020-07-01 09-58-53](https://user-images.githubusercontent.com/53208493/86191230-7f59fa80-bb81-11ea-819f-5bc8c024089a.png)

### (2) User
- Identity -> Users
- admin 사용자를 제외한 사용자들은 오픈스택 운영에 필요한 service 계정(service 프로젝트의 멤버로 속해 있음)
![Screenshot from 2020-07-01 12-11-27](https://user-images.githubusercontent.com/53208493/86199059-0fa13b00-bb94-11ea-9179-7d75d2a319e5.png)

### (3) Flavor
- Admin -> Compute -> Flavors
- 인스턴스 생성 시 인스턴스에서 사용할 하드웨어 사이즈를 정의해 놓은 프로파일
![Screenshot from 2020-07-01 11-30-36](https://user-images.githubusercontent.com/53208493/86202506-53e50900-bb9d-11ea-99e7-5183ea99b9e9.png)
- VCPUS: 인스턴스에 할당할 VCPU 개수
- RAM: 인스턴스에 할당할 메모리
- Disk
  - Root Disk: 운영체제가 설치될 디스크
  - Ephemeral Disk: 비어있는 디스크
  - Swap Disk: 인스턴스에서 사용할 Swap 디스크

### (4) Image(Public)
- Admin -> Compute -> Images
- 인스턴스 생성 시 사용할 디스크 템플릿 이미지
![Screenshot from 2020-07-01 11-26-11](https://user-images.githubusercontent.com/53208493/86202637-c524bc00-bb9d-11ea-9099-ee654b47251f.png)
- Image Source: 로컬에 파일 또는 이미지가 있는 url 주소 선택
- Format: qcow2
- Image Sharing Visibility: Public
  - public: 다른 프로젝트에서도 해당 이미지를 확인하고 사용 가능, 관리자 역할을 가진 사용자만 설정 가능
  - private: 해당 사용자의 프로젝트 내에서만 사용 가능

### (5) External Network
- Admin -> Network -> Networks
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
- Project -> Network -> Networks
- 내부 네트워크: 인스턴스가 사용할 네트워크   
![Screenshot from 2020-07-02 10-30-43](https://user-images.githubusercontent.com/53208493/86306438-3d978580-bc4f-11ea-8e4f-6ba8f06c19db.png)   
- Network Name   

![Screenshot from 2020-07-02 10-31-15](https://user-images.githubusercontent.com/53208493/86306444-3ec8b280-bc4f-11ea-8047-b25c5544f28b.png)   
- Subnet Name: 서브넷 이름
- Network Address: 내부 네트워크에서 사용할 네트워크 대역
- Gateway IP: 내부 네트워크에서 외부로 나갈 때 사용할 라우터의 게이트웨이 주소
![Screenshot from 2020-07-02 10-31-28](https://user-images.githubusercontent.com/53208493/86306445-3f614900-bc4f-11ea-9862-4668def87e4d.png)   
- Enable DHCP: Yes
- DNS Name Servers: DNS 서버 주소 

### (2) Router
- Project -> Network -> Routers
- 라우터: 내부 네트워크가 라우터를 통해 외부 네트워크와 연결됨
- 라우터 생성   
![Screenshot from 2020-07-02 10-34-25](https://user-images.githubusercontent.com/53208493/86306611-b1399280-bc4f-11ea-92cc-ed05c7d09b13.png)
- 인터페이스 추가   
![Screenshot from 2020-07-02 10-35-30](https://user-images.githubusercontent.com/53208493/86306660-d3cbab80-bc4f-11ea-9dcc-dd46e70d914b.png)
![Screenshot from 2020-07-02 10-35-41](https://user-images.githubusercontent.com/53208493/86306661-d4644200-bc4f-11ea-96ad-e88b7a1038a0.png)
                                                           
### (3) Security Group
- Project -> Network -> Security Groups
- 보안그룹: 인스턴스의 네트워크 필터링 정책
- Packstack으로 오픈스택 설치 시 "default"라는 이름의 보안 그룹이 미리 생성되어있음
- "default" 보안 그룹의 규칙 확인 및 SSH, ICMP 규칙 생성
![Screenshot from 2020-07-02 10-38-01](https://user-images.githubusercontent.com/53208493/86306865-46d52200-bc50-11ea-8c87-566136f833be.png)
- 보안 그룹 생성   
![Screenshot from 2020-07-02 10-38-20](https://user-images.githubusercontent.com/53208493/86306867-476db880-bc50-11ea-881b-69d8f43b3be9.png)   
- 규칙 생성   
  - HTTP
  - HTTPS
![Screenshot from 2020-07-02 10-39-01](https://user-images.githubusercontent.com/53208493/86306869-48064f00-bc50-11ea-8bdf-c4876a8d40be.png)
  
### (4) Floating IP
- Project -> Network -> Floating IPs
- 유동 IP: 인스턴스에 할당하여 외부 네트워크에서 인스턴스에 접근할 수 있도록 해주는 IP
- Floating IP 생성   
![Screenshot from 2020-07-02 10-41-04](https://user-images.githubusercontent.com/53208493/86307005-9e738d80-bc50-11ea-8f6b-0c8eeeae83ba.png)   

### (5) Image(Private)
- Project -> Compute -> Images
![Screenshot from 2020-07-02 10-47-02](https://user-images.githubusercontent.com/53208493/86307335-7afd1280-bc51-11ea-8d32-0f96d3d3514b.png)
- Image Source: 로컬에 파일 또는 이미지가 있는 url 주소 선택
- Format: qcow2

### (6) Key Pair
- Project -> Compute -> Key Pairs
- 인터넷에 공개된 대부분의 클라우드 이미지는 SSH 키 인증 방식으로만 접근 가능, 보안을 위해 SSH 패스워드 인증 방식으로 접근하지 못하도록 하기 위한 조치
- 키 페어: 인스턴스 생성 시 적용할 SSH 키를 등록하기 위해 키 쌍을 생성해주는 부분
- 키 페어 생성   
![Screenshot from 2020-07-02 10-50-02](https://user-images.githubusercontent.com/53208493/86307499-df1fd680-bc51-11ea-8dde-cbe86d0ac231.png)
![Screenshot from 2020-07-02 10-50-12](https://user-images.githubusercontent.com/53208493/86307501-e0510380-bc51-11ea-8957-c2174b038822.png)

### (7) Instance
- Project -> Compute -> Instances   
- 이미지 선택   
![Screenshot from 2020-07-02 10-51-55](https://user-images.githubusercontent.com/53208493/86307696-59e8f180-bc52-11ea-9400-e84b691c06f0.png)   
- flavor 선택   
![Screenshot from 2020-07-02 10-52-01](https://user-images.githubusercontent.com/53208493/86307698-5a818800-bc52-11ea-9c7f-1dda6760fd78.png)
- 네트워크 선택   
![Screenshot from 2020-07-02 10-52-08](https://user-images.githubusercontent.com/53208493/86307700-5a818800-bc52-11ea-9c53-0563ab54f681.png)
- 보안 그룹 선택   
![Screenshot from 2020-07-02 10-52-17](https://user-images.githubusercontent.com/53208493/86307702-5b1a1e80-bc52-11ea-80a2-d335930b2963.png)
- 키 페어 선택  
![Screenshot from 2020-07-02 10-52-24](https://user-images.githubusercontent.com/53208493/86307704-5bb2b500-bc52-11ea-88fc-cf7ce4e8447a.png)
- 유동 IP 연결   
![Screenshot from 2020-07-02 10-55-03](https://user-images.githubusercontent.com/53208493/86307761-8e5cad80-bc52-11ea-9cb8-c66c4e02150b.png)
- 키 생성 시 다운로드 받은 키 파일에 대한 권한 수정   
  - 소유자만이 읽고 쓰기가 가능한 상태여야 키 파일 사용 가능
![Screenshot from 2020-07-02 10-59-39](https://user-images.githubusercontent.com/53208493/86308033-2fe3ff00-bc53-11ea-9f5c-2eb867ba5e46.png)
- 키 파일을 이용한 ssh 인증    
![Screenshot from 2020-07-02 11-00-34](https://user-images.githubusercontent.com/53208493/86308085-4a1ddd00-bc53-11ea-8753-352c7d16e1d5.png)

### (8) Volume
- Project -> Volume -> Volumes
![Screenshot from 2020-07-02 11-02-54](https://user-images.githubusercontent.com/53208493/86308299-c57f8e80-bc53-11ea-91d8-5f6df5126910.png)
![Screenshot from 2020-07-02 11-03-30](https://user-images.githubusercontent.com/53208493/86308303-c6b0bb80-bc53-11ea-83e7-d66e2cf12b4f.png)
![Screenshot from 2020-07-02 11-03-57](https://user-images.githubusercontent.com/53208493/86308305-c6b0bb80-bc53-11ea-9d6d-196969545419.png)
![Screenshot from 2020-07-02 11-07-11](https://user-images.githubusercontent.com/53208493/86308468-40e14000-bc54-11ea-9322-dba2bcf5e9a5.png)   
![Screenshot from 2020-07-02 11-04-54](https://user-images.githubusercontent.com/53208493/86308407-198a7300-bc54-11ea-9f4c-fe3943929e9d.png)


### (9) Object Storage
- Project -> Object Store -> Containers
![Screenshot from 2020-07-02 11-09-50](https://user-images.githubusercontent.com/53208493/86308581-aaf9e500-bc54-11ea-8917-0066112da363.png)
![Screenshot from 2020-07-02 11-10-02](https://user-images.githubusercontent.com/53208493/86308583-ac2b1200-bc54-11ea-9db2-b16b30c84e5c.png)
![Screenshot from 2020-07-02 11-10-19](https://user-images.githubusercontent.com/53208493/86308587-acc3a880-bc54-11ea-99b2-8a9c20d18c03.png)
![Screenshot from 2020-07-02 11-10-26](https://user-images.githubusercontent.com/53208493/86308588-ad5c3f00-bc54-11ea-8ef2-3caac6c3d9e0.png)

### (10) Load Balancer
Load Balancer 구성을 위해 새로운 instance 생성
### Volume
- 이미지가 연결된 volume 생성   
![Screenshot from 2020-07-02 11-12-32](https://user-images.githubusercontent.com/53208493/86308895-3d01ed80-bc55-11ea-8d72-07fd5a1d2666.png)

### Instance 
- 이미지 연결한 volume으로 새 instance 생성   
![Screenshot from 2020-07-02 11-17-54](https://user-images.githubusercontent.com/53208493/86309193-1abc9f80-bc56-11ea-8f98-f27edd52c00e.png)
![Screenshot from 2020-07-02 11-18-00](https://user-images.githubusercontent.com/53208493/86309195-1b553600-bc56-11ea-83ea-61393578b136.png)
![Screenshot from 2020-07-02 11-18-09](https://user-images.githubusercontent.com/53208493/86309197-1bedcc80-bc56-11ea-9373-4dc148c53e79.png)
![Screenshot from 2020-07-02 11-18-16](https://user-images.githubusercontent.com/53208493/86309200-1c866300-bc56-11ea-8fae-e41e1ca52b69.png)
![Screenshot from 2020-07-02 11-18-56](https://user-images.githubusercontent.com/53208493/86309201-1d1ef980-bc56-11ea-88a4-23da3e04fadc.png)
![Screenshot from 2020-07-02 11-19-02](https://user-images.githubusercontent.com/53208493/86309202-1d1ef980-bc56-11ea-851e-c6ba2fa852da.png)
![Screenshot from 2020-07-02 11-19-43](https://user-images.githubusercontent.com/53208493/86309203-1db79000-bc56-11ea-8105-eb9eb2d2005b.png)

### Floating IP 연결
- floating ip 생성
- 새로 생성한 instance에 floating ip 연결
![Screenshot from 2020-07-02 11-23-02](https://user-images.githubusercontent.com/53208493/86309389-961e5100-bc56-11ea-8f00-545cf11b4a6b.png)
![Screenshot from 2020-07-02 11-23-24](https://user-images.githubusercontent.com/53208493/86309392-974f7e00-bc56-11ea-8359-f8de2c748603.png)

### Load Balancer
- load balancer 생성   
![Screenshot from 2020-07-02 11-44-58](https://user-images.githubusercontent.com/53208493/86311001-5bb6b300-bc5a-11ea-8491-20f6d24ac2fb.png)
![Screenshot from 2020-07-02 11-45-16](https://user-images.githubusercontent.com/53208493/86311004-5c4f4980-bc5a-11ea-9ace-ee518bd4c610.png)
![Screenshot from 2020-07-02 11-45-30](https://user-images.githubusercontent.com/53208493/86311005-5ce7e000-bc5a-11ea-9c27-6d32d7c7e8cf.png)
![Screenshot from 2020-07-02 11-45-52](https://user-images.githubusercontent.com/53208493/86311006-5ce7e000-bc5a-11ea-8201-a4c67cb842b5.png)
![Screenshot from 2020-07-02 11-46-10](https://user-images.githubusercontent.com/53208493/86311008-5d807680-bc5a-11ea-8919-6fd801a47cb2.png)
![Screenshot from 2020-07-02 11-47-40](https://user-images.githubusercontent.com/53208493/86311010-5e190d00-bc5a-11ea-8f32-baf34c3b0d25.png)

- load balancer에 floating ip 연결   
![Screenshot from 2020-07-02 11-49-57](https://user-images.githubusercontent.com/53208493/86311012-5e190d00-bc5a-11ea-9c33-23159674d1cb.png)
![Screenshot from 2020-07-02 11-50-08](https://user-images.githubusercontent.com/53208493/86311013-5eb1a380-bc5a-11ea-9c2a-9b13a7cfe057.png)
![Screenshot from 2020-07-02 11-50-28](https://user-images.githubusercontent.com/53208493/86311015-5eb1a380-bc5a-11ea-8e6c-5b0d36a8cb57.png)

- 연결한 floating ip로 웹 브라우저 접속 시 web1과 web2의 내용이 나타나는 것을 확인할 수 있음   
  - 테스트를 위해 web1과 web2의 컨텐츠 내용을 각각 다르게 지정함   
![Screenshot from 2020-07-02 11-54-52](https://user-images.githubusercontent.com/53208493/86311280-0038f500-bc5b-11ea-86c9-d918f0c972f6.png)
![Screenshot from 2020-07-02 11-54-58](https://user-images.githubusercontent.com/53208493/86311282-00d18b80-bc5b-11ea-9a67-b39292620594.png)
