## 2. Telemetry 서비스
### 1) Telemetry 서비스 아키텍쳐
![ceilo-arch](https://user-images.githubusercontent.com/53208493/87237491-f620ae80-c431-11ea-949e-9560dcc71752.png)
- Telemetry 서비스: Ceilometer 프로젝트
- 사용자 수준의 리소스에 대한 사용량 데이터를 측정하고 제공하는 서비스
- 사용량 측정을 위해 폴링 에이전트 및 알림 에이전트를 통해 데이터 수집 -> 에이전트에서 수집된 샘플 데이터는 데이터베이스에 저장됨
- 기존 Ceilometer 프로젝트의 기능을 Gnocchi, Panko, Aodh로 분리함으로써 확장성 및 성능이 향상됨
  - Gnocchi: 미터링 서비스
    - 데이터 저장을 분리해 효율성을 높이기 위한 기능
    - Telemetry 서비스에서 게시한 데이터를 Time-series 데이터베이스에 타임스탬프에 따라 데이터를 인덱싱하여 저장하는 기능
  - Panko: 이벤트 저장 서비스
  - Aodh: 알람 서비스
    - 알람 조건을 지정해 기준을 넘을 경우 알람이 트리거되어 다른 서비스에 특정 행동을 하도록 구성 가능

### 2) 알람 생성 및 관리
- 알람 생성 명령의 형식

|옵션|설명|
|:----:|:---:|
|--name|알람 이름|
|--description|알람 설명|
|--metric|미터 이름 지정|
|--aggregation-method|집계 방법|
|--operation-method|연산자|
|--threshold|미터의 임계값|
|--evaluation-periods|평가기간|
|--granularity|간격|
|--alarm-action|알람 액션|
|--resource-type|리소스 종류|
|--query|특정 리소스 지정 쿼리|

```
openstack alarm create 
--type gnocchi_aggregation_by_resources_threshold
--name <NAME>
--description <DESC>
--metric <METER>
--aggregation-method <METHOD>
--comparison-operator <OP>
--threshold <THRESHOLD>
--evaluation-periods <SECONDS>
--granularity <SECONDS>
--alarm-action <URL>
--resource-type <TYPE>
--query <QUERY>
```

- 알람 목록 확인
```
openstack alarm list
```

- 특정 알람의 알람 상태 확인
  - 알람 상태의 유형
    - alarm
    - ok
    - insufficient data
```
openstack alarm state get <ALARM-ID>
```

- 특정 알람의 상태 변화 히스토리 확인
```
openstack alarm-history show <ALARM-ID>
```

## 3. Orchestration 서비스
### 1) Orchestration 서비스 아키텍쳐
![Heat-Vision](https://user-images.githubusercontent.com/53208493/87237524-46980c00-c432-11ea-843b-9f4b94283276.png)
- Orchestration 서비스: Heat 프로젝트
- 개발자 및 시스템 관리자가 쉽고 반복적으로 인프라를 생성하고 관리할 수 있도록 도와주는 서비스
- 이 서비스는 인프라를 코드로 표현하며, 해당 코드를 Heat Engine을 이용해 스택 생성
- 스택은 AWS Cloudformation의 CFN 템플릿 형식과 HOT(Heat Orchestration Template) 템플릿을 지원하며, HOT 템플릿은 YAML 형식 사용

### 2) YAML 기본
HOT 템플릿은 YAML 언어를 사용해 작성
#### (1) 사전
- 해시, 배열이라고도 함
- 키 및 값을 가진 쌍
- 키 뒤에 콜론이 있어야 하며, 값 전에 하나의 공백을 가짐
```yaml
key: value
```

#### (2) 목록
타 언어의 배열과 비슷하며 대시 뒤에 반드시 하나의 공백을 가짐
```yaml
- list1
- list2
- list3
```
#### (3) 사전 / 목록
전체로는 사전 형식, 키의 값으로 목록을 가지는 형태
```yaml
key:
  - value1
  - value2
```

#### (4) 계층 구조
계층적인 구조는 들여쓰기로 표현, 들여쓰기는 스페이스 두 칸을 사용

### 3) HOT 템플릿
HOT 템플릿 파일은 YAML 언어로 다양한 인프라 리소스를 코드로 표현
#### (1) header
템플릿 버전 및 템플릿 설명을 지정
#### (2) parameters
리소스 생성 시 참조할 수 있는 파라미터를 지정
#### (3) resources
스택에 배포할 리소스를 정의   
리소스 유형은 `openstack orchestration resource type list` 명령으로 확인 
#### (4) outputs
스택 배포 후 스택을 사용하는 사용자가 참고할 수 있는 정보를 지정   
(ex. 할당된 IP 주소, 사용 방법 등)

- yaml 파일 예제
```yaml
heat_template_version: 2015-04-30
description: version 2017-09-01 created by HOT Generator at Thu, 09 Jul 2020 10:06:34 GMT.
parameters:
  iname:
    type: string
    default: centos7
  fname:
    type: string
    default: flavor1
  kname:
    type: string
    default: key1
resources: 
  Net_1: 
    type: OS::Neutron::Net
    properties: 
      admin_state_up: true
      name: network3
  Subnet_1: 
    type: OS::Neutron::Subnet
    properties: 
      dns_nameservers: 
        - 8.8.8.8
      network: { get_resource: Net_1 }
      name: subnet3
      ip_version: 4
      cidr: 192.168.122.0/24
      enable_dhcp: true
  Port_1: 
    type: OS::Neutron::Port
    properties: 
      admin_state_up: true
      security_groups: 
        - 8c5a2d33-e9be-4f8a-a89e-fc055391c1f4
      network: { get_resource: Net_1 }
  RouterInterface_1: 
    type: OS::Neutron::RouterInterface
    properties: 
      subnet: { get_resource: Subnet_1 }
      router: e83d1b03-16fd-4dc6-95ee-36823f7e3a44
  Server_1: 
    type: OS::Nova::Server
    properties: 
      security_groups: 
        - 10f22372-622f-423e-8830-7b5841ce580e
        - 8c5a2d33-e9be-4f8a-a89e-fc055391c1f4
      networks: 
        - network: { get_resource: Net_1 }
      name: web1
      flavor: { get_param: fname }
      image: { get_param: iname }
      availability_zone: nova
      key_name: { get_param: kname }
      user_data: |
        #!/bin/bash
        yum -y install httpd
        systemctl start httpd
        systemctl enable httpd
        echo "web1" > /var/www/html/index.html
```

### 4) 스택 배포
스택을 생성하는 명령어 형식
```
openstack stack create --environment <FILE> --template <FILE> [--enable-rollback] [--wait] <STACK-NAME>
```
- --environment
  - HOT 템플릿의 환경 파일
  - 반복적인 스택 배포 시 동일한 HOT 템플릿 파일의 재사용성을 높일 수 있음
  - 기존의 HOT 템플릿 파일에 정의된 파라미터를 덮어쓰기 위한 파라미터를 지정하는 파일
- --template
  - HOT 템플릿 파일을 지정
- --enable-rollback
  - 특정 리소스 생성 실패 시 이전까지 생성된 모든 리소스 삭제
- --wait
  - 스택 생성 완료 시까지 대기
