## 실습 1
- OpenStack에서 Heat 서비스를 이용해 유동 IP가 연결된 가상머신 만들기
![Untitled Diagram (1)](https://user-images.githubusercontent.com/53208493/86601355-2a344380-bfdc-11ea-8c15-73e16970a460.png)

### 내용
1. Heat 설치(controller node에서 진행)
```
yum -y install openstack-heat-ui
systemctl restart httpd
```
2. 각 resource에 대한 설정을 수정하기 전 연결부터 모두 진행  
![Screenshot from 2020-07-13 10-00-16](https://user-images.githubusercontent.com/53208493/87261071-b91af180-c4ef-11ea-82bc-f3b23234b5fa.png)

3. 네트워크 설정   
![Screenshot from 2020-07-09 19-30-40](https://user-images.githubusercontent.com/53208493/87029369-06ab0c00-c21b-11ea-91cb-5a7fe083a3e5.png)

4. 서브넷 설정   
![Screenshot from 2020-07-09 19-31-12](https://user-images.githubusercontent.com/53208493/87029373-0743a280-c21b-11ea-9534-302e5a400218.png)
![Screenshot from 2020-07-09 19-31-18](https://user-images.githubusercontent.com/53208493/87029376-07dc3900-c21b-11ea-8f1e-eb5ca615fab4.png)

5. 포트 설정   
![Screenshot from 2020-07-09 19-35-59](https://user-images.githubusercontent.com/53208493/87029769-8fc24300-c21b-11ea-9b83-06eff65dfd3f.png)

6. 라우터 설정   
![Screenshot from 2020-07-13 11-06-58](https://user-images.githubusercontent.com/53208493/87263735-03ed3700-c4f9-11ea-8d2e-021bfa9da542.png)

7. 유동 아이피 설정  
- 유동 아이피를 만들려면 **포트가 사용하는 서브넷**이 **라우터**에 연결되어야 함
![Screenshot from 2020-07-13 14-40-59](https://user-images.githubusercontent.com/53208493/87274538-ee870580-c516-11ea-9f55-a84475b878f1.png)

8. 서버 설정   
![Screenshot from 2020-07-13 14-41-48](https://user-images.githubusercontent.com/53208493/87274580-08284d00-c517-11ea-8608-a16c1ab2595b.png)
![Screenshot from 2020-07-13 14-41-55](https://user-images.githubusercontent.com/53208493/87274596-0eb6c480-c517-11ea-9216-15320f48ebc0.png)

9. volume attachment 설정   
- 가상 머신에 연결 시 볼륨 attachment 통해서 연결되어야 함
![Screenshot from 2020-07-13 14-42-48](https://user-images.githubusercontent.com/53208493/87274696-58071400-c517-11ea-8fe5-6d3f61d4f4db.png)

10. volume 설정   
![Screenshot from 2020-07-13 14-42-55](https://user-images.githubusercontent.com/53208493/87274698-589faa80-c517-11ea-8828-9951b0c07f52.png)

- 모든 resource 설정 및 연결 완료   
![Screenshot from 2020-07-13 14-47-03](https://user-images.githubusercontent.com/53208493/87274881-d19f0200-c517-11ea-8d4d-48b9a24c841e.png)

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









