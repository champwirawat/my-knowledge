# คู่มือเริ่มต้นใช้งาน Git Flow (Git Flow Beginner Guide)

คู่มือการเรียนรู้ **Git Flow** สำหรับผู้เริ่มต้นฉบับนี้จัดทำขึ้นเพื่อให้เข้าใจแนวคิดของ branching model แบบ Git Flow และการใช้งานเครื่องมือ `git-flow` CLI จริงแบบทีละขั้นตอน เพื่อให้สามารถนำไปประยุกต์ใช้กับโปรเจกต์พัฒนาซอฟต์แวร์ทั่วไปได้

---

## 1. Git Flow คืออะไร?

**Git Flow** คือ **branching model** — แบบแผนการตั้งชื่อและจัดการ branch ใน Git — ที่ Vincent Driessen นำเสนอไว้ในบทความ "A successful Git branching model" (2010) แนวคิดหลักคือแบ่ง branch ออกเป็นบทบาทชัดเจน แต่ละ branch มีหน้าที่ จุดเริ่มต้น และปลายทางของตัวเอง ทำให้ทีมรู้ตรงกันว่าโค้ดแต่ละส่วนอยู่ในสถานะไหน

Git Flow ประกอบด้วย branch 2 กลุ่ม:

### Branch ถาวร (มีตลอดอายุโปรเจกต์)

| Branch | บทบาท |
| --- | --- |
| `main` (หรือ `master`) | โค้ด **production** เท่านั้น — ทุก commit บน branch นี้คือเวอร์ชันที่ release แล้ว และควรถูกแปะ tag เวอร์ชันเสมอ |
| `develop` | โค้ด **กำลังพัฒนา** — จุดรวมของ feature ทั้งหมดที่จะออกใน release ถัดไป |

### Branch ชั่วคราว (สร้างเมื่อต้องใช้ จบงานแล้วลบ)

| Branch | แตกจาก | merge กลับเข้า | ใช้เมื่อ |
| --- | --- | --- | --- |
| `feature/*` | `develop` | `develop` | พัฒนา feature ใหม่ |
| `bugfix/*` | `develop` | `develop` | แก้บั๊กที่พบระหว่างพัฒนา |
| `release/*` | `develop` | `main` + `develop` | เตรียมออกเวอร์ชันใหม่ (bump version, แก้จุกจิกก่อนปล่อย) |
| `hotfix/*` | `main` | `main` + `develop` | แก้บั๊กด่วนบน production โดยไม่รอรอบ release |
| `support/*` | tag/commit เก่า | (ไม่ merge กลับ) | ดูแลเวอร์ชันเก่าที่ยังต้อง support (ใช้น้อยมาก) |

ภาพรวมการไหลของโค้ด:

```
main    ──●───────────────●──────────●──→   (ทุก ● = release + tag)
           \             ↗ \        ↗
release     \           ●   \      ●
             \         ↗     \    ↗ hotfix (แตกจาก main)
develop ──●──●──●──●──●───●──●──●──→
              \       ↗
feature        ●──●──●
```

**จุดแข็ง:** เหมาะกับซอฟต์แวร์ที่ออกเป็นเวอร์ชัน (มี release cycle ชัดเจน) และทีมที่ต้องการเส้นแบ่งชัดระหว่าง "กำลังทำ" กับ "ปล่อยแล้ว"

**จุดที่ควรรู้:** สำหรับทีมที่ deploy ต่อเนื่องวันละหลายครั้ง (continuous deployment) โมเดลนี้อาจซับซ้อนเกินจำเป็น — ทีมลักษณะนั้นมักใช้ GitHub Flow (branch เดียว + PR) แทน เลือกโมเดลให้ตรงกับจังหวะการ release ของโปรเจกต์

---

## 2. git-flow CLI คืออะไร?

