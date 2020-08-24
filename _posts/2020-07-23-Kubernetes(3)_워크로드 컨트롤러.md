## 1. 라이브니스 프로브
## 2. 레플리케이션 컨트롤러
## 3. 레플리카셋
### 1) 레플리카셋 소개
### 2) 레플리카셋과 레클리케이션 컨트롤러의 비교
### 3) 레플리카셋 생성
- mynapp-rs.yml
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mynapp-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mynapp-rs
  template:
    metadata:
      labels:
        app: mynapp-rs
    spec:
      containers:
      - name: mynapp
        image: minjulee226/myweb
        ports:
        - containerPort: 8080
```

`kubectl create -f mynapp-rs.yml`
[image]

### 4) 레플리카셋 확인
- 레플리카셋의 상태 목록 확인
`kubectl get replicasets`
[image]

- 파드의 목록 확인
`kubectl get pods`
[image]

### 5) 레플리카셋의 레이블 셀렉터 사용
#### (1) matchLabels 레이블 셀렉터
matchLabels로 레플리카셋을 생성하는 경우에는 replicationcontroller와 똑같이 동작한다.

#### (2) matchExpressions 레이블 셀렉터
```yaml
spec:
  replicas: 3
  selector:
    matchExpressions:
      - key: <string>
        operator: <In | NotIn | Exists | DoesNotExist>
        values: 
          - <string>
```
matchExpressions 필드가 레플리카셋의 기능이다.
- key: 선택할 레이블의 키 이름 지정

### 6) matchExpressions를 사용한 레플리카셋 생성
### 7) 레플리카셋 확인
### 8) 레플리카셋 삭제

## 4. 데몬셋
### 1) 데몬셋 소개
### 2) 데몬셋 생성
- mynapp-ds.yaml
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata: 
  name: mynapp-ds
spec:
  selector:
    matchLabels:
      app: mynapp-ds
  template:
    metadata:
      labels:
        app: mynapp-ds
    spec:
      containers:
      - name: mynapp
        image: minjulee226/myweb
        ports:
        - containerPort: 8080
```

전

### 3) 데몬셋 확인
### 4) 노드 레이블 지정
### 5) 데몬셋 확인
### 6) 노드 레이블 제거
#### (1) 노드 레이블 값 제거
#### (2) 노드 레이블 키 제거

### 7) 데몬셋 삭제

## 5. 잡
### 1) 잡 소개
### 2) 잡 컨트롤러 생성
- mynapp-job.yml
```yaml
apiVersion: batch/v1
kind: Job
metadata: 
  name: mynapp-job
spec:
  template:
    metadata:
      labels:
        app: mynapp-job
    spec:
      restartPolicy: OnFailure
      containers:
      - name: mynapp
        image: busybox
        command: ["sleep", "60"]
```
- job.spec.template.spec.restartPolicy 재시작 정책
  - Always: 종료/실패 시 항상 재시작
  - Onfailure: 실패 시 재시작
  - Never: 종료 또는 오류 시에도 절대 재시작하지 않음
 
- 사용한 이미지: busybox 리눅스 이미지
- 실행될 명령: sleep 60

위와 같이 구성한 이 애플리케이
  
- 잡 오브젝트 생성
`kubectl create -f mynapp-job.yml`
![Screenshot from 2020-07-23 13-16-54](https://user-images.githubusercontent.com/53208493/88251871-2fd19f00-cce7-11ea-9b9e-2e8877962171.png)

### 3) 잡 컨트롤러 및 파드 확인
- 잡 컨트롤러 확인
`kubectl get jobs.batch`
![Screenshot from 2020-07-23 13-17-21](https://user-images.githubusercontent.com/53208493/88251872-306a3580-cce7-11ea-9fab-f6f712c807d4.png)

- 파드 확인
`kubectl get po`
![Screenshot from 2020-07-23 13-18-59](https://user-images.githubusercontent.com/53208493/88251874-319b6280-cce7-11ea-95d1-bc006853e13e.png)

### 4) 잡 컨트롤러 삭제
`kubectl delete -f mynapp-job.yml`
![Screenshot from 2020-07-23 13-19-29](https://user-images.githubusercontent.com/53208493/88251876-319b6280-cce7-11ea-861f-a862a91e311a.png)

### 5) 다중 잡 컨트롤러 생성
- mynapp-job-comp.yml
```
apiVersion: batch/v1
kind: Job
metadata:
  name: mynapp-job-comp
spec:
  completions: 3
  template:
    metadata:
      labels:
        app: mynapp-job-comp
    spec:
      restartPolicy: OnFailure
      containers:
      - name: mynapp
        image: busybox
        command: ["sleep", "60"]
```

- 다중 잡 컨트롤러 생성
`kubectl create -f mynapp-job-comp.yml`
![Screenshot from 2020-07-23 13-21-33](https://user-images.githubusercontent.com/53208493/88252030-ba1a0300-cce7-11ea-8d7c-1f98a7cba4bf.png)

### 6) 다중 잡 컨트롤러 확인
`kubectl get jobs.batch`
![Screenshot from 2020-07-23 13-22-28](https://user-images.githubusercontent.com/53208493/88252032-ba1a0300-cce7-11ea-942e-e717a7569b27.png)

### 7) 다중 잡 컨트롤러 삭제
`kubectl delete -f mynapp-job-comp.yml`
![Screenshot from 2020-07-23 13-23-04](https://user-images.githubusercontent.com/53208493/88252033-bab29980-cce7-11ea-9b98-641879277218.png)

### 8) 병렬 다중 잡 컨트롤러 생성
- mynapp-job-comp-parallel.yml
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: mynapp-job-para
spec:
  completions: 6
  parallelism: 3
  template:
    metadata:
      labels:
        app: mynapp-job-para
    spec:
      restartPolicy: OnFailure
      containers:
      - name: mynapp
        image: busybox
        command: ["sleep", "10"]
```

- 병렬 다중 잡 컨트롤러 생성
`kubectl create -f mynapp-job-comp-parallel.yml`
![Screenshot from 2020-07-23 13-26-49](https://user-images.githubusercontent.com/53208493/88252208-4f1cfc00-cce8-11ea-9392-c7341e80cf8e.png)


### 9) 병렬 다중 잡 컨트롤러 확인
`kubectl get jobs --watch`
![Screenshot from 2020-07-23 13-27-02](https://user-images.githubusercontent.com/53208493/88252210-4fb59280-cce8-11ea-979f-76a16e299186.png)

### 10) 병렬 다중 잡 컨트롤러 삭제
`kubectl delete -f mynapp-job-comp-parallel.yml`
![Screenshot from 2020-07-23 13-29-14](https://user-images.githubusercontent.com/53208493/88252306-9d31ff80-cce8-11ea-93e1-4083474550db.png)

## 6. 크론잡



























