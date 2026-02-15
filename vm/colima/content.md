# Colima

Colima คือ container runtime สำหรับ macOS (และ Linux) ที่ตั้งค่าง่าย ชื่อย่อจาก **Container on Lima** เพราะใช้ [Lima](https://lima-vm.io/) เป็นฐาน ใช้สำหรับรัน Docker, Containerd หรือ Kubernetes บน macOS โดยใช้ VM ที่ Lima จัดการ ให้ใช้คำสั่ง Docker/containerd/kubectl ตามปกติได้ทันที

**ข้อดี:**

- ติดตั้งและใช้งานง่าย
- รองรับ Docker, Containerd และ Kubernetes (k3s)
- ใช้หลาย profile แยก env ได้
- รองรับทั้ง macOS (Intel และ Apple Silicon) และ Linux

--------------------------------------------------------------------------------

## 💡 Use Cases

- **รัน Docker บน Mac** - ใช้คำสั่ง Docker ได้ทันทีโดยไม่ต้องติดตั้ง Docker Desktop
- **พัฒนาแอปที่ใช้ container** - สร้าง VM ครั้งเดียวแล้วรัน containers ต่อได้
- **ทดสอบ Kubernetes ท้องถิ่น** - เปิด k3s พร้อม VM ได้ในคำสั่งเดียว
- **แยก environment** - ใช้หลาย profile เช่น dev, test

--------------------------------------------------------------------------------

## ⚙️ การติดตั้ง Colima

### 1\. ติดตั้ง Colima

```sh
brew install colima
```

ตรวจสอบว่าติดตั้งสำเร็จ:

```sh
colima version
```

### 2\. ติดตั้ง Docker CLI (ถ้าจะใช้ Docker)

ถ้ายังไม่มีคำสั่ง `docker` ให้ติดตั้งแยก:

```sh
brew install docker
```

--------------------------------------------------------------------------------

## ⚙️ เริ่มต้นใช้งาน Colima

### 1\. เริ่ม Colima

```sh
colima start
```

คำสั่งนี้จะสร้างและรัน VM โดยใช้ **Docker** เป็น runtime ตามค่าเริ่มต้น หลังรันเสร็จใช้คำสั่ง Docker ได้เลย เช่น:

```sh
docker ps
docker run hello-world
```

### 2\. ตรวจสอบสถานะ

```sh
colima status
```

ตัวอย่างผลลัพธ์:

```
INFO[0000] colima is running using QEMU
INFO[0000] arch: aarch64
INFO[0000] runtime: docker
INFO[0000] mountType: sshfs
INFO[0000] socket: unix:///Users/<user>/.colima/default/docker.sock
```

### 3\. Basic Commands

```sh
# เริ่ม / หยุด / รีสตาร์ท VM
colima start [profile]
colima stop [profile]
colima restart [profile]

# ดูสถานะ
colima status [profile]

# แสดง profile ทั้งหมด
colima list

# SSH เข้า VM
colima ssh [profile]
```

--------------------------------------------------------------------------------

## ⚙️ กำหนดทรัพยากรและตัวอย่างการใช้งาน

### 1\. กำหนดทรัพยากรตอน start

ค่าเริ่มต้น: CPU 2 cores, Memory 2 GiB, Disk 100 GiB

```sh
colima start --cpus 4 --memory 8 --disk 100
```

**Flags หลักของ `colima start`:**

| Flag | Short | ความหมาย | ค่าเริ่มต้น |
|------|-------|----------|-------------|
| `--cpus` | `-c` | จำนวน CPU | 2 |
| `--memory` | `-m` | Memory (GiB) | 2 |
| `--disk` | `-d` | ขนาด disk (GiB) | 100 |
| `--runtime` | `-r` | docker / containerd / incus | docker |
| `--kubernetes` | `-k` | เปิด Kubernetes (k3s) | false |

### 2\. รันด้วย Docker ตามค่าเริ่มต้น

```sh
colima start
docker run -p 8080:80 nginx
# เปิด http://localhost:8080
```

### 3\. รันพร้อม Kubernetes

```sh
colima start --kubernetes --cpus 4 --memory 8
kubectl get nodes
```

### 4\. ใช้หลาย profile

```sh
colima start dev --cpus 4 --memory 8
colima start test --cpus 2 --memory 4
colima list
colima stop dev
colima stop test
```

### 5\. แก้ไข config ก่อน start

```sh
colima start --edit
```

หรือกำหนด editor:

```sh
colima start --edit --editor code
```

ไฟล์ config อยู่ที่ `~/.colima/default/colima.yaml` (profile default)

### 6\. Docker context

Colima จะสร้าง Docker context ชื่อ `colima` ให้อัตโนมัติ:

```sh
docker context ls
docker context use colima
docker context use default
```

### 7\. Kubernetes ใน Colima

**เริ่ม/หยุด Kubernetes (เมื่อ VM รันอยู่แล้ว):**

```sh
colima kubernetes start
colima kubernetes stop
colima kubernetes reset
```

**ระบุเวอร์ชัน k3s:**

```sh
colima start --kubernetes --kubernetes-version v1.28.3+k3s1
```

--------------------------------------------------------------------------------

## 🔧 คำสั่งหลัก (สรุป)

| คำสั่ง | คำอธิบาย |
|--------|----------|
| `colima start [profile]` | เริ่ม VM |
| `colima stop [profile]` | หยุด VM |
| `colima restart [profile]` | รีสตาร์ท VM |
| `colima status [profile]` | ดูสถานะ |
| `colima list` | แสดง profile ทั้งหมด |
| `colima delete [profile]` | ลบ instance (ใช้ `--data` เพื่อลบ images/volumes ด้วย) |
| `colima ssh [profile]` | SSH เข้า VM |
| `colima version` | แสดงเวอร์ชัน |

--------------------------------------------------------------------------------

## 📚 Learning Resources

- **เว็บไซต์:** <https://colima.run/>
- [Getting Started](https://colima.run/docs/getting-started/) · [Commands](https://colima.run/docs/commands/) · [Configuration](https://colima.run/docs/configuration/)
- **GitHub:** <https://github.com/abiosoft/colima>
- [Lima: Bringing Linux VMs to macOS](/others/lima/content.md) — โปรเจกต์ที่ Colima ใช้เป็นฐาน
