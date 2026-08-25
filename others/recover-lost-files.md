# กู้ไฟล์ที่หายหรือลบพลาด (File Recovery Playbook)

ไฟล์หายมีหลายแบบ — `rm` พลาด, `git checkout` ทับ, stash pop ผิด, หรือหายไปเฉยๆ โดยไม่รู้ว่าใครลบ
วิธีกู้ **ขึ้นอยู่กับว่า git รู้จักไฟล์นั้นหรือไม่** ถ้าเดาผิดตั้งแต่ต้นจะเสียเวลาไล่ผิดทาง

บทความนี้เรียงวิธีตามลำดับที่ควรลอง — ถูกและเร็วก่อน ไล่ไปหาวิธีที่ยากขึ้น

---

## 0. แยกประเภทไฟล์ก่อน (สำคัญที่สุด)

```bash
# ไฟล์นี้ git track อยู่ไหม (ผ่าน = tracked)
git ls-files --error-unmatch path/to/file

# ไฟล์นี้โดน ignore อยู่ไหม (ตอบกลับมา = โดน ignore)
git check-ignore -v path/to/file
```

| ผลลัพธ์ | ความหมาย | ไปต่อที่ |
| --- | --- | --- |
| tracked | git มีประวัติไฟล์นี้ กู้ได้เกือบ 100% | ข้อ 1 |
| untracked / ignored | git ไม่เคยเห็นไฟล์นี้เลย กู้จาก git **ไม่ได้** | ข้อ 2 เป็นต้นไป |

> ไฟล์ประเภท `config.yaml`, `.env`, `local_config.yaml` มักอยู่ใน `.gitignore` — พวกนี้ git ช่วยอะไรไม่ได้เลย ต้องข้ามไปข้อ 2 ทันที

---

## 1. ไฟล์ที่ git track อยู่

### คำสั่งที่ใช้บ่อย

| สถานการณ์ | คำสั่ง |
| --- | --- |
| ลบไปแล้วแต่ยังไม่ commit | `git restore path/to/file` |
| แก้ทับไปแล้วอยากได้ของเดิม | `git checkout HEAD -- path/to/file` |
| ลบแล้ว commit ไปแล้ว | หา commit ที่ลบ แล้วดึงเนื้อไฟล์จาก commit ก่อนหน้า |
| เคย `git add` แต่ commit ไม่ทัน | `git fsck --lost-found` หา dangling blob |
| stash หาย / pop ผิด | `git fsck --unreachable` หา stash commit ที่หลุด |
| branch หาย / reset แรงไป | `git reflog` (เก็บย้อนหลัง ~90 วัน) |

### ตัวอย่างการใช้งาน

```bash
# ลบแล้ว commit ไปแล้ว — หา commit ที่ลบไฟล์นี้
git log --diff-filter=D --oneline -- path/to/file

# ดึงเนื้อไฟล์จาก commit "ก่อนหน้า" commit ที่ลบ (สังเกตเครื่องหมาย ^)
git show <sha>^:path/to/file > path/to/file

# เคย git add ไว้แล้วไฟล์หาย — blob ยังอยู่ใน object database
git fsck --lost-found            # ได้ list ของ dangling blob
git cat-file -p <blob-sha> > recovered.txt

# stash หาย (git stash list ว่างเปล่าแต่มั่นใจว่าเคย stash)
git fsck --unreachable | grep commit
git show <commit-sha>            # ดูว่าใช่ตัวที่ต้องการไหม
git stash apply <commit-sha>

# ย้อนดูทุกการเคลื่อนไหวของ HEAD (กู้ branch ที่ reset/delete ไป)
git reflog
git checkout -b recovered <sha>
```

---

## 2. Local History ของ Editor ⭐

**วิธีนี้กู้ไฟล์ที่ git ไม่ track ได้** — VS Code / Cursor เก็บ snapshot ของไฟล์ **ทุกครั้งที่กด save** โดยไม่สนใจ `.gitignore` เลย

