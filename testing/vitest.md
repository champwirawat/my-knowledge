# คู่มือเริ่มต้นใช้งาน Vitest (Vitest Beginner Guide)

คู่มือการเรียนรู้ **Vitest** สำหรับผู้เริ่มต้นฉบับนี้จัดทำขึ้นเพื่อให้เข้าใจแนวคิดพื้นฐาน วิธีการตั้งค่าคอนฟิก การเขียนเทส และแนวทางการแบ่งชั้นการทดสอบ (Testing Layers) ในโปรเจกต์เว็บแอปพลิเคชันยุคใหม่ (เช่น React และ Next.js)

---

## 1. Vitest คืออะไร?

**Vitest** คือ Test Runner สำหรับ JavaScript/TypeScript ที่ถูกสร้างขึ้นมาเพื่อให้ทำงานร่วมกับเครื่องมือ build ยอดนิยมอย่าง Vite ได้เป็นอย่างดี โดยมีชุดคำสั่งและ API ในการเรียกใช้งานที่คล้ายคลึงกับ Jest (เช่น `describe`, `it`, `expect`, `vi.mock`) แต่รวดเร็วกว่าเนื่องจากประมวลผลผ่าน ESM Native และทำงานได้อย่างมีประสิทธิภาพในโหมด Watch (ตรวจสอบไฟล์และรันเทสใหม่ทันทีที่กดบันทึกโค้ด)

คุณสมบัติเด่นที่ทำให้ Vitest เหมาะสมกับแอปพลิเคชันสมัยใหม่:
*   **รวดเร็ว** — รันซ้ำได้รวดเร็วใน Watch Mode โดยไม่ต้องเสียเวลาคอมไพล์โปรเจกต์ใหม่หมดทุกครั้ง
*   **รองรับ TypeScript ตั้งแต่เริ่มต้น** — ไม่ต้องติดตั้งปลั๊กอินตรวจสอบประเภทข้อมูล เช่น `ts-jest` แยกต่างหาก
*   **สลับมาใช้งานได้ง่าย** — โครงสร้างชุดคำสั่งรองรับรูปแบบของ Jest เกือบทั้งหมด ทำให้ย้ายระบบเทสเดิมมาได้ไม่ยาก
*   **จำลองสภาพแวดล้อมเบราว์เซอร์ได้** — สนับสนุนการรันร่วมกับ `jsdom` หรือ `happy-dom` เพื่อให้ทดสอบ React component ได้
*   **ระบบตรวจสอบความครอบคลุม (Built-in Coverage)** — ตรวจจับปริมาณโค้ดที่รันผ่านเทสได้ผ่านระบบ `@vitest/coverage-v8`

---

## 2. โครงสร้างการแบ่งระดับการทดสอบ (Testing Strategies)

ในการพัฒนาเว็บแอปพลิเคชันขนาดใหญ่ เรามักแบ่งระดับการทดสอบออกเป็น 3 เลเยอร์ เพื่อประสิทธิภาพในการตรวจสอบและความง่ายในการเขียนโค้ด:

| ระดับการทดสอบ | เครื่องมือที่ใช้ | ขอบเขตการทดสอบ |
| :--- | :--- | :--- |
| **Unit Test** | **Vitest** | ทดสอบความถูกต้องของ Logic ย่อย, ฟังก์ชันจำพวก Pure Functions, ระบบตรวจสอบเงื่อนไข (Validation Schemas) หรือ Helper functions |
| **Integration Test** | Vitest + Mock API / DB | ทดสอบการทำงานร่วมกันระหว่างฟังก์ชันหรือระบบ เช่น การเช็คข้อมูลร่วมกับ Local Database จำลอง หรือการเรียกใช้ Hooks ร่วมกับ API |
| **End-to-End (E2E) Test**| Playwright / Cypress | ทดสอบเส้นทางผู้ใช้งานจริงทั้งหมด (User Flows) ตั้งแต่ต้นจนจบ เช่น การสมัครสมาชิกไปจนถึงการชำระเงินสำเร็จบนหน้าจอจริง |

สำหรับการใช้ Vitest จะมุ่งเน้นไปที่การทำ **Unit Test** เป็นหลัก เพื่อทดสอบ Logic การแปลงค่าและการคัดกรองเงื่อนไขต่าง ๆ ให้รวดเร็วและแม่นยำ

---

