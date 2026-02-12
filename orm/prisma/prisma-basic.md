# Prisma คืออะไร

Prisma เป็น ORM (Object-Relational Mapping) สำหรับ Node.js/TypeScript ที่ช่วยให้จัดการฐานข้อมูลเชิงสัมพันธ์ (เช่น PostgreSQL, MySQL, SQLite, SQL Server) ได้ง่ายขึ้น โดยใช้ TypeScript type-safe queries และ auto-completion

**ข้อดีของ Prisma:**

- **Type Safety** - TypeScript types อัตโนมัติจาก schema
- **Auto-completion** - IDE support ที่ดี
- **Migration Management** - จัดการ database schema changes ได้ง่าย
- **Developer Experience** - เขียน queries ได้ง่ายและอ่านง่าย
- **Prisma Studio** - UI สำหรับจัดการข้อมูลในฐานข้อมูล

--------------------------------------------------------------------------------

## 💡 Use Cases

- **REST API Development** - สร้าง API backend ด้วย TypeScript/Node.js
- **Full-stack Applications** - Next.js, NestJS, Express.js
- **Database Migration** - จัดการ schema changes อย่างเป็นระบบ
- **Type-safe Database Access** - ลดข้อผิดพลาดจาก type mismatch
- **Rapid Prototyping** - สร้างและทดสอบ schema ได้เร็ว

--------------------------------------------------------------------------------

## ⚙️ การติดตั้งและเริ่มต้นใช้งาน

### 1\. ติดตั้ง Prisma

```bash
npm install prisma @prisma/client
```

### 2\. เริ่มต้นโปรเจ็กต์

```bash
npx prisma init
```

คำสั่งนี้จะสร้าง:

- `prisma/schema.prisma` - ไฟล์ schema หลัก
- `.env` - ไฟล์ environment variables สำหรับ database connection

### 3\. ตั้งค่า Database Connection

แก้ไขไฟล์ `.env`:

```env
# PostgreSQL
DATABASE_URL="postgresql://user:password@localhost:5432/mydb?schema=public"

# MySQL
DATABASE_URL="mysql://user:password@localhost:3306/mydb"

# SQLite
DATABASE_URL="file:./dev.db"
```

### 4\. กำหนด Schema

แก้ไขไฟล์ `prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql" // หรือ "mysql", "sqlite"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

### 5\. สร้าง Migration

```bash
npx prisma migrate dev --name init
```

คำสั่งนี้จะ:

- สร้าง migration files ใน `prisma/migrations/`
- รัน migration บนฐานข้อมูล
- สร้าง Prisma Client อัตโนมัติ

### 6\. Generate Prisma Client

```bash
npx prisma generate
```

คำสั่งนี้จะสร้าง Prisma Client ที่ใช้สำหรับ query ข้อมูล

--------------------------------------------------------------------------------

## 💻 การใช้งาน Prisma Client

### 1\. Import Prisma Client

```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()
```

### 2\. Create (เพิ่มข้อมูล)

```typescript
// สร้าง User ใหม่
const user = await prisma.user.create({
  data: {
    email: 'alice@example.com',
    name: 'Alice',
  },
})

// สร้าง Post พร้อม User
const post = await prisma.post.create({
  data: {
    title: 'My First Post',
    content: 'Hello, World!',
    published: true,
    author: {
      connect: { id: user.id }
    }
  },
})
```

### 3\. Read (อ่านข้อมูล)

```typescript
// หา User ทั้งหมด
const users = await prisma.user.findMany()

// หา User ตามเงื่อนไข
const user = await prisma.user.findUnique({
  where: { email: 'alice@example.com' }
})

// หา Post พร้อม User (include relation)
const posts = await prisma.post.findMany({
  include: {
    author: true
  }
})

// หา Post ที่ published = true
const publishedPosts = await prisma.post.findMany({
  where: {
    published: true
  },
  include: {
    author: true
  }
})
```

### 4\. Update (อัปเดตข้อมูล)

```typescript
// อัปเดต User
const updatedUser = await prisma.user.update({
  where: { id: 1 },
  data: { name: 'Alice Updated' }
})