Branching model ข้างต้นทำด้วยคำสั่ง Git ธรรมดา (`checkout -b`, `merge`, `tag`) ได้ทั้งหมด แต่ต้องพิมพ์หลายคำสั่งและจำลำดับให้ถูก **git-flow CLI** คือเครื่องมือที่ห่อขั้นตอนเหล่านั้นให้เหลือคำสั่งเดียว เช่น `git flow feature finish` จะ merge กลับเข้า `develop`, ลบ branch และสลับกลับไป `develop` ให้อัตโนมัติ

รุ่นที่แนะนำคือ **git-flow AVH Edition** (`gitflow-avh`) ซึ่งเป็น fork ที่ยังดูแลต่อเนื่องและมีคำสั่งครบกว่ารุ่นดั้งเดิม

### การติดตั้ง

```bash
# macOS (Homebrew — ได้ AVH Edition อยู่แล้ว)
brew install git-flow-avh

# Ubuntu/Debian
apt-get install git-flow

# ตรวจสอบการติดตั้ง
git flow version    # ควรเห็นเลขเวอร์ชันตามด้วย (AVH Edition)
```

---

## 3. เริ่มต้นใช้งาน: `git flow init`

รันครั้งเดียวต่อ repo (ทุกคนในทีมต้องรันในเครื่องตัวเอง เพราะ config เก็บใน `.git/config` ซึ่งไม่ถูก push ไปยัง repository):

```bash
git flow init
```

ระบบจะถามชื่อ branch และ prefixทีละตัว — กด Enter เพื่อรับค่า default ได้ทั้งหมด:

```
Branch name for production releases: [main]
Branch name for "next release" development: [develop]
Feature branches? [feature/]
Bugfix branches? [bugfix/]
Release branches? [release/]
Hotfix branches? [hotfix/]
Support branches? [support/]
Version tag prefix? []        ← ถ้าอยากให้ tag เป็น v1.2.3 ให้ใส่ "v"
```

หรือรันแบบไม่ต้องตอบคำถาม (รับค่าเริ่มต้นทั้งหมด):

```bash
git flow init -d
```

ถ้า repo ยังไม่มี branch `develop` คำสั่งนี้จะสร้างให้อัตโนมัติ สามารถตรวจสอบ config ที่ตั้งไว้ได้ด้วย:

```bash
git config --get-regexp gitflow
```

---

## 4. Feature: งานหลักในชีวิตประจำวัน

### 4.1 เริ่ม feature ใหม่

```bash
git flow feature start login-form
```

เทียบเท่ากับ `git checkout -b feature/login-form develop` — สังเกตว่าใส่แค่ชื่อ **ไม่ต้องใส่ prefix** (`feature/`) จากนั้นก็ commit งานตามปกติบน branch นี้

### 4.2 แชร์ branch ขึ้น remote (publish)

```bash
git flow feature publish login-form
```

เป็นการ push branch ขึ้น origin พร้อมตั้ง tracking ให้ — ใช้เมื่อต้องการให้เพื่อนร่วมทีมเห็น หรือเมื่อต้องการเปิด Pull Request

ฝั่งเพื่อนร่วมทีมที่ต้องการดึง branch นั้นมาทำต่อในเครื่องตัวเองให้รัน:

```bash
git flow feature track login-form
```

### 4.3 จบ feature

```bash
git flow feature finish login-form
```

คำสั่งเดียวทำ 3 อย่างอัตโนมัติ: merge `feature/login-form` เข้า `develop` → ลบ branch นี้ในเครื่อง → สลับกลับมา `develop`

> **ทางเลือกสำหรับทีมที่รีวิวโค้ดผ่าน Pull Request:** ไม่ต้องใช้ `finish` — ให้ใช้ `publish` แล้วเปิด PR จาก `feature/login-form` เข้า `develop` บน GitHub/GitLab แทน เมื่อ PR ถูก merge แล้วค่อยลบ branch ในเครื่องด้วย `git flow feature delete login-form` วิธีนี้ทำให้ได้ทำการ Code Review และผ่าน CI ก่อน merge ซึ่งจะปลอดภัยและเหมาะกับงานทีมมากกว่า

### 4.4 คำสั่งเสริมที่มีให้ทุก branch type

