# Docker Compose

Docker Compose คือเครื่องมือสำหรับกำหนดและรัน multi-container applications ด้วยไฟล์ YAML เดียว แทนที่จะรันคำสั่ง `docker run` หลายครั้ง สามารถกำหนด services, networks, volumes ในที่เดียวและรันด้วยคำสั่งเดียว

**ข้อดี:**

- กำหนด infrastructure เป็น code (Infrastructure as Code)
- รันหลาย containers พร้อมกันด้วยคำสั่งเดียว
- ตั้งค่า network, volume, environment ได้ในที่เดียว
- เหมาะกับ development และ testing

--------------------------------------------------------------------------------

## 💡 Use Cases

- **Development environment** - รัน app + database + Redis ด้วย `docker compose up`
- **ทดสอบแอปแบบเต็มสแตก** - สร้าง stack ชั่วคราวแล้วทิ้งด้วย `down`
- **Documentation** - ไฟล์ `docker-compose.yml` เป็นเอกสารบอกวิธีรันแอป
- **CI/CD** - ใช้รัน integration tests

--------------------------------------------------------------------------------

## ⚙️ การติดตั้ง

Docker Compose V2 มาพร้อมกับ Docker Desktop และ Docker Engine (plugin) แล้ว

ตรวจสอบว่าติดตั้งแล้ว:

```sh
docker compose version
```

**หมายเหตุ:** คำสั่งเก่า `docker-compose` (มีขีด) ยังใช้ได้ แต่แนะนำใช้ `docker compose` (ช่องว่าง) แทน

--------------------------------------------------------------------------------

## ⚙️ เริ่มต้นใช้งาน

### 1\. สร้างไฟล์ docker-compose.yml

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html

  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: myapp
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:
```

### 2\. รัน stack

```sh
# รันใน foreground (เห็น logs)
docker compose up

# รันใน background
docker compose up -d

# Build image จาก Dockerfile ก่อนรัน (ถ้ามี build ใน config)
docker compose up -d --build
```

### 3\. หยุดและลบ

```sh
# หยุด containers
docker compose stop

# หยุดและลบ containers, networks
docker compose down

# ลบรวม volumes ด้วย
docker compose down -v
```

### 4\. Basic Commands

```sh
# ดูสถานะ
docker compose ps

# ดู logs
docker compose logs
docker compose logs -f web    # follow logs ของ service ชื่อ web

# รันคำสั่งใน service
docker compose exec web sh
docker compose exec db mysql -uroot -p -e "SHOW DATABASES;"
```

--------------------------------------------------------------------------------

## 📄 โครงสร้าง docker-compose.yml

### Services

```yaml
services:
  app:
    image: myapp:1.0
    # หรือ build จาก Dockerfile
    build: .
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=mysql://db:3306/myapp
      - NODE_ENV=production
    env_file:
      - .env
    volumes:
      - ./src:/app/src
      - app-cache:/app/node_modules
    depends_on:
      - db
    networks:
      - backend

  db:
    image: mysql:8
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - backend
```

### Key fields

| Field | คำอธิบาย |
|-------|----------|
| `image` | ใช้ image จาก registry |
| `build` | build จาก Dockerfile |
| `ports` | map port `host:container` |
| `environment` / `env_file` | ตัวแปร environment |
| `volumes` | mount volumes |
| `depends_on` | รัน service อื่นก่อน |
| `networks` | เชื่อมกับ network |

### Volumes และ Networks

```yaml
volumes:
  db-data:
  app-cache:

networks:
  backend:
    driver: bridge
  frontend:
```

--------------------------------------------------------------------------------

## 🔧 คำสั่งที่ใช้บ่อย (สรุป)

| คำสั่ง | คำอธิบาย |
|--------|----------|
| `docker compose up -d` | รันทุก service ใน background |
| `docker compose down` | หยุดและลบ containers, networks |
| `docker compose down -v` | ลบรวม volumes |
| `docker compose ps` | แสดงสถานะ |
| `docker compose logs -f [service]` | ดู logs |
| `docker compose exec [service] sh` | เข้า shell ใน service |
| `docker compose build` | build images |
| `docker compose pull` | pull images |

--------------------------------------------------------------------------------

## 📚 Learning Resources

- **Documentation:** <https://docs.docker.com/compose/>
- [Compose file reference](https://docs.docker.com/compose/compose-file/)
- [Compose CLI reference](https://docs.docker.com/compose/reference/)
