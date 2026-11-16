---
name: figma-ui-update
description: อัปเดต Frontend UI ให้ตรงกับ design ล่าสุด (Figma export เป็น SVG/PNG/JPG หรือ Figma URL) แบบปลอดภัย โดยวิเคราะห์ส่วนต่างระหว่าง design กับโค้ดปัจจุบันก่อน รอผู้ใช้อนุมัติ แล้วจึงแก้เฉพาะส่วนต่างที่อนุมัติ ใช้เมื่อผู้ใช้พูดถึง "แก้ UI ให้ตรง Figma", "update UI ตาม design", "เทียบ design กับหน้าจอ", ส่งไฟล์ design มาพร้อมขอให้ปรับหน้า, page, component, modal หรือ dialog ใช้ได้กับทุก feature และทุก frontend project (React/Next/Vue ฯลฯ)
---

# Figma → UI Update

Workflow 2 phase สำหรับปรับ UI ที่มีอยู่แล้วให้ตรง design โดยไม่ทำลาย logic เดิม

- **Phase 1 — Analyze only**: อ่าน design และโค้ด แล้วสรุปส่วนต่างพร้อม to-do (ห้ามแก้ code)
- **Phase 2 — Implement**: แก้เฉพาะ to-do ที่ผู้ใช้อนุมัติ, typecheck แล้วรายงานผล

หลักใหญ่: **โค้ดปัจจุบันคือ baseline แก้เฉพาะ design delta ที่ตรวจพบจริง**

---

## Input

| Input | บังคับ | ตัวอย่าง |
|---|---|---|
| Design reference | ✅ | `type-job.svg`, `docs/design/users.png`, Figma URL |
| Feature name | ไม่บังคับ | `--feature="Type Job"` |
| Design directory | ไม่บังคับ | ค่าเริ่มต้น `docs/design/` |
| Report directory | ไม่บังคับ | ค่าเริ่มต้น `docs/ui-analysis/` |

ตัวอย่างการเรียก:

```text
/figma-ui-update
/figma-ui-update type-job.svg
/figma-ui-update --feature="Users" --design="docs/design/users.svg"
แก้หน้า Link Setting ให้ตรงกับ Link-Setting.svg
```

## Output

1. **Phase 1**: รายงาน `UI Change Analysis` (ตอบในแชตและบันทึกเป็นไฟล์ `<report-dir>/<feature-kebab>-change-analysis.md`) ตาม [references/phase1-report-template.md](references/phase1-report-template.md) ปิดท้ายด้วย `PHASE 1 STATUS: ANALYSIS COMPLETE`
2. **Phase 2**: โค้ดที่แก้แล้ว + รายงาน `UI Update Result` ตาม [references/phase2-report-template.md](references/phase2-report-template.md) ปิดท้ายด้วย `PHASE 2 STATUS: PASS` หรือ `BLOCKED`

---

## Language

- ตอบด้วยภาษาเดียวกับผู้ใช้ (ถ้า project หรือผู้ใช้กำหนดภาษาไว้ ให้ใช้ภาษานั้น)
- identifier ทางเทคนิคคงไว้ตามเดิมเสมอ เช่น code, path, route, ชื่อ component/type/field/endpoint และ label ใน template (`ID:`, `File:`)

---

## Step 0 — ตรวจ Project Context

ก่อนเริ่ม ให้หาจาก project จริง (ห้ามสมมติ):

- framework, UI library และ styling system (ดูจาก `package.json` และ import ในโค้ด)
- ตำแหน่ง source (`src/`, `app/` ฯลฯ) และวิธีกำหนด route
- script สำหรับตรวจ type และ lint จาก `package.json` (เช่น `typecheck`, `tsc --noEmit`, `lint`)
- design directory และ report directory (ถ้าไม่มี ให้ใช้ค่าเริ่มต้น และสร้าง report dir เฉพาะตอนต้องบันทึก)
- `CLAUDE.md` หรือ convention ของ project (ถ้ามี ให้ยึดตามนั้นก่อน skill นี้)

## Step 1 — หา Design File

| สถานการณ์ | ทำอะไร |
|---|---|
| ระบุ path มา | ใช้ไฟล์นั้น (ตรวจว่ามีอยู่จริง) |
| ระบุแค่ชื่อไฟล์ | ค้นใน design directory โดยเทียบชื่อจริง ไม่สนตัวพิมพ์เล็กใหญ่ |
| ระบุแค่ feature | ค้นไฟล์ชื่อใกล้เคียง → **แสดงรายการแล้วรอผู้ใช้ยืนยัน** |
| เจอหลายไฟล์ | แสดง path, ชื่อไฟล์ และเหตุผลที่เกี่ยว → **หยุดรอเลือก** ห้ามเลือกเอง |
| ไม่เจอ | รายงาน `ไม่พบ Design File ที่ยืนยันได้` แล้ว **หยุด** |
| Figma URL | ใช้ Figma MCP ถ้ามี ถ้าไม่มีให้ขอไฟล์ export |

วิธีอ่าน design:

- **PNG/JPG**: เปิดดูเป็นรูป
- **SVG ขนาดใหญ่** (เกิน ~1MB): ห้ามอ่านทั้งไฟล์ ให้ grep หา `<text>`/`<tspan>` เพื่อดึง label, หา `fill=`/`stroke=`/`font-size`/`font-family` เพื่อดึง token และดู `width`/`height`/`viewBox` เพื่อดูขนาด ถ้า project มี PNG คู่กันให้ใช้ประกอบ

## Step 2 — Design คือความจริงเฉพาะด้าน UI

Design เป็น source of truth สำหรับ: layout, spacing, sizing, typography, color, border, radius, shadow, icon, visual hierarchy, visible label และ UI state ที่แสดงไว้