// อัปเดตหลาย records
const updatedPosts = await prisma.post.updateMany({
  where: { published: false },
  data: { published: true }
})
```

### 5\. Delete (ลบข้อมูล)

```typescript
// ลบ User
const deletedUser = await prisma.user.delete({
  where: { id: 1 }
})

// ลบหลาย records
const deletedPosts = await prisma.post.deleteMany({
  where: { published: false }
})
```

### 6\. Advanced Queries

```typescript
// Pagination
const posts = await prisma.post.findMany({
  skip: 10,
  take: 5,
  orderBy: { createdAt: 'desc' }
})

// Filter และ Search
const posts = await prisma.post.findMany({
  where: {
    OR: [
      { title: { contains: 'prisma' } },
      { content: { contains: 'prisma' } }
    ],
    published: true
  }
})

// Aggregate
const count = await prisma.post.count({
  where: { published: true }
})
```

--------------------------------------------------------------------------------

## 🛠️ CLI Commands พื้นฐาน

### การจัดการ Schema

```bash
# สร้างโปรเจ็กต์ใหม่
npx prisma init

# ดึงโครงสร้างจากฐานข้อมูลจริงมาสร้าง schema
npx prisma db pull

# ส่ง schema ขึ้นฐานข้อมูล (ไม่สร้าง migration)
npx prisma db push

# ตรวจสอบ schema format
npx prisma format

# Validate schema
npx prisma validate
```

### การจัดการ Migration

```bash
# สร้างและรัน migration (development)
npx prisma migrate dev --name <migration-name>

# รัน migration ที่มีอยู่ (production)
npx prisma migrate deploy

# ดู migration status
npx prisma migrate status

# Reset database และรัน migration ใหม่
npx prisma migrate reset
```

### การจัดการ Prisma Client

```bash
# สร้าง Prisma Client จาก schema
npx prisma generate

# ดูข้อมูล Prisma Client
npx prisma --version
```

### Prisma Studio

```bash
# เปิด Prisma Studio (GUI สำหรับจัดการข้อมูล)
npx prisma studio

# เปิด Prisma Studio ที่ port อื่น
npx prisma studio --port 5555
```

### ตัวอย่างการใช้งาน

```bash
# สร้าง migration สำหรับเพิ่ม field ใหม่
npx prisma migrate dev --name add_user_phone

# Pull schema จากฐานข้อมูลที่มีอยู่แล้ว
npx prisma db pull

# Push schema ไปยังฐานข้อมูล (สำหรับ prototyping)
npx prisma db push

# เปิด Prisma Studio เพื่อดูและแก้ไขข้อมูล
npx prisma studio
```

--------------------------------------------------------------------------------

## 📝 Schema Examples

### One-to-Many Relationship

```prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  posts Post[]
}

model Post {
  id       Int  @id @default(autoincrement())
  title    String
  author   User @relation(fields: [authorId], references: [id])
  authorId Int
}
```

### Many-to-Many Relationship

```prisma
model Post {
  id       Int    @id @default(autoincrement())
  title    String
  tags     Tag[]
}

model Tag {
  id    Int    @id @default(autoincrement())
  name  String @unique
  posts Post[]
}
```

### Field Types และ Attributes

```prisma
model Product {
  id          Int      @id @default(autoincrement())
  name        String
  price       Float
  description String?
  isActive    Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Unique constraint
  sku         String   @unique

  // Index
  category    String   @index

  // Enum
  status      ProductStatus @default(ACTIVE)
}

enum ProductStatus {
  ACTIVE
  INACTIVE
  ARCHIVED
}
```

--------------------------------------------------------------------------------

## 📚 Learning Resources

- **Official Website**: <https://www.prisma.io/>
- **Documentation**: <https://www.prisma.io/docs>
- **Schema Reference**: <https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference>
