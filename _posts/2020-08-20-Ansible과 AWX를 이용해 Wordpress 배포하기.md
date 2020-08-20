# 1. 프로젝트의 목표

Ansible의 playbook 배포를 통해 로드밸런서로 접속하면 두 대의 데이터베이스가 연결되어 있는 워드프레

스 웹 서버에 접속할 수 있는 고가용성 시스템을 구성한 후, 해당 고가용성 시스템을 AWX에 올려보는 것을 

목표로 본 프로젝트를 진행하였다.

# 2. 시스템 구성

## 1) 시스템 구성

본 프로젝트를 위해 시스템은 총 6개의 노드로 구성하였다. 

6개의 노드는 하나의 제어 노드와 5개의 관리 노드로 나누어진다.  

- controller node
- manage node(lb, web1, web2, database1, database2)

## 2) 네트워크 구조

하나의 제어 노드 및 5개의 관리 노드의 네트워크 구조는 다음과 같다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/afc8805a-d9a4-4156-bc32-50d00dc5885a/Screenshot_from_2020-08-17_14-24-49.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/afc8805a-d9a4-4156-bc32-50d00dc5885a/Screenshot_from_2020-08-17_14-24-49.png)

## 3) 시스템 사양

- controller node
    - OS: CentOS 7
    - CPU: 2
    - RAM: 2048MB
- manage node
    - OS: CentOS 7
    - CPU: 2
    - RAM: 2048MB

# 3. AWX 설치

- AWX 설치를 위한 명령어

    AWX 설치는 controller 노드에서 진행한다.

    ```yaml
    # epel-release를 설치한다.
    yum install epel-release

    # AWX를 설치하기 위해 패키지를 설치한다.
    yum install ansible make git nodejs npm python3 python3-pip

    # 최신 Ansible 버전을 사용하기 위해 EPEL Repository를 추가한다.
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

    # Docker-ce 관련 패키지를 설치한다.
    yum install docker-ce docker-ce-cli containerd.io

    # Docker 서비스를 활성화하고 시작한다.
    systemctl enable --now docker

    # AWX Git 레포지토리를 적절한 디렉토리에 클론한다. 
    git clone -b 14.0.0 https://github.com/ansible/awx.git

    # pip3 모듈을 이용해 Docker 패키지를 설치한다.
    pip3 install docker docker-compose selinux

    cd awx/installer

    # awx/installer/inventory 파일의 140번째 줄 주석을 해제한다.
    vi inventory
    140 project_data_dir=/var/lib/awx/projects

    # install.yml 플레이북을 실행시켜 AWX를 설치한다.
    ansible-playbook -i inventory installer.yml -b
    ```

- 웹 브라우저를 통해 AWX에 접속한다.

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7f673698-5c7b-4538-b395-6d4589117542/Screenshot_from_2020-08-17_14-41-04.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7f673698-5c7b-4538-b395-6d4589117542/Screenshot_from_2020-08-17_14-41-04.png)

    - USERNAME: admin
    - PASSWORD: password

# 4. Wordpress 구조

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8f05194c-fe76-4539-a39e-7775ac1b6507/Untitled_Diagram(1).svg](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8f05194c-fe76-4539-a39e-7775ac1b6507/Untitled_Diagram(1).svg)

외부에서 로드밸런서를 통해 웹 서버로 접근하면 웹 서비스를 이용할 수 있고,  두 개의 DB는 Galera 

module을 통해 서로 데이터를 공유하게 된다.

# 5. Playbook 설명

## 1) 인벤토리

```yaml
lb ansible_host=192.168.123.51 
web1 ansible_host=192.168.123.52
web2 ansible_host=192.168.123.53
database1 ansible_host=192.168.123.54
database2 ansible_host=192.168.123.55

[lb]
lb

[web]
web1
web2

[db]
database1
database2

[db-master]
database1

[db-slave]
database2

[db-nfs]
database1
```

- 로드밸런서는 'lb' 1대로 설정한다.
- 웹 서버는 'web' 그룹 내에 web1, web2로 설정한다.
- Galera module을 통해 서로 공유되는 DB 두 대는 'db-master'와 'db-slave'로 설정한다.
- NFS 서버는 'db-nfs'로 설정한다.

