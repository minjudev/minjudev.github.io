## Assignment 1  
- OpenStack에서 wordpress 서비스 동작시키기

### 절차
1. web 인스턴스에 적용할 volume(web-vol - 1G) 생성
2. db 인스턴스에 적용할 volume(db-vol - 2G) 생성
3. 생성한 볼륨을 적용해 web 인스턴스 생성
4. 생성한 볼륨을 적용해 db 인스턴스 생성
5. 생성한 web 인스턴스에 적용한 web-vol을 /var/www로 마운트
6. 생성한 db 인스턴스에 적용한 db-vol을 /var/lib/mysql로 마운트
7. web 인스턴스에 http, php, wordpress 설치

### 내용
1. web 인스턴스에 적용할 volume(web-vol - 1G) 생성
2. db 인스턴스에 적용할 volume(db-vol - 2G) 생성
3. 생성한 볼륨을 적용해 web 인스턴스 생성
4. 생성한 볼륨을 적용해 db 인스턴스 생성
5. 생성한 web 인스턴스에 적용한 web-vol을 /var/www로 마운트
6. 생성한 db 인스턴스에 적용한 db-vol을 /var/lib/mysql로 마운트
7. web 인스턴스에 http, php, wordpress 설치
- selinux 끄기
```
setenforce 0
getenforce
```
- http 설치
```
yum -y install httpd
systemctl start httpd
systemctl enable httpd
systemctl status httpd
firewall-cmd --add-service=http --permanent
```
- php 설치(참고: https://rpms.remirepo.net/)
```
yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum -y install https://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum -y install yum-utils
yum-comfig-manager --enable remi-php74
yum -y install php php-mysql
systemctl restart httpd
```
- wordpress 설치
```
curl -o latest.tar.gz https://wordpress.org/latest.tar.gz
tar -xvzf latest.tar.gz -C /var/www/html
setfacl -Rm u:apache:rwX /var/www/html
```
8. db 인스턴스에 mariadb 설치(참고: https://mariadb.org/download/)
- selinux 끄기
```
setenforce 0
getenforce
```
- mariadb 설치
```
**1. http://yum.mariadb.org/ 에서 최신 버전 확인**
**2. 설치할 MariaDB 버전 입력**
vi /etc/yum.repos.d/MariaDB.repo
	[mariadb]
	name = MariaDB
	baseurl = http://yum.mariadb.org/10.5/centos7-amd64/
	gpgkey = https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
	gpgcheck = 1
yum repolist 해서 확인
**3. MariaDB 설치 및 버전 확인**
yum install -y MariaDB-server
mysql --version
**4. MariaDB 자동실행 등록 후 실행**
- 10.1.8 버전 이상일 경우 -
systemctl enable mariadb.service(mariadb)
systemctl start mariadb.service(mariadb)
firewall-cmd --add-service=mysql --permanent
**5. MariaDB 보안 설정(넘어가도 됨)**
mysql_secure_installation
	root의 비밀번호가 설정되지 않은 상태이므로 enter를 입력
	root의 비밀번호를 설정하기 위해 Y 입력
	나머지 4개 모두 Y 입력
	내용은 차레대로 대충 아래와 같다
	- 익명 사용자 삭제할 것인가
	- 원격으로 루트 로그인을 허용하지 않을 것인가.
	- 테스트 데이터베이스를 제거하고 액세스할 것인가.
	- 권한 테이블을 지금 다시로드할 것인가.
**6. Mariadb 재시작**
systemctl restart mariadb.service
**7. 계정 생성**
DB 콘솔 로그인
mysql -u root -p
	내부 계정 생성
	MariaDB [(none)]> "CREATE DATABASE wordpress_db"
	MariaDB [(none)]> "CREATE USER 'wp-admin'@'web서버 주소' IDENTIFIED BY 'dkagh1.'"
	MariaDB [(none)]> "GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp-admin'@'web서버 주소' IDENTIFIED BY 'dkagh1.'"
	MariaDB [(none)]> "FLUSH PRIVILEGES"
```
9. 웹 브라우저에서 'web 인스턴스 ip 주소'/wordpress로 접속하여 확인