```bash
git flow feature list              # ดูรายชื่อ feature ที่ค้างอยู่
git flow feature checkout login    # สลับไป feature/login (พิมพ์ชื่อย่อได้ถ้าไม่ซ้ำ)
git flow feature rebase login      # rebase branch ทับ develop ล่าสุด
git flow feature diff login        # ดู diff เทียบกับ develop
git flow feature delete login      # ลบ branch โดยไม่ merge (กรณีจะทิ้งงานนั้น)
```

สำหรับ `bugfix` จะมีคำสั่งชุดเดียวกันทุกตัว ต่างกันแค่เปลี่ยนคำว่า `feature` เป็น `bugfix` เท่านั้น

---

## 5. Release: ออกเวอร์ชันใหม่

เมื่อ `develop` มี feature ครบตามแผนที่จะออกเวอร์ชันใหม่แล้ว:

### 5.1 เริ่ม release

```bash
git flow release start 1.2.0
```

สร้าง `release/1.2.0` แตกจาก `develop` — ตั้งแต่จุดนี้ `develop` จะเริ่มทำ feature ใหม่ต่อได้ทันที ส่วนบน branch release นี้จะทำเฉพาะ:

*   Bump เลขเวอร์ชัน (เช่น แก้ไขใน `package.json`)
*   อัปเดต CHANGELOG
*   แก้ไขบั๊กจุกจิกที่พบระหว่างการทดสอบรอบสุดท้าย — **ห้ามเพิ่ม feature ใหม่เด็ดขาด**

### 5.2 จบ release

```bash
git flow release finish 1.2.0
```

คำสั่งนี้จะทำขั้นตอนต่อไปนี้ให้โดยอัตโนมัติ:

1.  merge `release/1.2.0` เข้า `main`
2.  สร้าง **tag** `1.2.0` บน `main`
3.  merge กลับเข้า `develop` (เพื่อป้องกันไม่ให้เวอร์ชันใหม่ที่แก้บน release หายไปเมื่อพัฒนาต่อ)
4.  ลบ branch `release/1.2.0`

ระหว่างทาง Git จะเปิด editor ขึ้นมาให้พิมพ์ commit message ของการ merge และ tag message — ถ้าอยากรันแบบไม่ interactive (เช่น ใน CI หรือ Script):

```bash
GIT_MERGE_AUTOEDIT=no git flow release finish -m "Release 1.2.0" 1.2.0
```

### 5.3 อย่าลืม push ผลลัพธ์ขึ้น server

คำสั่ง `finish` ทำงานเฉพาะในเครื่อง local เท่านั้น — เราต้องสั่ง push ทั้ง branch และ tag เองด้วย:

```bash
git push origin main develop --follow-tags
```

---

## 6. Hotfix: แก้ด่วนบน Production

ใช้เมื่อพบบั๊กร้ายแรงบน production แต่ `develop` มีงานใหม่ที่ยังไม่พร้อมปล่อยปะปนอยู่ จึงจำต้องแตก branch จาก `main` โดยตรง:

```bash
git flow hotfix start 1.2.1        # แตก hotfix/1.2.1 จาก main
# ...ทำการแก้ไขบั๊ก, commit, bump version เป็น 1.2.1...
git flow hotfix finish 1.2.1       # merge เข้า main + tag 1.2.1 + merge กลับ develop
git push origin main develop --follow-tags
```

ชื่อ hotfix นิยมใช้เลขเวอร์ชัน patch ถัดไป (เช่น 1.2.0 → 1.2.1) เพราะระบบจะเอาชื่อนี้ไปตั้งเป็น tag เมื่อ finish

---

## 7. สรุปวงจรชีวิต (Life Cycle) ของ Git Flow ครบ 1 รอบ

