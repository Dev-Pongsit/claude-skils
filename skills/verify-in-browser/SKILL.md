---
name: verify-in-browser
description: รัน tso-web-client แล้วตรวจพฤติกรรมจริง (หน้า, ปุ่ม, request/response, console) ตาม feature map ก่อนบอกว่างานเสร็จ หรือเพื่อ reproduce defect ที่ผู้ใช้/tester แจ้ง ใช้เมื่อ "ตรวจ/verify/ลองกดดู/reproduce/เช็คในแอปจริง" หรือก่อนรายงานว่า FIX/DEV เสร็จ
---

# verify-in-browser

lint/typecheck ผ่าน ≠ ใช้งานได้ skill นี้คือวิธีพิสูจน์ว่าหน้าจอทำงานจริง

## 0. กฎ

- **ห้ามรัน/แก้ backend** (`../tso-web-api` read-only) — ถ้าต้องใช้ backend แต่ไม่ได้เปิด ให้ถาม user
- **ห้าม kill dev server ของ user** — มีตัวรันอยู่แล้วให้ใช้ตัวนั้น
- **ห้ามพิมพ์ค่าใน `.env*`** — ดูได้แค่ว่ามี key อะไร (`grep -oE '^[A-Z_]+' .env`)
- mutation บน backend จริงเขียนลง **DB ที่ tester ใช้ร่วม** → สร้าง/แก้เฉพาะข้อมูลทดสอบ, ถามก่อนลบ/
  เปลี่ยนสถานะงานของคนอื่น
- ตรวจไม่ได้ → รายงานตรงๆ ว่า "ยังไม่ได้ verify" พร้อมเหตุผล ห้ามบอกว่าเสร็จ

## 1. เลือกโหมด

| โหมด | ใช้เมื่อ | เงื่อนไข |
|---|---|---|
| **real** (backend) | งาน contract, สิทธิ์, ข้อมูลจริง, defect จาก tester | `.env` มี `NEXT_PUBLIC_API_URL` และ backend ตอบ `/api/health` |
| **mock** (MSW) | UI ล้วน, backend ไม่พร้อม | ไม่ตั้ง `NEXT_PUBLIC_API_URL` (ต้อง restart dev server — ถาม user ก่อนสลับ) |

`NEXT_PUBLIC_*` ฝังตอน start → เปลี่ยน `.env` ต้อง restart dev server

## 2. รันแอป

```bash
# มี dev server อยู่แล้วไหม (Bash บน Windows)
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:3000/login
```
- ได้ `200` → ใช้ตัวนั้นเลย
- ไม่ได้ → `npm run dev` แบบ `run_in_background` แล้วรอจน `/login` ตอบ 200 (compile ครั้งแรก ~30–60s)
- โหมด real: เช็ค backend แล้วจดเวอร์ชันไว้ในรายงาน
  ```bash
  curl -s "<API_URL>/api/health"     # API_URL อ่านจาก .env ห้าม echo ค่าออกมาในรายงาน ถ้าเป็นโดเมนภายใน
  ```

## 3. Login ตาม role

หน้า `/login` (โหมด `NEXT_PUBLIC_AUTH_MODE=mock`) มี dropdown รวมบัญชี mock และบัญชี real backend
(`src/features/auth/components/MockLoginForm.tsx:38`) — ดูตาราง "บัญชีทดสอบ" ใน `feature-map.md`

- real: เลือกบัญชี `…@example.com` ใน dropdown (ยิง `POST {API_URL}/api/auth/login` body `{ email }`
  — dev-only bypass ไม่ใช้รหัสผ่าน ใช้ได้เฉพาะ backend ที่รัน Development)
- API ล้วน (ไม่ต้องดูจอ):
  ```bash
  curl -s -X POST "<API_URL>/api/auth/login" -H "Content-Type: application/json" \
       -d '{"email":"planner@example.com"}'          # → token ใน response ใช้เป็น Bearer (ดูชื่อ field จริงจาก response/authApi.ts)
  ```
- token เก็บใน `localStorage` (`src/lib/auth/tokenStorage.ts`) — สลับ role: logout (header → ยืนยัน) แล้ว login ใหม่
- งานแตะสิทธิ์: ตรวจ **2 role เสมอ** (ควรเห็น 1 + ไม่ควรเห็น 1) บนงานชิ้นเดียวกัน

## 4. ตรวจในเบราว์เซอร์

เครื่องมือ (เลือกตัวที่มีใน session): Claude in Chrome (`mcp__claude-in-chrome__*`, โหลด skill
`chrome-browser` ก่อน) → built-in browser (โหลด skill `built-in-browser`) → ถ้าไม่มีเลย ตรวจระดับ API ด้วย curl
และบอก user ว่าตรวจจอไม่ได้

1. เปิด `feature-map.md` หา route + ทางไปถึงส่วนที่แก้
2. เดินตาม path เหมือนผู้ใช้ (คลิกจริง ไม่พิมพ์ URL ข้ามขั้นถ้า defect เกิดจากการนำทาง)
3. ตรวจ 3 ชั้น:
   - **จอ**: ข้อความ/ปุ่ม/สถานะตรงที่คาด (ภาษาไทย, วันที่ พ.ศ./เวลา Bangkok)
   - **Network**: request ออกจริง, method/path/query/body ตรง contract, status code — ไม่มี request = บั๊ก FE
   - **Console**: ไม่มี error ใหม่ (React key, hydration, unhandled rejection)
4. Mutation: ต้องเห็น toast **Completed** / **Request Failed** มุมขวาบน แล้ว **reload** ยืนยันว่าข้อมูล persist
5. เปิดงานจากตาราง/การ์ด → ส่วนใหญ่เปิด **แท็บใหม่** (`window.open`) ต้องสลับไปแท็บนั้นก่อนตรวจ
6. Regression เบาๆ: เปิดอีก 1 หน้าที่ใช้ component/map เดียวกัน (ดูคอลัมน์ "ใช้ร่วมกับ" ใน feature map)

## 5. Reproduce defect จาก tester

1. ถามหรือหา: บัญชี/role, URL หรือเลขงาน, ขั้นตอน, สิ่งที่เห็น vs ที่คาด
2. defect สิทธิ์/ข้อมูล: เช็ค `/api/health` + หน่วยงานของบัญชี + `can_write`/`why_not_write` ก่อนแตะโค้ด
3. reproduce ให้ได้ก่อนแก้ — ทำไม่ได้ให้รายงานว่า reproduce ไม่ได้ และสภาพที่ลอง
4. แก้เสร็จ → ทำขั้นตอนเดิมซ้ำ ต้องผ่าน

## 6. รายงาน (แนบท้ายทุกงาน)

```
Verify: real (API <version>) | mock
Role: planner@example.com (Region Requester), approver@example.com (Approver)
หน้า: /plans/2009 → ปุ่ม Duplicate
ผล: ✅ requester หน่วยอื่นไม่เห็นปุ่ม · ✅ approver เห็น · Network: GET /api/work-plans/2009 can_write=false
ยังไม่ได้ตรวจ: <ถ้ามี + เหตุผล>
```
