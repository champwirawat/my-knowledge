# Python พื้นฐาน — `@contextmanager`

เอกสารชุดนี้รวบรวมหัวข้อ Python พื้นฐาน โดยเริ่มจาก **`contextlib.contextmanager`** (`@contextmanager`) ว่าใช้ทำอะไร และเมื่อไหร่ควรใช้

## สารบัญ

1. [Context manager คืออะไร](#_1-context-manager-คืออะไร)
2. [`@contextmanager` ใช้ทำอะไร](#_2-contextmanager-ใช้ทำอะไร)
3. [ลำดับการทำงาน (ก่อน / หลัง `yield`)](#_3-ลำดับการทำงาน-ก่อน--หลัง-yield)
4. [ตัวอย่างที่พบบ่อย](#_4-ตัวอย่างที่พบบ่อย)
5. [ข้อควรระวัง](#_5-ข้อควรระวัง)
6. [สรุป](#_6-สรุป)

---

## 1\. Context manager คืออะไร

**Context manager** คือออบเจ็กต์ที่กำหนด “ช่วงเวลา” ของการใช้ทรัพยากรหรือสถานะ — มีขั้น **เข้า** (setup) และ **ออก** (teardown) ที่จะรันแน่นอนเมื่อจบบล็อก `with`

ใน Python เราเขียนแบบนี้:

```python
with open("data.txt", encoding="utf-8") as f:
    text = f.read()
# หลังออกจาก with ไฟล์ถูกปิดให้อัตโนมัติ (แม้เกิดข้อผิดพลาดใน try ภายใน)
```

ตัว `open(...)` คืนค่า context manager — ภายในมี `__enter__` / `__exit__` (หรือเทียบเท่า) ที่รันตามลำดับ

---

## 2\. `@contextmanager` ใช้ทำอะไร

`@contextmanager` จากโมดูล **`contextlib`** ใช้ **เขียน context manager แบบฟังก์ชัน** โดยไม่ต้องสร้างคลาสที่มี `__enter__` และ `__exit__`

แนวคิดคือเขียน **ฟังก์ชัน generator** ที่:

1. รันโค้ด **ก่อน** `yield` → เทียบเท่า **`__enter__`** (เตรียมทรัพยากร / เปลี่ยนสถานะ)
2. ค่าที่ **yield** ออกไป → คือค่าที่ได้จาก `as ...` ใน `with ... as x:`
3. รันโค้ด **หลัง** `yield` (เมื่อออกจากบล็อก `with`) → เทียบเท่า **`__exit__`** (เก็บกวาด / คืนค่า)

```python
from contextlib import contextmanager


@contextmanager
def managed_resource():
    print("setup")
    try:
        yield "resource"  # ค่านี้ไปที่ตัวแปรหลัง as
    finally:
        print("teardown")


with managed_resource() as r:
    print("inside:", r)
# setup
# inside: resource
# teardown
```

**สรุปสั้นๆ:** `@contextmanager` ใช้ **แปลง generator ที่มี `yield` ตัวเดียว** ให้กลายเป็น context manager ที่ใช้กับ `with` ได้ — เหมาะเมื่อ logic เข้า–ออกอยู่ในฟังก์ชันเดียวและอ่านง่ายกว่าการเขียนคลาส

---

## 3\. ลำดับการทำงาน (ก่อน / หลัง `yield`)

| ลำดับ | เมื่อไหร่ | ทำอะไร |
|--------|-----------|--------|
| 1 | เข้า `with` | รันโค้ดจนถึง `yield` |
| 2 | อยู่ในบล็อก `with` | ใช้ค่าที่ `yield` และรันโค้ดของผู้เรียก |
| 3 | ออกจาก `with` (ปกติหรือมี exception) | รันส่วนหลัง `yield` (ควรใช้ `try` / `finally` เพื่อให้ teardown รันเสมอ) |

ถ้าใน `with` มี exception ที่ไม่ถูกดักใน generator ระบบจะ **ส่ง exception เข้า generator** ที่จุด `yield` — ถ้าไม่จัดการ อาจทำให้ส่วน teardown ไม่รันตามที่ต้องการ ดังนั้นมักใช้ **`try` / `finally`** รอบ `yield` เพื่อให้ cleanup แน่นอน

---

## 4\. ตัวอย่างที่พบบ่อย

### 4.1 จำกัดเวลา / วัดเวลา

```python
import time
from contextlib import contextmanager


@contextmanager
def timer(label: str):
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f"{label}: {elapsed:.3f}s")


with timer("heavy work"):
    time.sleep(0.1)
```

### 4.2 เปลี่ยนค่าชั่วคราวแล้วคืน (เช่น `os.chdir`)

```python
import os
from contextlib import contextmanager


@contextmanager
def chdir(path: str):
    prev = os.getcwd()
    os.chdir(path)
    try:
        yield
    finally:
        os.chdir(prev)
```

### 4.3 แทนที่ stdout ชั่วคราว (เทส / logging)

รูปแบบเดียวกัน: เก็บของเดิม → ตั้งค่าใหม่ → `yield` → ใน `finally` คืนค่าเดิม

---

## 5\. ข้อควรระวัง

- **`yield` ควรมีแค่ครั้งเดียว** ต่อการเรียก context หนึ่งครั้ง — ถ้า `yield` ซ้ำหรือไม่ `yield` จะไม่ตรงกับสัญญาของ context manager
- **Teardown ที่สำคัญ** ใส่ใน **`finally`** เพื่อให้รันแม้มี exception
- ถ้า logic ซับซ้อนมาก (หลายสถานะ, หลาย exception type) บางที **คลาสที่มี `__enter__` / `__exit__`** อ่านและควบคุมได้ชัดกว่า

---

## 6\. สรุป

- **Context manager** คือกลไกของ `with` สำหรับ setup/teardown ที่ปลอดภัยแม้มี error
- **`@contextmanager`** ใช้เขียน context manager แบบ **generator + `yield` จุดเดียว** แทนคลาส — ส่วนก่อน `yield` = เข้า context, หลัง `yield` = ออก context
- เหมาะกับงาน **จำกัดขอบเขตชัด** เช่น จับเวลา, สลับ working directory, mock ทรัพยากรชั่วคราว

หัวข้อถัดไปในชุด Python พื้นฐาน (ถ้าต้องการต่อ) สามารถเพิ่มในไฟล์อื่นหรือขยายไฟล์นี้ได้ เช่น `contextlib` ตัวอื่น (`closing`, `suppress`, `ExitStack`)
