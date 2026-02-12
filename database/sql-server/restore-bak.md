# การกู้คืนฐานข้อมูลจากไฟล์ .bak ใน SQL Server

การกู้คืนฐานข้อมูลจากไฟล์ `.bak` (backup file) เป็นกระบวนการสำคัญในการกู้คืนข้อมูลที่สูญหายหรือย้ายฐานข้อมูลไปยังเซิร์ฟเวอร์อื่น SQL Server รองรับการ restore ทั้งผ่าน GUI (SQL Server Management Studio) และ T-SQL commands

**ข้อดีของการใช้ไฟล์ .bak:**
- เก็บข้อมูลและโครงสร้างฐานข้อมูลไว้ครบถ้วน
- สามารถกู้คืนไปยังเซิร์ฟเวอร์อื่นได้
- รองรับการ restore แบบ full, differential, และ transaction log
- สามารถกำหนดตำแหน่งไฟล์ใหม่ได้ตามต้องการ

--------------------------------------------------------------------------------

## 💡 Use Cases

- **กู้คืนข้อมูลที่สูญหาย** - restore ฐานข้อมูลหลังเกิดปัญหา
- **ย้ายฐานข้อมูล** - ย้ายฐานข้อมูลไปยังเซิร์ฟเวอร์ใหม่
- **Clone ฐานข้อมูล** - สร้างฐานข้อมูลใหม่จาก backup
- **ทดสอบ restore process** - ทดสอบการกู้คืนก่อนใช้งานจริง
- **Migrate ไปยัง Docker** - ย้ายฐานข้อมูลไปยัง SQL Server container

--------------------------------------------------------------------------------

## 💻 การกู้คืนผ่าน T-SQL Commands

### 1. ดูไฟล์ภายใน Backup File

ก่อน restore ควรตรวจสอบไฟล์ภายใน backup เพื่อดูชื่อไฟล์เชิงตรรกะ (Logical Name):

```sql
RESTORE FILELISTONLY
FROM DISK = 'D:\backup\MyDatabase.bak';
```

**ผลลัพธ์จะแสดง:**
- `LogicalName` - ชื่อไฟล์เชิงตรรกะ (ใช้ในคำสั่ง `MOVE`)
- `PhysicalName` - ชื่อไฟล์จริงที่ใช้ใน backup
- `Type` - ประเภทไฟล์ (D = Data, L = Log)
- `Size` - ขนาดไฟล์

**หมายเหตุ:** ใช้ข้อมูลจากผลลัพธ์นี้เพื่อกำหนดค่า `MOVE` ในคำสั่ง `RESTORE DATABASE`

### 2. Restore Database ด้วย T-SQL

```sql
RESTORE DATABASE [DatabaseName]
FROM DISK = 'C:\backup\your-backup.bak'
WITH MOVE 'LogicalDataFileName' TO 'C:\MSSQL\Data\DatabaseName.mdf',
     MOVE 'LogicalLogFileName' TO 'C:\MSSQL\Log\DatabaseName_log.ldf',
     REPLACE;
```

**พารามิเตอร์สำคัญ:**
- `FROM DISK` - เส้นทางไฟล์ backup
- `MOVE` - ย้ายไฟล์จากตำแหน่งเดิมไปยังตำแหน่งใหม่ (ต้องมีทุกไฟล์)
- `REPLACE` - ทับฐานข้อมูลเดิม (ถ้ามี)
- `NORECOVERY` - restore หลายไฟล์ (full + differential + log)
- `RECOVERY` - restore เสร็จพร้อมใช้งาน (default)

### 3. Restore หลายไฟล์ (Full + Differential + Log)

```sql
-- Restore Full Backup
RESTORE DATABASE [DatabaseName]
FROM DISK = 'C:\backup\MyDatabase_Full.bak'
WITH MOVE 'MyDatabase' TO 'C:\MSSQL\Data\MyDatabase.mdf',
     MOVE 'MyDatabase_log' TO 'C:\MSSQL\Log\MyDatabase_log.ldf',
     NORECOVERY;

-- Restore Differential Backup
RESTORE DATABASE [DatabaseName]
FROM DISK = 'C:\backup\MyDatabase_Diff.bak'
WITH NORECOVERY;

-- Restore Transaction Log
RESTORE LOG [DatabaseName]
FROM DISK = 'C:\backup\MyDatabase_Log.trn'
WITH RECOVERY;
```

