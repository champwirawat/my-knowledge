# Get Start Kubernetes

บทความนี้จะพาทำความรู้จักกับ Kubernetes ตั้งแต่พื้นฐาน ครอบคลุม Core Concepts ทั้ง 9 ตัว ได้แก่ Pod, ReplicaSet, Deployment, Service, Ingress, ConfigMap, Secret, Namespace และ Context พร้อมตัวอย่าง YAML และคำสั่ง kubectl ที่ใช้บ่อย

## 🚀 Core Concepts

![K8S Flow](/images/k8s-flow.png)

### 1\. Pod

คือหน่วยที่เล็กที่สุดใน Kubernetes สำหรับใช้ run containers

- 1 Pod สามารถมี 1 หรือหลาย containers
- Containers ใน Pod เดียวกัน share network และ storage
- Pod มี IP address ของตัวเอง

### 2\. ReplicaSet

ทำหน้าที่รักษาจำนวน Pod ให้คงที่ตามที่กำหนด ถ้า Pod ตาย จะสร้างใหม่อัตโนมัติ

### 3\. Deployment

ใช้สำหรับจัดการ lifecycle ของแอป (deploy, scale, rolling updates, rollback)

- จัดการ ReplicaSet ให้อัตโนมัติ
- รองรับ Rolling Update และ Rollback

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment # กำหนด label ของ deployment
spec:
  replicas: 2
  selector:
    matchLabels: # กำหนด label ของ pod ที่จะดูแล
      app: nginx-app
  template: # กำหนด template spec ของ pod ที่จะดูแล
    metadata:
      labels:
        app: nginx-app
    spec:
      containers: # docker image ที่จะใช้งาน
        - name: nginx-app # กำหนดชื่อของ container
          image: nginx:1.14.2
          ports:
            - containerPort: 80 # กำหนด port ของ container
```

### 4\. Service

ทำหน้าที่ expose Pod ให้เข้าถึงได้

Type         | Description
------------ | -----------------------------------
ClusterIP    | เข้าถึงได้เฉพาะใน cluster (default)
NodePort     | เปิด port บน Node ทุกตัว
LoadBalancer | ใช้ external load balancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-app # label ของ pod ที่จะให้วิ่งไปหา
  ports:
    - protocol: TCP
      port: 81 # port ที่ให้ภายในคุยกัน
      targetPort: 80 # port ของ pod ที่จะให้วิ่งไปหา ต้องตรงกับ containerPort ที่กำหนดใน Deployment
      nodePort: 30180 # port ที่จะให้ภายนอกเข้ามาได้
```

**อธิบาย Ports:**

Port         | คำอธิบาย                                 | ใช้กับ Service Type
------------ | ---------------------------------------- | ---------------------------------
`port`       | Port ที่ Service expose ให้ภายใน cluster | ClusterIP, NodePort, LoadBalancer
`targetPort` | Port ของ container ใน Pod                | ClusterIP, NodePort, LoadBalancer
`nodePort`   | Port ที่เปิดบน Node (30000-32767)        | NodePort, LoadBalancer

### 5\. Ingress

ตัวจัดการ routing HTTP/HTTPS จากภายนอกไปยัง Service ภายใน cluster

**Flow:** `Client → Ingress → Service:port → Pod:targetPort`

**ตัวอย่าง Nginx Ingress**

```sh
# Installation
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/baremetal/deploy.yaml

# เมื่อ run ทั้ง 2 อันเสร็จแล้วจะได้ ingress-nginx มา
$ kubectl get namespace

# ดู port ที่ให้ภายนอกเข้ามาได้
$ kubectl get svc -n ingress-nginx
```

pathType               | Description
---------------------- | ----------------------------------------------------
Prefix                 | match path ที่ขึ้นต้นด้วย prefix ที่กำหนด
Exact                  | match path ที่ตรงเป๊ะๆ เท่านั้น
ImplementationSpecific | ขึ้นอยู่กับ IngressClass ที่ใช้ (แต่ละตัวอาจต่างกัน)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /testpath
            pathType: Prefix
            backend:
              service:
                name: nginx-service # ชื่อ service ต้องตรงกับที่เราสร้าง
                port:
                  number: 81 # เลข port ของ service ต้องตรงกับที่เราสร้าง ไม่ใช่ targetPort หรือ nodePort
```

### 6\. ConfigMap

เก็บข้อมูล config ที่ ไม่ใช่ความลับ (plain text) เช่น ไฟล์ config, environment variables, flags สามารถ mount เป็นไฟล์ใน container หรือ inject เป็น env vars ได้

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configmap
data:
  nginx.conf: |
    user nginx;
    worker_processes  3;
    error_log  /var/log/nginx/error.log;
    events {
      worker_connections  10240;
    }
    http {
      log_format  main
              'remote_addr:$remote_addr\t'
              'time_local:$time_local\t'
              'method:$request_method\t'
              'uri:$request_uri\t'
              'host:$host\t'
              'status:$status\t'
              'bytes_sent:$body_bytes_sent\t'
              'referer:$http_referer\t'
              'useragent:$http_user_agent\t'
              'forwardedfor:$http_x_forwarded_for\t'
              'request_time:$request_time';
      access_log    /var/log/nginx/access.log main;
      server {
          listen       80;
          server_name  _;
          location / {
              root   html;
              index  index.html index.htm;
          }
      }
    }
```

