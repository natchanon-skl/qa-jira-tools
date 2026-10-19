---
name: qa-requirement-understanding
description: >
  เครื่องมือ Jira สำหรับ QA 3 คำสั่ง ใช้ skill นี้ทุกครั้งที่ user พิมพ์ /jx, /df หรือ /jd
  /jx = อธิบายการ์ด Jira (Story, Bug, Task) ให้ QA เข้าใจเร็ว ตอบสั้น 3 หัวข้อ
  (Overview, Actual vs Expected, Impact) แล้วให้เลือกดูหัวข้อเพิ่มด้วย checkbox
  เช่น /jx https://skilllane.atlassian.net/browse/MICA2-1637
  /df = format การ์ดเป็น 3 บรรทัดพร้อม copy ไปวางใน Discord (OPEN Defect P1 / Summary / URL)
  เช่น /df https://skilllane.atlassian.net/browse/MICA2-1647 หรือ /df URL P1
  /jd = draft การ์ด Bug จากหัวข้อสั้นๆ ไว้ copy ไปสร้างเอง (Title, Description, Environment, Steps, Expected, Actual)
  เช่น /jd EVMS Questionbank กดสร้างข้อสอบไม่ได้
  skill นี้อ่าน Jira อย่างเดียว ไม่สร้างหรือแก้ไขการ์ด
  ไม่ใช้เมื่อ user ขอ Test Case, Test Scenario, test plan หรือ automation script
---

# QA Jira Tools (/jx, /df, /jd)

| คำสั่ง | ทำอะไร | ไปที่ |
|---|---|---|
| `/jx <URL หรือ key>` | อธิบายการ์ด | ส่วน A |
| `/df <URL หรือ key> [P1-P4]` | format 3 บรรทัดไว้ copy | ส่วน B |
| `/jd <หัวข้อการ์ด>` | draft การ์ด Bug | ส่วน C |

ทุกคำสั่งทำงานเฉพาะเมื่อมี `/jx`, `/df` หรือ `/jd` นำหน้า แค่ส่ง Jira URL มาเฉยๆ ไม่ถือว่าเรียก skill นี้

## 🔒 Read-only: อ่าน Jira เท่านั้น

skill นี้ **อ่าน Jira ได้อย่างเดียว ห้ามเขียนหรือแก้ไขใดๆ ทั้งสิ้น** ไม่ว่า user จะสั่งหรือยืนยันก็ตาม
เพราะการ์ดใน Jira ถูกใช้ร่วมกันทั้งทีม การเปลี่ยนค่าจะแจ้งเตือนทุกคนและย้อนกลับยาก

- ใช้ได้เฉพาะ tool อ่าน: `getJiraIssue`, `searchJiraIssuesUsingJql`, `getConfluencePage`, `searchConfluenceUsingCql`, `fetch`
- ห้ามใช้: `createJiraIssue`, `editJiraIssue`, `transitionJiraIssue`, `addCommentToJiraIssue`, `addWorklogToJiraIssue`, `createIssueLink`, `createConfluencePage`, `updateConfluencePage` และ tool เขียนอื่นทั้งหมด
- ถ้า user ขอให้สร้าง แก้ เปลี่ยน status หรือ comment ให้ตอบ 1 บรรทัดว่า skill นี้อ่านได้อย่างเดียว และให้ทำเองใน Jira

---

# ส่วน A — /jx อธิบายการ์ด

ช่วย QA เข้าใจว่าการ์ดทำอะไร ระบบควรทำงานอย่างไร และกระทบส่วนไหน
เหมือน BA อธิบายให้ฟัง **ไม่สร้าง Test Case, Test Steps, Test Data หรือ script**
เพราะ skill นี้คือขั้นทำความเข้าใจก่อนออกแบบ test ถ้าสร้าง TC ปนมา QA จะแยกไม่ออกว่าอะไรคือ requirement จริง

