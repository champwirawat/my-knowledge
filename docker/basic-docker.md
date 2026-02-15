# Basic Docker

Docker คือแพลตฟอร์มสำหรับ build, ship และรัน applications ใน containers ช่วยให้แอปทำงานเหมือนกันทั้ง development และ production environment โดย encapsulate code พร้อม dependencies ทั้งหมดใน container

**ข้อดี:**

- รันแอปได้สม่ำเสมอกันทุก environment
- แยก dependencies ไม่ให้ชนกับระบบหลัก
- สร้างและลบได้ง่าย ไม่เหลือขยะบนเครื่อง
- รองรับ image registry แชร์และ deploy ได้สะดวก

--------------------------------------------------------------------------------

## 💡 Use Cases

- **พัฒนาแอป** - รัน database, Redis, message queue ฯลฯ บนเครื่อง dev
- **ทดสอบ** - สร้าง environment ชั่วคราวแล้วทิ้งหลังทดสอบเสร็จ
- **Deploy** - build image แล้ว deploy ไปยัง server หรือ Kubernetes
- **CI/CD** - ใช้ Docker เป็น runtime สำหรับ pipeline

--------------------------------------------------------------------------------

## 🐳 Image

Image คือ template สำหรับสร้าง container มัก pull จาก Docker Hub หรือ registry อื่น

### คำสั่งที่ใช้บ่อย

| คำสั่ง | คำอธิบาย |
|--------|----------|
| `docker images` | แสดง images ทั้งหมด |
| `docker pull <image>` | ดาวน์โหลด image จาก registry |
| `docker build -t <name>:<tag> .` | build image จาก Dockerfile |
| `docker rmi <image>` | ลบ image |
| `docker image prune` | ลบ dangling images ที่ไม่ถูกใช้ |

### ตัวอย่าง

```sh
# Pull image จาก Docker Hub
docker pull nginx:alpine
docker pull node:20-alpine

# Build image จาก Dockerfile ในโฟลเดอร์ปัจจุบัน
docker build -t myapp:1.0 .

# ลบ image
docker rmi nginx:alpine
docker image prune -a   # ลบทุก image ที่ไม่มี container ใช้อยู่
```

--------------------------------------------------------------------------------

## 📦 Container

Container คือ instance ที่รันจาก image

### คำสั่งที่ใช้บ่อย

| คำสั่ง | คำอธิบาย |
|--------|----------|
| `docker ps` | แสดง containers ที่รันอยู่ |
| `docker ps -a` | แสดง containers ทั้งหมดรวมที่หยุดแล้ว |
| `docker run <image>` | สร้างและรัน container ใหม่ |
| `docker start <container>` | เริ่ม container ที่หยุดอยู่ |
| `docker stop <container>` | หยุด container |
| `docker rm <container>` | ลบ container |
| `docker exec -it <container> sh` | เข้า shell ใน container ที่รันอยู่ |
| `docker logs <container>` | ดู logs |

### ตัวอย่าง

```sh
# รัน container (detached mode)
docker run -d --name web -p 8080:80 nginx:alpine

# รันแบบ interactive + รันทันทีลบเมื่อหยุด
docker run -it --rm alpine sh

# รันและ map volume
docker run -d -v $(pwd)/data:/app/data myapp:1.0

# เข้า shell ใน container
docker exec -it web sh

# Copy ไฟล์เข้า/ออก container
docker cp ./file.txt web:/tmp/
docker cp web:/tmp/file.txt ./
```

### Flags ที่ใช้บ่อยของ `docker run`

| Flag | คำอธิบาย |
|------|----------|
| `-d` | รันแบบ background (detached) |
| `-it` | interactive + TTY (สำหรับ shell) |
| `--rm` | ลบ container อัตโนมัติเมื่อหยุด |
| `--name <name>` | ตั้งชื่อ container |
| `-p <host>:<container>` | map port (เช่น `-p 8080:80`) |
| `-v <host>:<container>` | mount volume |
| `-e KEY=value` | ตั้ง environment variable |

--------------------------------------------------------------------------------

## 💾 Volume

Volume คือการเก็บข้อมูลถาวร นอกเหนือจาก container ที่มี lifecycle แยกต่างหาก

### คำสั่งที่ใช้บ่อย

| คำสั่ง | คำอธิบาย |
|--------|----------|
| `docker volume ls` | แสดง volumes ทั้งหมด |
| `docker volume create <name>` | สร้าง volume |
| `docker volume rm <name>` | ลบ volume |
| `docker volume prune` | ลบ volumes ที่ไม่มี container ใช้ |

### ตัวอย่าง

```sh
# สร้าง volume
docker volume create mysql-data

# ใช้ volume ตอนรัน container
docker run -d -v mysql-data:/var/lib/mysql mysql:8

# Bind mount (โฟลเดอร์บนเครื่อง)
docker run -d -v $(pwd)/html:/usr/share/nginx/html nginx:alpine

# ดูรายละเอียด volume
docker volume inspect mysql-data
```

--------------------------------------------------------------------------------

## 🌐 Network

Network ใช้สำหรับให้ containers ติดต่อกันได้

### คำสั่งที่ใช้บ่อย

| คำสั่ง | คำอธิบาย |
|--------|----------|
| `docker network ls` | แสดง networks ทั้งหมด |
| `docker network create <name>` | สร้าง network |
| `docker network rm <name>` | ลบ network |
| `docker network inspect <name>` | ดูรายละเอียด network |

### ตัวอย่าง

```sh
# สร้าง network
docker network create mynet

# รัน containers ใน network เดียวกัน
docker run -d --name db --network mynet mysql:8
docker run -d --name app --network mynet -p 3000:3000 myapp:1.0

# containers ใน network เดียวกัน resolve ชื่อกันได้ (เช่น db:3306)
```

### Networks ที่มีให้โดย default

| Network | คำอธิบาย |
|---------|----------|
| `bridge` | default network สำหรับ containers |
| `host` | ใช้ network ของ host โดยตรง |
| `none` | ไม่มี network |

--------------------------------------------------------------------------------

## 🔧 คำสั่งที่ใช้บ่อย (สรุป)

```sh
# ตรวจสอบสถานะ
docker ps
docker images
docker volume ls
docker network ls

# ลบของที่ไม่ใช้แล้ว
docker container prune   # ลบ stopped containers
docker image prune      # ลบ dangling images
docker volume prune     # ลบ unused volumes
docker system prune -a  # ลบทุกอย่างที่ไม่ได้ใช้ (ระวัง!)
```

--------------------------------------------------------------------------------

## 📚 Learning Resources

- **Documentation:** <https://docs.docker.com/>
- [Docker CLI Reference](https://docs.docker.com/reference/cli/docker/)
- [Dockerfile Reference](https://docs.docker.com/reference/dockerfile/)
- **Docker Hub:** <https://hub.docker.com/>
