# Kubernetes Basics

**Kubernetes (K8s)** เป็น container orchestration platform สำหรับจัดการ container applications

## ✨ ข้อดี

- **Container Orchestration** - จัดการ container หลายตัวอัตโนมัติ
- **Self-healing** - สร้าง Pod ใหม่อัตโนมัติเมื่อ Pod หยุดทำงาน
- **Auto Scaling** - เพิ่ม/ลด Pod ตาม CPU/Memory
- **Load Balancing** - แจกจ่าย traffic ไปยัง Pod หลายตัว
- **Rolling Updates & Rollback** - อัปเดตโดยไม่หยุดระบบ และ rollback ได้ทันที

## 🏗️ Architecture

แบ่งเป็น 2 ส่วน: **Control Plane** และ **Worker Nodes**

![Kubernetes Architecture](/images/k8s-architecture.png)

### Control Plane

ควบคุมทุกอย่างใน Cluster

- **kube-apiserver** - จุดศูนย์กลางการสื่อสาร (kubectl ใช้ตัวนี้)
- **etcd** - เก็บ config และ state ของ cluster
- **kube-scheduler** - เลือก Node สำหรับ Pod
- **kube-controller-manager** - ตรวจสอบและจัดการ cluster

### Worker Node

เครื่องที่รัน containers

- **kubelet** - ตัวแทนใน Node รับคำสั่งจาก kube-apiserver
- **kube-proxy** - จัดการ networking/routing ให้ Pod สื่อสารกัน
- **container runtime** - รัน container (Docker, containerd, CRI-O)

## 🚀 ส่วนประกอบพื้นฐาน

![K8S Flow](/images/k8s-flow.png)

### 1\. Pod

หน่วยที่เล็กที่สุดสำหรับรัน containers

- 1 Pod = 1 หรือหลาย containers
- Containers ใน Pod เดียวกัน share network และ storage
- Pod มี IP address ของตัวเอง

### 2\. ReplicaSet

รักษาจำนวน Pod ให้คงที่ตามที่กำหนด (สร้างใหม่อัตโนมัติเมื่อ Pod ตาย)

### 3\. Deployment

จัดการ lifecycle ของแอป (deploy, scale, rolling updates, rollback)

- จัดการ ReplicaSet อัตโนมัติ
- รองรับ Rolling Update และ Rollback

#### ตัวอย่าง: สร้าง Deployment

**Step 1: สร้างไฟล์ `deployment.yaml`**

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
```

**Step 2: Apply Deployment**

```sh
kubectl apply -f deployment.yaml
```

**Step 3: ตรวจสอบ Pods**

```sh
kubectl get pods -o wide
```

**ผลลัพธ์:**

```
NAME                                READY   STATUS    RESTARTS   AGE     IP              NODE
nginx-deployment-5d7869c5b5-nx7jm   1/1     Running   0          8m43s   10.244.186.70   lima-k8s-worker-1
nginx-deployment-5d7869c5b5-rkdtp   1/1     Running   0          8m43s   10.244.186.69   lima-k8s-worker-1
```

### 4\. Service

Expose Pod ให้เข้าถึงได้จากภายนอก

**Service Types:**

- **ClusterIP** - เข้าถึงได้เฉพาะใน cluster (default)
- **NodePort** - เปิด port บน Node ทุกตัว
- **LoadBalancer** - ใช้ external load balancer

**Ports:**

- `port` - Port ที่ Service expose ให้ภายใน cluster
- `targetPort` - Port ของ container ใน Pod
- `nodePort` - Port ที่เปิดบน Node (30000-32767) สำหรับ NodePort/LoadBalancer

#### ตัวอย่าง: สร้าง Service

**Step 1: สร้างไฟล์ `service.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-app
  ports:
    - protocol: TCP
      port: 81
      targetPort: 80
      nodePort: 30180
```

**Step 2: Apply Service**

```sh
kubectl apply -f service.yaml
```

**Step 3: ตรวจสอบ Service**

```sh
kubectl get services
```

**ผลลัพธ์:**

```
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx-service   NodePort    10.106.68.180   <none>        81:30180/TCP   108s
```

**Step 4: ดูรายละเอียด Service**

```sh
kubectl describe svc nginx-service
```

**ผลลัพธ์:**

```
Name:                     nginx-service
Selector:                 app=nginx-app
Type:                     NodePort
Port:                     81/TCP
TargetPort:               80/TCP
NodePort:                 30180/TCP
Endpoints:                10.244.186.70:80,10.244.186.69:80
```

**หมายเหตุ:**

- `Endpoints` = IP ของ Pods ที่ Service เชื่อมต่อ
- เข้าถึงได้ที่ `http://localhost:30180`

### 5\. Ingress

จัดการ routing HTTP/HTTPS จากภายนอกไปยัง Service

**Flow:** `Client → Ingress → Service:port → Pod:targetPort`

**pathType:**

- **Prefix** - match path ที่ขึ้นต้นด้วย prefix
- **Exact** - match path ที่ตรงเป๊ะๆ
- **ImplementationSpecific** - ขึ้นอยู่กับ IngressClass

#### ตัวอย่าง: สร้าง Ingress

**Step 1: ติดตั้ง ingress-nginx (ต้องทำก่อน)**

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/baremetal/deploy.yaml
```

**Step 2: ตรวจสอบ ingress-nginx**

```sh
kubectl get namespace
kubectl get svc -n ingress-nginx
```

**ผลลัพธ์:**

```
NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.103.196.23    <none>        80:31544/TCP,443:32658/TCP   30s
```

**Step 3: สร้างไฟล์ `ingress.yaml`**

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
                name: nginx-service
                port:
                  number: 81
```

