# คู่มือเริ่มต้นใช้งาน Supabase (Supabase Beginner Guide)

คู่มือการเรียนรู้ **Supabase** สำหรับผู้เริ่มต้นฉบับนี้จัดทำขึ้นเพื่อให้เข้าใจสถาปัตยกรรม แนวคิดพื้นฐาน ระบบความปลอดภัย และขั้นตอนการเริ่มต้นใช้งานจริงแบบทีละขั้นตอน เพื่อนำไปประยุกต์ใช้ในการพัฒนาซอฟต์แวร์ทั่วไปได้

---

## 1. Supabase คืออะไร?

**Supabase** คือแพลตฟอร์ม **Backend-as-a-Service (BaaS)** แบบ Open-source ที่ช่วยให้นักพัฒนาสร้างระบบ Backend ได้อย่างรวดเร็วโดยไม่ต้องเขียนโค้ดฝั่ง Server เองทั้งหมด ตั้งแต่ระบบจัดการฐานข้อมูล ไปจนถึงการจัดการสิทธิ์การเข้าถึงข้อมูลและการอัปโหลดไฟล์ โดยมีบริการหลักดังนี้:

*   **Database (PostgreSQL):** ฐานข้อมูลแบบ Relational Database ที่ทรงพลังและมีเสถียรภาพสูง รองรับฟีเจอร์ขั้นสูง เช่น SQL Views, Triggers, Functions และ Indexes
*   **Authentication:** ระบบสมัครสมาชิกและจัดการผู้ใช้ (ผ่าน Email/Password, Social OAuth เช่น Google/GitHub หรือการเข้ารหัสด้วย OTP)
*   **Row Level Security (RLS):** ระบบความปลอดภัยระดับฐานข้อมูล ช่วยจำกัดและคัดกรองสิทธิ์การเข้าถึงข้อมูลแต่ละแถว (Row) ของผู้ใช้งาน
*   **Storage:** บริการฝากไฟล์และสื่อมีเดียต่าง ๆ เช่น รูปโปรไฟล์ ภาพสินค้า หรือวิดีโอ
*   **Edge Functions & Realtime:** ระบบรันโค้ด Serverless ฝั่ง Backend และการเชื่อมต่อสื่อสารข้อมูลแบบสองทางเรียลไทม์ (Websockets)

---

## 2. แนวคิดฐานข้อมูลที่ต้องเข้าใจ (Database Concepts)

### 2.1 PostgreSQL
หัวใจหลักของ Supabase คือฐานข้อมูล **PostgreSQL** (หรือ Postgres) ทุกคำสั่งในการสร้างตาราง, การเขียนฟังก์ชัน หรือกฎสิทธิ์ความปลอดภัยในระบบ จึงต้องเขียนด้วยไวยากรณ์ SQL มาตรฐาน

ตัวอย่างการสร้างตารางขั้นพื้นฐาน:
```sql
create table workspaces (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  created_at timestamptz not null default now()
);
```
*คำอธิบาย:* สร้างตารางชื่อ `workspaces` โดยคอลัมน์ `id` จะสุ่มรหัส UUID ขึ้นมาโดยอัตโนมัติ คอลัมน์ `name` ห้ามเป็นค่าว่าง และ `created_at` บันทึกวันเวลาปัจจุบันอัตโนมัติทันทีที่สร้างแถวใหม่

### 2.2 ระบบ Auth ของ Supabase และตาราง `auth.users`
เมื่อมีผู้ใช้สมัครสมาชิก ข้อมูลการล็อกอินจะถูกจัดเก็บแยกใน Schema ของระบบที่ชื่อ `auth` ภายในตาราง `auth.users`

เมื่อต้องการเชื่อมข้อมูลผู้ใช้เข้ากับตารางปกติของเรา (เช่น การทำฟิลด์ระบุตัวตนคนบันทึกข้อมูล) ให้ใช้วิธีโยง Foreign Key ไปยังตาราง `auth.users(id)` เสมอ:
```sql
created_by uuid references auth.users(id) on delete set null
```
*คำอธิบาย:* คอลัมน์ `created_by` จะบันทึก ID ของผู้ใช้งานที่ทำรายการ หากในอนาคตผู้ใช้งานรายนี้ถูกลบออกจากระบบ คอลัมน์นี้จะเปลี่ยนเป็นค่าว่าง (`null`) เพื่อคงข้อมูลหลักไว้ไม่ให้โดนลบตามไป (Data Integrity)

