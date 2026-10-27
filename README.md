# QA Jira Tools — Claude Skill

Claude skill สำหรับงาน QA กับ Jira (อ่านอย่างเดียว)

| คำสั่ง | ทำอะไร | ตัวอย่าง |
|---|---|---|
| `/jx <URL หรือ key>` | อธิบายการ์ด Jira: Overview, Actual, Expected, วิธี Test คร่าวๆ แล้วให้เลือกดูหัวข้อเพิ่มด้วย checkbox | `/jx https://skilllane.atlassian.net/browse/MICA2-1637` |
| `/df <URL หรือ key> [P1-P4]` | format การ์ดเป็น 3 บรรทัดพร้อม copy ไปวางใน Discord | `/df https://skilllane.atlassian.net/browse/MICA2-1647 P1` |
| `/jd <หัวข้อการ์ด>` | draft การ์ด Bug (Title, Description, Environment, Steps, Expected, Actual) ไว้ copy ไปสร้างใน Jira | `/jd EVMS Questionbank กดสร้างข้อสอบไม่ได้` |

## สิ่งที่ต้องมี
- Claude (claude.ai) ที่เชื่อม connector **Atlassian Rovo** ด้วยบัญชี Jira ของตัวเอง
- สิทธิ์อ่าน Jira project MICA2 และ Confluence space PLUT

## วิธีติดตั้ง
1. ดาวน์โหลดไฟล์ `qa-requirement-understanding.skill` จาก Releases (หรือ zip โฟลเดอร์ `qa-requirement-understanding/` เอง)
2. Claude → Settings → Capabilities → Skills → Upload skill

## ความปลอดภัย
skill นี้ **อ่านได้อย่างเดียว** ไม่สร้าง แก้ไข เปลี่ยน status หรือ comment ใน Jira/Confluence

## โครงสร้าง
```
qa-requirement-understanding/
└── SKILL.md
```
