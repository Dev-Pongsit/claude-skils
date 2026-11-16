# figma-ui-update

AI Skill สำหรับปรับ UI ที่มีอยู่แล้วให้ตรงกับ design ล่าสุดจาก Figma อย่างปลอดภัย

## ปัญหาที่แก้

เวลาสั่ง AI ว่า "แก้หน้านี้ให้ตรง Figma" มักเจอปัญหาเหล่านี้:

- AI แก้โค้ดทันทีโดยไม่สรุปก่อนว่าจะแก้อะไร ทำให้รีวิวยาก
- เขียนหน้าใหม่ทั้งหน้า หรือ copy SVG มาใส่ใน source
- แตะ business logic, API หรือ route เพื่อให้ภาพตรง ทำให้ฟีเจอร์เดิมพัง
- เดา field หรือ endpoint ที่ไม่มีอยู่จริง
- แก้ shared component แล้วกระทบหน้าอื่นโดยไม่รู้ตัว
- ไม่บอกว่าต้องไปทดสอบที่หน้าไหน และรายงานว่า "ผ่าน" ทั้งที่ยังไม่ได้ตรวจ

Skill นี้บังคับให้ AI ทำงานเป็น 2 phase: **วิเคราะห์ → รออนุมัติ → แก้เฉพาะส่วนต่าง** และรายงานผลในรูปแบบเดิมทุกครั้ง

## โครงสร้าง

```
figma-ui-update/
├── SKILL.md                          # workflow หลัก (AI อ่าน)
├── README.md                         # ไฟล์นี้ (คนอ่าน)
├── references/
│   ├── change-classification.md      # checklist design, กลุ่มส่วนต่าง, ป้ายผลกระทบ, priority
│   ├── phase1-report-template.md     # template รายงานวิเคราะห์
│   └── phase2-report-template.md     # template รายงานหลังแก้
└── evaluation/
    └── comparison.md                 # ผลเทียบ AI ปกติกับ AI ที่มี skill
```

## ติดตั้ง

clone repo แล้ว copy โฟลเดอร์ `skills/figma-ui-update` ไปไว้ในโฟลเดอร์ skills ตามขอบเขตที่ต้องการ:

```bash
git clone https://github.com/Dev-Pongsit/claude-skills.git

# ใช้ได้ทุก project ในเครื่อง
cp -r claude-skills/skills/figma-ui-update ~/.claude/skills/

# ใช้เฉพาะ project เดียว
cp -r claude-skills/skills/figma-ui-update <project>/.claude/skills/
```

หรือดาวน์โหลดเป็น zip แล้วแตกไฟล์ไปไว้ที่ path เดียวกัน

ใช้ได้กับ Claude Code และ agent อื่นที่รองรับมาตรฐาน Agent Skills ไม่ต้องติดตั้ง dependency เพิ่ม

## วิธีใช้

### 1. เตรียม design

export หน้าจอจาก Figma เป็น SVG หรือ PNG แล้วไว้ใน `docs/design/` (หรือส่ง Figma URL มาก็ได้ ถ้ามี Figma MCP)

### 2. เรียก skill

```text
/figma-ui-update Link-Setting.svg
/figma-ui-update --feature="Users" --design="docs/design/users.svg"
```

หรือพิมพ์บอกเป็นภาษาปกติ เช่น "แก้หน้า Link Setting ให้ตรงกับ design" แล้ว AI จะเรียก skill ให้เอง

### 3. อ่านรายงาน Phase 1

AI จะยังไม่แก้โค้ด แต่จะส่งรายงาน `UI Change Analysis` มาก่อน และบันทึกไว้ที่ `docs/ui-analysis/<feature>-change-analysis.md` ในรายงานมี:

- ส่วนต่างระหว่าง design กับโค้ด แบ่งเป็นกลุ่ม A–F พร้อมป้าย `UI-only` / `New behavior` ฯลฯ
- ผลกระทบต่อ API
- รายการ MUST CHANGE / SHOULD CHANGE / BLOCKED / MUST NOT CHANGE
- to-do ที่มี ID และ priority
- วิธีเปิดดูแต่ละจุดในแอป

### 4. อนุมัติ

```text
ยืนยัน Phase 2
แก้เฉพาะ ID 1, 2, 4
```

### 5. ตรวจผล Phase 2

AI จะแก้เฉพาะ ID ที่อนุมัติ รัน typecheck และ lint แล้วส่งรายงาน `UI Update Result` พร้อม manual test checklist ให้นำไปทดสอบในแอปต่อ

## Input / Output โดยย่อ

| | รายละเอียด |
|---|---|
| **Input** | ไฟล์ design (SVG/PNG/JPG หรือ Figma URL) และชื่อ feature ถ้ามี |
| **Output Phase 1** | รายงานวิเคราะห์ + to-do (ยังไม่แก้โค้ด) |
| **Output Phase 2** | โค้ดที่แก้แล้ว + รายงานผล + ผล typecheck/lint + manual test checklist |

## ใช้ซ้ำกับ project อื่น

- skill ไม่ได้ผูก path, framework หรือชื่อ feature ไว้ แต่จะอ่าน `package.json` และโครง project เองใน Step 0
- ถ้า design หรือ report อยู่ที่อื่น ให้บอก path ตอนเรียกใช้
- ถ้า project มี `CLAUDE.md` กำหนด convention ไว้ skill จะยึดตามนั้นก่อน

## ข้อจำกัด

- ไฟล์ SVG ขนาดใหญ่มาก (หลาย MB) อ่านได้แค่ข้อความและ token ถ้ามี PNG คู่กันจะวิเคราะห์ได้แม่นขึ้น
- skill ไม่เปิด browser เอง ผลด้าน responsive จะถูกรายงานเป็น `NOT VERIFIED` จนกว่าจะมีคนทดสอบจริง
- skill ไม่แก้ backend ถ้า design ต้องใช้ข้อมูลใหม่ จะถูกจัดเป็น BLOCKED