### ตำแหน่งเก็บข้อมูล (macOS)

| Editor | Path |
| --- | --- |
| Cursor | `~/Library/Application Support/Cursor/User/History/` |
| VS Code | `~/Library/Application Support/Code/User/History/` |
| JetBrains | `~/Library/Application Support/JetBrains/<IDE>/LocalHistory/` |

แต่ละโฟลเดอร์ย่อยคือ 1 ไฟล์ ข้างในมี `entries.json` บอก path จริงกับ timestamp ของทุก snapshot

### ตัวอย่างการใช้งาน

```bash
# 1) หาโฟลเดอร์ history ของไฟล์ที่ต้องการ (ใส่ path บางส่วนก็พอ)
grep -rl "canalone-nex-ai/config.yaml" \
  ~/Library/Application\ Support/Cursor/User/History

# 2) ดูรายการ snapshot ทั้งหมดพร้อมเวลา
cat ~/Library/Application\ Support/Cursor/User/History/88e5a1e/entries.json | jq .

# 3) เทียบว่า snapshot ไหนใช่ แล้ว copy กลับ
cp ~/Library/Application\ Support/Cursor/User/History/88e5a1e/wRu3.yaml ./config.yaml
```

ถ้าไม่อยากใช้เทอร์มินัล: คลิกขวาที่ไฟล์ใน editor → **Timeline** (VS Code/Cursor) หรือ **Local History → Show History** (JetBrains)

> **ข้อจำกัด:** เก็บเฉพาะไฟล์ที่ "เซฟผ่าน editor" เท่านั้น — ถ้าแก้ด้วย `vim`, `sed`, หรือ heredoc ในเทอร์มินัล การแก้รอบนั้นจะไม่มีใน history

---

## 3. ถังขยะและ Snapshot ของ macOS

```bash
# ลบผ่าน Finder จะอยู่ในถังขยะ (rm ในเทอร์มินัล "ไม่" เข้าถังขยะ)
ls -la ~/.Trash

# APFS local snapshot — มีอยู่แม้ไม่ได้ตั้งปลายทาง Time Machine ไว้
tmutil listlocalsnapshots /
```

เจอ snapshot ที่ต้องการแล้ว mount ขึ้นมาอ่าน:

```bash
mkdir /tmp/snap
mount_apfs -s com.apple.TimeMachine.2026-08-21-120000 / /tmp/snap
cp /tmp/snap/Users/<user>/path/to/file ./file
umount /tmp/snap
```

---

## 4. กวาดหาสำเนาทั้งเครื่อง

```bash
# Spotlight เร็วที่สุด (ใช้ index ไม่ต้องไล่ดิสก์)
mdfind -name config.yaml

# ไล่หาเองเมื่อ Spotlight ไม่ครอบคลุม (เช่นโฟลเดอร์ที่ถูก exclude)
find ~ -name 'config*.yaml' -not -path '*/node_modules/*' 2>/dev/null
```

จุดที่มักมีสำเนาซ่อนอยู่โดยไม่รู้ตัว:

- โฟลเดอร์ชั่วคราวในโปรเจกต์เอง — `.temp/`, `.bak/`, `*.example`
- repo ฝั่ง deploy / infra ที่เก็บ config ของ environment
- container ที่ยังรันอยู่ — `docker cp <container>:/app/config.yaml .`
- เครื่อง staging / server ที่ deploy ตัวเดียวกันไว้
- iCloud Drive / Dropbox / Google Drive — มี version history ของตัวเอง

---

## 5. เศษที่หลงเหลือในบันทึกต่างๆ

ถ้าเคยเปิดอ่านไฟล์นั้นมาก่อน เนื้อไฟล์อาจยังค้างอยู่ในบันทึกเหล่านี้:

- **Terminal scrollback** — เลื่อนขึ้นไปหา `cat` / `less` ครั้งล่าสุด
- **Transcript ของ AI agent** — `~/.claude/projects/<project>/*.jsonl` (ถ้าเคยให้ agent อ่านไฟล์นั้น เนื้อไฟล์อยู่ในนั้น)
- **Log ของเครื่องมือ** เช่น RTK — `~/Library/Application Support/rtk/tee/`