```bash
# --- พัฒนา feature ---
git flow feature start awesome-thing
git commit -am "Implement awesome feature"
git flow feature publish awesome-thing  # (ส่งไปเปิด PR บน GitHub)
git flow feature finish awesome-thing   # (หรือ merge ผ่าน PR แล้วลบ branch)

# --- เตรียมออกเวอร์ชันใหม่ ---
git flow release start 1.0.0
# อัปเดต version/CHANGELOG + commit
git flow release finish -m "Release 1.0.0" 1.0.0
git push origin main develop --follow-tags

# --- เกิดเหตุบั๊กด่วนบน Production ---
git flow hotfix start 1.0.1
# ทำการแก้ไข + commit
git flow hotfix finish -m "Hotfix 1.0.1" 1.0.1
git push origin main develop --follow-tags
```

---

## 8. ข้อควรระวังและเทคนิค (Tips)

*   **คำสั่ง `finish` เขียนทับ branch หลักโดยตรง** — `release finish` และ `hotfix finish` จะ merge ลง `main` และ `develop` ทันที หากทีมมีการป้องกัน branch (branch protection) ไว้บน GitHub/GitLab จะส่งผลให้ push ไม่ผ่าน ควรเปลี่ยนไปใช้วิธีเปิด Pull Request แทน
*   **อย่าใส่ prefix ในคำสั่ง** — รัน `git flow feature start feature/login` จะส่งผลให้ได้ branch ซ้อนกันเป็น `feature/feature/login`
*   **ตั้งชื่อ branch แบบ kebab-case** — เช่น `user-profile-page` ซึ่งเป็นมาตรฐานที่อ่านง่ายที่สุด
*   **Configuration เก็บแยกรายเครื่อง** — สมาชิกใหม่ในทีมเมื่อ clone repo ไปแล้ว ต้องรัน `git flow init -d` ก่อนหนึ่งครั้งในเครื่องตัวเอง
*   **branch release ควรมีอายุสั้น** — ยิ่งเปิดค้างไว้นาน `develop` จะยิ่งห่างออกไปเรื่อย ๆ ทำให้เกิด conflict ตอน merge กลับยาก
*   **การจัดการเมื่อเจอ Conflict ตอน Finish** — ถ้า merge ชนกัน ระบบจะหยุดและแจ้งเตือน ให้เราเข้าไปแก้ conflict ตามปกติ → รัน `git commit` → จากนั้นรันคำสั่ง finish เดิมซ้ำอีกครั้ง ระบบจะทำงานส่วนที่เหลือต่อให้อัตโนมัติ

---

## 9. ตารางเปรียบเทียบคำสั่ง git-flow กับ Git ธรรมดา

การเข้าใจเบื้องหลังจะช่วยให้แก้ไขปัญหาเฉพาะหน้าได้ง่ายขึ้นเมื่อระบบทำงานอัตโนมัติของ CLI ไม่สมบูรณ์:

| คำสั่ง git-flow | เทียบเท่า Git ธรรมดา (โดยประมาณ) |
| --- | --- |
| `git flow feature start x` | `git checkout -b feature/x develop` |
| `git flow feature publish x` | `git push -u origin feature/x` |
| `git flow feature finish x` | `git checkout develop && git merge --no-ff feature/x && git branch -d feature/x` |
| `git flow release finish 1.0.0` | merge เข้า `main` → `git tag 1.0.0` → merge กลับ `develop` → ลบ branch |

*หมายเหตุ: git-flow ใช้ `--no-ff` (no fast-forward) เสมอตอน merge เพื่อให้ประวัติของ Git แสดงเป็นกลุ่มก้อน (merge commit) ชัดเจนว่าแต่ละ feature ถูกรวมเข้ามาเมื่อใด*

---

## 10. แหล่งอ้างอิงและศึกษาเพิ่มเติม

*   บทความต้นฉบับ: [A successful Git branching model — Vincent Driessen](https://nvie.com/posts/a-successful-git-branching-model/)
*   git-flow AVH Edition: [github.com/petervanderdoes/gitflow-avh](https://github.com/petervanderdoes/gitflow-avh)
*   Cheatsheet สรุปภาพประกอบ: [danielkummer.github.io/git-flow-cheatsheet](https://danielkummer.github.io/git-flow-cheatsheet/)