## 3. ชุดเครื่องมือที่จำเป็นสำหรับการติดตั้ง (Dependencies)

ในการเริ่มต้นใช้งาน Vitest ร่วมกับ React หรือ Next.js มักต้องติดตั้งแพ็กเกจต่อไปนี้เป็น dev dependencies:

| แพ็กเกจ | หน้าที่การทำงาน |
| :--- | :--- |
| `vitest` | เครื่องมือรันการทดสอบ (Test Runner) ตัวหลัก |
| `@vitejs/plugin-react` | ปลั๊กอินแปลงคำสั่ง JSX/TSX ให้พร้อมรันในเทส |
| `jsdom` | สภาพแวดล้อมจำลองเบราว์เซอร์ (DOM API) ในเครื่องเพื่อเทส Component |
| `@testing-library/react` | เครื่องมือช่วย Render Component และสืบค้นตรวจสอบเนื้อหาบนหน้าจอ |
| `@testing-library/jest-dom` | ชุดคำสั่งตรวจสอบสิทธิ์เฉพาะ (Matchers) เช่น `.toBeInTheDocument()` |
| `@testing-library/user-event` | จำลองพฤติกรรมการพิมพ์และการคลิกของผู้ใช้งานให้ใกล้เคียงจริง |
| `@vitest/coverage-v8` | เครื่องมือทำรายงานความครอบคลุมของโค้ด (Code Coverage) |

### การวางตำแหน่งไฟล์ทดสอบ
ในการจัดระเบียบโครงการ มักจะนิยมวางโฟลเดอร์สำหรับเก็บไฟล์เทสไว้แยกต่างหาก เช่น สร้างโฟลเดอร์ `tests/` ไว้ที่ Root ของโปรเจกต์ (หรือวางไฟล์นามสกุล `.test.ts` คู่ไปกับไฟล์โค้ดจริงใน `src/` ตามรูปแบบโครงสร้างที่ทีมถนัด)

ตัวอย่างสคริปต์ในไฟล์ `package.json`:
```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  }
}
```

---

## 4. การเขียน Unit Test หน้าแรก

### 4.1 โครงสร้างไฟล์เทสพื้นฐาน
ไฟล์สำหรับรันคำสั่งทดสอบจะต้องใช้นามสกุลไฟล์เป็น `.test.ts` หรือ `.test.tsx`

ตัวอย่างการเทสฟังก์ชันคำนวณราคาเฉลี่ยทั่วไป (Pure Function):
```typescript
import { describe, expect, it } from "vitest";

import { calculateDiscount } from "@/lib/price/discount";

describe("calculateDiscount", () => {
  it("ลดราคาสินค้า 10% ได้ถูกต้อง", () => {
    const finalPrice = calculateDiscount(100, 10);
    expect(finalPrice).toBe(90);
  });

  it("ราคาสินค้าไม่ติดลบหากใส่เปอร์เซ็นต์ส่วนลดเกิน 100", () => {
    const finalPrice = calculateDiscount(100, 120);
    expect(finalPrice).toBe(0);
  });
});
```

### 4.2 ชุดคำสั่งและ API ยอดนิยม
*   `describe(name, fn)`: ใช้สำหรับจัดหมวดหมู่กลุ่มกรณีทดสอบที่เกี่ยวข้องกัน
*   `it(name, fn)` หรือ `test(name, fn)`: ใช้เขียนกรณีการทำงานข้อใดข้อหนึ่ง
*   `expect(value)`: คำสั่งเช็คผลลัพธ์ว่าตรงกับที่คาดหวังหรือไม่
*   `beforeEach(fn)`: สั่งให้รันชุดคำสั่งตั้งค่าเริ่มต้นก่อนที่จะเริ่มทดสอบกรณีถัดไป
*   `vi.fn()`: ใช้สำหรับสร้างฟังก์ชันจำลอง (Mock Function) เพื่อดักจับยอดการเรียกใช้งาน
*   `vi.mock(path)`: จำลองการทำงานของไฟล์หรือโมดูลภายนอกทั้งโมดูล

