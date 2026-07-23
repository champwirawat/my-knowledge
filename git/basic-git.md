# Git Basic Commands (คู่มือคำสั่ง Git พื้นฐาน)

**Git** คือระบบ Version Control System (VCS) ที่ใช้ควบคุมและบันทึกการเปลี่ยนแปลงของไฟล์ในโปรเจกต์การพัฒนาซอฟต์แวร์ ช่วยให้สามารถย้อนกลับไปดูประวัติการแก้ไขโค้ด ทำงานร่วมกับผู้อื่นได้ง่าย และป้องกันโค้ดทับซ้อนกัน

บทความนี้ได้รวบรวมคำสั่ง Git พื้นฐานที่จำเป็นสำหรับการทำงานในชีวิตประจำวัน ตั้งแต่เริ่มต้นตั้งค่า ไปจนถึงการจัดการสาขาโค้ด (Branch) และการใช้งานร่วมกับ Remote Repository (เช่น GitHub, GitLab, Bitbucket)

---

## 1. การเริ่มต้นและตั้งค่าพื้นฐาน (Setup & Initialization)

ก่อนเริ่มใช้งาน Git บนเครื่องคอมพิวเตอร์เป็นครั้งแรก หรือต้องการสร้างโปรเจกต์ใหม่ ต้องระบุตัวตนและสร้าง Git Repository ก่อน

### คำสั่งที่ใช้บ่อย

| คำสั่ง | คำอธิบาย | ตัวอย่างการใช้งาน |
| --- | --- | --- |
| `git config` | ตั้งค่าโปรไฟล์ผู้ใช้งานของ Git | `git config --global user.name "Your Name"` |
| `git init` | สร้าง Git Repository ใหม่ในโฟลเดอร์ปัจจุบัน | `git init` |
| `git clone` | คัดลอกโปรเจกต์จาก Remote Server ลงมายังเครื่องคอมพิวเตอร์ | `git clone <url>` |

### ตัวอย่างการใช้งาน

```bash
# ตั้งค่าชื่อและอีเมล (ทำครั้งแรกครั้งเดียว เพื่อระบุตัวตนใน commit)
git config --global user.name "Wirawat Khawluang"
git config --global user.email "your.email@example.com"

# ตรวจสอบการตั้งค่าทั้งหมด
git config --list

# เริ่มต้นสร้างโปรเจกต์ใหม่ด้วย Git
mkdir my-new-project
cd my-new-project
git init  # จะสร้างโฟลเดอร์ซ่อนชื่อ .git ขึ้นมา

# คัดลอกโปรเจกต์ที่มีอยู่แล้วจาก GitHub ลงมาที่เครื่อง
git clone https://github.com/user/repository.git
```

---

## 2. ขั้นตอนการทำงานประจำวัน (Basic Workflow)

นี่คือรอบวงจรชีวิตพื้นฐานของการทำงาน (Working Directory -> Staging Area -> Local Repository)

```
Working Directory  ──( git add )──>  Staging Area  ──( git commit )──>  Local Repo (.git)
```

### คำสั่งที่ใช้บ่อย

| คำสั่ง | คำอธิบาย | Option ยอดฮิต |
| --- | --- | --- |
| `git status` | ตรวจสอบสถานะการเปลี่ยนแปลงของไฟล์ในโปรเจกต์ | - |
| `git add` | เพิ่มไฟล์ที่เตรียมจะบันทึกเข้าสู่ Staging Area | `git add .` (เพิ่มทั้งหมด), `git add <filename>` |
| `git commit` | บันทึกประวัติการเปลี่ยนแปลงลงใน Repository | `-m "message"` (ใส่ข้อความอธิบาย), `-am` (add + commit ทันทีสำหรับไฟล์ที่เคย track แล้ว) |
| `git log` | แสดงประวัติการ Commit ทั้งหมดที่ผ่านมา | `--oneline` (แสดง 1 บรรทัดต่อ 1 commit), `--graph` (แสดงเป็นกราฟ) |
| `git diff` | เปรียบเทียบความแตกต่างของโค้ดก่อน/หลังการบันทึก | `--staged` (เปรียบเทียบใน Staging Area) |

### ตัวอย่างการใช้งาน

