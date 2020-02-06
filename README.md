# docker-alpine-ftp-server

Small and flexible docker image with vsftpd server

## Usage
```
docker run --name ftp -d -p 21:21 -p 21000-21004:21000-21004 \
-e USERS="User1|Password|/ftp/User1 User2|Password|/ftp/User2" \
-e ADDRESS=kluster.tk -e IP=192.168.1.245 akaronte/alpine-ftp
```

## Configuration

Environment variables:
- `USERS` - space and `|` separated list (optional, default: `ftp|alpineftp`)
  - format `name1|password1|[folder1][|uid1] name2|password2|[folder2][|uid2]`
- `ADDRESS` - external address witch clients can connect passive ports 
- `IP` - The ip public of external resolution ADDRESS
- `MIN_PORT` - minamal port number may be used for passive connections (optional, default `21000`)
- `MAX_PORT` - maximal port number may be used for passive connections (optional, default `21004`)

## USERS examples

- `user|password foo|bar|/home/foo`
- `user|password|/home/user/dir|10000`
- `user|password||10000`

Example YAMEL
```
apiVersion: v1
kind: Namespace
metadata:
  name: ftp
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: ftp-config
  namespace: ftp
data:
  ADDRESS: kluster.tk
  USERS: "User1|Password|/ftp/User1 User2|Password|/ftp/User2"
  IP: "192.168.1.245"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ftp-data-disk
  namespace: ftp
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ftp-deployment
  namespace: ftp
  labels:
    app: ftp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ftp
  template:
    metadata:
      labels:
        app: ftp
    spec:
      containers:
        - name: ftp
          image: akaronte/alpine-ftp:latest
          imagePullPolicy: Always
          ports:
            - containerPort: 21
            - containerPort: 21000
            - containerPort: 21001
            - containerPort: 21002
            - containerPort: 21003
            - containerPort: 21004
          volumeMounts:
            - mountPath: "/ftp"
              subPath: "ftp"
              name: ftp-data
          env:
            - name: USERS
              valueFrom:
                configMapKeyRef:
                  name: ftp-config
                  key: USERS
            - name: ADDRESS
              valueFrom:
                configMapKeyRef:
                  name: ftp-config
                  key: ADDRESS
            - name: IP
              valueFrom:
                configMapKeyRef:
                  name: ftp-config
                  key: IP
      volumes:
        - name: ftp-data
          persistentVolumeClaim:
            claimName: ftp-data-disk
---
apiVersion: v1
kind: Service
metadata:
  name: ftp-service
  namespace: jenkins-test
spec:
  type: LoadBalancer
  selector:
    app: ftp
  ports:
  - name: ftp
    protocol: TCP
    port: 21
    targetPort: 21
  - name: ftp0
    protocol: TCP
    port: 21000
    targetPort: 21000
  - name: ftp1
    protocol: TCP
    port: 21001
    targetPort: 21001
  - name: ftp2
    protocol: TCP
    port: 21002
    targetPort: 21002
  - name: ftp3
    protocol: TCP
    port: 21003
    targetPort: 21003
  - name: ftp4
    protocol: TCP
    port: 21004
    targetPort: 21004
```
