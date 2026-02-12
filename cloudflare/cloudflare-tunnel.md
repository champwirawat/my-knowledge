# Cloudflare Tunnel

Cloudflare Tunnel คือบริการจาก Cloudflare ที่ช่วยให้เราสามารถเชื่อมต่อและเปิด access เครื่องเซิร์ฟเวอร์หรือเว็บ local ของเราออกอินเทอร์เน็ตได้อย่างปลอดภัย โดยไม่ต้องเปิด port firewall หรือ expose IP จริงของเครื่อง แค่รัน agent ชื่อ `cloudflared` เครื่องเราก็จะเชื่อมต่อกับ Cloudflare อัตโนมัติ พร้อมใช้งานผ่าน domain ได้ทันที

**ข้อดี:**

- ไม่ต้องเปิด port ใน firewall
- ไม่ต้อง expose IP จริง
- ได้ HTTPS/SSL ฟรีจาก Cloudflare
- ปลอดภัยกว่าเปิด port โดยตรง

--------------------------------------------------------------------------------

## 💡 Use Cases

- **ทดสอบเว็บ local บน mobile device** - เข้าถึงจากมือถือผ่าน internet
- **แชร์ development server** - ให้ทีมหรือ client ดูงานที่กำลังทำ
- **เชื่อมต่อ service ภายใน** - ไม่ต้องเปิด port ใน firewall
- **Access internal tools** - เข้าถึงเครื่องมือภายในองค์กรจากที่ไหนก็ได้

--------------------------------------------------------------------------------

## ⚙️ ใช้ Cloudflare Tunnel กับเครื่อง Local

### 1\. ติดตั้ง cloudflared

```sh
brew install cloudflared
```

ตรวจสอบว่าติดตั้งสำเร็จ:

```sh
cloudflared --version
```

### 2\. Login เพื่อผูกกับ Account

```sh
cloudflared tunnel login
```

คำสั่งนี้จะเปิดเบราว์เซอร์ให้เรา login เข้า Cloudflare account และอนุญาตให้ `cloudflared` เชื่อมต่อกับ account ของเรา หลังจาก login สำเร็จ จะมีไฟล์ `cert.pem` ถูกสร้างไว้ใน `~/.cloudflared/`

### 3\. สร้าง Tunnel

```sh
cloudflared tunnel create my-local-tunnel
```

คำสั่งนี้จะสร้าง tunnel ใหม่ชื่อ `my-local-tunnel` และจะได้ Tunnel ID (เช่น `xxxxx-xxxx-xxxx-xxxx`) พร้อมเก็บ config files ไว้ใน `~/.cloudflared/`:

- `<tunnel_id>.json` - credentials file สำหรับ tunnel
- `cert.pem` - certificate สำหรับ authentication (ถ้ายังไม่มี)

### 4\. เช็ค Tunnel List

```sh
cloudflared tunnel list
```

คำสั่งนี้จะแสดงรายการ tunnel ทั้งหมดที่สร้างไว้ พร้อม Tunnel ID

### 5\. สร้าง Config File

สร้างไฟล์ config ที่ `~/.cloudflared/config.yml`:

```yaml
tunnel: <tunnel_id>
credentials-file: /Users/<username>/.cloudflared/<tunnel_id>.json

ingress:
  - hostname: local-web.example.com
    service: http://localhost:3000
  - hostname: local-api.example.com
    service: http://localhost:3001
  - service: http_status:404
```

**หมายเหตุ:**

- แทนที่ `<tunnel_id>` ด้วย Tunnel ID ที่ได้จากขั้นตอนที่ 3
- แทนที่ `<username>` ด้วยชื่อ user ของคุณ
- `ingress` เป็นการกำหนด routing rules
- Rule สุดท้าย `service: http_status:404` เป็น default route สำหรับ request ที่ไม่ match กับ hostname ใดๆ
- ถ้า domain อยู่ใน Cloudflare แล้ว ไม่ต้องใส่ `origincert`

### 6\. ลงทะเบียน DNS Route

ก่อนที่จะใช้งานได้ ต้องสร้าง DNS record ใน Cloudflare เพื่อชี้ domain ไปที่ tunnel:

