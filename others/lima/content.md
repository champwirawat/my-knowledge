# Lima

Lima คือเครื่องมือ Open Source สำหรับรัน Linux Virtual Machines บน macOS มีการแชร์ไฟล์และ port forwarding อัตโนมัติ คล้ายกับ WSL2 บน Windows

- เว็บไซต์: <https://lima-vm.io/>
- โปรเจกต์อยู่ภายใต้ CNCF (Cloud Native Computing Foundation) ระดับ Incubating

## Lima คืออะไร

Lima เปิดตัวเมื่อเดือนพฤษภาคม 2021 เริ่มจากเป้าหมายส่งเสริม containerd และ nerdctl ให้ผู้ใช้ Mac ต่อมาขยายรองรับ container engines อื่นๆ และแอปที่ไม่ใช่ container

**เทคโนโลยี:** ใช้ QEMU กับ HVF (Hypervisor Framework) รองรับทั้ง Mac แบบ Intel และ Apple Silicon

## คุณสมบัติหลัก

- การแชร์ไฟล์อัตโนมัติ (reverse SSHFS)
- Port forwarding อัตโนมัติ
- รองรับ containerd และ container engines อื่นๆ
- หลายสถาปัตยกรรม: Intel/ARM, multi-arch
- หลาย Linux distro: Ubuntu (default), AlmaLinux, Alpine, Arch, Debian, Fedora, openSUSE, Oracle Linux, Rocky ฯลฯ

โปรเจกต์ที่ใช้ Lima เป็นฐาน: Rancher Desktop, **Colima**, Finch, Podman Desktop

## การติดตั้ง

**Homebrew (แนะนำ):**

```sh
brew install lima
```

เวอร์ชันล่าสุดจาก master: `brew install lima --HEAD`