### 4.3 คำสั่งเปรียบเทียบผลลัพธ์ (Matchers)
```typescript
expect(value).toBe(50);                    // เปรียบเทียบค่าข้อมูลทั่วไป (strict equality ===)
expect(object).toEqual({ key: "val" });    // เปรียบเทียบเนื้อหาภายใน Object หรือ Array
expect(array).toContain("apple");          // ตรวจสอบว่าใน Array มีข้อมูลชิ้นนั้นอยู่หรือไม่
expect(mockFn).toHaveBeenCalledOnce();     // ตรวจว่าฟังก์ชันจำลองโดนเรียกไป 1 ครั้งถูกต้อง
expect(element).toBeInTheDocument();       // (Testing Library) ตรวจว่า Component แสดงผลบนจอจริง
```

---

## 5. การทดสอบ Validation Schema (Zod)

การเช็คความถูกต้องของ Schema (เช่น การใช้ Zod เพื่อตรวจสอบสิทธิ์การป้อนฟอร์ม) เป็นจุดที่ทำ Unit Test ได้ดีและง่ายที่สุดเนื่องจากไม่ต้องทำการ Mock ข้อมูลของภายนอกเลย:

```typescript
import { describe, expect, it } from "vitest";

import { loginSchema } from "@/lib/validation/auth";

describe("loginSchema", () => {
  it("ผ่านเงื่อนไขเมื่อป้อนอีเมลและรหัสผ่านถูกต้อง", () => {
    const result = loginSchema.safeParse({
      email: "test@example.com",
      password: "password123"
    });
    expect(result.success).toBe(true);
  });

  it("ไม่ผ่านเงื่อนไขเมื่อใส่อีเมลผิดรูปแบบ", () => {
    const result = loginSchema.safeParse({
      email: "not-an-email",
      password: "password123"
    });
    expect(result.success).toBe(false);
  });
});
```
*คำแนะนำ:* แนะนำให้ใช้ `.safeParse()` แทน `.parse()` เพราะจะส่งคืนสถานะเป็น Boolean ในช่อง `.success` ทำให้เขียน assertions ได้ง่ายขึ้นโดยไม่ต้องครอบคำสั่งดักจับ error (try-catch)

---

## 6. การทดสอบ React Component (Testing Library)

เมื่อต้องการทดสอบ Component บนหน้าจอ (เช่น ปุ่ม, แบบฟอร์ม) เราจะจำลองการแสดงผลเพื่อตรวจวัดพฤติกรรมการใช้งานจริงของผู้ใช้ มากกว่าการเข้าไปแงะทดสอบสถานะการทำงานภายใน (Internal State)

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, expect, it, vi } from "vitest";

import { CustomButton } from "@/components/core/custom-button";

describe("CustomButton", () => {
  it("เมื่อผู้ใช้กดคลิกปุ่ม จะส่งสัญญาณเรียก callback สำเร็จ", async () => {
    const user = userEvent.setup();
    const handleClick = vi.fn();

    render(<CustomButton label="ยืนยันการทำรายการ" onClick={handleClick} />);

    // ค้นหาและกดคลิกปุ่มที่มีข้อความระบุตาม role
    await user.click(screen.getByRole("button", { name: /ยืนยันการทำรายการ/i }));

    expect(handleClick).toHaveBeenCalledOnce();
  });
});
```

### 6.1 กฎเหล็กในการเขียนทดสอบ Component
1.  **ค้นหา element ด้วยสิทธิ์การเข้าถึง (Accessibility Roles):** พยายามค้นหาด้วย `getByRole` หรือ `getByLabelText` ก่อนที่จะเรียกใช้ `getByTestId` เพื่อให้ใกล้เคียงกับการที่โปรแกรมอ่านหน้าจอ (Screen Reader) ค้นหาข้อมูลจริง
2.  **หลีกเลี่ยงการเช็คสเตตตรง ๆ:** แทนที่จะเข้าไปตรวจเช็คว่าตัวแปร `useState` เปลี่ยนเป็นค่าอะไร ให้เน้นตรวจสอบว่าตัวอักษร Error หรือปุ่มบนหน้าจอเปลี่ยนสถานะไปตามพฤติกรรมที่เรากดหรือไม่
3.  **ใช้งาน userEvent:** จำลองการทำงานด้วย `@testing-library/user-event` เสมอแทนการสั่งยิง event ตรงด้วย `fireEvent` เนื่องจากมีการสั่งรัน event ย่อยประกอบให้อัตโนมัติ (เช่น การพิมพ์อักษรจะมีการสั่ง keyDown, keyUp ตามจริง)

---

## 7. กลยุทธ์การเขียนเทสโดยการจำลองชั้นข้อมูล (Testing Layers)

ในสถาปัตยกรรมของโปรเจกต์เว็บสมัยใหม่ เรามักแยกชั้นความรับผิดชอบของไฟล์ออกจากกัน เพื่อให้โค้ดทดสอบได้ง่าย โดยการเลือกทำการ **Mock เฉพาะชั้นถัดไปด้านล่างเพียง 1 ชั้นเท่านั้น**:

```
[Component หน้าจอ] ──(เรียกใช้)──▶ [Custom Hooks] ──(เรียกใช้)──▶ [API Functions] ──(ดึงข้อมูล)──▶ [Client SDK / Database]
```

### 7.1 การทดสอบ Component ด้วยการ Mock Custom Hook
เมื่อต้องการเทสความถูกต้องของ Component ไม่จำเป็นต้องให้ไฟล์เทสยิงขอข้อมูลจาก API จริง ให้ใช้การ Mock ผลลัพธ์ที่ตอบกลับมาจาก Custom Hook แทน:
```typescript
// จำลองพฤติกรรม Hook ของฟีเจอร์สินค้า
vi.mock("@/features/items/hooks/use-get-items", () => ({
  useGetItems: () => ({
    items: [{ id: "1", name: "นมสดจืด" }],
    isLoading: false,
    error: null
  })
}));
```

### 7.2 การทดสอบ Custom Hook ด้วย `renderHook`
หาก Custom Hook มีการสลับ logic ข้างในบ่อยครั้ง สามารถรันเทสเฉพาะตัว Hook ได้โดยเรียกใช้งานฟังก์ชัน `renderHook`:
```typescript
import { renderHook, waitFor } from "@testing-library/react";
import { describe, expect, it, vi } from "vitest";