### 2.3 ตัวแปรสภาพแวดล้อมและ API Keys (.env)
ในการตั้งค่า Frontend เพื่อเชื่อมต่อกับระบบของ Supabase จะต้องระบุ Keys 2 ชุด ซึ่งควรบันทึกไว้ในไฟล์สิ่งแวดล้อม เช่น `.env` หรือ `.env.local` เพื่อความปลอดภัย:
```bash
NEXT_PUBLIC_SUPABASE_URL=https://your-project-ref.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anonymous-public-key
```

> [!IMPORTANT]
> **ประเภทของ Keys ใน Supabase:**
> 1.  **Anon Key:** คีย์สาธารณะสำหรับเชื่อมต่อจากฝั่ง Frontend (ฝั่ง Browser/Client) มีความปลอดภัยเนื่องจากคีย์นี้จะถูกคัดกรองด้วยนโยบายความปลอดภัย **Row Level Security (RLS)** ของฐานข้อมูลเสมอ
> 2.  **Service Role Key:** คีย์ที่มีสิทธิ์ของระบบผู้ดูแลสูงสุด (Admin) ซึ่งจะ **ข้าม (Bypass) กฎความปลอดภัย RLS ทั้งหมด** ห้ามนำคีย์นี้ไปวางไว้ฝั่ง Client หรือเปิดเผยต่อภายนอกโดยเด็ดขาด ควรเรียกใช้เฉพาะในระบบ Backend ที่ปลอดภัยเท่านั้น เช่น ใน Server-side scripts หรือ Edge Functions

---

## 3. ระบบความปลอดภัยระดับแถว: Row Level Security (RLS)

### 3.1 RLS คืออะไร?
**Row Level Security (RLS)** เป็นคุณสมบัติเด่นของ PostgreSQL ที่ยินยอมให้เราเขียนนโยบายความปลอดภัย (Policy) ควบคุมการเข้าถึงข้อมูลรายแถวได้โดยตรงจากระดับฐานข้อมูล

หากปิดใช้งาน RLS เมื่อมีผู้ส่งคำสั่งดึงข้อมูล เช่น `select * from tasks;` ระบบจะส่งคืนตารางงานของทุกคนกลับไปทั้งหมด ซึ่งผิดหลักความปลอดภัย

เมื่อเปิดใช้งาน RLS เราจะสามารถเขียน Policy คัดกรองข้อมูลได้ เช่น:
```sql
create policy "Users can manage tasks in their workspaces"
on tasks
for all
to authenticated
using (is_workspace_member(workspace_id))
with check (is_workspace_member(workspace_id));
```
*ส่วนประกอบสำคัญ:*
*   `on tasks`: ใช้บังคับกับตาราง `tasks`
*   `for all`: บังคับครอบคลุมทุกคำสั่ง (SELECT, INSERT, UPDATE, DELETE)
*   `to authenticated`: ตรวจสอบสิทธิ์เฉพาะผู้ใช้ที่ผ่านการยืนยันตัวตน (Login) เข้าระบบแล้วเท่านั้น
*   `using`: เงื่อนไขที่ใช้ในการคัดกรองข้อมูลเมื่อผู้ใช้พยายามจะ **อ่าน (Read)** หรือทำการ **แก้ไข/ลบ (Update/Delete)** ข้อมูลเก่า
*   `with check`: เงื่อนไขในการตรวจสอบข้อมูลเมื่อผู้ใช้ต้องการ **เขียนเพิ่ม (Insert)** หรือเขียนทับข้อมูลใหม่เข้าไปในตาราง

### 3.2 ความสำคัญของ RLS ต่อแอปพลิเคชันสมัยใหม่
แอปพลิเคชันสมัยใหม่มักเขียนคำสั่งดึงฐานข้อมูลด้วย Supabase SDK จากฝั่ง Browser (Client) โดยตรง หากไม่เปิดใช้งาน RLS หรือเขียนเงื่อนไขไม่รัดกุม ผู้ใช้ที่มีความรู้สามารถแก้ไขสคริปต์ในคอนโซลของบราวเซอร์เพื่อเข้าถึงหรือทำลายข้อมูลของคนอื่นในระบบได้โดยง่าย RLS จึงทำหน้าที่เป็นเหมือนประตูด่านความปลอดภัยด่านสุดท้ายที่อยู่ติดกับข้อมูลจริง

### 3.3 การเข้าถึงสิทธิ์ผู้ใช้งานปัจจุบันด้วยฟังก์ชัน `auth.uid()`
Supabase เตรียมฟังก์ชันสำเร็จรูปที่ชื่อ `auth.uid()` ไว้เพื่อส่งคืนค่า ID (UUID) ของผู้ใช้ปัจจุบันที่กำลังล็อกอินและส่งคำสั่งขอดึงข้อมูลเข้ามา ทำให้เรานำไปเปรียบเทียบใน Policy ได้ง่ายขึ้น

