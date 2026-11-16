---
name: sync-backend
description: อ่าน commit ล่าสุดของ backend repo (read-only) แล้ววิเคราะห์ผลกระทบเชิง API contract ต่อ frontend — phase 1 วิเคราะห์อย่างเดียว รอผู้ใช้ยืนยัน แล้ว phase 2 จึงแก้ frontend ให้ตรง backend จริง ใช้เมื่อผู้ใช้สั่ง /sync-backend, "sync backend", "backend เปลี่ยนอะไร", "เช็ค commit ใหม่ของ backend กระทบ FE ไหม" ใช้ได้กับทุกโปรเจคที่มี frontend + backend แยก repo
---

# Sync Backend → Frontend

ตอบผู้ใช้เป็นภาษาไทยเสมอ (identifier, path, code คงภาษาอังกฤษ)

เป้าหมาย: อ่านการเปลี่ยนแปลงล่าสุดของ backend → บอกว่ากระทบ frontend ตรงไหน → (หลังยืนยัน) แก้ frontend ให้ตรง backend จริง

---

## ขั้น 0: โหลด config ของโปรเจค (ห้ามข้าม)

อ่าน `CLAUDE.md` ของ frontend repo ก่อนเสมอ แล้วหาหัวข้อ **`## Backend sync config`** ซึ่งควรระบุ:

| Key | ความหมาย | ตัวอย่าง |
|---|---|---|
| Backend repo | path ของ backend (read-only) | `../my-api` |
| Backend stack | ภาษา/framework | .NET, Node, Go |
| Integration doc | note ที่ทีม backend เขียนให้ FE (อ่านก่อนเสมอ) | `docs/frontend-integration.md` |
| Contract paths | โฟลเดอร์ใน backend ที่แตะ = กระทบ contract | `Controllers/`, `Models/`, `routes/`, `dto/` |
| Ignore paths | แตะแค่นี้ = ไม่กระทบ FE | `Service/`, migration, jobs |
| FE API layer | ที่ FE เรียก endpoint | `src/features/*/api/`, `src/services/` |
| Feature map | route → หน้า → ปุ่ม → API (ถ้ามี) | `docs/feature-map.md` |
| Checks | คำสั่งตรวจ FE | `npm run lint`, `npm run typecheck` |
| Verify | วิธีตรวจในแอปจริง (skill/คำสั่ง) | skill `verify-in-browser` |
| Health endpoint | เช็คว่า backend ที่ deploy เป็น commit ไหน | `GET /api/health` |
| Last sync line | บรรทัดที่ต้องอัปเดตทุกรอบ sync | "Backend sync ล่าสุด: ..." |

**ไม่พบหัวข้อนี้** → เดาจากโครง workspace (หา repo พี่น้องที่เป็น backend) แล้ว **ถามผู้ใช้ยืนยัน backend repo ก่อน**
และเสนอร่าง `## Backend sync config` ให้ใส่ใน CLAUDE.md — ห้ามเดาเงียบๆ

อ่านกฎอื่นใน CLAUDE.md ด้วย (เช่น pagination/filter, signature endpoint ที่มีหลายผู้เรียก) — กฎโปรเจคชนะ skill นี้

---

## กฎตายตัว

- **Backend repo = READ-ONLY**: ห้ามแก้/สร้างไฟล์, ห้าม build/run, ห้าม git write (อนุญาต `git fetch`, `git log`, `git show`, `git diff` เท่านั้น)
- **Frontend ปรับตาม backend จริง** ไม่ใช่ขอให้ backend ปรับตาม frontend
- **Sync = data/contract เท่านั้น ห้ามแก้ UI** — ห้ามเพิ่ม/ซ่อนปุ่ม, dialog, layout แม้ backend เปิด edge/action ใหม่
  ให้รายงานเป็นข้อมูล "backend อนุญาต X, UI คงเดิม" ไม่ใช่ to-do
- ห้าม filter/paginate ฝั่ง frontend แทน backend — ยิง query param จริง
- Endpoint ที่มีผู้เรียกหลายจุด ห้ามแก้ signature ตรงๆ → เพิ่ม endpoint/hook ใหม่แยก
- อย่าเดา behavior backend — diff บอกว่าเพิ่ม param แต่ไม่ชัดว่า filter จริงหรือแค่รับเฉยๆ → เตือน "ต้อง curl ยืนยันก่อน" อย่าเพิ่งแก้