## 2) 플레이북

- 다음 플레이북은 모든 노드에 공통으로 적용된다.

```yaml
# 모든 노드에 epel-release package와 libesemanage-python package를 설치한다. 
# ansible을 설치한 시스템의 OS가 CentOS일 경우 다음 명령어를 실행한다.
- block:
  - name: Install epel-release
    yum: 
      name: epel-release 
      state: latest
  - name: Install libsemanage-python for seboolean
    yum: 
      name: libsemanage-python 
      state: latest
  when: ansible_distribution == 'CentOS'
# ansible을 설치한 시스템의 OS가 Ubuntu일 경우 다음 명령어를 실행한다.
- block:
  - name: Update apt cache
    apt:
      update_cache: yes
  when: ansible_distribution == 'Ubuntu'
```

- DB 서버 두 대에 MariaDB를 설치한 후 워드프레스 관련 설정을 진행한다.

```yaml
# MariaDB 설치를 위해 레포지토리를 추가한다.
- name: Add yum_repository for mariadb
  yum_repository: 
    name: MariaDB 
    baseurl: "{{ mariadb['repo']['baseurl'] }}"
    gpgkey: "{{ mariadb['repo']['gpgkey'] }}"
    gpgcheck: 1 
    description: MariaDB
# MariaDB를 설치한다.
- name: Install mariadb
  yum: 
    name: MariaDB-server 
    enablerepo: MariaDB 
    state: latest
- name: Copy mariadb configuration
  template:
    src: my.cnf.j2
    dest: /etc/my.cnf.d/server.cnf
  notify:
  - Restart mariadb
# MariaDB 서비스를 시작한다.
- name: Start mariadb
  service: 
    name: mariadb 
    state: started 
    enabled: true
# MySQL-python을 설치한다.
- name: Install library for DB
  yum: 
    name: MySQL-python 
    state: latest
# root password를 설정한다.
- name: Set root password
  mysql_user: 
    login_user: root 
    login_password: '' 
    user: root 
    password: dkagh1. 
    state: present
# DB의 익명 사용자를 삭제한다.
- name: Delete anonymous user in DB
  mysql_user: 
    login_user: root 
    login_password: dkagh1. 
    name: '' 
    host_all: yes 
    state: absent
# DB의 test db를 삭제한다.
- name: Delete test db in DB
  mysql_db: 
    login_user: root 
    login_password: dkagh1. 
    db: test 
    state: absent
# 워드프레스에서 사용할 새로운 DB를 생성한다.
- name: Create DB for wordpress
  mysql_db: 
    login_user: root 
    login_password: dkagh1. 
    name: "{{ mariadb['wp']['name'] }}"
    state: present
# 워드프레스에서 사용할 새로운 사용자를 생성하고 권한을 설정한다.
- name: Create User for wordpress
  mysql_user: 
    login_user: root 
    login_password: dkagh1. 
    name: "{{ mariadb['wp']['user'] }}" 
    password: "{{ mariadb['wp']['pwd'] }}"
    priv: "{{ mariadb['wp']['priv'] }}" 
    host: "{{ mariadb['wp']['host'] }}" 
    state: present
# MariaDB와 관련된 방화벽을 열어준다.
- name: Open mariadb port
  firewalld: 
    port: "{{ mariadb['port'] }}/tcp"
    permanent: yes 
    state: enabled 
    immediate: yes
# MariaDB와 관련된 seboolean을 활성화시킨다.
- name: Active seboolean for mysql
  seboolean: 
    name: mysql_connect_any 
    state: yes 
    persistent: yes
```

- DB 서버 한 대에 NFS 서버를 구축하고 워드프레스 설치를 위한 설정을 진행한다.