**วิธีอื่น:** MacPorts (`sudo port install lima`), Nix (`nix-env -i lima`), หรือดาวน์โหลด binary จาก [Lima Releases](https://github.com/lima-vm/lima/releases)

---

## ตัวอย่างการใช้งาน

### ตัวอย่างที่ 1: สร้าง VM เดี่ยวจาก template

ดู template ที่มี → สร้าง instance → เริ่ม → เข้า shell

```sh
limactl create --list-templates
limactl create --name ubuntu-test template:ubuntu
limactl start ubuntu-test
limactl shell ubuntu-test
```

ออกจาก shell แล้วหยุด VM:

```sh
limactl stop ubuntu-test
```

### ตัวอย่างที่ 2: สร้าง instance จากไฟล์ YAML (ปรับ CPU, memory, network)

ใช้เมื่อต้องการแก้ config ก่อนสร้าง (เช่น ใช้กับ k8s-control / k8s-worker)

```sh
# คัดลอง template มาเป็น YAML แล้วแก้ไฟล์ตามต้องการ
limactl template copy --fill template:ubuntu ubuntu.yaml
# หลังจากแก้ k8s-control.yaml, k8s-worker.yaml แล้ว
limactl create --name k8s-control k8s-control.yaml
limactl create --name k8s-worker-1 k8s-worker.yaml
limactl start k8s-control
limactl start k8s-worker-1
```

รันคำสั่งใน VM โดยไม่เข้า shell (เช่น ดู IP):

```sh
limactl shell k8s-control ip addr show
limactl shell k8s-worker-1 ip addr show
```

### ตัวอย่างที่ 3: SSH และ copy ไฟล์เข้า VM

Lima สร้าง `ssh.config` ให้แต่ละ instance อยู่ที่ `~/.lima/<ชื่อ-instance>/ssh.config`

**SSH เข้า VM:**

```sh
ls -la ~/.lima/k8s-control/ssh.config
ssh -F ~/.lima/k8s-control/ssh.config lima-k8s-control
ssh -F ~/.lima/k8s-worker-1/ssh.config lima-k8s-worker-1
```

**Copy ไฟล์เข้า VM แล้วรันสคริปต์ — Control Plane:**

```sh
scp -F ~/.lima/k8s-control/ssh.config ./setup-control-plane.sh lima-k8s-control:~/
ssh -F ~/.lima/k8s-control/ssh.config lima-k8s-control
# ใน VM:
# chmod +x ~/setup-control-plane.sh && ./setup-control-plane.sh
```

**Copy ไฟล์เข้า VM แล้วรันสคริปต์ — Worker (ใช้ join command จาก Control Plane):**

```sh
scp -F ~/.lima/k8s-worker-1/ssh.config ./setup-worker.sh lima-k8s-worker-1:~/
ssh -F ~/.lima/k8s-worker-1/ssh.config lima-k8s-worker-1
# ใน VM:
# chmod +x ~/setup-worker.sh && ./setup-worker.sh
```

### ตัวอย่างที่ 4: เครือข่าย (socket_vmnet) และ Port Forwarding

**ให้ VM ออกเน็ต / เครื่องอื่นเข้าถึง VM ได้ — ใช้ [socket_vmnet](https://github.com/lima-vm/socket_vmnet):**

```sh
brew install socket_vmnet
sudo brew services start socket_vmnet
ls -al /opt/homebrew/var/run/socket_vmnet   # ตรวจสอบว่า socket ทำงาน
```

ใส่ใน config ของ instance (`~/.lima/<ชื่อ>/lima.yaml` หรือใน YAML ตอนสร้าง):

```yaml
networks:
  - socket: "/opt/homebrew/var/run/socket_vmnet"
# หรือ
networks:
  - lima: shared
```

**ส่งพอร์ตจาก Mac ไปยังพอร์ตใน VM (Local Port Forwarding):**

```sh
ssh -F ~/.lima/k8s-control/ssh.config lima-k8s-control -L 8080:localhost:31544
# จากนั้นเข้า http://localhost:8080 บน Mac
```

### ตัวอย่างที่ 5: ตั้งค่าให้ kubectl exec ใช้ได้ (node IP จริง)

ให้ kubelet รายงาน node IP จริงในทุก node — ทำบน **ทุก node** (control + worker):

```sh
# 1. SSH เข้า node แล้วแก้
sudo vi /etc/default/kubelet
# ใส่ (แทน [NODE_IP] ด้วย IP จริงของ node นั้น จาก ip addr show):
# KUBELET_EXTRA_ARGS=--node-ip=[NODE_IP]

# 2. รีโหลดและรีสตาร์ท
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

ตรวจจากเครื่องที่รัน kubectl:

```sh
kubectl get nodes -o wide
```

---

## คำสั่งที่ใช้บ่อย (สรุป)

| คำสั่ง | คำอธิบาย |
|--------|----------|
| `limactl create --list-templates` | แสดง template ทั้งหมด |
| `limactl list` | แสดง instance และสถานะ |
| `limactl create --name <ชื่อ> template:<template>` | สร้าง instance จาก template |
| `limactl template copy --fill template:<ชื่อ> <ไฟล์>.yaml` | คัดลอง template เป็น YAML |
| `limactl create --name <ชื่อ> <ไฟล์>.yaml` | สร้าง instance จาก YAML |
| `limactl start / stop [ชื่อ]` | เริ่ม / หยุด instance |
| `limactl shell [ชื่อ]` | เปิด shell ใน VM |
| `limactl shell [ชื่อ] <คำสั่ง>` | รันคำสั่งใน VM แล้วออก |
| `lima <คำสั่ง>` | รันคำสั่งใน instance ชื่อ default |
| `limactl edit [ชื่อ]` | แก้ไข config |
| `limactl delete [ชื่อ]` | ลบ instance |

**Shell completion:** Bash → `source <(limactl completion bash)` ใน `~/.bash_profile` · Zsh → `limactl completion zsh --help`

## อ้างอิง

- [Lima Documentation](https://lima-vm.io/docs/)
- [Lima GitHub](https://github.com/lima-vm/lima)
- [Installation](https://lima-vm.io/docs/installation/) · [Usage](https://lima-vm.io/docs/usage/) · [Configuration](https://lima-vm.io/docs/config/) · [SSH](https://lima-vm.io/docs/usage/ssh/)