---

## ขั้นเตรียม: หาช่วง diff เอง (อย่าถามผู้ใช้)

1. `git -C <backend repo> fetch`
2. อ่าน Integration doc (ถ้ามี)
3. `git -C <backend repo> log --oneline -15` — เริ่มจาก commit ถัดจาก "Last sync line" ใน CLAUDE.md ถ้ามี
4. โฟกัส commit ที่แตะ Contract paths · ข้าม commit ที่แตะแค่ Ignore paths / docs ล้วน
5. ไม่แน่ใจจุดเริ่ม → ดู 15 commit ล่าสุด เลือกเฉพาะที่กระทบ API contract

---

## เฟส 1: วิเคราะห์ (ห้ามแก้ไฟล์ใดๆ)

1. สรุปการเปลี่ยนแปลงเชิง API contract:
   - endpoint (method + path) เปลี่ยนอะไร
   - param เพิ่ม/ลบ/เปลี่ยนชื่อ — ชื่อจริงตรงเป๊ะ (snake_case/camelCase)
   - response/status/shape เปลี่ยนยังไง
   - ⚠️ ทำเครื่องหมาย breaking change ที่ทำให้ FE เดิมพัง
2. เทียบกับ FE: grep ใน FE API layer หาจุดที่เรียก endpoint ที่เปลี่ยน — กระทบกี่จุด กี่ component
3. ระบุ **UI location** ของแต่ละจุด ให้ผู้ใช้เดินตามไปดูได้:
   - route/URL (ระบุว่า id ใน URL เป็นตัวอย่าง)
   - ต้องกดปุ่มอะไร / เปิด modal ไหน / ทำ action อะไร
   - เริ่มจาก Feature map (ถ้ามี) แล้วยืนยันกับโค้ดจริง — map ผิด/ไม่มี → ใส่ to-do "อัปเดต feature map"
   - หา route จริงไม่เจอ → บอกตรงๆ ห้ามเดา URL
4. ออกเป็น to-do list เรียงตามความสำคัญ
5. **หยุด รอผู้ใช้พิมพ์ยืนยัน**
   - FE sync อยู่แล้ว → บอก "ไม่ต้องแก้อะไร" พร้อมเหตุผล ห้ามสร้างงานที่ไม่จำเป็น
   - backend เปลี่ยนแค่ internal → บอก "ไม่มีผลกับ frontend"

---

## เฟส 2: ลงมือแก้ (หลังผู้ใช้ยืนยันเท่านั้น)

6. แก้ทีละจุดตาม to-do ที่ยืนยัน
7. รัน Checks ของโปรเจค — ต้องไม่มี error ตกค้าง
8. Verify ทุกจุดที่มี UI:
   - เช็ค Health endpoint ว่า backend ที่ใช้ตรวจมี commit ที่ sync แล้ว — ยังไม่ deploy → บอก
     "ตรวจกับ backend จริงไม่ได้ เพราะยังเป็นเวอร์ชันเก่า" ห้ามสรุปว่าผ่าน
   - ใช้วิธี Verify ของโปรเจค ดู request/response ว่าตรง contract ใหม่ (ชื่อ field, query param)
   - ไม่มีเครื่องมือเบราว์เซอร์ / backend ไม่เปิด → ตรวจระดับ API ด้วย curl แล้วบอกว่าจอยังไม่ได้ตรวจ
9. อัปเดต Last sync line ใน CLAUDE.md เป็น commit ล่าสุดที่วิเคราะห์ (+ วันที่) และอัปเดต Feature map ถ้า endpoint/ทางไปเปลี่ยน
10. สรุปผล 3 ส่วน:
    - **(ก) ไฟล์ที่แก้** — ชื่อไฟล์ + แก้อะไร
    - **(ข) ผล verify** — จุดที่ตรวจแล้ว (role, หน้า, request/response สำคัญ, ผ่าน/ไม่ผ่าน) + จุดที่ยังไม่ได้ตรวจพร้อมเหตุผล
    - **(ค) 🧭 วิธีทดสอบด้วยตา** — เฉพาะจุดที่ agent ตรวจเองไม่ได้: เปิด URL ไหน → กดอะไรตามลำดับ → ควรเห็นอะไร
      จุดที่ไม่มี UI (แก้ type ล้วน) → "ไม่มีจอให้ดู ยืนยันด้วย typecheck พอ"