```yaml
# NFS 관련 패키지를 설치한다.
- name: Install nfs-utils
  yum:
    name: nfs-utils
    state: latest
# export할 디렉토리를 생성한다.
- name: Create a directory for nfs exports
  file:
    path: "{{ nfs['exports']['directory'] }}"
    state: directory
    mode: '0775'
- block:
  # 새로운 파티션을 생성한다.
  - name: Create a new primary partition for LVM
    parted:
      device: "{{ nfs['block']['device'] }}"
      number: 1
      flags: [ lvm ]
      state: present
      part_start: 5GiB
  # 파일시스템을 생성한다.
  - name: Create a filesystem
    filesystem:
      fstype: "{{ nfs['block']['fs_type'] }}"
      dev: "{{ nfs['block']['device'] }}1"
  # export 하기 위해 만든 디렉토리로 마운트한다.
  - name: mount /dev/vdb1 on /wordpress
    mount:
      path: "{{ nfs['exports']['directory'] }}"
      src: "{{ nfs['block']['device'] }}1"
      fstype: "{{ nfs['block']['fs_type'] }}"
      state: mounted
  when: nfs['block']['device'] is defined
- name: Create exports to webserver
  template:
    src: exports.j2
    dest: /etc/exports
  notify:
  - Re-export all directories
# wordpress url을 변수로 설정한다.
- name: Set wordpress url
  set_fact:
    wp_url: "https://ko.wordpress.org/wordpress-{{ wordpress['source']['version'] }}-{{ wordpress['source']['language'] }}.tar.gz"
    wp_filename: "wordpress-{{ wordpress['source']['version'] }}-{{ wordpress['source']['language'] }}.tar.gz"
# 워드프레스를 다운받는다.
- name: Download wordpress sources
  get_url: 
    url: "{{ wp_url }}"
    dest: "/tmp/{{ wp_filename }}"
# 다운받은 워드프레스 파일의 압축을 해제한다.
- name: Unarchive wordpress archive
  unarchive: 
    src: "/tmp/{{ wp_filename }}"
    dest: "{{ nfs['exports']['directory'] }}"
    remote_src: yes 
    owner: root 
    group: root
# 워드프레스 관련 설정을 수정하기 위해 wp-config-sample.php 파일을 복사한다.
- name: Copy wp-config.php
  template:
    src: wp-config.php.j2
    dest: "{{ nfs['exports']['directory'] }}/wordpress/wp-config.php"
# NFS 서비스를 활성화시킨다.
- name: Start nfs service
  service:
    name: nfs
    enabled: true
    state: started
# NFS 관련 방화벽을 열어준다.
- name: Allow port for nfs, rpc-bind, mountd
  firewalld:
    service: "{{ item }}"
    permanent: yes
    state: enabled
    immediate: yes
  with_items: "{{ firewall_nfs_lists }}"
```

- 웹서버 두 대에 NFS 클라이언트 설정을 진행한다.

```yaml
# NFS 패키지를 설치한다.
- name: Install nfs-utils
  yum:
    name: nfs-utils
    state: latest
# httpd를 설치한다
- name: Install httpd
  yum: 
    name: httpd 
    state: latest
- name: Copy configuration
  template:
    src: apache.conf.j2
    dest: /etc/httpd/conf/httpd.conf
  notify:
  - Restart httpd
- name: Delegate collecting facts for mariadb
  setup:
  delegate_to: node4
- name: Set facts for mariadb private ip
  set_fact:
    db_private_ip: "{{ ansible_eth1.ipv4.address }}"
# 공유 디렉토리를 /var/www/html에 마운트한다.
- name: Mount nfs share
  mount:
    path: /var/www/html
    src: "{{ db_private_ip }}:{{ nfs['exports']['directory'] }}"
    fstype: nfs
    state: mounted
# http 방화벽을 열어준다. 
- name: Open http port
  firewalld: 
    port: "{{ apache['port'] }}/tcp"
    permanent: yes 
    state: enabled 
    immediate: yes
# httpd와 관련된 seboolean을 활성화시킨다.
- name: Active seboolean for httpd
  seboolean:
    name: "{{ item }}"
    state: yes
    persistent: yes
  with_items: "{{ sebool_httpd_lists }}"
# php를 설치한다.
- name: Install remi-release-7 for php74
  yum: 
    name: "{{ php['repo']['pkg'] }}"
    state: latest
- name: Install php and php-mysql 
  yum: 
    name: php,php-mysql 
    enablerepo: remi-php74 
    state: latest
# apache 서비스를 시작한다.
- name: Start httpd
  service: 
    name: httpd 
    state: started 
    enabled: true
```

- 로드밸런서 한 대에 haproxy를 설치한다.

