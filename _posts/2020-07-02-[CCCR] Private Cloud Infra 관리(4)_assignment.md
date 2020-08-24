## Assignment 1  
- 새로운 내부 네트워크 생성 후 db라는 이름의 Instance 만들기

### 절차
1. 새로운 내부 네트워크 생성
2. 새 내부 네트워크와 라우터 연결하기
3. 보안그룹 추가(web이라는 보안그룹에서만 mysql 3306/tcp 추가)
4. 새로운 내부 네트워크로 db라는 Instance 만들기

### 내용
1. 새로운 내부 네트워크 생성   
![Screenshot from 2020-07-02 14-23-21](https://user-images.githubusercontent.com/53208493/86319557-bb6b8900-bc6f-11ea-9ab0-0e137c714a50.png)
![Screenshot from 2020-07-02 14-23-36](https://user-images.githubusercontent.com/53208493/86319559-bc9cb600-bc6f-11ea-83c0-dbdb802fd246.png)
![Screenshot from 2020-07-02 14-23-46](https://user-images.githubusercontent.com/53208493/86319561-bc9cb600-bc6f-11ea-87cb-f0b665773aa4.png)

2. 새롭게 생성한 내부 네트워크와 라우터 연결하기   
![Screenshot from 2020-07-02 14-25-39](https://user-images.githubusercontent.com/53208493/86319647-efdf4500-bc6f-11ea-8886-f1321212093e.png)

3. 보안그룹 추가(web이라는 보안그룹에서만 mysql 추가)   
![Screenshot from 2020-07-02 14-26-33](https://user-images.githubusercontent.com/53208493/86319687-100f0400-bc70-11ea-9eca-e376ad4d5981.png)

4. 새로운 내부 네트워크로 db라는 Instance 만들기
![Screenshot from 2020-07-02 14-28-00](https://user-images.githubusercontent.com/53208493/86319837-5ebc9e00-bc70-11ea-92dd-94b46b76119a.png)
![Screenshot from 2020-07-02 14-28-09](https://user-images.githubusercontent.com/53208493/86319839-5f553480-bc70-11ea-8119-462afd5e73f6.png)
![Screenshot from 2020-07-02 14-28-17](https://user-images.githubusercontent.com/53208493/86319841-5fedcb00-bc70-11ea-9afe-0e96ad831f94.png)
![Screenshot from 2020-07-02 14-28-26](https://user-images.githubusercontent.com/53208493/86319843-60866180-bc70-11ea-89f3-e95275600391.png)
![Screenshot from 2020-07-02 14-28-34](https://user-images.githubusercontent.com/53208493/86319845-611ef800-bc70-11ea-96ba-3be204f9ccf9.png)
![Screenshot from 2020-07-02 14-28-39](https://user-images.githubusercontent.com/53208493/86319847-611ef800-bc70-11ea-9669-e04f63730969.png)

5. db 인스턴스에 floating ip 할당
![Screenshot from 2020-07-02 16-23-41](https://user-images.githubusercontent.com/53208493/86329060-9df2eb00-bc80-11ea-9b7a-3104d9895925.png)

## Assignment 2
- 새로운 인스턴스 'storage'의 볼륨을 이용해 web1과 web2가 사용하는 공유 디렉토리 만들기

### 절차
1. storage라는 이름의 새 인스턴스 만들기
2. 새 인스턴스에 volume(webvol - 5G) 연결하기
3. 해당 볼륨 xfs로 포맷하기
4. /web에 마운트하기
5. storage 인스턴스에서 /web을 nfs 공유 디렉토리로 설정
6. web1과 web2 인스턴스에서 /var/www를 마운트 포인트로 설정
7. /var/www를 storage 인스턴스의 /web과 연결시키기

### 내용
1. storage라는 이름의 새 인스턴스 만들기
![Screenshot from 2020-07-02 16-43-17](https://user-images.githubusercontent.com/53208493/86330808-2bcfd580-bc83-11ea-988a-fb825c18bbb4.png)

2. 새 인스턴스에 volume(webvol - 5G) 연결하기
![Screenshot from 2020-07-02 16-44-25](https://user-images.githubusercontent.com/53208493/86330949-5cb00a80-bc83-11ea-8bec-6019e58d5cb7.png)
![Screenshot from 2020-07-02 17-55-50](https://user-images.githubusercontent.com/53208493/86338445-4e66ec00-bc8d-11ea-86aa-bbaaeb09bb77.png)

3. 해당 볼륨 xfs로 포맷하기
```
fdisk /dev/vdb
mkfs -t xfs /dev/vdb1
```
![Screenshot from 2020-07-02 17-58-48](https://user-images.githubusercontent.com/53208493/86338804-be757200-bc8d-11ea-808f-f0707df1281e.png)

4. /web에 마운트하기
```
mkdir /web
mount /dev/vdb1 /web
vi /etc/fstab
  UUID="0dfa459e-3bd8-48e5-b21d-87b158dd5810"   /web   xfs   defaults   0   0
mount -a
```
![Screenshot from 2020-07-02 18-08-53](https://user-images.githubusercontent.com/53208493/86339912-1f517a00-bc8f-11ea-812f-fcb1f6f72069.png)

5. storage 인스턴스에서 /web을 nfs 공유 디렉토리로 설정
```
yum -y install nfs-utils rpcbind
chmod 777 /web
vi /etc/exports
  /web   192.168.122.0/24(rw,sync,sec=sys)
exportfs -r
systemctl start nfs-server
systemctl enable nfs-server
yum -y install firewalld
systemctl start firewalld
systemctl enable firewalld
systemctl status firewalld
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload
```

6. web1과 web2 인스턴스에서 /var/www를 마운트 포인트로 설정
```
yum -y install nfs-utils
showmount -e 192.168.122.119
mount -o rw,sync,sec=sys 192.168.122.119:/web /var/www
vi /etc/fstab
  192.168.122.119:/web   /var/www   nfs   rw,sync,sec=sys
mount -a
```

7. /var/www를 storage 인스턴스의 /web과 연결시키기   
![Screenshot from 2020-07-02 18-27-21](https://user-images.githubusercontent.com/53208493/86341834-c800d900-bc91-11ea-835e-e57a128762f9.png)
![Screenshot from 2020-07-02 18-27-44](https://user-images.githubusercontent.com/53208493/86341841-c8996f80-bc91-11ea-8311-a54624053906.png)

## Assignment 3
- Bootable Volume의 데이터가 영구적으로 남는지 확인하기

### 절차
1. Bootable Volume으로 가상머신 VM1 생성
2. cat /root/filea.txt 실행하면 "like and subscribe my youtube channel" 나오게 하기
3. VM1 삭제한 후 남은 Bootable Volume으로 VM2 만들기
4. 해당 가상머신에서 /root/filea.txt 확인하기

### 내용
1. Bootable Volume으로 가상머신 VM1 생성
2. cat /root/filea.txt 실행하면 "like and subscribe my youtube channel" 나오게 하기
![Screenshot from 2020-07-02 18-50-22](https://user-images.githubusercontent.com/53208493/86344184-eae0bc80-bc94-11ea-8ce9-59a91634d017.png)

3. VM1 삭제한 후 남은 Bootable Volume으로 VM2 만들기
![Screenshot from 2020-07-02 18-53-13](https://user-images.githubusercontent.com/53208493/86344493-4f9c1700-bc95-11ea-8d15-6f8ab13a5463.png)

4. 해당 가상머신에서 /root/filea.txt 확인하기   
![Screenshot from 2020-07-02 18-54-03](https://user-images.githubusercontent.com/53208493/86344629-75292080-bc95-11ea-8a26-1764d768d30a.png)