**หมายเหตุ:**
- ใช้ `NORECOVERY` สำหรับทุกไฟล์ยกเว้นไฟล์สุดท้าย
- ใช้ `RECOVERY` สำหรับไฟล์สุดท้ายเพื่อให้ฐานข้อมูลพร้อมใช้งาน

--------------------------------------------------------------------------------

## 🐳 การกู้คืนผ่าน Docker (SQL Server Linux Container)

### 1. คัดลอกไฟล์ Backup เข้า Container

**วิธีที่ 1: ใช้ docker cp**

```bash
docker cp D:\backup\MyDatabase.bak sqlserver:/var/opt/mssql/backup/MyDatabase.bak
```

**วิธีที่ 2: ใช้ Volume Mount (แนะนำ)**

สร้าง volume หรือ mount directory ที่มีไฟล์ backup:

```bash
# สร้าง volume
docker volume create mssql_backup

# คัดลอกไฟล์เข้า volume (ถ้าใช้ named volume)
docker cp D:\backup\MyDatabase.bak sqlserver:/var/opt/mssql/backup/
```

### 2. ตรวจสอบไฟล์ใน Backup

```bash
docker exec -it sqlserver /opt/mssql-tools/bin/sqlcmd \
  -S localhost -U SA -P "YourStrong!Passw0rd" \
  -Q "RESTORE FILELISTONLY FROM DISK = '/var/opt/mssql/backup/MyDatabase.bak';"
```

### 3. Restore Database

```bash
docker exec -it sqlserver /opt/mssql-tools/bin/sqlcmd \
  -S localhost -U SA -P "YourStrong!Passw0rd" \
  -Q "RESTORE DATABASE [MyDatabase]
      FROM DISK = '/var/opt/mssql/backup/MyDatabase.bak'
      WITH MOVE 'MyDatabase' TO '/var/opt/mssql/data/MyDatabase.mdf',
           MOVE 'MyDatabase_log' TO '/var/opt/mssql/data/MyDatabase_log.ldf',
           REPLACE;"
```

**หมายเหตุ:**
- แทนที่ `sqlserver` ด้วยชื่อ container ของคุณ
- แทนที่ `YourStrong!Passw0rd` ด้วยรหัสผ่าน SA ของคุณ
- แทนที่ `MyDatabase`, `MyDatabase_log` ด้วยชื่อ Logical Name จากผลลัพธ์ `RESTORE FILELISTONLY`
- ตรวจสอบว่าเส้นทางไฟล์ถูกต้องตามโครงสร้างของ container

### 4. ตัวอย่าง Docker Compose

```yaml
version: "3.8"

services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: sqlserver
    restart: unless-stopped
    environment:
      ACCEPT_EULA: "Y"
      SA_PASSWORD: "P@ssw0rd!"
      MSSQL_PID: "Developer"
    ports:
      - "1433:1433"
    volumes:
      - mssql_data:/var/opt/mssql/data
      - mssql_backup:/var/opt/mssql/backup
    networks:
      - local-network

networks:
  local-network:
    external: true

volumes:
  mssql_data:
  mssql_backup:
```

**การใช้งาน:**

```bash
# สร้าง network (ถ้ายังไม่มี)
docker network create local-network

# เริ่ม container
docker-compose up -d

# คัดลอกไฟล์ backup เข้า container
docker cp ./backup/MyDatabase.bak sqlserver:/var/opt/mssql/backup/

# Restore database
docker exec -it sqlserver /opt/mssql-tools/bin/sqlcmd \
  -S localhost -U SA -P "P@ssw0rd!" \
  -Q "RESTORE DATABASE [MyDatabase] FROM DISK = '/var/opt/mssql/backup/MyDatabase.bak' WITH REPLACE;"
```

**หมายเหตุ:**
- `mssql_backup` volume ใช้เก็บไฟล์ backup
- `mssql_data` volume ใช้เก็บไฟล์ฐานข้อมูล
- ปรับพอร์ตและรหัสผ่านตามต้องการ
- ตรวจสอบว่า volume `mssql_backup` มีไฟล์ `.bak` อยู่แล้วก่อน restore

--------------------------------------------------------------------------------

## 📚 Learning Resources

- **Microsoft Documentation**: <https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/restore-a-database-backup-using-ssms>
- **RESTORE DATABASE Syntax**: <https://learn.microsoft.com/en-us/sql/t-sql/statements/restore-statements-transact-sql>