Design **ไม่ใช่** source of truth สำหรับ: API contract, business logic, permission, workflow, data model, auth

ถ้า design ต้องการ behavior หรือ data ที่ระบบยังไม่มี → จัดเป็น `BLOCKED / NEED CONFIRMATION` ห้ามเดา implementation

## Safety Rules (ใช้ทั้งสอง phase)

ห้าม:

- แก้ไฟล์ใดๆ ใน Phase 1 ยกเว้นไฟล์รายงาน
- rebuild หรือ recreate ทั้งหน้า หรือ copy SVG/path data ลง source
- เปลี่ยน business logic, API contract, route, permission, auth, workflow หรือ state management เพื่อให้ภาพตรง
- refactor ใหญ่, rename ไฟล์จำนวนมาก หรือเพิ่ม dependency โดยไม่จำเป็น
- ใช้ `any`/`as any`, suppress error, comment code ทิ้ง หรือลบ code ที่ยังใช้งานอยู่
- แก้ shared component โดยไม่ตรวจว่ามีที่ไหนใช้บ้าง
- แก้ feature ที่ไม่เกี่ยวข้อง

---

## PHASE 1 — ANALYZE ONLY

1. **Target**: สรุป `Target Feature / Design File / Design File Path / Related Feature Guess / Confidence` ถ้าไม่แน่ใจให้บอกตรงๆ ห้ามเดา route
2. **อ่าน design**: เช็คตาม checklist ใน [references/change-classification.md](references/change-classification.md) และเขียน `ไม่พบข้อมูลส่วนนี้ใน Design File` สำหรับส่วนที่ไม่มี ห้ามสรุปจาก layer name อย่างเดียว
3. **หา implementation**: ใช้ keyword จาก feature name, label ที่เห็นใน design, ชื่อ entity และ route แล้วตามดู route config → page → components → dialogs → hooks และ API calls → types → mocks → tests **ต้องยืนยันจาก import/usage จริง** ห้ามเชื่อแค่ชื่อไฟล์
4. **API impact**: ทุกจุดที่ข้อมูลเกี่ยวข้อง ให้ระบุ endpoint, method, field, frontend type และ component ที่ใช้ แล้วสรุปว่าเป็น `UI-only change — ไม่กระทบ API contract` หรือ `BLOCKED` ถ้าหาไม่เจอให้เขียน `ไม่พบ endpoint/data source ที่ยืนยันได้จาก source code`
5. **เทียบส่วนต่าง**: จัดกลุ่มเป็น A–F (Layout / Visual / Content / Interaction / State / Responsive) และติดป้ายทุกจุดตามระดับผลกระทบ ดูรายละเอียดใน [references/change-classification.md](references/change-classification.md)
6. **UI Location**: ทุกจุดต้องบอกวิธีเปิดดูในแอป (route, ปุ่มที่ต้องกด, modal) ถ้าเป็น shared component ให้ระบุว่ามีหน้าไหนใช้บ้าง
7. **จัดกลุ่มงาน**: `MUST CHANGE` / `SHOULD CHANGE` / `BLOCKED / NEED CONFIRMATION` / `MUST NOT CHANGE`
8. **To-do**: เรียงตาม priority ทุกข้อต้องมีหลักฐานจาก design หรือโค้ด
9. **รายงาน**: เขียนตาม template, บันทึกไฟล์ แล้วจบด้วย

```text
PHASE 1 STATUS: ANALYSIS COMPLETE
ยังไม่มีการแก้ไข application code
รอผู้ใช้ยืนยันก่อนเริ่ม PHASE 2
```

แล้ว **หยุดทันที**

---

## PHASE 2 — IMPLEMENT (หลังได้รับอนุมัติเท่านั้น)

เริ่มได้เมื่อผู้ใช้ยืนยันชัดเจน เช่น "ยืนยัน Phase 2", "แก้ ID 1, 3, 4" ถ้ายังไม่มีคำยืนยัน **ห้ามแก้ไฟล์**

1. **Confirm scope**: แยกรายการที่อนุมัติกับรายการที่ไม่อนุมัติ แล้วแก้เฉพาะรายการที่อนุมัติ
2. **อ่านไฟล์เต็มก่อนแก้**: ดู imports, exports, props, types, state, hooks และการใช้ shared component ห้ามแก้จาก snippet ที่อ่านไม่ครบ
3. **แก้ทีละ to-do**: ใช้ component, style system และ design token เดิมก่อน รักษา state, API hook และ business logic เดิม
4. **เจอปัญหา** (API ไม่รองรับ, type ไม่ตรง, กระทบหน้าอื่น, ต้องแก้ backend): หยุดรายการนั้นและรายงาน

   ```text
   BLOCKED ITEM:
   - To-do ID:
   - File:
   - Problem:
   - Evidence:
   - Risk:
   - Options:
   ```

5. **Verify**: รัน typecheck และ lint script ที่ project มีจริง ห้ามแก้ script หรือ config เพื่อให้ผ่าน
6. **Manual test checklist**: ทุก to-do ให้ระบุ URL, ขั้นตอนเปิด, ผลที่ควรเห็น และสถานะ `PASS/FAIL/NOT VERIFIED`
7. **Responsive และ regression**: ถ้ายังไม่ได้เปิด browser จริง ให้เขียน `ยังไม่ได้ยืนยันด้วย browser จริง` ห้ามรายงานว่าผ่านโดยไม่มีหลักฐาน
8. **รายงาน**: เขียนตาม template แล้ว **หยุด** ห้ามแก้ไฟล์เพิ่มหรือเริ่ม feature อื่น