**Step 4: Apply Ingress**

```sh
kubectl apply -f ingress.yaml
```

**Step 5: ตรวจสอบ Ingress**

```sh
kubectl get ing
kubectl describe ing nginx-ingress
```

**ผลลัพธ์:**

```
NAME            CLASS   HOSTS   ADDRESS   PORTS   AGE
nginx-ingress   nginx   *                 80      8s
```

**Step 6: ดู port ของ ingress-nginx**

```sh
kubectl get svc -n ingress-nginx
```

**เข้าถึงได้ที่:** `http://localhost:32658/testpath` (ใช้ port จาก ingress-nginx-controller)

### 6\. ConfigMap

เก็บข้อมูล config (plain text) เช่น ไฟล์ config, environment variables

- Mount เป็นไฟล์ใน container
- Inject เป็น env vars

#### ตัวอย่าง: สร้าง ConfigMap

**Step 1: สร้างไฟล์ `configmap.yaml`**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configmap
data:
  nginx.conf: |
    worker_processes 1;

    events {
      worker_connections 1024;
    }

    http {
      access_log /dev/stdout;
      error_log  /dev/stderr notice;

      server {
        listen 80;

        location / {
          return 200 "CONFIGMAP VERSION: v1\n";
        }
      }
    }
```

**Step 2: Apply ConfigMap**

```sh
kubectl apply -f configmap.yaml
```

**Step 3: ตรวจสอบ ConfigMap**

```sh
kubectl get configmap
kubectl describe configmap nginx-configmap
```

#### ตัวอย่าง: ใช้ ConfigMap ใน Deployment

**Step 1: สร้างไฟล์ `deployment.yaml` (พร้อม mount ConfigMap)**

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
            - mountPath: /etc/nginx
              readOnly: true
              name: nginx-volumes
            - mountPath: /var/log/nginx
              name: log
      volumes:
        - name: nginx-volumes
          configMap:
            name: nginx-configmap
            items:
              - key: nginx.conf
                path: nginx.conf
        - name: log
          emptyDir: {}
```

**Step 2: Apply Deployment**

```sh
kubectl apply -f deployment.yaml
kubectl get pods -w
```

**Step 3: ทดสอบ**

```sh
# เข้าถึงผ่าน Ingress
# http://localhost:32658/testpath
# จะเห็น: CONFIGMAP VERSION: v1
```

#### อัปเดต ConfigMap

**Step 1: แก้ไข `configmap.yaml`**

```yaml
# เปลี่ยนจาก: return 200 "CONFIGMAP VERSION: v1\n";
# เป็น: return 200 "CONFIGMAP VERSION: v2\n";
```

**Step 2: Apply และ Restart**

```sh
kubectl apply -f configmap.yaml
kubectl rollout restart deployment nginx-deployment
```

**หมายเหตุ:** nginx ต้อง restart Pod เพื่อใช้ config ใหม่ (nginx ไม่ reload อัตโนมัติ)

### 7\. Namespace

แบ่งพื้นที่ใน cluster สำหรับจัดระเบียบ resources (แยก environment/project)

**ตัวอย่าง:**

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

### 8\. Context

โปรไฟล์การเชื่อมต่อกับ Kubernetes cluster (สลับระหว่าง dev/staging/prod)

เก็บไว้ใน `~/.kube/config`

## 💻 คำสั่ง kubectl พื้นฐาน

### Get Resources

```sh
# Nodes
kubectl get node

# Pods
kubectl get po
kubectl get pod -l app=nginx
kubectl get pods -o wide

# Deployments
kubectl get deployment
kubectl get deployment -n kube-system -o wide

# Services
kubectl get svc
kubectl get svc -n ingress-nginx

# ReplicaSets
kubectl get replicaset -n kube-system

# ConfigMaps
kubectl get configmap

# Namespaces
kubectl get namespace

# Ingress
kubectl get ing

# All resources in namespace
kubectl get all -n <namespace>
```

### Describe & Logs

```sh
# Describe resource
kubectl describe pod <podname>
kubectl describe svc <servicename>
kubectl describe deployment <deploymentname>

# Logs
kubectl logs <podname>
kubectl logs -f <podname>  # follow logs
```

### Apply & Delete

```sh
# Apply single file
kubectl apply -f <filepath>
kubectl delete -f <filepath>

# Apply all files in directory
kubectl apply -f .
kubectl delete -f .

# Apply kustomization
kubectl apply -k .
kubectl delete -k .
```

### Exec & Debug

```sh
# Shell into pod
kubectl exec -it <podname> -- /bin/sh
kubectl exec -it <podname> -- /bin/bash
```

### Rollout

```sh
# Restart deployment
kubectl rollout restart deployment <deploymentname>

# Check rollout status
kubectl rollout status deployment <deploymentname>

# Rollback
kubectl rollout undo deployment <deploymentname>
```

### Context & Config

```sh
# View config
kubectl config view
kubectl config view -o jsonpath='{.users[*].name}'

# Context
kubectl config get-contexts
kubectl config current-context
kubectl config use-context <context_name>

# Clusters & Users
kubectl config get-clusters
kubectl config get-users

# Delete
kubectl config delete-context <context-name>
kubectl config delete-cluster <cluster-name>
kubectl config unset users.<user-name>
```
