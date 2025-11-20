<div style="text-align: right; color: #6c757d; font-size: 12px; margin-bottom: 20px;">
Updated at 16/12/2025
</div>

# Kubernetes Components

Kubernetes ใช้หลาย components ทำงานร่วมกันเพื่อจัดการ containerized applications โดยมี flow การทำงานจากภายนอกเข้าไปยัง application ดังนี้:

![k8s flow](https://firebasestorage.googleapis.com/v0/b/a6dd-1e710cb4332d.firebasestorage.app/o/k8s%2Fbasic-overview.png?alt=media&token=f80fe497-48ca-42c6-a668-e2fd5ab1a8f0)

--------------------------------------------------------------------------------

## 🏗️ Components หลัก

### 1\. Deployment

**Deployment** เป็น Kubernetes resource ที่ใช้สร้างและจัดการ Pod

**หน้าที่หลัก:**

- สร้างและจัดการ Pod ตามจำนวนที่กำหนด (replicas)
- จัดการ lifecycle ของ Pod (create, update, delete)
- รับประกันว่า Pod จะทำงานตามจำนวนที่ต้องการ (self-healing)
- รองรับ rolling updates และ rollbacks

**ตัวอย่างการใช้งาน:**

- กำหนดจำนวน Pod ที่ต้องการ (เช่น 3 replicas)
- กำหนด container image ที่จะใช้
- กำหนด resource limits (CPU, Memory)
- Deployment จะสร้าง Pod ตามที่กำหนดและดูแลให้ Pod ทำงานอยู่เสมอ

**ความสัมพันธ์:**

- Deployment → สร้างและจัดการ Pod
- Deployment → ใช้ ConfigMap เพื่อ inject configuration

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

--------------------------------------------------------------------------------

### 2\. Pod

**Pod** เป็นหน่วยพื้นฐานที่สุดใน Kubernetes ที่รัน container

**หน้าที่หลัก:**

- รัน application containers จริงๆ
- แต่ละ Pod มี IP address ของตัวเอง
- Pod สามารถมีหลาย containers ที่ทำงานร่วมกัน

**ลักษณะสำคัญ:**

- Pod เป็น ephemeral (ชั่วคราว) - ถ้า Pod หยุดทำงาน มันจะถูกสร้างใหม่
- แต่ละ application สามารถมี Pod หลายตัวเพื่อ load balancing และ high availability
- Pod แต่ละตัวมี IP address แต่ IP นี้จะเปลี่ยนเมื่อ Pod ถูกสร้างใหม่

**ตัวอย่าง:**

- App A อาจมี Pod หลายตัว: `app-a-pod-1`, `app-a-pod-2`, `app-a-pod-3`
- แต่ละ Pod รัน container เดียวกัน แต่เป็น instance แยกกัน

**ความสัมพันธ์:**

- Pod ถูกสร้างโดย Deployment
- Pod ถูก expose ผ่าน Service
- Pod ใช้ ConfigMap เพื่อรับ configuration

--------------------------------------------------------------------------------

### 3\. Service

**Service** เป็น abstraction layer ที่ให้ stable network endpoint สำหรับ Pod

**หน้าที่หลัก:**

- ให้ stable IP address และ DNS name สำหรับ Pod (แม้ว่า Pod IP จะเปลี่ยน)
- Route traffic จากภายนอกเข้าไปยัง Pod
- Load balance traffic ไปยัง Pod หลายตัว
- ใช้ target port เพื่อกำหนดว่าจะส่ง traffic ไปที่ port ไหนของ Pod

**ปัญหาที่ Service แก้ไข:**

- Pod IP เปลี่ยนทุกครั้งที่ Pod ถูกสร้างใหม่
- ต้องมีวิธีเข้าถึง Pod หลายตัวที่ทำงานเหมือนกัน
- ต้องมี load balancing ระหว่าง Pod

**ประเภทของ Service:**

- **ClusterIP**: Service ที่เข้าถึงได้ภายใน cluster เท่านั้น (default)
- **NodePort**: Expose service ผ่าน port ของ node
- **LoadBalancer**: Expose service ผ่าน cloud provider's load balancer

**ตัวอย่าง:**

- Service `app-a-service` จะ route traffic ไปยัง Pod `app-a-pod-1`, `app-a-pod-2`, `app-a-pod-3`
- Service ใช้ selector เพื่อหา Pod ที่จะ route traffic ไป

**ความสัมพันธ์:**

- Service → route traffic ไปยัง Pod (หลายตัว)
- Ingress → route traffic ไปยัง Service

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
      targetPort: 80 # port ของ pod ที่จะให้วิ่งไปหา
      nodePort: 30180 # port ที่จะให้ภายนอกเข้ามาได้
```

--------------------------------------------------------------------------------

### 4\. Ingress

**Ingress** เป็น API object ที่จัดการ external access เข้าสู่ services ใน cluster

**หน้าที่หลัก:**

- Map HTTP/HTTPS path เพื่อ route ไปยัง Service ที่เหมาะสม
- จัดการ routing ระดับ HTTP/HTTPS (path-based routing)
- กำหนด external access rules และ SSL/TLS termination
- รองรับ virtual hosting (หลาย domains บน IP เดียว)

**ปัญหาที่ Ingress แก้ไข:**

- ต้องการ route traffic ไปยัง Service หลายตัวตาม path หรือ domain
- ต้องการจัดการ SSL/TLS certificates
- ต้องการ expose services หลายตัวผ่าน IP address เดียว

**ตัวอย่างการใช้งาน:**

```
example.com/api → app-a-service
example.com/web → app-b-service
example.com/admin → app-c-service
```

**Ingress Controller:**

- Ingress ต้องมี Ingress Controller (เช่น NGINX, Traefik) เพื่อทำงานจริง
- Ingress Controller จะอ่าน Ingress rules และ configure load balancer ตามนั้น

**ความสัมพันธ์:**

- Ingress → route traffic ไปยัง Service (หลายตัว)
- External → เข้าถึงผ่าน Ingress

--------------------------------------------------------------------------------

## 🔄 Flow การทำงานแบบละเอียด

### External → Kubernetes Flow

1. **External Request** - Request จากภายนอก (เช่น browser, API client)
2. **Ingress** - รับ request และ route ตาม path/domain ไปยัง Service ที่เหมาะสม
3. **Service** - รับ traffic จาก Ingress และ load balance ไปยัง Pod หลายตัว
4. **Pod** - รับ request และประมวลผล (รัน application จริง)
5. **Deployment** - สร้างและจัดการ Pod ให้ทำงานตามที่กำหนด

### ตัวอย่างการทำงาน

**Scenario: User เข้าถึง `example.com/api/users`**

1. Request มาถึง **Ingress**
2. Ingress ตรวจสอบ path `/api` และ route ไปยัง **Service** `api-service`
3. **Service** `api-service` load balance request ไปยัง **Pod** ตัวใดตัวหนึ่ง (เช่น `api-pod-2`)
4. **Pod** `api-pod-2` ประมวลผล request และส่ง response กลับ
5. **Deployment** `api-deployment` ดูแลให้มี Pod ทำงานอยู่เสมอ (ถ้า Pod หายไป จะสร้างใหม่)

--------------------------------------------------------------------------------

## 🔧 Components เพิ่มเติม

### ConfigMap

**ConfigMap** เก็บ configuration data สำหรับ Pod และ Deployment

**หน้าที่:**

- เก็บ configuration แบบ key-value
- สามารถ "set config" ให้กับ Pod ผ่าน environment variables
- สามารถ "mount" เป็น volume เข้าไปใน Pod

**ตัวอย่าง:**

- เก็บ database connection string
- เก็บ API keys
- เก็บ application settings

**ความสัมพันธ์:**

- ConfigMap → ใช้โดย Pod และ Deployment

--------------------------------------------------------------------------------

### Helm

**Helm** เป็น package manager สำหรับ Kubernetes

**หน้าที่:**

- Package Kubernetes applications (charts)
- จัดการ dependencies
- รองรับ templating และ values

**ตัวอย่างการใช้งาน:**

- ติดตั้ง RabbitMQ: `helm install rabbitmq`
- ติดตั้ง PostgreSQL: `helm install postgresql`
- Deploy application ด้วย custom values

**ความสัมพันธ์:**

- Helm → จัดการ Deployment, Service, Ingress และ resources อื่นๆ

## ⚙️ Basic Command

### 1️⃣ Context Management (การจัดการ Context)

**Context** คือโปรไฟล์การเชื่อมต่อ Kubernetes ที่ใช้เลือกว่าจะเชื่อมต่อกับ cluster ไหน

```sh
# ดู context ทั้งหมดที่มี
kubectl config get-contexts

# ดู context ที่กำลังใช้งานอยู่
kubectl config current-context

# เปลี่ยน context ที่จะใช้งาน
kubectl config use-context <context_name>
```

**คำอธิบาย:**

- `get-contexts` - แสดงรายการ context ทั้งหมด พร้อมเครื่องหมาย `*` บอก context ที่กำลังใช้งาน
- `current-context` - แสดงชื่อ context ที่กำลังใช้งานอยู่
- `use-context` - เปลี่ยนไปใช้ context อื่น (เช่น เปลี่ยนจาก dev cluster ไป prod cluster)

--------------------------------------------------------------------------------

### 2️⃣ Get Commands (คำสั่งดึงข้อมูล)

คำสั่งสำหรับดูรายการ resources ต่างๆ ใน cluster

```sh
# ดูรายการ Nodes (เครื่องที่รัน Kubernetes)
kubectl get nodes
kubectl get no  # shorthand

# ดูรายการ Pods
kubectl get pods
kubectl get po  # shorthand

# ดู Pods พร้อมข้อมูลเพิ่มเติม (IP, Node ที่รัน)
kubectl get pods -o wide

# ดู Pods ตาม label
kubectl get pod -l <label>
# ตัวอย่าง: kubectl get pod -l app=nginx

# ดูรายการ Deployments
kubectl get deployment
kubectl get deployment -o wide  # พร้อมข้อมูลเพิ่มเติม

# ดูรายการ Services
kubectl get service
kubectl get svc  # shorthand
```

**คำอธิบาย:**

- `get nodes` - ดูรายการ nodes (worker nodes และ master nodes) ใน cluster
- `get pods` - ดูรายการ pods ทั้งหมด พร้อมสถานะ (Running, Pending, Error)
- `-o wide` - แสดงข้อมูลเพิ่มเติม เช่น IP address, node ที่ pod รันอยู่
- `-l <label>` - กรอง pods ตาม label (เช่น `app=nginx`, `env=production`)
- `get deployment` - ดูรายการ deployments และจำนวน replicas
- `get service` - ดูรายการ services และ ClusterIP/NodePort

--------------------------------------------------------------------------------

### 3️⃣ Describe Commands (คำสั่งดูรายละเอียด)

คำสั่งสำหรับดูรายละเอียดแบบละเอียดของ resource แต่ละตัว

```sh
# ดูรายละเอียด Deployment
kubectl describe deployment <deployment_name>
# ตัวอย่าง: kubectl describe deployment nginx-deployment

# ดูรายละเอียด Service
kubectl describe service <service_name>
# ตัวอย่าง: kubectl describe service nginx-service

# ดูรายละเอียด Pod
kubectl describe pod <pod_name>
```

**คำอธิบาย:**

- `describe deployment` - แสดงรายละเอียด deployment เช่น replicas, image, labels, events, และ pods ที่เกี่ยวข้อง
- `describe service` - แสดงรายละเอียด service เช่น ClusterIP, ports, selector, และ endpoints (pods ที่เชื่อมต่อ)
- `describe pod` - แสดงรายละเอียด pod เช่น IP, node, containers, events, และ logs

--------------------------------------------------------------------------------

### 4️⃣ Logs Commands (คำสั่งดู Logs)

คำสั่งสำหรับดู logs ของ Pod

```sh
# ดู logs ของ Pod
kubectl logs <pod_name>

# ดู logs แบบ real-time (follow)
kubectl logs -f <pod_name>

# ดู logs ของ container เฉพาะ (ถ้า Pod มีหลาย containers)
kubectl logs <pod_name> -c <container_name>

# ดู logs ล่าสุด (เช่น 100 บรรทัดล่าสุด)
kubectl logs <pod_name> --tail=100
```

**คำอธิบาย:**

- `logs <pod_name>` - แสดง logs ของ pod (stdout และ stderr)
- `-f` - follow mode ดู logs แบบ real-time (เหมือน `tail -f`)
- `-c <container_name>` - ระบุ container ใน pod (กรณี pod มีหลาย containers)
- `--tail=100` - แสดง logs ล่าสุด N บรรทัด

--------------------------------------------------------------------------------

### 5️⃣ Exec Commands (คำสั่งเข้าไปใน Pod)

คำสั่งสำหรับเข้าไปรันคำสั่งภายใน Pod

```sh
# เข้าไปใน Pod ด้วย shell
kubectl exec -it <pod_name> -- sh
# หรือ
kubectl exec -it <pod_name> -- bash

# รันคำสั่งใน Pod โดยไม่เข้าไปใน shell
kubectl exec <pod_name> -- <command>
# ตัวอย่าง: kubectl exec nginx-pod -- ls -la

# รันคำสั่งใน container เฉพาะ (ถ้า Pod มีหลาย containers)
kubectl exec -it <pod_name> -c <container_name> -- sh
```

**คำอธิบาย:**

- `exec -it` - execute command แบบ interactive terminal (`-i` = stdin, `-t` = tty)
- `-- sh` หรือ `-- bash` - ใช้ shell ที่ต้องการ (sh, bash, zsh)
- `-- <command>` - รันคำสั่งเดียวแล้วออก (ไม่เข้า shell)
- `-c <container_name>` - ระบุ container ใน pod (กรณี pod มีหลาย containers)

--------------------------------------------------------------------------------

### 6️⃣ Apply Commands (คำสั่งสร้าง/อัปเดต Resources)

คำสั่งสำหรับสร้างหรืออัปเดต resources จากไฟล์ YAML

```sh
# สร้างหรืออัปเดต resource จากไฟล์ YAML
kubectl apply -f <file.yaml>
# ตัวอย่าง: kubectl apply -f nginx-deployment.yaml

# สร้างหรืออัปเดต resources จากหลายไฟล์
kubectl apply -f <directory>
# ตัวอย่าง: kubectl apply -f ./k8s/

# สร้าง resource (จะ error ถ้ามีอยู่แล้ว)
kubectl create -f <file.yaml>

# อัปเดต resource (จะ error ถ้ายังไม่มี)
kubectl replace -f <file.yaml>
```

**คำอธิบาย:**

- `apply -f` - สร้าง resource ใหม่ถ้ายังไม่มี หรืออัปเดตถ้ามีอยู่แล้ว (declarative)
- `create -f` - สร้าง resource ใหม่เท่านั้น (จะ error ถ้ามีอยู่แล้ว)
- `replace -f` - แทนที่ resource ที่มีอยู่ (จะ error ถ้ายังไม่มี)
- `-f <directory>` - apply ทุกไฟล์ YAML ใน directory

--------------------------------------------------------------------------------

### 7️⃣ Delete Commands (คำสั่งลบ Resources)

คำสั่งสำหรับลบ resources ออกจาก cluster

```sh
# ลบ Pod
kubectl delete pod <pod_name>
kubectl delete po <pod_name>  # shorthand

# ลบ Deployment (จะลบ Pods ที่เกี่ยวข้องด้วย)
kubectl delete deployment <deployment_name>
# ตัวอย่าง: kubectl delete deployment nginx-deployment

# ลบ Service
kubectl delete service <service_name>
kubectl delete svc <service_name>  # shorthand

# ลบ resource จากไฟล์ YAML
kubectl delete -f <file.yaml>

# ลบ resources หลายตัวพร้อมกัน
kubectl delete pod <pod1> <pod2> <pod3>

# ลบ resources ตาม label
kubectl delete pod -l app=nginx
```

**คำอธิบาย:**

- `delete pod` - ลบ pod ตัวเดียว (ถ้า pod ถูกสร้างโดย deployment จะถูกสร้างใหม่อัตโนมัติ)
- `delete deployment` - ลบ deployment และ pods ที่เกี่ยวข้องทั้งหมด
- `delete service` - ลบ service (จะไม่กระทบ pods)
- `delete -f <file.yaml>` - ลบ resources ตามที่กำหนดในไฟล์ YAML
- `-l <label>` - ลบ resources ตาม label (เช่น ลบ pods ทั้งหมดที่มี label `app=nginx`)

--------------------------------------------------------------------------------

<div style="display: flex; justify-content: space-between; margin-top: 40px;"><a href="#/k8s/what-k8s" style="padding: 10px 20px; background-color: #6c757d; color: white; text-decoration: none; border-radius: 5px;">
        Previous<br>
        <i>What is OpenSearch?</i></a>
  <a href="#/k8s/k8s-example" style="padding: 10px 20px; background-color: #6c757d; color: white; text-decoration: none; border-radius: 5px;">
        Next<br>
        <i>Kubernetes Example</i></a></div>