คำสั่ง `/jx` มีลำดับสูงกว่า workflow เขียน TC ใน user preferences:
เมื่อเจอ `/jx` ให้ทำตาม skill นี้อย่างเดียว ไม่ต้องสรุป req, risk หรือถาม clarification แบบ workflow TC

## A1. รับ input

- ดึง **issue key** (เช่น `MICA2-1637`) และ **site** (เช่น `skilllane.atlassian.net`) จาก URL
- ถ้าได้แค่ key ให้ใช้ site เดิมที่เคยใช้ในบทสนทนา หรือถามถ้าไม่รู้
- หลาย key: สรุปแยกทีละการ์ด

## A2. อ่านข้อมูล

ใช้ Atlassian Rovo connector (`getJiraIssue`) อ่านก่อนเสมอ:
- เรียกด้วย `fields: ["*all"]` และ `expand: "names"` เพื่อให้รู้ชื่อจริงของ custom field
  (เช่น Severity, Test Status) แทนที่จะเห็นแค่ `customfield_11368`
- อ่าน Summary, Type, Status, Priority, Fix Version, Description, AC, Comments, Issue Links, Subtasks
- Linked issue / Epic / Confluence: อ่าน **1 hop เท่านั้น** และเฉพาะเมื่อจำเป็นต่อความเข้าใจ
  ถ้ามีมากกว่า 5 ใบ ให้ list ชื่อไว้ อย่าอ่านทั้งหมด (context บวม คำตอบช้า)

### อ่าน PRD ใน Confluence ทุกครั้ง

Jira หลายใบมี description น้อยหรือว่าง PRD จึงเป็นแหล่งหลักของ Expected และ Business Rules

- หน้าดัชนี: `https://skilllane.atlassian.net/wiki/spaces/PLUT/pages/3693379606/Product+Requirement+Documents` (page id `3693379606`)
- โครงสร้าง: ดัชนีรวม → ดัชนีระบบย่อย (OLS / ELMS / CBMS / EvMS) → PRD 1 หน้าต่อ 1 Epic
  ชื่อหน้าเป็น `[Subsystem] PRD - [EPIC-KEY] Feature` และ PRD ยาวจะแบ่งเป็นหน้า `(ต่อ 1)`, `(ต่อ 2)`, ...
- วิธีหา PRD ที่ตรงการ์ด:
  1. หา Epic ของการ์ดจาก field Parent (หรือ Parent ของ Parent ถ้าเป็น subtask)
  2. ค้นด้วย `searchConfluenceUsingCql`: `ancestor = 3693379606 AND type = page AND title ~ "EPIC-KEY"`
     แล้วเลือกหน้าที่ตรงกับเรื่องในการ์ด (ดูจาก excerpt) อย่าอ่านทุกหน้า `(ต่อ N)`
  3. ถ้า Parent ไม่ใช่ Epic ฟีเจอร์ (เช่น Epic "Regression Golive 68") ให้ค้นด้วย keyword ภาษาอังกฤษจาก Summary
     เช่น `ancestor = 3693379606 AND text ~ "Center Admin"` เพราะการค้นข้อความภาษาไทยใน Confluence มักไม่เจอ
- ถ้าเจอ PRD: ใช้เป็นแหล่งของ Expected / Business Rules และอ้างชื่อหน้า + หัวข้อ เช่น `[PRD MICA2-630 §3.2]`
- ถ้าไม่เจอหรือไม่แน่ใจว่าตรง: บอกตรงๆ ว่าไม่พบ PRD ที่เกี่ยวข้อง และ Expected ยังเป็น "(คาดว่า)"
- ถ้า PRD ขัดกับ Jira: รายงานทั้งสองฝั่งใน Notes & Questions ไม่ตัดสินเอง

ถ้าอ่าน Jira ไม่ได้ ให้บอกตรงๆ และขอให้ user paste เนื้อหามา
**ห้ามสรุปจากชื่อการ์ดอย่างเดียว** เพราะ QA จะเชื่อว่าเป็นข้อมูลจริง
ถ้ายังไม่ได้เปิดไฟล์แนบ/ลิงก์ (รูป, JSON, วิดีโอ, Drive) อย่าอ้างว่าอ่านแล้ว

