# Codex Skills Inventory

Updated: 2026-06-30

ไฟล์นี้เก็บรายชื่อ skill ที่มีในเครื่องนี้ เพื่อใช้เป็น checklist เวลาย้ายไปตั้งค่า Codex บนเครื่องอื่น

## รายชื่อแบบสั้น

- imagegen
- openai-docs
- plugin-creator
- skill-creator
- skill-installer
- browser:control-in-app-browser
- creating-mermaid-diagrams
- defuddle
- diagram-design
- documents:documents
- drawio-skill
- excalidraw
- it-governance-ops
- json-canvas
- obsidian
- obsidian-bases
- obsidian-cli
- obsidian-markdown
- pdf:pdf
- presentations:Presentations
- spreadsheets:Spreadsheets
- svg-diagram
- template-creator:template-creator

## Checklist สำหรับติดตั้ง/ตรวจสอบบนเครื่องอื่น

| สถานะ | Skill | ใช้สำหรับ | แหล่งที่มาบนเครื่องนี้ |
|---|---|---|---|
| [ ] | imagegen | สร้างหรือแก้ไขภาพ bitmap ด้วย AI | `C:/Users/kanit/.codex/skills/.system/imagegen/SKILL.md` |
| [ ] | openai-docs | ค้น/อ้างอิงเอกสารทางการของ OpenAI และ Codex | `C:/Users/kanit/.codex/skills/.system/openai-docs/SKILL.md` |
| [ ] | plugin-creator | สร้างหรือปรับปรุง Codex plugin ส่วนตัว | `C:/Users/kanit/.codex/skills/.system/plugin-creator/SKILL.md` |
| [ ] | skill-creator | สร้างหรือปรับปรุง Codex skill | `C:/Users/kanit/.codex/skills/.system/skill-creator/SKILL.md` |
| [ ] | skill-installer | ติดตั้ง skill จาก curated list หรือ GitHub repo | `C:/Users/kanit/.codex/skills/.system/skill-installer/SKILL.md` |
| [ ] | browser:control-in-app-browser | ควบคุม/ทดสอบ browser ภายใน Codex | `C:/Users/kanit/.codex/plugins/cache/openai-bundled/browser/26.623.61825/skills/control-in-app-browser/SKILL.md` |
| [ ] | creating-mermaid-diagrams | สร้าง Mermaid diagram และ export เป็นไฟล์ภาพ | `C:/Users/kanit/.codex/skills/mermaid-skill/SKILL.md` |
| [ ] | defuddle | ดึงเนื้อหา Markdown สะอาดจากเว็บเพจ | `C:/Users/kanit/.codex/skills/defuddle/SKILL.md` |
| [ ] | diagram-design | สร้าง diagram เป็น HTML/SVG แบบ standalone | `C:/Users/kanit/.codex/skills/diagram-design/SKILL.md` |
| [ ] | documents:documents | สร้าง/แก้ไข/ตรวจ `.docx` และเอกสาร Word | `C:/Users/kanit/.codex/plugins/cache/openai-primary-runtime/documents/26.623.12021/skills/documents/SKILL.md` |
| [ ] | drawio-skill | สร้าง diagram ด้วย draw.io XML และ export | `C:/Users/kanit/.codex/skills/drawio-skill/SKILL.md` |
| [ ] | excalidraw | สร้าง diagram/flowchart แบบ Excalidraw | `C:/Users/kanit/.codex/skills/excalidraw-skill/SKILL.md` |
| [ ] | it-governance-ops | งาน IT operations, ISO 27001, audit, compliance, healthcare IT | `C:/Users/kanit/.codex/skills/it-governance-ops/SKILL.md` |
| [ ] | json-canvas | สร้าง/แก้ไข Obsidian JSON Canvas `.canvas` | `C:/Users/kanit/.codex/skills/json-canvas/SKILL.md` |
| [ ] | obsidian | ทำงานกับ Obsidian vault, notes, tags, properties, settings | `C:/Users/kanit/.codex/skills/obsidian/SKILL.md` |
| [ ] | obsidian-bases | สร้าง/แก้ไข Obsidian Bases `.base` | `C:/Users/kanit/.codex/skills/obsidian-bases/SKILL.md` |
| [ ] | obsidian-cli | ใช้ Obsidian CLI จัดการ vault และ debug plugin/theme | `C:/Users/kanit/.codex/skills/obsidian-cli/SKILL.md` |
| [ ] | obsidian-markdown | เขียน/แก้ไข Obsidian Flavored Markdown | `C:/Users/kanit/.codex/skills/obsidian-markdown/SKILL.md` |
| [ ] | pdf:pdf | อ่าน/สร้าง/ตรวจ/แปลง PDF | `C:/Users/kanit/.codex/plugins/cache/openai-primary-runtime/pdf/26.623.12021/skills/pdf/SKILL.md` |
| [ ] | presentations:Presentations | สร้างหรือแก้ไข PowerPoint/Google Slides | `C:/Users/kanit/.codex/plugins/cache/openai-primary-runtime/presentations/26.623.12021/skills/presentations/SKILL.md` |
| [ ] | spreadsheets:Spreadsheets | สร้าง/แก้ไข/วิเคราะห์ spreadsheet, CSV, Excel | `C:/Users/kanit/.codex/plugins/cache/openai-primary-runtime/spreadsheets/26.623.12021/skills/spreadsheets/SKILL.md` |
| [ ] | svg-diagram | สร้าง SVG diagram สำหรับเอกสารและ GitHub | `C:/Users/kanit/.codex/skills/svg-diagram/SKILL.md` |
| [ ] | template-creator:template-creator | สร้าง reusable artifact-template skill จาก Word/PowerPoint/Excel | `C:/Users/kanit/.codex/plugins/cache/openai-primary-runtime/template-creator/26.623.12021/skills/template-creator/SKILL.md` |

## หมายเหตุการย้ายเครื่อง

- Skill ที่อยู่ใต้ `.codex/skills/.system` มักมากับ Codex หรือระบบหลักของ Codex
- Skill ที่อยู่ใต้ `.codex/skills/<ชื่อ skill>` เป็น skill ส่วนตัว/ติดตั้งเพิ่ม ควร backup หรือ install ซ้ำบนเครื่องใหม่
- Skill ที่อยู่ใต้ `.codex/plugins/cache/...` มาจาก plugin/runtime ที่ติดตั้งไว้ใน Codex เครื่องนี้
- ถ้าต้องการย้ายแบบ manual ให้เทียบรายการนี้กับโฟลเดอร์ `%USERPROFILE%/.codex/skills` และ plugin ที่ติดตั้งใน Codex