```bash
# 1. ตรวจสอบดูว่ามีไฟล์ไหนเปลี่ยนไปบ้าง
git status

# 2. เพิ่มไฟล์เฉพาะเจาะจง หรือไฟล์ทั้งหมดเข้าไปที่ Staging Area
git add src/index.js      # เพิ่มเฉพาะไฟล์ index.js
git add .                 # เพิ่มทุกไฟล์ที่มีการแก้ไข/สร้างใหม่

# 3. บันทึกประวัติการเปลี่ยนแปลงพร้อมเขียนอธิบายสั้นๆ (ควรเขียนให้เข้าใจง่ายและได้ใจความ)
git commit -m "feat: add login page validation"

# 4. ดูประวัติ commit ย้อนหลังแบบกระชับและแสดงเส้นเชื่อมต่อของ Branch
git log --oneline --graph --decorate -n 10

# 5. ดูว่าไฟล์มีการแก้ไขตรงไหนบ้างเทียบกับ commit ล่าสุด
git diff
```

---

## 3. การจัดการสาขาโค้ด (Branch Management)

Branch (กิ่งหรือสาขา) ช่วยให้เราสามารถแยกการพัฒนาฟีเจอร์ต่าง ๆ ออกจากโค้ดหลักได้โดยไม่ปะปนกัน เพื่อความปลอดภัยในการทดลองและพัฒนาโค้ด

### คำสั่งที่ใช้บ่อย

| คำสั่ง | คำอธิบาย | ตัวอย่างการใช้งาน |
| --- | --- | --- |
| `git branch` | ดูรายชื่อ Branch ทั้งหมด หรือสร้าง Branch ใหม่ | `git branch <branch-name>` |
| `git checkout` | สลับไปยัง Branch ที่ต้องการ หรือสลับไฟล์ | `git checkout <branch-name>` |
| `git switch` | สลับไปยัง Branch (แนะนำให้ใช้แทน checkout สำหรับการสลับ branch) | `git switch <branch-name>` |
| `git merge` | รวมโค้ดจาก Branch อื่นเข้ามายัง Branch ปัจจุบัน | `git merge <branch-name>` |

### ตัวอย่างการใช้งาน

```bash
# ดู Branch ทั้งหมดในเครื่อง (เครื่องหมาย * คือ branch ที่เรากำลังอยู่)
git branch

# สร้าง Branch ใหม่ชื่อ 'feature/login' (แตกยอดมาจาก branch ปัจจุบัน)
git branch feature/login

# สลับการทำงานไปยัง branch 'feature/login'
git switch feature/login
# หรือสร้างพร้อมสลับไปทันที
git switch -c feature/register  # (-c ย่อมาจาก create)

# เมื่อพัฒนาเสร็จ และต้องการรวมโค้ดเข้าหา branch หลัก (เช่น main)
git switch main                # 1. สลับกลับมาที่ branch main ก่อน
git merge feature/login        # 2. ทำการ merge feature/login เข้ามา

# ลบ Branch ที่ไม่มีการใช้งานแล้ว (หลังจาก merge เรียบร้อย)
git branch -d feature/login
# บังคับลบ Branch แม้จะยังไม่ได้ merge (ระวัง!)
git branch -D feature/abandoned
```

> [!TIP]
> ใน Git เวอร์ชันใหม่ ๆ แนะนำให้ใช้ `git switch` แทน `git checkout` สำหรับสลับ Branch และใช้ `git restore` แทนสำหรับดึงไฟล์กลับมา เพื่อป้องกันความสับสนเนื่องจากคำสั่ง `checkout` ทำหน้าที่สองอย่างที่ต่างกันมาก

---

## 4. การทำงานกับเซิร์ฟเวอร์ปลายทาง (Remote Repository)

เมื่อทำงานร่วมกับทีม เราต้องทำการซิงก์ข้อมูลระหว่างเครื่องเรา (Local Repository) และเซิร์ฟเวอร์ส่วนกลาง (Remote Repository เช่น GitHub, GitLab)

### คำสั่งที่ใช้บ่อย

