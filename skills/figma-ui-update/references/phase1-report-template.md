# Phase 1 Report Template

บันทึกเป็นไฟล์ `<report-dir>/<feature-kebab>-change-analysis.md` และตอบสรุปในแชต

````markdown
# UI Change Analysis — <Feature>

## 1. Target Feature
Target Feature:
Design File:
Design File Path:
Related Feature Guess:
Confidence: High | Medium | Low

## 2. Design File ที่ตรวจสอบ
- วิธีอ่าน (เปิดเป็นรูป / grep SVG / Figma MCP)
- ส่วนที่พบใน design

## 3. Frontend Files ที่ตรวจสอบ
| File | บทบาท | ยืนยันจาก |
|---|---|---|

## 4. Current Implementation Summary

## 5. Design Differences
| # | กลุ่ม (A–F) | Current | Design | ป้าย | File |
|---|---|---|---|---|---|

## 6. API/Data Impact
| Field/Data | Endpoint | Frontend type | Component | Impact |
|---|---|---|---|---|
สรุป: UI-only change — ไม่กระทบ API contract | BLOCKED

## 7. UI Location สำหรับ Manual Testing
UI Location:
- Route/URL:
- Route source:
- วิธีเปิด:
- Modal/Dialog:
- จุดที่ต้องสังเกต:

Shared Component (ถ้ามี):
- Component:
- ไฟล์:
- ถูกใช้งานที่:
- Risk ต่อหน้าอื่น:

## 8. MUST CHANGE
## 9. SHOULD CHANGE
## 10. BLOCKED / NEED CONFIRMATION
## 11. MUST NOT CHANGE

## 12. Ordered To-do List
ID:
Priority: P1 | P2 | P3
File:
Component:
Current Behavior:
Expected Behavior:
Change Type: (ป้าย)
API Impact:
Business Logic Impact:
Risk:
Manual Test Location:

## 13. Risk Assessment
## 14. Proposed Files to Change
## 15. Confirmed Unchanged Files/Features

```text
PHASE 1 STATUS: ANALYSIS COMPLETE
ยังไม่มีการแก้ไข application code
รอผู้ใช้ยืนยันก่อนเริ่ม PHASE 2
```
````