## A3. คำตอบรอบแรก (สั้นเสมอ)

```
### [KEY] · [Type] · [Status]

**Overview**
2 ประโยค: ใคร ทำอะไร เกิดอะไรขึ้น / ต้องการอะไร

**Actual vs Expected**        ← Bug
| จุด | Actual | Expected |    ← เฉพาะจุดสำคัญ 3–5 แถว

**Expected Behavior**         ← Story / Task (ใช้แทน Actual vs Expected)
2–4 ข้อสั้น

**Impact**
2–3 ข้อ: flow / role / module ที่กระทบ และที่ไม่กระทบ (ถ้า Jira ระบุ)
```

- ไม่ใส่ Business Rules, Root Cause, Notes, Open Questions ในรอบนี้
- อิงสิ่งที่ Jira ระบุเท่านั้น ถ้าเป็นการคาดเดาให้เขียนว่า "(คาดว่า)"
- ใช้ศัพท์ระบบเดิม (Private, Wizard, 403) ไม่ต้องอธิบายเพิ่ม

## A4. Checkbox ให้เลือกดูต่อ

หลังคำตอบรอบแรก ให้เรียก `ask_user_input_v0` (multi_select) ทันที เครื่องมือรับได้สูงสุด 4 ตัวเลือก:

**Bug:**
- `Business Rules — กฎและสิทธิ์ที่ระบบต้องทำตาม`
- `Root Cause — สาเหตุที่ทำให้เกิด Bug`
- `Notes & Questions — จุดที่ต้องระวัง และข้อที่ยังต้องถามทีม`
- `ไม่ต้องการ — จบแค่นี้`

**Story / Task:** เปลี่ยนตัวที่ 2 เป็น `State/Flow — สถานะและลำดับการทำงานของฟีเจอร์`

คำถาม: `[KEY] — ดูอะไรต่อ?`
ถ้าไม่มีเครื่องมือนี้ ให้แสดงตัวเลือกเดียวกันเป็นข้อความ 1 บรรทัด

ถ้า user เลือก "ไม่ต้องการ" ให้ตอบสั้นๆ ว่าจบการ์ดนี้ ไม่ต้องเสนออะไรต่อ

## A5. หัวข้อเพิ่มเติม (เมื่อถูกเลือก)

แสดงเฉพาะที่เลือก เรียงตามนี้ แต่ละหัวข้อ 2–4 ข้อสั้น

- **Business Rules** — กฎที่เปลี่ยนความหมายของฟีเจอร์: สิทธิ์ เงื่อนไขสถานะ ข้อจำกัด
- **Root Cause** — อธิบายแบบคนทั่วไปก่อน แล้วค่อยระบุ API/field ที่จำเป็น
  ใช้เฉพาะเมื่อ Jira มีหลักฐาน ถ้าไม่มีให้เขียนว่า "Jira ไม่ได้ระบุสาเหตุ"
  ถ้าเป็นสมมติฐานให้เขียนว่า "สาเหตุที่คาดว่าเป็นไปได้"
- **State/Flow** — สถานะและการเปลี่ยนสถานะ หรือลำดับขั้นของ flow ที่ requirement พูดถึง
- **Notes & Questions**
  - Notes: ข้อจำกัด สถานะการทดสอบตาม comment (ระบุ environment และวันที่), Fix Version released หรือยัง
  - Questions: ข้อมูลที่ขัดกันหรือขาด ระบุว่าอะไรขัดกับอะไร และควรถามใคร (PO / Dev / BA)
  - ถ้าไม่มี ให้เขียนว่า "ไม่พบประเด็นที่ต้องยืนยันเพิ่ม"

## A6. หลักความถูกต้อง

