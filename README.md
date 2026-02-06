# 📚 N8N Skills Library

> ไลบรารี Skills สำหรับ n8n workflow automation platform - เอกสารครบถ้วนพร้อม JSON workflow generation

## 🎯 ภาพรวม

ไลบรารีนี้รวบรวมความรู้ทั้งหมดเกี่ยวกับ **n8n** (อ่านว่า "เอ็น-แปด-เอ็น") ซึ่งเป็นแพลตฟอร์ม Workflow Automation แบบ fair-code ที่รวมความสามารถ AI เข้ากับการสร้าง Business Automation

### n8n คืออะไร?

n8n เป็นเครื่องมือที่ช่วยให้คุณ:
- ✅ **เชื่อมต่อแอปต่างๆ** - เชื่อมต่อ API ของแอปใดก็ได้เข้าด้วยกัน
- ✅ **Automate งาน** - สร้าง Workflow อัตโนมัติแบบ Visual
- ✅ **AI Integration** - รวม LangChain และ AI Models เข้าใน Workflow
- ✅ **Self-host ได้** - ติดตั้งบน Server ของตัวเองเพื่อความปลอดภัย
- ✅ **JSON-based** - สร้าง, Import, และ Version Control ได้ผ่าน JSON

### ⭐ ความสามารถหลัก (v2.0.0)

- 🔧 **JSON Node Configs** — ข้อมูล type identifiers และ parameters ครบทุก node
- 📦 **Importable Workflows** — 7+ ตัวอย่าง workflow พร้อม import ใช้งานได้ทันที
- 🤖 **AI Agent JSON** — Config สำหรับ OpenAI, Anthropic, Gemini, Ollama, Tools, Memory
- 🔗 **Connection Patterns** — รูปแบบการเชื่อมต่อทั้ง main และ AI special types
- 📊 **400+ Integrations** — เอกสาร built-in nodes ครบถ้วน

---

## 📁 โครงสร้างไฟล์

```
Skills_N8N/
│
├── SKILL.md                    # ไฟล์ Skills หลัก (สำคัญที่สุด)
├── CLAUDE.md                   # สรุปสำหรับ AI (พร้อม node type identifiers)
├── README.md                   # ไฟล์นี้ (เอกสารภาษาไทย)
│
└── resources/                  # เอกสารเพิ่มเติม
    ├── core_concepts.md        # Nodes, Triggers, Flow Logic (พร้อม JSON configs)
    ├── integrations.md         # การเชื่อมต่อ (400+ nodes, credentials)
    ├── advanced_ai.md          # AI/LangChain (พร้อม JSON configs ทุก model)
    ├── code_and_expressions.md # Code Node, Expressions, JMESPath
    ├── hosting_and_deployment.md # Docker, Configuration, Scaling
    ├── api_reference.md        # REST API Reference
    ├── workflow_patterns.md    # Patterns + Importable Workflow JSON
    └── troubleshooting.md      # แก้ไขปัญหา
```

---

## 🚀 การเริ่มต้นใช้งาน

### สำหรับ AI (Antigravity/Claude)

1. อ่านไฟล์ `SKILL.md` เป็นหลัก
2. ใช้ `CLAUDE.md` สำหรับ Quick Reference (มี node type identifiers)
3. ลงรายละเอียดในไฟล์ `resources/` ตามหัวข้อที่ต้องการ
4. ใช้ `resources/workflow_patterns.md` สำหรับตัวอย่าง importable JSON

### สำหรับผู้ใช้งาน

| หัวข้อ | ไฟล์ | คำอธิบาย |
|--------|------|----------|
| **พื้นฐาน** | `core_concepts.md` | Nodes, Triggers, Flow Logic พร้อม JSON configs |
| **การเชื่อมต่อ** | `integrations.md` | 400+ Nodes, Community Nodes, Credentials |
| **AI** | `advanced_ai.md` | LangChain, AI Agents, Memory, Tools พร้อม JSON |
| **โค้ด** | `code_and_expressions.md` | Expressions, Code Node, JMESPath |
| **Hosting** | `hosting_and_deployment.md` | Docker, Configuration, Scaling |
| **API** | `api_reference.md` | REST API Endpoints |
| **Patterns** | `workflow_patterns.md` | Patterns + Importable Workflow JSON |
| **แก้ปัญหา** | `troubleshooting.md` | Common Issues และวิธีแก้ |

---

## 📖 หัวข้อสำคัญ

### 1. Workflow คืออะไร?

Workflow คือชุดของ Nodes ที่เชื่อมต่อกัน เป็น JSON object:

```
Trigger → Action → Action → Output
```