| คำสั่ง | คำอธิบาย | ตัวอย่างการใช้งาน |
| --- | --- | --- |
| `git remote` | จัดการลิงก์เชื่อมต่อไปยัง Server ปลายทาง | `git remote add origin <url>` |
| `git fetch` | ดึงประวัติการอัปเดตล่าสุดจาก Server มายังเครื่อง แต่ยังไม่รวมไฟล์ | `git fetch origin` |
| `git pull` | ดึงการอัปเดตล่าสุดจาก Server และรวมไฟล์เข้าเครื่องเราทันที | `git pull origin main` |
| `git push` | ส่ง Commit จากเครื่องเราขึ้นไปยัง Server | `git push origin main` |

### ตัวอย่างการใช้งาน

```bash
# ตรวจสอบลิงก์ Remote Repository ปัจจุบัน
git remote -v

# เพิ่ม Remote ปลายทาง (ปกติระบุชื่อว่า origin)
git remote add origin https://github.com/champwirawat/my-knowledge.git

# ส่งการเปลี่ยนแปลงบน branch ปัจจุบันขึ้นไปยัง origin
# (ใส่ -u ในครั้งแรก เพื่อสร้าง upstream link ครั้งต่อไปรันแค่ git push ได้เลย)
git push -u origin main

# ดึงโค้ดล่าสุดจากเครื่องเพื่อนร่วมงานบน Server มารวมเข้ากับโค้ดเครื่องเรา
git pull origin main

# ดึงประวัติล่าสุดเพื่อเช็กความเคลื่อนไหว โดยยังไม่เอาโค้ดมารวม
git fetch origin
```

---

## 5. การแก้ไขข้อผิดพลาดและการยกเลิกการเปลี่ยนแปลง (Undoing Changes)

เป็นเรื่องธรรมดาที่การเขียนโค้ดอาจเกิดข้อผิดพลาด Git มีคำสั่งต่าง ๆ ที่ช่วยย้อนเวลากลับไปแก้ไขได้

### คำสั่งที่ใช้บ่อย

| คำสั่ง | คำอธิบาย | ตัวอย่างการใช้งาน |
| --- | --- | --- |
| `git restore` | ยกเลิกการแก้ไขไฟล์ปัจจุบัน (กลับไปเป็นเหมือน commit ล่าสุด) | `git restore <filename>` |
| `git reset` | ย้อนกลับประวัติ commit (โค้ดเก่าอาจหายไปตาม option) | `git reset --hard HEAD~1` |
| `git revert` | สร้าง commit ใหม่ที่เป็นการกลับค่า (ย้อนกระบวนการ) ของ commit ก่อนหน้า | `git revert <commit-hash>` |
| `git stash` | เก็บซ่อนงานที่ยังทำไม่เสร็จไว้ชั่วคราว เพื่อสลับไปทำงานด่วนอื่น | `git stash` |

### ตัวอย่างการใช้งาน

```bash
# 1. ต้องการยกเลิกการแก้ไขล่าสุดในไฟล์ (ที่ยังไม่ได้ add)
git restore src/App.js

# 2. ถอนไฟล์ออกจาก Staging Area (แต่โค้ดยังอยู่เหมือนเดิม)
git restore --staged src/App.js

# 3. การเก็บซ่อนงานชั่วคราว (เช่น กำลังแก้โค้ดค้างไว้ แต่ต้องรีบไปแก้บั๊กบน branch อื่น)
git stash             # เก็บโค้ดปัจจุบันที่ค้างอยู่ลงลิ้นชักชั่วคราว
# ... สลับ branch ไปแก้บั๊ก ...
git switch develop
git stash pop         # ดึงโค้ดที่ซ่อนกลับมาทำต่อ

# 4. ย้อนกลับ Commit
# แบบ Revert (สร้าง commit ใหม่เพื่อลบล้างโค้ดตัวเก่า - ปลอดภัยที่สุดสำหรับการทำงานร่วมกัน)
git revert a1b2c3d4   # ระบุ hash ของ commit ที่ต้องการลบล้าง

# แบบ Reset (ลบ commit ออกไปจากประวัติเลย - ห้ามทำกับ branch ที่ผู้อื่นใช้ร่วมกันแล้ว)
git reset --soft HEAD~1  # ย้อนกลับ 1 commit โดยโค้ดที่ทำไปยังอยู่ใน Staging Area
git reset --hard HEAD~1  # ย้อนกลับ 1 commit และลบโค้ดที่ทำทิ้งทั้งหมดไปทันที!
```