- ห้ามแต่ง requirement, expected result หรือ root cause เพราะ QA จะใช้ข้อมูลนี้กำหนด scope ของ regression
- **แนวทางแก้ที่ Jira เสนอ ≠ สิ่งที่แก้จริง** ถ้าไม่มี comment/PR ยืนยัน ให้ใส่ไว้ใน Questions
- Comment ใหม่ไม่ได้ชนะ AC เสมอ ถ้าขัดกันให้รายงานทั้งสองฝั่ง
- Status กับหลักฐานทดสอบ: รายงานทั้งสองอย่าง ไม่สรุปเองว่าปิดงานแล้ว
  (เช่น comment บอก pass แต่ status ยัง Ready to Test)
- Priority ต้องมาจาก field จริง ถ้า Summary บอก P1 แต่ field เป็น Medium ให้ใส่ใน Questions

## A7. ข้อมูลส่วนบุคคล

ห้ามแสดง: อีเมล, User ID/UUID ของบุคคล, ชื่อบัญชีทดสอบ, ชื่อโรงเรียน/ลูกค้าในบัญชีทดสอบ
อ้างถึงคนด้วยบทบาทแทนชื่อ เช่น "Dev (assignee)", "QA", "PO"
UUID ในตัวอย่าง API แสดงได้แค่ระดับรูปแบบ เช่น "ได้ UUID" ไม่ต้องใส่ค่าจริง

## ตัวอย่าง (แสดงสไตล์เท่านั้น — ต้องอ่าน Jira จริงทุกครั้ง)

Input: `/jx PROJ-123`

```
### PROJ-123 · Bug · Ready to Test

**Overview**
ผู้ใช้ที่เปลี่ยนภาษาเป็นอังกฤษแล้ว refresh หน้า ระบบกลับเป็นภาษาไทย ต้องเลือกใหม่ทุกครั้ง

**Actual vs Expected**
| จุด | Actual | Expected |
|---|---|---|
| ภาษาหลัง refresh | กลับเป็นไทย | คงเป็นอังกฤษ |
| ภาษาหลัง login ใหม่ | กลับเป็นไทย | ใช้ภาษาที่เลือกล่าสุด |

**Impact**
- ทุกหน้าที่มีตัวเลือกภาษา
- ไม่กระทบผู้ใช้ที่ใช้ภาษาไทยเป็นค่าเริ่มต้น
```

→ ตามด้วย checkbox

---

# ส่วน B — /df Discord format

ใช้ส่งการ์ดเข้า Discord ของทีมแบบ copy ได้ทันที ทำงานเลยโดยไม่ต้องถามยืนยัน

## B1. ขั้นตอน

1. ดึง issue key จาก URL (ตัด query parameter ทิ้ง)
2. อ่าน issue ผ่าน Atlassian Rovo (`getJiraIssue`) ต้องได้ Summary, Issue Type, Priority จริง **ห้ามเดา**
3. แปลงค่าตามกฎ B2 แล้วตอบเป็น code block 3 บรรทัด

## B2. กฎการแปลง

**บรรทัด 1: `OPEN [Type] [P]`**
- `OPEN` ใส่เสมอ ไม่ว่า status จริงจะเป็นอะไร (เป็น convention ของทีม)
- Type: `Bug` → `Defect` ส่วน type อื่นใช้ตาม Jira (Story, Task, ...)
- Priority เลือกตามลำดับนี้:
  1. user ระบุมาในคำสั่ง (`/df <URL> P1`) → ใช้ค่านั้น
  2. Summary มี P1–P4 (เช่น `[P2]` หรือ ` P1 `) → ใช้ค่าจาก Summary
  3. ไม่มีทั้งสองแบบ → แปลงจาก Priority field: Highest/High = P1, Medium = P2, Low = P3, Lowest = P4
  4. ถ้า field เป็นชื่ออื่นที่ไม่อยู่ในตารางข้อ 3 → ถาม user ห้ามเดา

**บรรทัด 2:** Summary ตรงตัวตาม Jira ทุกตัวอักษร (ไทย, วงเล็บ, เครื่องหมาย, ช่องว่าง, P ที่อยู่ใน Summary) ห้ามตัดหรือจัดใหม่

