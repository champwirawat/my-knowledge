<div style="text-align: right; color: #6c757d; font-size: 12px; margin-bottom: 20px;">
Updated at 16/12/2025
</div>

# What is Kubernetes?

**Kubernetes (K8s)** เป็น open-source container orchestration platform ที่ใช้สำหรับจัดการ containerized applications อัตโนมัติ

- **Container Orchestration** - จัดการและควบคุม container หลายตัวใน cluster
- **Cluster Management** - จัดการหลาย nodes (servers) ให้ทำงานร่วมกันเป็นระบบเดียว
- **Automation** - ทำหน้าที่ต่างๆ อัตโนมัติ เช่น deployment, scaling, healing
- **Portable** - รันได้บน cloud providers หลายตัว (AWS, GCP, Azure) หรือ on-premise

**ทำไมต้องใช้ Kubernetes?**

- เมื่อมี container หลายตัวที่ต้องจัดการ
- ต้องการระบบที่ scale ได้ง่าย
- ต้องการ high availability และ reliability
- ต้องการ automate deployment และ management

--------------------------------------------------------------------------------

## ✨ ข้อดีของ Kubernetes

- **Container Orchestration** - จัดการ container หลายตัวได้อัตโนมัติ
- **Self-healing** - สร้าง Pod ใหม่อัตโนมัติเมื่อ Pod หยุดทำงาน
- **Scaling** - เพิ่ม/ลดจำนวน Pod ตาม demand
- **Load Balancing** - แจกจ่าย traffic ไปยัง Pod หลายตัว
- **Rolling Updates** - อัปเดต application โดยไม่หยุด service

--------------------------------------------------------------------------------

## 💡 Use Cases

- **Microservices Architecture** - จัดการ services หลายตัวในระบบเดียว
- **Containerized Applications** - รันและจัดการ Docker containers
- **High Availability** - รัน application หลาย instances เพื่อความเสถียร
- **Auto-scaling** - เพิ่ม/ลด resources ตาม demand อัตโนมัติ
- **CI/CD Pipelines** - Deploy applications อย่างต่อเนื่อง
- **Multi-cloud Deployment** - รัน application บน cloud providers หลายตัว

--------------------------------------------------------------------------------

<div style="display: flex; justify-content: end; margin-top: 40px;">
  <a href="#/k8s/k8s-components" style="padding: 10px 20px; background-color: #6c757d; color: white; text-decoration: none; border-radius: 5px;">
        Next<br>
        <i>Kubernetes Components</i></a>
</div>