> [!CAUTION]
> การใช้งาน `git reset --hard` จะลบโค้ดปัจจุบันที่คุณไม่ได้ทำการ commit หรือ stash ทิ้งทั้งหมดอย่างถาวร กรุณาตรวจสอบให้แน่ใจก่อนรันคำสั่งนี้

---

## 6. ตารางสรุปคำสั่ง Git ยอดนิยมด่วน (Git Cheat Sheet)

| คำสั่ง | วัตถุประสงค์การใช้งาน |
| --- | --- |
| `git init` | เริ่มต้นโปรเจกต์ Git ในโฟลเดอร์ปัจจุบัน |
| `git clone <url>` | ดึงโปรเจกต์จาก Remote ลงเครื่อง |
| `git status` | ตรวจสอบไฟล์ที่มีการสร้าง แก้ไข หรือลบออก |
| `git add <file>` | จัดเตรียมไฟล์ (Stage) เพื่อเตรียมเซฟ |
| `git commit -m "<msg>"` | บันทึกประวัติและข้อความอธิบายการแก้ไข |
| `git log --oneline` | แสดงประวัติ Commit สั้น ๆ กระชับ |
| `git branch -a` | ดูรายชื่อ branch ทั้งหมด ทั้งในเครื่องและเซิร์ฟเวอร์ |
| `git switch <branch>` | สลับไปยัง branch อื่น |
| `git switch -c <branch>` | สร้าง branch ใหม่และสลับไปทันที |
| `git merge <branch>` | รวมโค้ดจากสาขาอื่นเข้ามาที่สาขาเราในปัจจุบัน |
| `git pull` | ดึงโค้ดเวอร์ชันล่าสุดจากเซิร์ฟเวอร์ลงมาที่เครื่อง |
| `git push` | อัปโหลด commit บนเครื่องเราขึ้นเซิร์ฟเวอร์ |
| `git stash` | พักการเปลี่ยนแปลงปัจจุบันลงกล่องเก็บโค้ดชั่วคราว |
| `git stash pop` | ดึงโค้ดที่เก็บพักเอาไว้ชั่วคราวกลับมาทำงานต่อ |
| `git restore <file>` | ดึงโค้ดดั้งเดิมกลับมา (ล้างความเปลี่ยนแปลงที่เพิ่งพิมพ์ไป) |

---

## 7. แนวทางปฏิบัติที่ดีในการใช้งาน Git (Git Best Practices)

*   **Commit บ่อย ๆ แต่เฉพาะเรื่องเดี่ยว (Commit Early, Commit Often)**: หลีกเลี่ยงการรวมงานหลายฟีเจอร์ไว้ใน 1 commit เดียว การ commit ทีละชิ้นงานย่อยจะทำให้ตรวจสอบความผิดพลาดและย้อนกลับโค้ดได้ง่ายขึ้นมาก
*   **เขียน Commit Message ให้มีความหมาย**: ใช้รูปแบบที่เป็นมาตรฐาน เช่น Conventional Commits (เช่น `feat: ...`, `fix: ...`, `docs: ...`) เพื่อให้ทุกคนในทีมเข้าใจว่าแต่ละ commit ทำอะไรไป
*   **อัปเดตโค้ดเสมอด้วย `git pull`**: ก่อนเริ่มลงมือทำฟีเจอร์ใหม่ ๆ หรือก่อน merge งาน ให้ดึงโค้ดล่าสุดจาก branch หลักมาอัปเดตในเครื่องของคุณเสมอเพื่อหลีกเลี่ยง Conflict บานปลาย
*   **อย่า Commit ไฟล์เก็บความลับ (Secrets)**: เผื่อแผ่ความปลอดภัยโดยหลีกเลี่ยงการ commit ไฟล์อย่าง `.env`, API token หรือ Key ต่างๆ ขึ้นสู่ repository ให้เพิ่มไฟล์เหล่านี้ใน `.gitignore` ทุกครั้งก่อนตั้งต้นโปรเจกต์
*   **ใช้งาน Branch ให้เหมาะสม**: แยกการทำงานไปทำบน branch ย่อย เช่น `feature/my-feature` หรือ `bugfix/issue-name` เสมอ ห้าม commit ทับลงบน branch `main` หรือ `master` ตรง ๆ โดยเด็ดขาด
