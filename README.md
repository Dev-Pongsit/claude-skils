# claude-skils

ตัวอย่าง Agent Skills ที่ใช้กับงาน frontend (Claude Code และ agent ที่รองรับมาตรฐาน Agent Skills)

| Skill | ใช้ทำอะไร | ขอบเขต |
|---|---|---|
| [figma-ui-update](skills/figma-ui-update/) | ปรับ UI ที่มีอยู่ให้ตรง design ล่าสุดจาก Figma — วิเคราะห์ส่วนต่าง → รออนุมัติ → แก้เฉพาะส่วนที่อนุมัติ | ทุก frontend project |
| [sync-backend](skills/sync-backend/) | อ่าน commit ล่าสุดของ backend (read-only) → วิเคราะห์ผลกระทบ API contract ต่อ frontend → แก้ FE หลังยืนยัน | ทุกโปรเจคที่ FE/BE แยก repo |
| [verify-in-browser](skills/verify-in-browser/) | รันแอปแล้วตรวจพฤติกรรมจริง (จอ, network, console) ตาม feature map ก่อนบอกว่างานเสร็จ หรือเพื่อ reproduce defect | ตัวอย่างจากโปรเจค Next.js หนึ่ง — ต้องปรับ `feature-map.md` และคำสั่งให้ตรงโปรเจคตัวเอง |

## ติดตั้ง

```bash
git clone https://github.com/Dev-Pongsit/claude-skils.git

# ทุก project ในเครื่อง
cp -r claude-skils/skills/<skill-name> ~/.claude/skills/

# เฉพาะ project เดียว
cp -r claude-skils/skills/<skill-name> <project>/.claude/skills/
```

ไม่ต้องติดตั้ง dependency เพิ่ม เรียกใช้ด้วย `/<skill-name>` หรือพูดถึงงานที่ตรงกับ description ของ skill
