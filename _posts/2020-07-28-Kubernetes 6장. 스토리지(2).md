## 실습
- 워드프레스, mysql에 동적 볼륨 연결하기

- mynapp-svc-wordpress.yml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mynapp-svc-wordpress
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: mynapp-rs-dynamicvol-wordrpess 
```

- mynapp-rs-wordpress.yml
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mynapp-rs-dynamicvol-wordpress
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mynapp-rs-dynamicvol-wordpress
  template:
    metadata:
      labels:
        app: mynapp-rs-dynamicvol-wordpress
    spec:
      containers:
      - name: mynapp-wordpress
        image: wordpress
        volumeMounts:
        - name: web-content
          mountPath: /var/www/html
        ports:
        - containerPort: 80
      volumes:
      - name: web-content
        persistentVolumeClaim:
          claimName: mynapp-pvc-dynamic
```

- mynapp-svc-wordpress_db.yml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mynapp-svc-wordpress-db
spec:
  type: ClusterIP
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: mynapp-rs-dynamicvol-wordpress-db

```

- mynapp-rs-wordpress_db.yml
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mynapp-rs-dynamicvol-wordpress-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mynapp-rs-dynamicvol-wordpress-db
  template:
    metadata:
      labels:
        app: mynapp-rs-dynamicvol-wordpress-db
    spec:
      containers:
      - name: mynapp-mysql
        image: mysql:5.7
        volumeMounts:
        - name: mysql-vol
          mountPath: /var/lib/mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: dkagh1.  
        ports:
         - containerPort: 3306
      volumes:
      - name: mysql-vol
        persistentVolumeClaim:
          claimName: mynapp-pvc-dynamic-db
```
