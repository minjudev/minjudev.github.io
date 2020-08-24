### PackStack이란?
- Redhat 계열의 OpenStack 자동화 설치 툴

</br>

### PackStack 사양
- VM: CentOS 7
- RAM: 10G
- CPU: 4(Copy Host CPU Configuration 설정 필요)
- NIC: nat / priv(virtio)

</br>

### PackStack 설치
- PackStack 설치 명령어
```bash
sudo yum update -y
sudo yum install -y centos-release-openstack-train
sudo yum update -y
# packstack 설치
sudo yum install -y openstack-packstack
# 응답 파일 만들기
packstack --gen-answer-file=answers.txt
```

- 웹 브라우저에서 PackStack을 설치한 시스템의 아이피 주소로 접속
  - ID: admin
  - PASSWD: `grep -i keystone_admin_pw answers.txt`로 확인
