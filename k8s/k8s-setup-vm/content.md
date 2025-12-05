# Setup VM for Kubernetes

ตั้งค่า VM 2 เครื่อง (Control Plane + Worker Node) สำหรับเรียนรู้ Kubernetes

## 🚀 การตั้งค่า

> 💡 **หมายเหตุ:** หากยังไม่มี VM สำหรับรัน Kubernetes สามารถไปดูวิธีการรัน Ubuntu บน Mac ได้ที่ [Run Ubuntu on Mac](/others/ubuntu-on-mac/content.md)

### 1. ตั้งค่าเบื้องต้น

ทำทั้ง 2 เครื่อง

**อัปเดตระบบ:**
```sh
sudo apt update && sudo apt upgrade -y
```

**ปิด Swap (kubeadm ต้องการ):**
```sh
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

**เปิดใช้งาน network modules:**
```sh
sudo tee /etc/modules-load.d/k8s.conf <<EOF
br_netfilter
EOF

sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

### 2. ติดตั้ง Container Runtime

ทำทั้ง 2 เครื่อง

**ติดตั้ง containerd:**
```sh
sudo apt install -y containerd
```

**สร้าง config:**
```sh
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

**เปิดใช้งาน Systemd cgroup:**
```sh
sudo nano /etc/containerd/config.toml
```

แก้ไขโดยเพิ่มบรรทัดนี้ในส่วน `[plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]`:
```toml
SystemdCgroup = true
```

**รีสตาร์ทและเปิดใช้งาน:**
```sh
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### 3. ติดตั้ง kubeadm, kubelet, kubectl

ทำทั้ง 2 เครื่อง

```sh
# ติดตั้ง dependencies
sudo apt-get install -y apt-transport-https ca-certificates curl

# เพิ่ม Kubernetes repository
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# ติดตั้ง Kubernetes tools
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### 4. Setup Control Plane

ทำบน VM-1 เท่านั้น

**Init cluster:**
```sh
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

เมื่อเสร็จแล้ว จะมีคำสั่ง `kubeadm join` แสดงออกมา (เก็บไว้ใช้สำหรับ Worker join):
```
kubeadm join <CONTROL_PLANE_IP>:6443 --token <TOKEN> \
    --discovery-token-ca-cert-hash sha256:<HASH>
```

**ตั้งค่า kubectl:**
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**ติดตั้ง Network Plugin (Flannel):**
```sh
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

**ตรวจสอบ:**
```sh
kubectl get nodes
```

### 5. Join Worker Node

ทำบน VM-2 เท่านั้น

ใช้คำสั่ง `kubeadm join` ที่ได้จากขั้นตอนที่ 4:

```sh
sudo kubeadm join <CONTROL_PLANE_IP>:6443 --token <TOKEN> \
    --discovery-token-ca-cert-hash sha256:<HASH>
```

**ตัวอย่าง:**
```sh
sudo kubeadm join 192.168.x.x:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### 6. ตรวจสอบ Cluster

ทำบน VM-1 (Control Plane)

```sh
kubectl get nodes -o wide
```

**ผลลัพธ์ที่ควรเห็น:**
```
NAME           STATUS   ROLES           AGE   VERSION
control-node   Ready    control-plane   10m   v1.30.x
worker-node    Ready    <none>          2m    v1.30.x
```

## 💻 ใช้งาน Kubernetes จาก Mac

ตั้งค่าให้ใช้ `kubectl` บน Mac เพื่อควบคุม Kubernetes cluster ที่รันบน VM

### 1. Copy kubeconfig จาก Control Plane

ดาวน์โหลด config จาก VM-1 (Control Plane) มาเก็บไว้บน Mac:

```sh
ssh <User>@<VM-1_IP> "cat ~/.kube/config" > ~/.kube/k8s-vm.conf
```

**ตัวอย่าง:**
```sh
ssh admini@192.168.138.137 "cat ~/.kube/config" > ~/.kube/k8s-vm.conf
```

### 2. Merge กับ config เดิม

รวม config ใหม่กับ config เดิม (ถ้ามี):

```sh
KUBECONFIG=~/.kube/config:~/.kube/k8s-vm.conf kubectl config view --merge --flatten > ~/.kube/config-merged

mv ~/.kube/config ~/.kube/config.bak
mv ~/.kube/config-merged ~/.kube/config
```

> 💡 **หมายเหตุ:** สามารถแก้ไขชื่อ cluster/context ได้ที่ไฟล์ `~/.kube/config`

### 3. ตรวจสอบ config

ดูรายการ clusters, users และ contexts:

```sh
kubectl config get-clusters
kubectl config get-users
kubectl config get-contexts
```

### 4. Switch context

เปลี่ยน context เพื่อใช้งาน cluster ที่ต้องการ:

```sh
kubectl config use-context <ชื่อ-context>
```

**ตรวจสอบว่าเชื่อมต่อได้:**
```sh
kubectl get nodes
```