อยากรู้ว่าไฟล์หายตอนไหนเพราะอะไร:

```bash
grep -n "rm \|mv " ~/.zsh_history | tail -30
```

---

## 6. ป้องกันไว้ก่อน (คุ้มกว่ากู้มาก)

```bash
# สำเนา config ที่ gitignore ไว้นอก repo
mkdir -p ~/.config/myproject
cp config.yaml ~/.config/myproject/config.yaml.bak

# ล็อกไฟล์กัน rm พลาด (ปลดล็อกด้วย nouchg)
chflags uchg config.yaml
chflags nouchg config.yaml
```

**วิธีที่ยั่งยืนกว่า:** ทำ repo ส่วนตัวเก็บ config ของทุกโปรเจกต์ (เช่น `~/dotfiles-work`) แล้ว symlink เข้าไปในโปรเจกต์ — ในโปรเจกต์ยังเป็นไฟล์ที่ถูก ignore เหมือนเดิม แต่มี git ของตัวเองคุ้มประวัติให้

```bash
ln -s ~/dotfiles-work/nex-ai/config.yaml ./config.yaml
```

> ถ้า config มี secret อยู่ด้วย ให้เข้ารหัสก่อน commit — ดู [SOPS](/others/sops)

---

## เคสจริง: `config.yaml` ที่หายไป

**อาการ:** รัน `python main.py` แล้วเจอ

```
psycopg2.OperationalError: connection to server at "localhost" (::1), port 5432 failed:
FATAL:  password authentication failed for user "ioc"
```

**การวิเคราะห์:** user `ioc` กับ host `localhost` ไม่ตรงกับที่ตั้งไว้ในโปรเจกต์เลย ไปดูโค้ดโหลด config เจอว่า

```python
if not config_path.exists():
    return AppConfig()      # ตกไปใช้ค่า default ทั้งชุด
```

`AppConfig()` default คือ `localhost:5432` user `ioc` — **ตรงกับ error เป๊ะ** แปลว่าไฟล์ config หายไปจริง ไม่ใช่รหัสผ่านผิด

**ผลตรวจ:** ทั้ง `config.yaml` และ `local_config.yaml` หายจากโฟลเดอร์ และทั้งคู่อยู่ใน `.gitignore` → git กู้ไม่ได้ (ข้อ 0)

**วิธีที่ใช้กู้:** Cursor Local History (ข้อ 2) — เจอ snapshot ล่าสุด 262 บรรทัด ยืนยันว่าใช่ตัวจริงด้วยการเทียบเนื้อหาบรรทัดที่จำได้ แล้ว copy กลับ ใช้เวลารวมไม่ถึง 2 นาที

**บทเรียน:**

1. อ่าน error ให้ออกว่า "ค่าที่โผล่มาคือค่า default หรือค่าที่เราตั้ง" — ถ้าเป็น default แปลว่า config ไม่ถูกโหลด ไม่ใช่ config ผิด
2. ไฟล์ที่ `.gitignore` คือไฟล์ที่ **ไม่มีตาข่ายรองรับ** ต้อง backup แยกเสมอ (ข้อ 6)
3. `git stash -u` **ไม่เก็บไฟล์ที่ถูก ignore** (ต้องใช้ `-a` ถึงจะเก็บ) — ดังนั้น stash ไม่ใช่ผู้ต้องสงสัยในเคสนี้

---

## 📚 Learning Resources

- [Basic Git](/git/basic-git) — คำสั่ง git พื้นฐาน
- [git-scm: Data Recovery](https://git-scm.com/book/en/v2/Git-Internals-Maintenance-and-Data-Recovery)
- [VS Code: Timeline View](https://code.visualstudio.com/docs/editor/versioncontrol#_timeline-view)
- [Apple: tmutil man page](https://ss64.com/mac/tmutil.html)