**วิธีใช้ ConfigMap ใน Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
        - name: nginx-app
          image: nginx:1.14.2
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: /etc/nginx # mount nginx-configmap volumn to /etc/nginx -> จะเอาไฟล์ไหนไปใส่ไว้ใน container นั้นๆ
              readOnly: true
              name: nginx-volumes
            - mountPath: /var/log/nginx
              name: log
      volumes:
        - name: nginx-volumes
          configMap:
            name: nginx-configmap # ต้องเหมือนกับใน file configmap
            items:
              - key: nginx.conf
                path: nginx.conf
        - name: log
          emptyDir: {}
```

### 7\. Secret

เก็บข้อมูลที่เป็นความลับ เช่น password, API key, token (base64 encoded)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  password: cGFzc3dvcmQxMjM=  # base64 encoded
  api-key: bXlzZWNyZXRrZXk=
```

**วิธี encode base64:**

```sh
$ echo -n "password123" | base64
# output: cGFzc3dvcmQxMjM=
```

**วิธีใช้ Secret ใน Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
        - name: my-app
          image: my-app:latest
          # วิธีที่ 1: ใช้เป็น Environment Variables
          env:
            - name: DB_PASSWORD # ชื่อ env var ที่จะใช้ใน container
              valueFrom:
                secretKeyRef:
                  name: my-secret # ชื่อ Secret ที่สร้างไว้
                  key: password # key ใน Secret
            - name: API_KEY
              valueFrom:
                secretKeyRef:
                  name: my-secret
                  key: api-key
          # วิธีที่ 2: Mount เป็นไฟล์
          volumeMounts:
            - name: secret-volume
              mountPath: /etc/secrets # path ที่จะ mount
              readOnly: true
      volumes:
        - name: secret-volume
          secret:
            secretName: my-secret # ชื่อ Secret ที่สร้างไว้
```

--------------------------------------------------------------------------------

### 8\. Namespace

คือการแบ่งพื้นที่ภายใน Kubernetes cluster เอาไว้ใช้จัดระเบียบ resource ต่าง ๆ แยกกัน เช่น แยก environment หรือ project เป็นต้น

แต่ละ namespace จะมี resource ของตัวเอง เช่น:

```
dev:
  - deployment: api
  - service: api
  - configmap: app-config

prod:
  - deployment: api
  - service: api
  - configmap: app-config
```

### 9\. Context

คือโปรไฟล์การเชื่อมต่อกับ Kubernetes cluster ช่วยให้เมื่อมีหลาย cluster เช่น dev / staging / prod จะสามารถสลับได้ง่าย

Context จะถูกเก็บไว้ในไฟล์ `~/.kube/config`

--------------------------------------------------------------------------------

## 💻 คำสั่ง kubectl พื้นฐาน

```sh
# ดูว่ามีเครื่อง node กี่อัน
$ kubectl get node

# ดูว่ามีกี่ pod
$ kubectl get po
# query หา pod ตาม label
$ kubectl get pod -l app=nginx

# ดู logs
$ kubectl logs <podname>

# ดูว่ามีกี่ namespace
$ kubectl get namespace

# ดูว่ามีกี่ configmap
$ kubectl get configmap

# shell เข้า pod
$ kubectl exec -it <podname>

# apply เพื่อสร้างของ
$ kubectl apply -f <filepath>
$ kubectl delete -f <filepath>
# apply เพื่อสร้างของทั้งหมดใน folder
$ kubectl apply -f .
$ kubectl delete -f .
# apply ไฟล์ทั้งหมดใน kustomization.yaml
$ kubectl apply -k .
$ kubectl delete -k .

# ดูทั้งหมดใน namespace
$ kubectl get all -n ingress-nginx
# ใส่ -n <namespace> , ถ้าไม่ใส่ -n มันจะเอาใน default

# รายชื่อ pod ที่ run อยู่บน namespace:kube-system
$ kubectl get po -n kube-system

# รายชื่อ replicaset ที่ run อยู่บน namespace:kube-system
$ kubectl get replicaset -n kube-system

# รายชื่อ deployment ที่ run อยู่บน namespace:kube-system
$ kubectl get deployment -n kube-system
# -o wide ใช้เพื่อแสดงผลลัพธ์แบบละเอียดมากขึ้น
$ kubectl get deployment -n kube-system -o wide

# ดูรายละเอียด
$ kubectl describe pod coredns-668d6bf9bc-b4hgb -n kube-system

# เช็คว่า ingress nginx เปิด port ไหน
$ kubectl get svc -n ingress-nginx

# --- helm ---
$ helm list
# helm install -f rb-value.yaml <release_name> <repository> 
$ helm install -f rb-value.yaml rabbitmq bitnami/rabbitmq 
# helm uninstall <release_name>
$ helm uninstall rabbitmq

# --- Config ---
kubectl config view
kubectl config view -o jsonpath='{.users[*].name}'

# --- Context ---
kubectl config get-clusters
kubectl config get-users
kubectl config get-contexts
kubectl config current-context
kubectl config use-context <context_name>
# --- Delete context,cluster,users ---
kubectl config delete-cluster <cluster-name>
kubectl config unset users.<user-name>
```