ตัวอย่างการเขียนฟังก์ชันเช็คการเป็นสมาชิกของพื้นที่ทำงาน (Workspace):
```sql
create or replace function is_workspace_member(target_workspace_id uuid)
returns boolean
language sql
security definer
stable
set search_path = public
as $$
  select exists (
    select 1
    from workspace_members
    where workspace_id = target_workspace_id
      and user_id = auth.uid()
  );
$$;
```
*คำอธิบาย:* ฟังก์ชันนี้จะไปสืบค้นข้อมูลในตาราง `workspace_members` เพื่อหาว่า ID ของผู้ใช้ปัจจุบัน (`auth.uid()`) เป็นสมาชิกในพื้นที่ทำงานดังกล่าวหรือไม่ หากมีข้อมูลอยู่จริงจะตอบกลับว่า `true`

---

## 4. Database Functions และ Remote Procedure Call (RPC)

### 4.1 RPC คืออะไร?
**Remote Procedure Call (RPC)** บน Supabase คือการยินยอมให้โค้ดฝั่ง Client สามารถเรียกใช้งานฟังก์ชัน (Stored Procedure) ที่เขียนด้วยภาษา SQL/PLpgSQL บนฐานข้อมูลได้โดยตรงผ่าน API SDK

**โค้ดบนฐานข้อมูล (Database):**
```sql
create or replace function get_active_tasks_count(p_workspace_id uuid)
returns bigint
language sql
security invoker
stable
as $$
  select count(*)
  from tasks
  where workspace_id = p_workspace_id and status = 'active';
$$;
```

**โค้ดฝั่งแอปพลิเคชัน (JavaScript/TypeScript):**
```typescript
const { data, error } = await supabase.rpc('get_active_tasks_count', {
  p_workspace_id: 'some-workspace-uuid'
});
```

### 4.2 เหตุผลในการใช้งาน RPC
1.  **ความปลอดภัย (Security Context):** ใช้กับงานที่ค่อนข้างละเอียดอ่อนและเราไม่อยากเปิดเผยตารางนั้นออกไปตรง ๆ เช่น การป้อนโค้ดเชิญเข้าร่วมกลุ่ม เราสามารถปิดสิทธิ์ตารางนั้นและเปิดฟังก์ชัน RPC เพื่อรับโค้ดไปเปรียบเทียบและประมวลผลแทนได้
2.  **ประสิทธิภาพความเร็ว (Performance):** การประมวลผลข้อมูลที่ซับซ้อน เช่น การนับจำนวนหรือการ Join ข้อมูลหลายหน้า หากดึงไฟล์ดิบทั้งหมดมาประมวลผลบน Client จะกินเน็ตมาก การรันด้วยฟังก์ชันบนฐานข้อมูลและส่งเฉพาะผลลัพธ์สุดท้ายกลับมาจึงรวดเร็วและใช้แบนด์วิดท์น้อยกว่ามาก
3.  **ควบคุมธุรกรรมฐานข้อมูล (Transaction Control):** กรณีที่ต้องรันสเต็ป SQL หลายขั้นตอนร่วมกัน (เช่น หักยอดเงิน และ บันทึกใบเสร็จ) หากขั้นตอนใดล้มเหลวต้องการให้ยกเลิกทั้งหมด (Rollback) การควบคุมด้วยฟังก์ชันบนฐานข้อมูลจะเสถียรกว่า

### 4.3 ตารางเปรียบเทียบ `security invoker` และ `security definer`

| คุณลักษณะ | `security invoker` (ค่าเริ่มต้น) | `security definer` |
| :--- | :--- | :--- |
| **สิทธิ์ที่ใช้รัน** | รันด้วยสิทธิ์ของผู้ใช้งานปัจจุบันที่ล็อกอินอยู่ | รันด้วยสิทธิ์ของ **ผู้สร้าง** ฟังก์ชัน (ปกติคือแอดมินหรือระบบ `postgres`) |
| **การตรวจ RLS** | **ตรวจสอบ RLS** (มองเห็นเฉพาะแถวที่ผู้ใช้มีสิทธิ์ตาม policy เท่านั้น) | **ข้าม RLS** (มองเห็นและจัดการข้อมูลในตารางทั้งหมดได้โดยไม่สน RLS) |
| **ระดับความปลอดภัย** | สูงมาก ไม่เกิดการทำงานล้นสิทธิ์ของระบบผู้ใช้งาน | ปานกลางถึงต่ำ (เสี่ยงที่จะมีช่องโหว่ให้ดึงข้อมูลสำคัญหากเขียนโค้ดตรวจสอบเงื่อนไขไม่รัดกุมพอ) |
| **ตัวอย่างการใช้** | ฟังก์ชันคำนวณสถิติและเรียกดูรายงานส่วนบุคคล | ขั้นตอนสมัครสมาชิกใหม่ หรือการเคลียร์รหัสชั่วคราวที่ถูก RLS บล็อกไว้ |

