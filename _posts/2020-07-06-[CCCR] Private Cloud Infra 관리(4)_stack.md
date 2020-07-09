## 실습 1
- OpenStack에서 Heat 서비스를 이용해 유동 IP가 연결된 가상머신 만들기
![Untitled Diagram (1)](https://user-images.githubusercontent.com/53208493/86601355-2a344380-bfdc-11ea-8c15-73e16970a460.png)

### 순서
1. Heat 설치(controller node에서 진행)
2. 네트워크와 서브넷 연결
3. 네트워크와 포트 연결
4. 서브넷과 포트 연결
5. 포트와 서버 연결
6. 라우터와 서브넷 연결
7. 포트와 유동 아이피 연결
8. 볼륨을 서버에 연결

### 내용
1. Heat 설치(controller node에서 진행)
```
yum -y install openstack-heat-ui
systemctl restart httpd
```

2. 네트워크와 서브넷 연결   
- 네트워크 설정   
![Screenshot from 2020-07-09 19-30-40](https://user-images.githubusercontent.com/53208493/87029369-06ab0c00-c21b-11ea-91cb-5a7fe083a3e5.png)
- 서브넷 설정   
![Screenshot from 2020-07-09 19-31-12](https://user-images.githubusercontent.com/53208493/87029373-0743a280-c21b-11ea-9534-302e5a400218.png)
![Screenshot from 2020-07-09 19-31-18](https://user-images.githubusercontent.com/53208493/87029376-07dc3900-c21b-11ea-8f1e-eb5ca615fab4.png)
![Screenshot from 2020-07-09 19-31-27](https://user-images.githubusercontent.com/53208493/87029377-0874cf80-c21b-11ea-8670-ab0756df2522.png)
3. 네트워크와 포트 연결  
- 포트 설정   
![Screenshot from 2020-07-09 19-35-59](https://user-images.githubusercontent.com/53208493/87029769-8fc24300-c21b-11ea-9b83-06eff65dfd3f.png)
![Screenshot from 2020-07-09 19-36-16](https://user-images.githubusercontent.com/53208493/87029772-90f37000-c21b-11ea-8cc0-d182d06575f7.png)


4. 서브넷과 포트 연결
5. 포트와 서버 연결
6. 라우터와 서브넷 연결
7. 포트와 유동 아이피 연결
8. 볼륨을 서버에 연결

- 유동 아이피를 만들려면 **포트가 사용하는 서브넷**이 **라우터**에 연결되어있어야 한다.
- 라우터 만들기, 라우터와 서브넷 연결, 포트에 유동 아이피 연결
- 볼륨 만들기, 볼륨을 가상머신에 연결하기
- vm3에 원격 접속하기
- 라우터 만들 때 라우터1 로 진행하기

볼륨

- 가상 머신에 연결 시 볼륨 attachment 통해서 연결되어야함

네트워크

- 하나의 네트워크에 하나의 서브넷을 사용한다고 생각하기
- 서브넷을 하나 가지고 있다면 네트워크와 가상머신을 바로 연결
- 서브넷을 두 개 가지고 있다면 아이피 두 개가 생김

라우터

- 라우터를 **네트워크와 서브넷 중 어디에 연결하는지가 매우 중요**
- 라우터와 서브넷 연결하기

## 실습 2
- OpenStack에서 Heat 서비스를 이용해 2개의 인스턴스를 만든 후 로드밸런서 구성하기
![Untitled Diagram (2)](https://user-images.githubusercontent.com/53208493/86653792-6c7f7400-c020-11ea-8e06-0656dae5c96a.png)

### 순서
1. Template Generator를 이용해 2개의 웹 인스턴스 띄우기
    1) 네트워크와 서브넷 연결
    2) 네트워크와 포트 연결
    3) 서브넷과 포트 연결
    4) 네트워크와 웹 서버 2개 연결
    5) 라우터와 서브넷 연결
    6) 포트와 유동 아이피 연결
  
2. yaml 파일을 작성해 로드밸런서 구성하기


### 내용