```sh
cloudflared tunnel route dns my-local-tunnel local-web.example.com
cloudflared tunnel route dns my-local-tunnel local-api.example.com
```

คำสั่งนี้จะสร้าง CNAME record ใน Cloudflare DNS โดยอัตโนมัติ

**ตรวจสอบใน Cloudflare DNS:** ![CheckDNS](https://firebasestorage.googleapis.com/v0/b/a6dd-1e710cb4332d.firebasestorage.app/o/cloudflare%2Fcloudflare-tunnel%2FSCR-20251114-nzpd.png?alt=media&token=a73933ef-9397-4be7-ba9a-f8194a06a3d4)

**หมายเหตุ:** Domain ที่ใช้ต้องอยู่ใน Cloudflare account ของคุณก่อน

### 7\. รัน Tunnel

```sh
cloudflared tunnel run my-local-tunnel
```

คำสั่งนี้จะเริ่มรัน tunnel และเชื่อมต่อกับ Cloudflare เมื่อเห็นข้อความว่า `Registered tunnel connection` แสดงว่าพร้อมใช้งานแล้ว

ลองเข้าจากอินเทอร์เน็ตได้เลย:

- `https://local-web.example.com` → `http://localhost:3000`
- `https://local-api.example.com` → `http://localhost:3001`

### 8\. Basic Commands

```sh
# สร้าง tunnel
cloudflared tunnel create <tunnel_name>

# รัน tunnel
cloudflared tunnel run <tunnel_name>

# ลบ tunnel
cloudflared tunnel delete <tunnel_name>

# แสดงรายการ tunnel
cloudflared tunnel list

# สร้าง DNS route
cloudflared tunnel route dns <tunnel_name> <hostname>
```

--------------------------------------------------------------------------------

## 🔄 Run Tunnel ใน Background (macOS LaunchDaemon)

เมื่อติดตั้งด้วย `brew` จะสร้างไฟล์ plist สำหรับ LaunchDaemon ให้แล้วที่ `/Library/LaunchDaemons/com.cloudflare.cloudflared.plist` แต่เราต้องแก้ไขให้ถูกต้อง

### 1\. แก้ไขไฟล์ plist

แก้ไขไฟล์ `/Library/LaunchDaemons/com.cloudflare.cloudflared.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.cloudflare.cloudflared</string>

    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/cloudflared</string>
        <string>tunnel</string>
        <string>run</string>
        <string>my-local-tunnel</string>
    </array>

    <key>EnvironmentVariables</key>
    <dict>
        <key>HOME</key>
        <string>/Users/{username}</string>
    </dict>

    <key>RunAtLoad</key>
    <true/>

    <key>KeepAlive</key>
    <true/>

    <key>StandardOutPath</key>
    <string>/Library/Logs/com.cloudflare.cloudflared.out.log</string>

    <key>StandardErrorPath</key>
    <string>/Library/Logs/com.cloudflare.cloudflared.err.log</string>
</dict>
</plist>
```

**หมายเหตุ:**

- แทนที่ `my-local-tunnel` ด้วยชื่อ tunnel ของคุณ (หรือใช้ tunnel ID ก็ได้)
- แทนที่ `{username}` ด้วยชื่อ user ของคุณ
- `RunAtLoad` - รันอัตโนมัติเมื่อ boot
- `KeepAlive` - restart อัตโนมัติถ้า crash
- `EnvironmentVariables` - จำเป็นเพราะ cloudflared ต้องรู้ HOME path เพื่อหา config files

### 2\. ตรวจสอบสิทธิ์ไฟล์

สิทธิ์ไฟล์ LaunchDaemon ต้องถูกต้อง:

```sh
ls -l /Library/LaunchDaemons/com.cloudflare.cloudflared.plist

# ต้องเป็น
-rw-r--r--  root  wheel  ...

# ถ้าไม่ใช่แบบนี้ให้แก้ไข
sudo chown root:wheel /Library/LaunchDaemons/com.cloudflare.cloudflared.plist
sudo chmod 644 /Library/LaunchDaemons/com.cloudflare.cloudflared.plist
```

### 3\. โหลด Service เข้าระบบ

**ถ้าครั้งแรก (ยังไม่ได้ bootstrap):**

```sh
sudo launchctl bootstrap system /Library/LaunchDaemons/com.cloudflare.cloudflared.plist

# เปิดใช้งาน service
sudo launchctl enable system/com.cloudflare.cloudflared
```

**ถ้าแก้ไขไฟล์แล้วอยาก reload:**

```sh
# unload service
sudo launchctl bootout system /Library/LaunchDaemons/com.cloudflare.cloudflared.plist

# load service ใหม่
sudo launchctl bootstrap system /Library/LaunchDaemons/com.cloudflare.cloudflared.plist
```

### 4\. เช็คสถานะและ Logs

**เช็คสถานะ:**

```sh
sudo launchctl list | grep cloudflared
```

**เช็ค logs:**

```sh
# stdout log
tail -f /Library/Logs/com.cloudflare.cloudflared.out.log

# stderr log
tail -f /Library/Logs/com.cloudflare.cloudflared.err.log
```

### 5\. Basic Commands สำหรับ LaunchDaemon

```sh
# start service
sudo launchctl bootstrap system /Library/LaunchDaemons/com.cloudflare.cloudflared.plist

# stop service
sudo launchctl bootout system /Library/LaunchDaemons/com.cloudflare.cloudflared.plist

# restart service (kill + start)
sudo launchctl kickstart -k system/com.cloudflare.cloudflared

# check status
sudo launchctl list | grep cloudflared

# check logs
tail -f /Library/Logs/com.cloudflare.cloudflared.out.log
tail -f /Library/Logs/com.cloudflare.cloudflared.err.log
```

--------------------------------------------------------------------------------

## 🔍 Troubleshooting

### Tunnel ไม่สามารถเชื่อมต่อได้

1. **ตรวจสอบว่า tunnel ถูกสร้างแล้ว:**

  ```sh
  cloudflared tunnel list
  ```

2. **ตรวจสอบ config file:**

  ```sh
  cat ~/.cloudflared/config.yml
  ```

  ตรวจสอบว่า `tunnel` ID และ `credentials-file` path ถูกต้อง

3. **ตรวจสอบ DNS records:**

  - เข้า Cloudflare Dashboard → DNS
  - ตรวจสอบว่า CNAME records ถูกสร้างแล้ว
  - ตรวจสอบว่า domain อยู่ใน Cloudflare account ของคุณ

4. **ตรวจสอบว่า local service ทำงานอยู่:**

  ```sh
  curl http://localhost:3000
  ```

### LaunchDaemon ไม่ทำงาน

1. **ตรวจสอบ logs:**

  ```sh
  tail -f /Library/Logs/com.cloudflare.cloudflared.err.log
  tail -f /Library/Logs/com.cloudflare.cloudflared.out.log
  ```

2. **ตรวจสอบว่า HOME path ถูกต้อง:**

  - ตรวจสอบใน plist ว่า `EnvironmentVariables.HOME` ถูกต้อง
  - ตรวจสอบว่า config files อยู่ใน `~/.cloudflared/`

3. **ตรวจสอบสิทธิ์ไฟล์:**

  ```sh
  ls -l /Library/LaunchDaemons/com.cloudflare.cloudflared.plist
  sudo chown root:wheel /Library/LaunchDaemons/com.cloudflare.cloudflared.plist
  sudo chmod 644 /Library/LaunchDaemons/com.cloudflare.cloudflared.plist
  ```

### DNS ไม่ resolve

1. **ตรวจสอบว่า DNS route ถูกสร้างแล้ว:**

  ```sh
  cloudflared tunnel route dns my-local-tunnel local-web.example.com
  ```

2. **รอ DNS propagation** (อาจใช้เวลา 1-5 นาที)

3. **ตรวจสอบใน Cloudflare Dashboard** ว่า CNAME record ถูกสร้างแล้ว

--------------------------------------------------------------------------------

## 📚 Learning Resources

- **Official Documentation**: <https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel>
