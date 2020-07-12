## 2. Telemetry 서비스
### 1) Telemetry 서비스 아키텍쳐
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




























