# Change Classification

## 1. Design Checklist (Phase 1 ข้อ 2)

ตรวจเฉพาะส่วนที่มีอยู่ใน design จริง ถ้าไม่มีให้เขียน `ไม่พบข้อมูลส่วนนี้ใน Design File`

- **โครงหน้า**: page layout, header, breadcrumb, title, action buttons, tabs
- **เนื้อหา**: cards, tables, forms, detail fields, badges/status, timeline, attachments
- **Overlay**: dialogs, modals, drawers, tooltips, popovers
- **States**: empty, loading, error, success, disabled
- **Tokens**: typography (font, size, weight), color, spacing, radius, border, shadow
- **อื่นๆ**: icons, interaction hints, responsive clues

## 2. กลุ่มส่วนต่าง (Phase 1 ข้อ 5)

| กลุ่ม | ตัวอย่าง |
|---|---|
| A. Layout | position, order, width/height, column/row, alignment, spacing |
| B. Visual | typography, color, border, radius, shadow, icon, button style, badge |
| C. Content | label เปลี่ยน, field ที่เพิ่ม/ลบ/ย้าย, conditional display |
| D. Interaction | click, modal/dialog/drawer, tab, expand/collapse, edit/save/cancel/delete, upload/download |
| E. State | loading, empty, error, success, disabled, selected, hover, focus |
| F. Responsive | desktop/tablet/mobile, breakpoint, wrapping, overflow |

## 3. ป้ายระดับผลกระทบ (บังคับติดทุกจุด)

| ป้าย | ความหมาย | ปกติจัดเป็น |
|---|---|---|
| `UI-only` | แก้ style หรือ markup อย่างเดียว | MUST/SHOULD CHANGE |
| `UI + existing state` | ใช้ state ที่มีอยู่แล้ว | MUST/SHOULD CHANGE |
| `UI + existing API data` | แสดง field ที่ API ส่งมาอยู่แล้ว (ยืนยันจาก type หรือ response จริง) | MUST/SHOULD CHANGE |
| `New behavior` | interaction หรือ flow ที่ระบบยังไม่มี | BLOCKED |
| `Business logic impact` | ต้องแก้ logic, API หรือ permission | BLOCKED |

## 4. เกณฑ์ Priority

- **P1**: ผู้ใช้เห็นชัดและเป็นจุดหลักของ design (layout หลัก, label ผิด, field ขาด)
- **P2**: visual token (สี, ระยะ, font) ที่ต่างจาก design ชัดเจน
- **P3**: รายละเอียดเล็กหรือ state รอง
