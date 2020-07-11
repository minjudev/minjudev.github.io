# Auto Scaling
## 1. 로드 밸런서 구성 및 관리
- 로드 밸런서: 특정 인스턴스로 몰리는 부하를 분산하는 장치
- 오픈스택에서는 LBaaS(Load Balancer as a Servicee)로 제공됨

### 1) LBaaS 구조
- LBaaS: 로드 밸런서 기능을 제공하는 서비스, 네트워크 트래픽을 분산하여 처리
- LBaaS 구성 시 필요한 요소
  - Load Balancer: 포트를 가지고 있으며, 서브넷에서 IP가 할당됨
  - Listener: 포트에서 대기하고 있는 요청
  - Pool: 로드 밸런서에 소속될 머신의 범위
  - Pool Memeber: 로드 밸런서에 소속될 머신들
  - Health Monitor: 멤버의 상태를 모니터링하는 요소

### 2) LBaaS 생성
#### (1) 웹 서버 생성
- 로드 밸런서를 생성하고 테스트하기 위해 http 서비스가 실행 중인 인스턴스 Server 1과 Server 2를 생성
- security-httpd 보안 그룹은 새롭게 생성하여 http 허용 규칙 추가해주기   
|속성|값|
|:-----:|:-----:|
|이름|Server 1, Server 2|
|Image|centos7|
|Flavor|m2.small|
|Key pair|key2|
|Security group|security-httpd|
|Network|Internal2|
|Subnet|Internal2-subnet|

```
// httpd 생성을 위한 shell script
vi configure-httpd
  #!/bin/bash
  yum -y install httpd
  systemctl start httpd
  systemctl enable httpd
  echo "Hello World!" > /var/www/html/index.html
  
// 두 개의 웹 서버 생성을 위한 openstack command  
openstack server create --image centos7 --flavor m2.small --network Internal2 --secuiry-group security-httpd --key-name key2 --user-data configure-httpd Server1
openstack server create --image centos7 --flavor m2.small --network Internal2 --secuiry-group security-httpd --key-name key2 --user-data configure-httpd Server2
```

#### (2) 로드 밸런서 생성
- 로드 밸런서 생성을 위해선 서브넷을 입력해야 함
- <VIP_SUBNET>에는 로드 밸런서에서 사용하는 서브넷을 입력해야 하며, 이 서브넷은 멤버가 사용하는 서브넷과 동일해야 함
```
openstack loadbalancer create --name lb1 --vip-subnet-id Internal2-subnet
```

#### (3) 리스너 생성
- 리스너 생성을 위해 필요한 옵션
|옵션|설명|
|:-----:|:-----:|
|--name|리스너의 이름|
|--protocol|리스너 트래픽의 프로토콜|
|--protocol-port|리스터 트래픽의 프로토콜 포트|

```
openstack loadbalancer listener create --name listener1 --protocol HTTP --protocol-port 80 lb1
```

#### (4) 풀 생성
- 리스너 생성을 위해 필요한 옵션
|옵션|설명|
|:---:|:---:|
|--name|풀의 이름|
|--lb-algorithm|풀의 로드 밸런싱 알고리즘|
|--listener|풀의 리스너|
|--protocol|풀의 프로토콜|

- 로드 밸런싱 알고리즘
|알고리즘|설명|
|:---:|:---:|
|ROUND_ROBIN|패킷을 순차적으로 포워딩|
|LEAST_CONNECTIONS|가장 연결이 적은 곳으로 포워딩|
|SOURCE_IP|특정 IP 주소에서 들어오는 패킷에 대해 포워딩|

```
openstack loadbalancer pool create --name pool1 --lb-algorithm ROUND_ROBIN --listener listener1 --protocol HTTP
```

#### (5) 풀 멤버 생성
- 풀 멤버 생성을 위해 필요한 옵션
|옵션|설명|
|:-----:|:-----:|
|--subnet-id|멤버의 서브넷|
|--address|멤버의 IP 주소|
|--protocol-port|멤버가 로드 밸런싱할 트래픽의 포트|

```
openstack loadbalancer member create --subnet-id Internal2-subnet --address 192.0.2.10 --protocol-port 80 pool1
openstack loadbalancer member create --subnet-id Internal2-subnet --address 192.0.2.11 --protocol-port 80 pool1
```

#### (6) 상태 모니터 생성
- 상태 모니터 생성을 위해 필요한 옵션
|옵션|설명|
|:-----:|:-----:|
|--delay|멤버의 상태 체크 주기 지정(초)|
|--max-retries|멤버 연결 실패 허용 횟수|
|--timeout|멤버와 연결을 확인하는 제한 시간(초)|
|--type|상태 모니터의 모니터링 타입 지정|

```
openstack loadbalancer healthmonitor create --delay 5 --max-retries 4 --timeout 10 --type PING pool1
```

### 3) 로드 밸런서 유동 IP 연결
- 유동 IP 생성(외부 네트워크 이름: External)
```
openstack floating ip create External
```
- 로드 밸런서의 포트 확인
```
openstack loadbalancer show 
```
- 로드 밸런서와 유동 IP 연결
```
openstack floating ip set --port <load_balancer_vip_port_id> <floating_ip_id>
```

### 4) 로드 밸런서 삭제
#### (1) 멤버 삭제
- openstack loadbalancer memeber delete 명령으로 멤버를 풀에서 제거
```
openstack loadbalancer member delete pool1 <member>
```

#### (2) 상태 모니터 및 풀 삭제
- 상태 모니터가 풀을 모니터링하고 있으므로 상태 모니터를 삭제한 뒤 풀을 삭제해야 함
- 상태 모니터 삭제
```
openstack loadbalancer healthmonitor delete <healthmonitor>
```
- 풀 삭제
```
openstack loadbalancer pool delete <pool>
```
#### (3) 리스너 삭제
```
openstack loadbalancer listener delete <listener>
```

#### (4) 로드 밸런서 삭제
```
openstack loadbalancer delete <loadbalancer>
```


## 2. Telemetry 서비스
## 3. Orchestration 서비스
## 4. Auto Scaling 구성