ตัวอย่าง:
- 📧 รับ Email → ดึงข้อมูล → บันทึกลง Database
- 🔔 Webhook → ประมวลผลคำสั่ง → ส่งแจ้งเตือน Slack
- ⏰ ตั้งเวลา → ดึงข้อมูล API → สร้างรายงาน
- 🤖 Chat Trigger → AI Agent → ตอบคำถามอัตโนมัติ

### 2. Node Types

| ประเภท | คำอธิบาย | ตัวอย่าง Type Identifier |
|--------|----------|--------------------------|
| **Trigger** | เริ่มต้น Workflow | `n8n-nodes-base.webhook` |
| **Action** | ทำงานบางอย่าง | `n8n-nodes-base.httpRequest` |
| **Core** | ควบคุม Flow | `n8n-nodes-base.if` |
| **AI** | ประมวลผล AI | `@n8n/n8n-nodes-langchain.agent` |

### 3. Expressions

การเข้าถึงข้อมูลแบบ Dynamic:

```javascript
// เข้าถึงข้อมูลปัจจุบัน
{{ $json.fieldName }}

// ข้อมูลจาก Node อื่น
{{ $('Node Name').item.json.field }}

// Environment Variable
{{ $env.API_KEY }}

// เวลาปัจจุบัน
{{ $now.toFormat('yyyy-MM-dd') }}

// $fromAI() - ให้ AI Agent กำหนดค่าเอง
{{ $fromAI('paramName', 'description', 'string') }}
```

### 4. AI Integration

n8n รองรับ AI ผ่าน LangChain:

```
Chat Trigger → AI Agent → Response
                  ↑
             Chat Model (GPT-4/Claude/Gemini)
                  ↑
             Memory + Tools
```

รองรับ: OpenAI, Anthropic, Google Gemini, Ollama (local)

---

## 💻 การติดตั้ง n8n

### Docker (แนะนำ)

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

### npm

```bash
npm install n8n -g
n8n start
```

### Cloud

ใช้ https://app.n8n.cloud สำหรับ Hosted Version

---

## 🔧 ตัวอย่าง Workflow

### 1. Webhook + Database

```
Webhook → Set Node → PostgreSQL Insert → Respond to Webhook
```

### 2. Schedule + API Sync

```
Schedule (ทุกชั่วโมง) → HTTP GET API → Transform → Filter → Update CRM
```

### 3. AI Chatbot with Tools

```
Chat Trigger → AI Agent → Response
                 ↑
           OpenAI GPT-4o
                 ↑
          Memory + Calculator + HTTP Tool
```

### 4. Data ETL Pipeline

```
Schedule (2AM) → Extract → Code (Transform) → Filter → Remove Duplicates → Load DB
```

> 💡 ดูตัวอย่าง JSON ที่ import ได้ทันทีใน `resources/workflow_patterns.md`

---

## 📚 แหล่งข้อมูลเพิ่มเติม

| แหล่ง | ลิงก์ |
|-------|-------|
| **Official Docs** | https://docs.n8n.io/ |
| **Workflow Templates** | https://n8n.io/workflows/ |
| **Community Forum** | https://community.n8n.io/ |
| **GitHub** | https://github.com/n8n-io/n8n |
| **Discord** | https://discord.gg/n8n |

---

## 📝 หมายเหตุ

เอกสารนี้สร้างขึ้นจากการอ่าน **Official Documentation** ของ n8n (https://docs.n8n.io/) อย่างละเอียด และรวมเนื้อหาจาก JSON workflow builder reference เพื่อให้ครอบคลุมทั้งความรู้เชิงแนวคิดและ JSON configs ที่ใช้งานจริง

**สร้างเมื่อ**: มกราคม 2026  
**อัพเดทล่าสุด**: กุมภาพันธ์ 2026  
**เวอร์ชัน**: 2.0.0  
**อ้างอิง**: n8n Documentation v1.19.4+

---

## 🤝 การใช้งานกับ AI

ไลบรารีนี้ออกแบบมาให้ AI (Antigravity, Claude, etc.) สามารถ:

1. **สร้าง Workflow JSON** ที่ import เข้า n8n ได้ทันที
2. **อ่านและเข้าใจ** node type identifiers ครบทุกตัว
3. **สร้าง AI Agent Workflows** พร้อม LangChain integration
4. **แก้ไขปัญหา** ที่เกี่ยวกับ n8n
5. **ให้คำแนะนำ** Best Practices

เมื่อต้องการข้อมูลเกี่ยวกับ n8n ให้อ่าน `SKILL.md` เป็นอันดับแรก จากนั้นค้นหารายละเอียดเพิ่มเติมในโฟลเดอร์ `resources/`