> [!WARNING]
> การสร้างฟังก์ชันแบบ `security definer` ต้องระบุคำสั่ง `set search_path = public` เสมอ เพื่อป้องกันความเสี่ยงในการทำ Search Path Hijacking (การสร้าง Schema ปลอมเพื่อหลอกเรียกฟังก์ชันอื่นของระบบ)

---

## 5. อ็อบเจกต์พื้นฐานของฐานข้อมูล (Database Objects)

### 5.1 Tables & Relationships (ตารางและความสัมพันธ์)
การออกแบบระบบที่ดีมักเชื่อมโยงตารางเข้าหากัน เช่น:
*   **ตารางหลัก (Primary Table):** เก็บข้อมูลชิ้นหลัก เช่น `workspaces` (พื้นที่ทำงาน)
*   **ตารางกลางความสัมพันธ์ (Membership/Association Table):** เก็บความสัมพันธ์แบบ Many-to-Many เช่น `workspace_members` เพื่อเชื่อมว่า user รายใดอยู่กลุ่มใดบ้าง
*   **ตารางย่อย/ตารางลูก (Child Table):** เก็บข้อมูลปลีกย่อย เช่น `tasks` ที่อ้างอิงอยู่กับไอดีของ `workspaces`

### 5.2 SQL Views
**View** เป็นเหมือนตารางเสมือน (Virtual Table) ที่บันทึกสเต็ปคิวรี SQL ที่ใช้บ่อยไว้ เพื่อให้กดเรียกใช้งานเปรียบเสมือนตารางปกติตารางหนึ่ง ช่วยซ่อนความซับซ้อนของคำสั่งคิวรีในฝั่งโค้ดได้ดี

```sql
create view high_priority_tasks
with (security_invoker = true)
as
  select id, title, due_date, workspace_id
  from tasks
  where priority = 'high' and status != 'completed';
```
> [!TIP]
> ใน PostgreSQL แนะนำให้เติมคำสั่ง `with (security_invoker = true)` เสมอตอนสร้าง View เพื่อให้นโยบาย RLS ของตารางต้นฉบับยังคงทำงานคัดกรองข้อมูลผู้ใช้อยู่เมื่อมีการขอดึงข้อมูลผ่าน View นี้นั่นเอง

### 5.3 Indexes และ Constraints
*   **Indexes (ดัชนี):** ทำหน้าที่จัดทำสารบัญให้คอลัมน์ที่ถูกเรียกค้นหาข้อมูลบ่อยครั้ง ช่วยให้ฐานข้อมูลสืบค้นได้เร็วขึ้นโดยไม่ต้องแสกนตารางทั้งหมด
*   **Unique Constraint:** บล็อกไม่ให้มีข้อมูลซ้ำซ้อนในฐานข้อมูล เช่น ป้องกันอีเมลซ้ำ หรือบล็อกไม่ให้มีการตั้งชื่อประเภทซ้ำกันภายในพื้นที่ทำงานเดียวกัน:
    ```sql
    create unique index unique_category_name_per_workspace
    on categories (workspace_id, lower(name));
    ```

### 5.4 Triggers (ตัวดักจับสัญญาณเพื่อรันคำสั่งอัตโนมัติ)
**Trigger** คือชุดคำสั่งที่ฐานข้อมูลจะรันอัตโนมัติทันทีที่มีเหตุการณ์เขียนข้อมูลเกิดขึ้น เช่น INSERT, UPDATE, DELETE

**ตัวอย่าง:** ปรับเวลา `updated_at` อัตโนมัติทุกครั้งที่มีการกดแก้ไขแถวข้อมูล
```sql
-- 1. เขียนฟังก์ชันตั้งค่าเวลาปัจจุบัน
create or replace function update_updated_at_column()
returns trigger as $$
begin
  new.updated_at = now();
  return new;
end;
$$ language plpgsql;

-- 2. สร้าง Trigger ผูกกับตารางเมื่อมีการ UPDATE
create trigger set_tasks_updated_at
before update on tasks
for each row
execute function update_updated_at_column();
```

---

