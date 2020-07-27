- Ingress Controller
```yaml
kind: Ingress
metadata:
  name: mynapp-ing-wordpress
spec:
  rules:
  - host: mydomain-minju.com
    http:
      paths:
      - path: /wordpress
        backend: 
          serviceName: mynapp-svc-wordpress 
          servicePort: 80

```

- Web Service
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
    app: mynapp-rs-wordpress

```

- Web Pod
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mynapp-rs-wordpress
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mynapp-rs-wordpress
  template:
    metadata:
      labels:
        app: mynapp-rs-wordpress
    spec:
      containers:
      - name: mynapp-wordpress
        image: wordpress
        ports:
        - containerPort: 80

```

- Database Service
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
    app: mysql-rs-wordpress-db

```

- Database Pod
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mynapp-rs-wordpress-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mynapp-rs-wordpress-db
  template:
    metadata:
      labels:
        app: mynapp-rs-wordpress-db
    spec:
      containers:
      - name: mynapp-mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: dkagh1.
        ports:
        - containerPort: 3306

```
