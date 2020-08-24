# Kubespray를 이용해 Ansible로 Kubernetes 설치하기
## all-in-one
- 설치 환경
  - Master Node 1대
  - CentOS 7
  - RAM: 3G
  - CPU: 2
  - Copy Host CPU Configuration
  
- 네트워크 설정
  - eth0(NAT)
    - IP: 192.168.122.21/24
    - GW: 192.168.122.1
    - DNS: 8.8.8.8

- Kubespray 설치 명령어
```bash
#1 master 버전 명시한 후 git clone하기
git clone --branch release-2.13 https://github.com/kubernetes-sigs/kubespray.git

#2 python 및 pip 설치
yum install python3 python3-pip

# 버전 설치 조건 명시된 파일
more kubespray/requirements.txt

#3 requirements.txt 로 의존성 파일 설치
sudo pip3 install -r requirements.txt

#4 inventory/sample 파일들을 inventory/mycluster에 복사
cp -rfp inventory/sample inventory/mycluster

#5 inventory 빌더로 Ansible inventory 파일 업데이트
declare -a IPS=(192.168.122.21)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

#6 inventory 파일 수정
vi kubespray/inventory/mycluster/inventory.ini

[all]
kube-node1 ansible_host=192.168.122.21 ip=192.168.122.21 ansible_connection=local
kube-node2 ansible_host=192.168.122.22 ip=192.168.122.22 ansible_user=student

[kube-master]
kube-node1

[etcd]
kube-node1

[kube-node]
kube-node1

[calico-rr]

[k8s-cluster:children]
kube-master
kube-node
calico-rr

#7 ssh 키 생성
ssh-keygen

#8 자신의 host로 ssh 키 복사
ssh-copy-id localhost

#9 NOPASSWD 설정
sudo visudo

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL
student        ALL=(ALL)       NOPASSWD: ALL

#10 Ansible Playbook으로 Kubespray 배포
# The option `--become` is required, as for example writing SSL keys in /etc/,
# installing packages and interacting with various systemd daemons.
# Without --become the playbook will fail to run!
ansible-playbook -i inventory/mycluster/hosts.yaml --become --become-user=root cluster.yml
```
</br>

다음과 같은 순서로 node1에 kubernetes를 설치한 후, `kubectl get ns` 명령어를 입력하면 네임스페이스의 목록이 출력된다.  
![Screenshot from 2020-08-21 16-47-51](https://user-images.githubusercontent.com/53208493/90866070-4fcfad80-e3ce-11ea-91f0-bfadc5e3279b.png)

</br>

## multi-node
- 설치 환경
  - Master Node 1대, Slave Node 1대
  - CentOS 7
  - RAM: 3G
  - CPU: 2
  - Copy Host CPU Configuration
  
- 네트워크 설정
  - Master Node
    - eth0(NAT)
      - IP: 192.168.122.21/24
      - GW: 192.168.122.1
      - DNS: 8.8.8.8
  - Slave Node
    - eth0(NAT)
      - IP: 192.168.122.22/24
      - GW: 192.168.122.1
      - DNS: 8.8.8.8

- Kubespray 설치 명령어
```bash
#1 master 버전 명시한 후 git clone하기
git clone --branch release-2.13 https://github.com/kubernetes-sigs/kubespray.git

#2 python 및 pip 설치
yum install python3 python3-pip

# 버전 설치 조건 명시된 파일
more kubespray/requirements.txt

#3 requirements.txt 로 의존성 파일 설치
sudo pip3 install -r requirements.txt

#4 inventory/sample 파일들을 inventory/mycluster에 복사
cp -rfp inventory/sample inventory/mycluster

#5 inventory 빌더로 Ansible inventory 파일 업데이트
declare -a IPS=(192.168.122.21,192.168.122.22)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

#6 inventory 파일 수정
vi kubespray/inventory/mycluster/inventory.ini

[all]
kube-node1 ansible_host=192.168.122.21 ip=192.168.122.21 ansible_user=student
kube-node2 ansible_host=192.168.122.22 ip=192.168.122.22 ansible_user=student

[kube-master]
kube-node1

[etcd]
kube-node1

[kube-node]
kube-node2

[calico-rr]

[k8s-cluster:children]
kube-master
kube-node
calico-rr

#7 ssh 키 생성
ssh-keygen

#8 자신의 host로 ssh 키 복사
ssh-copy-id localhost

#9 NOPASSWD 설정
sudo visudo

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL
student        ALL=(ALL)       NOPASSWD: ALL

#10 Ansible Playbook으로 Kubespray 배포
# The option `--become` is required, as for example writing SSL keys in /etc/,
# installing packages and interacting with various systemd daemons.
# Without --become the playbook will fail to run!
ansible-playbook -i inventory/mycluster/inventory.ini scale.yml --limit=kube-node2 -b -e ignore_assert_errors=yes -e etch_retries=10
```

하지만, 위 명령어(`ansible-playbook -i inventory/mycluster/inventory.ini scale.yml --limit=kube-node2 -b -e ignore_assert_errors=yes -e etch_retries=10`)를 실행하면 다음과 같은 오류가 발생한다.
![Screenshot from 2020-08-21 16-30-59](https://user-images.githubusercontent.com/53208493/90864563-dd5dce00-e3cb-11ea-8168-68fc08f29b66.png)