## 6. ขั้นตอนการตั้งค่าเริ่มต้นใช้งาน (Setup Workflow)

### 6.1 สมัครโครงการบน Supabase Cloud
1.  เปิดเว็บ [supabase.com](https://supabase.com) และลงชื่อเข้าใช้
2.  กดสร้างโปรเจกต์ใหม่ (**New Project**)
3.  ตั้งชื่อ ระบุรหัสผ่านฐานข้อมูล (Database Password)
4.  เลือกตำแหน่งเซิร์ฟเวอร์ (**Region**) ใกล้กลุ่มผู้ใช้ เช่น *Singapore*
5.  กดสร้างและรอให้หน้า Dashboard เตรียมระบบฐานข้อมูลให้พร้อมทำงาน

### 6.2 พัฒนาด้วย Supabase CLI
การจัดการฐานข้อมูลระดับมืออาชีพจะใช้วิธีเขียนชุดโครงสร้างเป็นโค้ดเพื่อควบคุมเวอร์ชัน (Infrastructure as Code) ผ่านการใช้เครื่องมือควบคุมคำสั่ง CLI:

**1. ติดตั้งและล็อกอินเข้าระบบ**
```bash
npx supabase login
```

**2. ตั้งค่าโครงสร้างในเครื่อง**
```bash
# สร้างไดเรกทอรีการตั้งค่าของ Supabase ในโฟลเดอร์โปรเจกต์ของเรา
npx supabase init

# ลิงก์ผูกโปรเจกต์ local เข้ากับ Cloud (ใช้รหัส Project Ref จาก Dashboard URL)
npx supabase link --project-ref your_project_ref_here
```

**3. จัดการโครงสร้างฐานข้อมูลผ่านไฟล์ SQL Migration**
สร้างบันทึกประวัติการเปลี่ยนตารางในฐานข้อมูลเป็นแต่ละสเต็ป:
```bash
# สั่งสร้างไฟล์ Migration ใหม่
npx supabase migration new init_db_schema
```
เราสามารถเข้าไปเขียนคำสั่ง SQL ในไฟล์ประวัติที่ระบบสร้างไว้ในโฟลเดอร์ `supabase/migrations/` จากนั้นกดบันทึกแล้วดันโครงสร้างขึ้นคลาวด์จริงด้วยคำสั่ง:
```bash
# ส่งประวัติการอัปเกรดฐานข้อมูลขึ้นระบบ Cloud
npx supabase db push
```

---

## 7. ข้อควรระวังและแนวทางรักษาความปลอดภัย (Security Best Practices)

1.  **ห้ามเปิดเผย Service Role Key:** เก็บเป็นความลับเสมือนรหัสผ่านหลักของฐานข้อมูล ห้ามใส่ไว้ในโค้ดฝั่งหน้าบ้าน
2.  **สั่ง ENABLE RLS ทุกตาราง:** ทุกครั้งที่เปิดตารางใหม่ RLS จะไม่ทำงานโดยอัตโนมัติ ต้องพิมพ์คำสั่งเปิดใช้งานก่อนเสมอ:
    ```sql
    alter table my_table enable row level security;
    ```
3.  **ระวังการ recursion ใน Policy:** พยายามหลีกเลี่ยงการเขียน RLS Policy ที่อ้างอิงและคิวรีย้อนกลับมาหาตารางเดิมหลายชั้นเกินไป เพราะอาจทำให้คิวรีทำงานช้ามากหรือติดลูปไม่สิ้นสุด (Infinite Loop)

---

## 8. รายการเช็คลิสต์ตรวจความพร้อม (Database Readiness Checklist)

*   [ ] โปรเจกต์ในระบบ Supabase Cloud ถูกสร้างเสร็จสมบูรณ์
*   [ ] บันทึก URL และ Anon Key ลงในตัวแปรสิ่งแวดล้อมฝั่ง Client ถูกต้อง
*   [ ] เลือกเปิดใช้งานผู้ให้บริการล็อกอิน (Auth Providers) เฉพาะวิธีที่ต้องใช้ และปิดช่องทางที่ไม่ใช้งาน
*   [ ] ตารางฐานข้อมูลทั้งหมดได้รับการสั่งเปิดใช้งาน RLS ครบถ้วน
*   [ ] เขียนนโยบายคัดกรองข้อมูลของแต่ละตารางและทดสอบระบบสิทธิ์การเข้าถึงว่าปลอดภัยจริง
*   [ ] บันทึกประวัติและโครงสร้างของฐานข้อมูลลงในสคริปต์ SQL Migration เรียบร้อยพร้อมใช้งานในสภาพแวดล้อมอื่น (Staging/Production)