**บรรทัด 3:** `https://<site>/browse/<KEY>` ไม่ใช้ markdown link

## B3. Output

ตอบเป็น code block อย่างเดียว ไม่มีคำทักทาย คำอธิบาย หรือ emoji:

```text
OPEN Defect P1
[BUG][EVMS][คาบสอบ]  P1 ชุดข้อสอบส่วนตัวของครูเอง ...
https://skilllane.atlassian.net/browse/MICA2-1637
```

ข้อยกเว้นที่ต้องเขียนเพิ่ม 1 บรรทัดใต้ code block:
- user override P → `P ใน Jira ยังเป็น [ค่าเดิม] (ไม่ได้แก้ใน Jira)`
- หา issue ไม่เจอ/ไม่มีสิทธิ์ → บอกสั้นๆ ไม่สร้าง output

ห้ามแก้ค่าใน Jira (ดูกฎ Read-only)

หลาย URL ใน `/df` เดียว → ตอบเป็น code block เดียว คั่นแต่ละการ์ดด้วยบรรทัดว่าง

---

# ส่วน C — /jd Jira draft

user พิมพ์หัวข้อการ์ดสั้นๆ (อาจสะกดผิด ผสมไทย/อังกฤษ) แล้วให้ draft การ์ด Bug ให้เลย ไม่ต้องถามก่อน

## C1. หลักการเดา

- เดาได้จากหัวข้อและความรู้ทั่วไปเรื่อง flow ของระบบ แต่ **ทุกส่วนที่เดาต้องมี 🔸 กำกับ**
  เพราะการ์ดจะถูกส่งให้ Dev ถ้าส่วนที่เดาดูเหมือนข้อเท็จจริง Dev จะไล่หาผิดจุด
- สิ่งที่รู้ไม่ได้จากหัวข้อ (URL, User, ข้อความ error จริง) ให้เว้นว่าง ห้ามแต่ง
- **Actual Result** มักกำกวม (เช่น "กดไม่ได้" อาจเป็น ไม่มีอะไรเกิดขึ้น / ขึ้น error / ค้างโหลด / บันทึกไม่ผ่าน / ปุ่ม disable)
  ให้เขียนประโยคหลักแล้วใส่ตัวเลือกใน `[...]` ให้ user เลือก
- แก้คำสะกดผิดใน Title ให้ (เช่น Qustionbank → Question Bank / คลังข้อสอบ)

## C2. Template

ใช้ตามนี้เท่านั้น ห้ามเพิ่ม Severity, Priority หรือ field อื่นใน Environment

```
**Title:** `[BUG][<Module>][<Feature ภาษาไทย>] <อาการสั้น ชัด>`

**Description**
1–2 ประโยค: ใครทำอะไรไม่ได้ / เกิดอะไรผิด 🔸

**Environment**
URL :
User :

**Steps to Reproduce** 🔸
1. ...  (1 step = 1 action ขึ้นต้นด้วย verb)

**Expected Result** 🔸
ผลที่สังเกตได้ชัด ห้ามใช้คำว่า "ทำงานถูกต้อง" / "ปกติ"

**Actual Result**
<ประโยคหลัก> `[ตัวเลือก 1 / ตัวเลือก 2 / ...]`

**Attachment:** `[screenshot / video / log]`
```

Title convention: `[BUG][Module][Feature]` ตาม prefix ที่ทีมใช้ (เช่น `[BUG][EVMS][คาบสอบ]`)

## C3. หลัง draft

- `/jd` สร้าง draft ใน chat เท่านั้น **ไม่สร้างการ์ดใน Jira** (ดูกฎ Read-only)
- จบด้วยบรรทัดเดียว: `copy ไปสร้างการ์ดใน Jira ได้เลย`
- ถ้ายังมี `[...]` หรือ URL/User ว่าง ให้เตือนในบรรทัดเดียวกัน เช่น `copy ไปสร้างการ์ดได้เลย (ยังต้องเลือก Actual และเติม URL/User)`