import * as api from "@/features/items/api";
import { useCreateItem } from "@/features/items/hooks/use-create-item";

vi.mock("@/features/items/api");

describe("useCreateItem", () => {
  it("แสดงสถานะ pending เป็นจริง ขณะกำลังประมวลผล", async () => {
    // จำลองให้ API ตอบช้าเล็กน้อยเพื่อเช็คสถานะหมุนโหลด
    vi.mocked(api.createNewItem).mockImplementation(
      () => new Promise((resolve) => setTimeout(resolve, 50))
    );

    const { result } = renderHook(() => useCreateItem());

    result.current.submit({ name: "สมุดบันทึก" });

    expect(result.current.isPending).toBe(true);
  });
});
```

---

## 8. การแก้ไขปัญหาเฉพาะหน้า (Troubleshooting)

*   **ฟ้องข้อความ Error: `document is not defined`**
    *   *สาเหตุ:* ไฟล์เทสกำลังเรียกใช้ DOM API ของเบราว์เซอร์ แต่เทสปัจจุบันรันอยู่บนโหมด Node.js
    *   *แก้ไข:* ตรวจสอบว่าในไฟล์ตั้งค่า `vitest.config.ts` ได้กำหนดค่า `environment: "jsdom"` หรือใส่คำสั่งคอมเมนต์ `@vitest-environment jsdom` ไว้ที่หัวข้อไฟล์เทสนั้น
*   **ฟ้องข้อความ Error: `toBeInTheDocument is not a function`**
    *   *สาเหตุ:* ไม่ได้นำเข้า matchers ของ Testing Library เข้าสู่สภาพแวดล้อม Vitest
    *   *แก้ไข:* ยืนยันว่ามีการเขียนคำสั่ง `import "@testing-library/jest-dom/vitest"` หรือโหลดไฟล์ setup ล่วงหน้าก่อนเริ่มกระบวนการรันเทส
*   **คำสั่ง Mock ไม่ทำงาน (ยังส่งคำสั่งดึงข้อมูลจริงอยู่)**
    *   *สาเหตุ:* สั่งรันคำสั่ง `vi.mock()` หลังจากที่ทำการอิมพอร์ตโมดูลนั้นเข้ามาใช้งานแล้ว
    *   *แก้ไข:* ขยับคำสั่ง `vi.mock("...")` ไปไว้ที่บรรทัดบนสุดของไฟล์เสมอ หรือทำการเขียนสืบค้นชื่อโมดูลให้เป็นที่อยู่สัมพันธ์แบบสัมบูรณ์ (Absolute Paths) ให้ตรงตามไฟล์จริงที่หน้าเพจเรียกใช้
