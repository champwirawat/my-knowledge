<div style="text-align: right; color: #6c757d; font-size: 12px; margin-bottom: 20px;">
Updated at 26/11/2025
</div>

# What is Kubernetes?

**Kubernetes (K8s)** เป็น open-source container orchestration platform ที่ใช้สำหรับจัดการ container applications

--------------------------------------------------------------------------------

## ✨ ข้อดีของ Kubernetes

- **Container Orchestration** - จัดการ container หลายตัวได้อัตโนมัติ
- **Self-healing** - สร้าง Pod ใหม่อัตโนมัติเมื่อ Pod หยุดทำงาน
- **Auto Scaling** - เพิ่ม/ลดจำนวน Pod ตาม CPU/Memory หรือ Metrics อื่น
- **Load Balancing** - แจกจ่าย traffic ไปยัง Pod หลายตัว
- **Rolling Updates & Rollback** - อัปเดต application โดยไม่ให้ระบบหยุดทำงาน หากมีปัญหา Rollback กลับเวอร์ชันเดิมได้ทันที

--------------------------------------------------------------------------------

## 🏗️ Kubernetes Architecture

แบ่งเป็น 2 ส่วนใหญ่ๆ คือ **Control Plane** และ **Worker Nodes** ![Kubernetes Architecture](/images/k8s-architecture.png)<br>
_Image source: [faun.pub](https://faun.pub/kubernetes-chronicles-k8s-01-introduction-to-kubernetes-architecture-18cad51d270f)_

### 1\. Control Plane

ทำหน้าที่ควบคุมทุกอย่างใน Cluster

ประกอบด้วย:

- kube-apiserver : เป็นจุดศูนย์กลางในการสื่อสารทั้งหมด (kubectl คุยกับ cluster ผ่านตัวนี้)
- etcd : เก็บ config และ state ทั้งหมดของ cluster
- kube-scheduler : เลือกว่าจะให้ Pod ไปลงที่ Node ไหน
- kube-controller-manager : ทำหน้าที่คอยเช็คว่า cluster ปัญหาหรือป่าว

### 2\. Worker Node

คือเครื่องที่รัน containers  

ประกอบด้วย:  
- kubelet : ตัวแทนในแต่ละ Node คอยสั่งให้ container ทำงานตามที่ kube-apiserver บอก
- kube-proxy : ดูแล networking / rules / routing ให้ Pod สื่อสารกัน
- container runtime : ทำงานในการ run container เช่น Docker, containerd, CRI-O
--------------------------------------------------------------------------------

<div style="display: flex; justify-content: end; margin-top: 40px;">
  <a href="#/k8s/k8s-get-start" style="padding: 10px 20px; background-color: #6c757d; color: white; text-decoration: none; border-radius: 5px;">
        Next<br>
        <i>Get Start Kubernetes</i></a>
</div>
