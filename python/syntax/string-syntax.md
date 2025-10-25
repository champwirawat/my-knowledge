<div style="text-align: right; color: #6c757d; font-size: 12px; margin-bottom: 20px;">
Updated at 25/10/2025
</div>

# String Syntax

--------------------------------------------------------------------------------

## `"""..."""` Multi-line String

**คุณสมบัติ:**

- เก็บข้อความหลายบรรทัด
- รองรับการขึ้นบรรทัดใหม่ (line breaks)
- ใช้เก็บ docstrings, comments หลายบรรทัด

```python
text = """
นี่คือข้อความ
ที่สามารถครอบคลุมหลายบรรทัด
ไม่มีตัวประมวลผลพิเศษ
"""
print(text)

# Output:
นี่คือข้อความ
ที่สามารถครอบคลุมหลายบรรทัด
ไม่มีตัวประมวลผลพิเศษ
```

--------------------------------------------------------------------------------

## `r"""..."""` Raw Multi-line String

**คุณสมบัติ:**

- ทำงานเหมือน `"""..."""` แต่ไม่แปลอักขระพิเศษ
- `\n`, `\t`, `\\` ไม่ถูกแปลง
- เหมาะสำหรับ regex patterns, Windows paths

```python
path = r"""
C:\Users\username\Documents
\n= newline
\t= tab
"""
print(path)

# Output: 
C:\Users\username\Documents
\n= newline
\t= tab
```

--------------------------------------------------------------------------------

## `f"""..."""` Formatted Multi-line String

**คุณสมบัติ:**

- ทำงานเหมือน `"""..."""` แต่สามารถฝังโค้ด Python
- `{variable}`, `{expression}` จะถูกประมวลผล
- ใช้ `f"""..."""` หรือ `rf"""..."""` (raw + f-string)

```python
name = "สมชาย"
age = 25
info = f"""
ชื่อ: {name}
อายุ: {age}
ปีเกิด: {2024 - age}
"""
print(info)

# Output:
ชื่อ: สมชาย
อายุ: 25
ปีเกิด: 1999
```