```yaml
- name: Deploy to CentOS
  block:
  # haproxy를 설치한다.
  - name: Install haproxy
    yum:
      name: haproxy
      state: latest
  # haproxy의 프론트엔드 포트를 설정한다.
  - name: Open http port
    firewalld: 
      port: "{{ haproxy['frontend']['port'] }}/tcp" 
      permanent: yes 
      state: enabled 
      immediate: yes    
  # http와 관련된 seboolean을 활성화시킨다.
  - name: Active seboolean for httpd  
    seboolean: 
      name: haproxy_connect_any
      state: yes 
      persistent: yes
  - name: Set facts for haproxy public ip
    set_fact:
      haproxy_public_ip: "{{ ansible_eth0.ipv4.address }}"
  - name: Delegate collecting facts for wordpress1
    setup:
    delegate_to: node2
  - name: Set facts for wordpress1 private ip
    set_fact:
      wordpress1_private_ip: "{{ ansible_eth1.ipv4.address }}"
  - name: Delegate collecting facts for wordpress2
    setup:
    delegate_to: node3
  - name: Set facts for wordpress2 private ip
    set_fact:
      wordpress2_private_ip: "{{ ansible_eth1.ipv4.address }}"
  - name: Copy haproxy configuration
    template:
      src: centos/haproxy.cfg.j2
      dest: /etc/haproxy/haproxy.cfg
    notify:
    - Restart haproxy service
  when: ansible_distribution == "CentOS"
# haproxy 서비스를 시작한다.
- name: Start haproxy service
  service:
    name: haproxy
    enabled: true
    state: started
```

## 3) 변수

- mariadb

```yaml
mariadb:
  # mariadb 설치를 위해 추가할 레포지토리의 주소를 변수로 설정해준다.
  repo:
    baseurl: http://mirror.yongbok.net/mariadb/yum/10.5/centos7-amd64
    gpgkey: http://mirror.yongbok.net/mariadb/yum/RPM-GPG-KEY-MariaDB 
  # mariadb에서 지정할 워드프레스 관련 설정을 변수로 만들어주었다. 
  wp:
    name: wordpress_db
    user: admin
    pwd: dkagh1.
    priv: wordpress_db.*:ALL,GRANT
    host: '192.168.123.%'
  port: 3306
```

- nfs

```yaml
nfs:
  # nfs 서버에서 export할 디렉토리와 관련된 정보들을 변수로 설정해준다.
  exports:
    directory: /wordpress
    subnet: 192.168.123.0/24
    options: rw,sync,no_root_squash
  # nfs 서버에서 새롭게 생성할 블록 장치의 파일시스템 타입을 설정해준다.
  block:
    #device: /dev/vdb
    fs_type: ext4
```

- apache

```yaml
# apache 웹 서버에서 허용할 seboolean을 반복문으로 지정해주기 위해 다음과 같이 설정한다.
sebool_httpd_lists:
  - httpd_can_network_connect
  - httpd_can_network_connect_db
  - httpd_use_nfs

# apache 웹 서버에서 허용할 포트를 지정한다.
apache:
  port: 80

# php 설치를 위해 repo 주소를 지정한다.
php:
  repo: 
    pkg: "https://rpms.remirepo.net/enterprise/remi-release-7.rpm "
```

- haproxy

```yaml
# haproxy 서버에서 내부로 접근하기 위해 다음과 같이 포트를 지정한다.
haproxy:
  frontend:
    port: 80
  backend:
    name: wordpress
    balance_type: roundrobin
    wordpress1:
      port: 80
    wordpress2:
      port: 80
```

# 6. AWX 구성

## 1) 자격증명

자격증명은 제어 노드에서 관리 노드에 접근할 때 사용하는 인증 값이다.

다음과 같은 순서로 자격증명을 진행할 수 있다.

1. AWX 대시보드의 왼쪽 메뉴 중 Credential로 들어온 후 [+] 버튼(Create a new credential)을 클릭한다.

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a2c734b4-af89-4658-8b6d-b7fb78394017/Screenshot_from_2020-08-19_18-09-26.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a2c734b4-af89-4658-8b6d-b7fb78394017/Screenshot_from_2020-08-19_18-09-26.png)

