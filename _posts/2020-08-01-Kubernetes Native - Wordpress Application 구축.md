# 실습 과제
- Kubernetes Native를 이용해 Wordpress Application 구축하기  

## 1. 전체 구성도
- 구성도  
![Untitled Diagram (3)](https://user-images.githubusercontent.com/53208493/89103490-38e01000-d44d-11ea-8927-20b203dde2c2.png)

- 설명
  - Ingress TLS Termination  
  쿠버네티스 클러스터 외부에서 TLS 종료 프록시 기능을 구성한 인그레스 컨트롤러를 이용해 클러스터 내부로 들어오게 된다. 
  - Service - Wordpress(ClusterIP)  
  클러스터 내부에는 워드프레스 디플로이먼트에 접속하기 위한 클러스터 내부 서비스가 존재한다. 해당 서비스는 워드프레스 디플로이먼트와 통신을 하면 되는 것이므로 ClusterIP를 이용해 서비스를 구성해준다.
  - Deployment - Wordpress  
  워드프레스 디플로이먼트에는 동적 볼륨 프로비저닝을 구성하여 PVC를 생성하면 PV가 자동으로 생성되게 해준다. 
  - Service - MySQL(Headless)  
  아래의 스테이트풀셋에서 데이터베이스를 실행하는 파드를 마스터(읽기/쓰기) 파드와 슬레이브(읽기 전용) 파드로 구성할 것이므로 데이터베이스에 쓰기를 해야하는 상황을 고려해 데이터베이스 서비스는 헤드리스 서비스로 구성해준다.
  - StatefulSet - MySQL  
  데이터베이스는 스테이트풀셋으로 구성해 파드를 두 개 생성해준다. 

## 2. YAML 코드
1. Ingress TLS Termination  
- Ingress TLS Termination에 사용할 secret
    ```yaml
    # 인그레스 컨트롤러를 위한 TLS 인증서와 키를 생성한 뒤 시크릿에 저장한다.
     
    kubectl create secret tls wordpress-tls-secret
    --key=wordpress-tls/wordpress-tls.key
    --cert=wordpress-tls/wordpress-tls.crt
    ```

- mynapp-ing-tls-termination.yml
    ```yaml
    apiVersion: networking.k8s.io/v1beta1
    kind: Ingress
    metadata:
      name: mynapp-ing-tls-termination
    spec:
      tls:
      - hosts:
        - mynapp.example.com 
        secretName: wordpress-tls-secret 
      rules:
      - host: mynapp.example.com
        http:
          paths:
          - path: /
            backend:
              serviceName: mynapp-svc-wordpress
              servicePort: 80
    ```

    - ingress.spec.tls: TLS 구성
    - ingress.spec.tls.hosts: 사용할 host의 FQDN을 지정한다.
    - ingress.spec.tls.secretName: TLS 인증서 및 키가 저장된 시크릿을 지정한다.  

2. Service: ClusterIP
- mynapp-svc-wordpress.yml
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: mynapp-svc-wordpress 
    spec:
      type: ClusterIP 
      ports:
      - port: 80
        targetPort: 80
      selector:
        app: mynapp-deploy-wordpress
    ```

    - service.spec.type: 워드프레스 디플로이먼트와 통신을 할 것이므로 ClusterIP로 지정한다.  

3. Deployment: Wordpress(Replica:2, Liveness, Readiness)
- mynapp-deploy-wordpress.yml
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: mynapp-deploy-wordpress
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: mynapp-deploy-wordpress
      template:
        metadata:
          labels:
            app: mynapp-deploy-wordpress
        spec:
          affinity:
            podAntiAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
              - labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                      - mynapp-deploy-wordpress
                topologyKey: "kubernetes.io/hostname"
            podAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
              - labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                      - mydb
                topologyKey: "kubernetes.io/hostname"          
          containers:
          - name: mynapp-wordpress
            image: wordpress
            ports:
            - containerPort: 80
            livenessProbe:
              httpGet:
                path: /
                port: 8080
            readinessProbe:
              httpGet:
                path: /
                port: 8080
            volumeMounts:
            - name: web-content
              mountPath: /var/www/html
            env:
            - name: WORDPRESS_DB_HOST
              value: mysql:3306
            - name: WORDPRESS_DB_USER
              value: wp-admin
            - name: WORDPRESS_DB_PASSWORD
              value: dkagh1.
            - name: WORDPRESS_DB_NAME
              value: wordpress_db
          volumes:
          - name: web-content
            persistentVolumeClaim:
              claimName: mynapp-pvc-dynamic1                                                                                                               
    ```

    - deployment.spec.template.spec.affinity  
        워드프레스 파드와 데이터베이스 파드를 한 쌍씩 같은 노드에 배치(wp/db ↔ wp/db)시키기 위해 위와 같은 설정을 해준다.  
    - deployment.spec.template.spec.containers.livenessProbe  
        HTTP GET 라이브니스 프로브를 통해 워드프레스 파드에 오류가 발생하면 컨테이너를 재시작할 수 있도록 설정한다.   
    - deployment.spec.template.spec.containers.readinessProbe  
        HTTP GET 레디니스 프로브를 통해 워드프레스 파드의 애플리케이션의 동작이 준비되었는지 확인한 후 준비가 되었다면 엔드포인트에 파드 주소를 등록할 수 있도록 설정한다.   
    - deployment.spec.template.spec.volumes  
        PVC가 워드프레스 파드에 연결될 수 있도록 설정한다.    

4. PVC: StorageClass(cephfs)
- mynapp-pvc-wordpress.yml
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata: 
      name: mynapp-pvc-dynamic1
    spec:
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 1Gi
      storageClassName: rook-cephfs
    ```

    - persistentvolumeclaim.spec.storageClassName
        워드프레스 파드에 동적 볼륨을 연결해주기 위한 storageClassName을 설정한다.  

5. HPA: Deployment
- mynapp-hpa-cpu-wordpress.yml

    ```yaml
    apiVersion: autoscaling/v1
    kind: HorizontalPodAutoscaler
    metadata:
      name: mynapp-hpa-cpu-wordpress
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: mynapp-deploy-wordpress
      minReplicas: 1
      maxReplicas: 4
      targetCPUUtilizationPercentage: 70
    ```

6. Service: Headless
- mydb-svc-write.yml

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: mydb
      labels:
        app: mydb
    spec:
      ports:
      - port: 3306
      clusterIP: None
      selector:
        app: mydb
    ```

- mydb-svc-read.yml
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: mydb-read
      labels:
        app: mydb
    spec:
      ports:
      - name: mysql
        port: 3306
      selector:
        app: mydb
    ```

7. Statefulset: Mysql(Replica:2, Liveness, Readiness)
- mydb-cm-mysql.yml(ConfigMap)

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: mydb-config
      labels:
        app: mydb
    data:
      master.cnf: |
        [mysqld]
        log-bin
      slave.cnf: |
        [mysqld]
        super-read-only
    ```

- mydb-sts-mysql.yml
    ```yaml
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: mydb
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: mydb
      serviceName: mydb
      template:
        metadata:
          labels:
            app: mydb
        spec:
          affinity:
            podAntiAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
              - labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                      - mydb
                topologyKey: "kubernetes.io/hostname"
          initContainers:
          - name: init-mysql
            image: mysql:5.7
            command:
            - bash
            - "-c"
            - |
              set -ex
              # Generate mysql server-id from pod ordinal index.
              [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
              ordinal=${BASH_REMATCH[1]}
              echo [mysqld] > /mnt/conf.d/server-id.cnf
              # Add an offset to avoid reserved server-id=0 value.
              echo server-id=$((100 + $ordinal)) >> /mnt/conf.d/server-id.cnf
              # Copy appropriate conf.d files from config-map to emptyDir.
              if [[ $ordinal -eq 0 ]]; then
                cp /mnt/config-map/master.cnf /mnt/conf.d/
              else
                **cp /mnt/config-map/slave.cnf /mnt/conf.d/
              fi
            volumeMounts:
            - name: conf
              mountPath: /mnt/conf.d
            - name: config-map
              mountPath: /mnt/config-map
          - name: clone-mysql
            image: gcr.io/google-samples/xtrabackup:1.0
            command:
            - bash
            - "-c"
            - |
              set -ex
              [[ -d /var/lib/mysql/mysql ]] && exit 0
              [[ `hostname` =~ -([0-9]+)$ ]] || exit 1
              ordinal=${BASH_REMATCH[1]}
              [[ $ordinal -eq 0 ]] && exit 0
              ncat --recv-only mydb-$(($ordinal-1)).mydb 3307 | xbstream -x -C /var/lib/mysql
              xtrabackup --prepare --target-dir=/var/lib/mysql
            volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
              subPath: mysql
            - name: conf
              mountPath: /etc/mysql/conf.d
          containers:
          - name: mynapp-mysql
            image: mysql:5.7
            env:
            - name: MYSQL_ALLOW_EMPTY_PASSWORD
              value: "1" 
            - name: MYSQL_DATABASE
              value: wordpress_db  
            - name: MYSQL_USER
              value: wp-admin
            - name: MYSQL_ROOT_HOST
              value: %
            - name: MYSQL_PASSWORD
              value: dkagh1.
            ports:
            - name: mysql
              containerPort: 3306
            volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
              subPath: mysql
            - name: conf
              mountPath: /etc/mysql/conf.d
            livenessProbe:
              exec:
                command: ["mysqladmin", "ping"]
              initialDelaySeconds: 30
              periodSeconds: 10
              timeoutSeconds: 5
            readinessProbe:
              exec:
                command: ["mysql", "-h", "127.0.0.1", "-e", "SELECT 1"]
              initialDelaySeconds: 5
              periodSeconds: 2
              timeoutSeconds: 1
          - name: xtrabackup
            image: gcr.io/google-samples/xtrabackup:1.0
            ports:
            - name: xtrabackup
              containerPort: 3307
            command:
            - bash
            - "-c"
            - |
              set -ex
              cd /var/lib/mysql

              if [[ -f xtrabackup_slave_info && "x$(<xtrabackup_slave_info)" != "x" ]]; then
                cat xtrabackup_slave_info | sed -E 's/;$//g' > change_master_to.sql.in
                rm -f xtrabackup_slave_info xtrabackup_binlog_info
              elif [[ -f xtrabackup_binlog_info ]]; then
                [[ `cat xtrabackup_binlog_info` =~ ^(.*?)[[:space:]]+(.*?)$ ]] || exit 1
                rm -f xtrabackup_binlog_info xtrabackup_slave_info
                echo "CHANGE MASTER TO MASTER_LOG_FILE='${BASH_REMATCH[1]}',\
                      MASTER_LOG_POS=${BASH_REMATCH[2]}" > change_master_to.sql.in
              fi

              if [[ -f change_master_to.sql.in ]]; then
                echo "Waiting for mysqld to be ready (accepting connections)"
                until mysql -h 127.0.0.1 -e "SELECT 1"; do sleep 1; done
                echo "Initializing replication from clone position"
                mysql -h 127.0.0.1 \
                      -e "$(<change_master_to.sql.in), \
                              MASTER_HOST='mydb-0.mydb', \
                              MASTER_USER='root', \
                              MASTER_PASSWORD='', \
                              MASTER_CONNECT_RETRY=10; \
                            START SLAVE;" || exit 1
                mv change_master_to.sql.in change_master_to.sql.orig
              fi
              exec ncat --listen --keep-open --send-only --max-conns=1 3307 -c \
                "xtrabackup --backup --slave-info --stream=xbstream --host=127.0.0.1 --user=root"
            volumeMounts:
            - name: data
              mountPath: /var/lib/mysql
              subPath: mysql
            - name: conf
              mountPath: /etc/mysql/conf.d
          volumes:
          - name: conf
            emptyDir: {}
          - name: config-map
            configMap:
              name: mydb-config
      volumeClaimTemplates:
      - metadata:
          name: data
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 1Gi
          storageClassName: rook-ceph-block
    ```

8. HPA: Statefulset
- mynapp-hpa-cpu-db.yml

    ```yaml
    apiVersion: autoscaling/v1
    kind: HorizontalPodAutoscaler
    metadata: 
      name: mynapp-hpa-cpu-db
    spec:
    	scaleTargetRef:
    	  apiVersion: apps/v1
    	  kind: StatefulSet
    	  name: mydb
    	minReplicas: 1
    	maxReplicas: 4
    	targetCPUUtilizationPercentage: 70
    ```

## 3. 생성된 리소스 확인
1. Ingress TLS Termination  
![Screenshot_from_2020-07-31_17-45-24](https://user-images.githubusercontent.com/53208493/89103946-8dd15580-d450-11ea-870b-50841ee36746.png)  

2. Service: ClusterIP  
![Screenshot_from_2020-07-31_17-43-14](https://user-images.githubusercontent.com/53208493/89103945-8d38bf00-d450-11ea-8df5-5897b7c36512.png)  

3. Deployment: Wordpress(Replica:2, Liveness, Readiness)    
![Screenshot_from_2020-07-31_17-47-03](https://user-images.githubusercontent.com/53208493/89103944-8d38bf00-d450-11ea-9f15-fca122911460.png)    

4. PVC: StorageClass(cephfs)    
![Screenshot_from_2020-07-31_17-50-13](https://user-images.githubusercontent.com/53208493/89103943-8c079200-d450-11ea-8039-1e9d5ffcbe0f.png)    

5. HPA: Deployment    
![Screenshot_from_2020-07-31_17-51-02](https://user-images.githubusercontent.com/53208493/89103942-8c079200-d450-11ea-92c4-84fd58875fed.png)    

6. Service: Headless    
![Screenshot_from_2020-07-31_17-52-30](https://user-images.githubusercontent.com/53208493/89103941-8b6efb80-d450-11ea-96c7-0f4070c6ddc5.png)    

7. Statefulset: Mysql(Replica:2, Liveness, Readiness)    
![Screenshot_from_2020-07-31_17-53-47](https://user-images.githubusercontent.com/53208493/89103940-8ad66500-d450-11ea-90e2-250c27033fdb.png)    

8. PVC: StorageClass(rbd)    
![Screenshot_from_2020-07-31_17-54-36](https://user-images.githubusercontent.com/53208493/89103939-8a3dce80-d450-11ea-8650-87bccdb2dfcd.png)    

9. HPA: Statefulset    
![Screenshot_from_2020-07-31_17-55-30](https://user-images.githubusercontent.com/53208493/89103938-8a3dce80-d450-11ea-8060-f824dbdb5143.png)    

## 4. 결과    
![Screenshot_from_2020-07-31_17-58-22](https://user-images.githubusercontent.com/53208493/89103928-86aa4780-d450-11ea-8ff3-415f5ca900cb.png)    
HTTP 503 오류가 발생하며 워드프레스가 제대로 동작하지 않았다. 
이를 해결하기 위해서 보완할 점은 다음과 같다.
- Ingress TLS Termination을 통한 Wordpress 접속 오류 수정
- Deployment: Wordpress
  - affinity 필드 수정
  - Liveness, Readiness 필드 수정
- Statefulset: Mysql
  - affinity 필드 수정