2. Name, Organization, Credential Type, Username을 순서대로 입력한다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5d91366d-eef6-48d8-8853-6ec2edb3e8d4/Screenshot_from_2020-08-19_17-52-53.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5d91366d-eef6-48d8-8853-6ec2edb3e8d4/Screenshot_from_2020-08-19_17-52-53.png)

3. 컨트롤 머신에서 ssh private key를 확인한 후 SSH PRIVATE KEY 칸에 입력한다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/42ba3555-ea34-4118-9d92-e26379a776c3/Screenshot_from_2020-08-19_17-52-27.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/42ba3555-ea34-4118-9d92-e26379a776c3/Screenshot_from_2020-08-19_17-52-27.png)

4. NAME 칸에 입력한 이름(new student)과 같게 새로운 자격증명이 생성된다. 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c68d79a0-bd03-4d96-ad18-9fc945749491/Screenshot_from_2020-08-19_17-54-31.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c68d79a0-bd03-4d96-ad18-9fc945749491/Screenshot_from_2020-08-19_17-54-31.png)

## 2) 프로젝트

AWX의 프로젝트는 플레이북의 모음이다.

다음과 같은 순서로 새로운 프로젝트를 생성할 수 있다.

1. AWX 대시보드의 왼쪽 메뉴 중 Projects로 들어온 후 [+] 버튼(Create a new credential)을 클릭한다.

    ![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a8eb076b-ea1c-432d-8fd8-fe7c8fb35946/Screenshot_from_2020-08-19_18-11-46.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a8eb076b-ea1c-432d-8fd8-fe7c8fb35946/Screenshot_from_2020-08-19_18-11-46.png)

2. 새로운 프로젝트를 생성하기 위해 Name, Organization, SCM Type, Playbook Directory를 순서대로 입력한다. 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/06a3f0db-c83d-4e1c-b5ee-aeee0cb6fd4c/Screenshot_from_2020-08-19_18-06-14.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/06a3f0db-c83d-4e1c-b5ee-aeee0cb6fd4c/Screenshot_from_2020-08-19_18-06-14.png)

Project Base Path는 플레이북이 존재하는 디렉토리 경로로 Project Base Path를 지정하기 위해서 

tower가 설치된 머신의 /var/lib/awx/project에 디렉토리를 생성해야한다.

/var/lib/awx/project 밑에 /test_project 디렉토리를 생성하였고 다음과 같이 지정해주었다.

3. 새로운 프로젝트가 생성되었다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1e7a8e57-6e6b-404b-bfb4-eb8de24c9038/Screenshot_from_2020-08-19_18-16-24.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1e7a8e57-6e6b-404b-bfb4-eb8de24c9038/Screenshot_from_2020-08-19_18-16-24.png)

## 3) 인벤토리

AWX의 인벤토리는 Ansible의 인벤토리와 같고, AWX에서는 인벤토리를 그래픽으로 관리할 수 있다는 

장점이 있다. Ansible의 컨트롤 머신에서 생성한 인벤토리 파일의 내용과 동일하게 AWX에서도 다음과 같

이 그룹과 호스트를 지정해주었다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/993e7f43-12b1-48f6-a532-6c705ef95f88/Screenshot_from_2020-08-19_18-17-27.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/993e7f43-12b1-48f6-a532-6c705ef95f88/Screenshot_from_2020-08-19_18-17-27.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/344432d9-bc0f-4189-bb6f-47c2185cca1b/Screenshot_from_2020-08-19_18-17-55.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/344432d9-bc0f-4189-bb6f-47c2185cca1b/Screenshot_from_2020-08-19_18-17-55.png)

## 4) 템플릿

# 7. 실행 결과

본 프로젝트는 미완성된 프로젝트로 보완해야할 점이 많이 남아있다.

보완이 필요한 사항은 첫번째, AWX에서 자격증명, 프로젝트, 인벤토리 생성은 완료했지만 컨트롤 머신에서 

제작한 role 디렉토리와 관련된 템플릿은 올려보지 못했다는 점이다.

두번째, 컨트롤 머신에서 작성한 yaml, 템플릿, 변수 파일은 강사님의 코드를 대부분 참고하여 작성한 파일

들로 스스로 해당 파일의 내용을 작성해볼 수 있도록 ansible의 활용과 관련된 여러 주제들을 다시 복습해야

할 필요가 있다.
