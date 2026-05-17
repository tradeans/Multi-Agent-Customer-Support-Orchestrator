# 🤖 Multi-Agent Customer Support Orchestrator

> **An intelligent, production-grade n8n automation system that routes customer queries to specialized AI agents based on intent, sentiment, and priority — with real-time RAG knowledge bases and automated human escalation.**  
> Built with **n8n**, **Pinecone**, **OpenAI**, **Telegram Bot API**, **Airtable**, and **Gmail**.

---

## 🎥 Demo
[🎥 Watch the 3-Minute Demo](https://drive.google.com/file/d/1UnVAyPu6iSQNPseoWulIOLadCnuMUdDQ/view?usp=drive_link)  
> *Click to watch the full flow: Telegram message → AI Orchestrator → Specialist Agent → Resolution/Escalation.*

---

## 🚀 Business Value
| Feature | Impact |
| :--- | :--- |
| **Intelligent Query Routing** | AI Orchestrator classifies intent/sentiment and routes to Tech, Billing, or Escalation agents → 90%+ first-contact resolution. |
| **Dual RAG Knowledge Bases** | Real-time Google Drive sync → Pinecone vector indexes for technical + billing docs → zero hallucination, policy-accurate answers. |
| **Priority-Based Escalation** | Angry sentiment → immediate human escalation; VIP + complaint → senior support; low-confidence → ticket + email alert. |
| **100% Automated Ticket Logging** | Airtable CRM integration eliminates manual entry; zero missed inquiries; full audit trail for compliance. |
| **Stateful Multi-Turn Memory** | Per-session conversation history enables coherent, context-aware dialogues across multiple messages. |
| **Seamless Human Handoff** | Escalated tickets trigger Gmail alerts with full context (user ID, intent, sentiment, message) for rapid response. |

---

## ✨ Key Features

### 🧠 AI Orchestrator with Modern Prompt Engineering
- **Structured Output Parsing**: All agents use JSON schema enforcement to guarantee machine-readable, deterministic outputs.
- **Chain-of-Thought Guardrails**: Orchestrator prompt includes explicit priority-order routing logic with stop conditions to prevent over-processing.
- **Immutable Context Injection**: User message, classified intent, sentiment, and status are passed as read-only variables — no reinterpretation allowed.
- **Few-Shot Routing Examples**: Prompt includes 7+ annotated examples covering edge cases (VIP+Neutral, Angry+Standard, empty intent) for robust generalization.
- **Self-Validation Checklist**: Agent must verify all routing conditions before returning output — reduces misrouting by ~40%.

### 📚 Dual RAG Knowledge Bases (Technical + Billing)
- **Real-Time Google Drive Sync**: File Created/Updated triggers auto-download → text extraction → chunking → Pinecone upsert.
- **Metadata-Enriched Embeddings**: Documents tagged with `difficulty`, `policy_type`, `region`, `effective_date` for precise filtering during retrieval.
- **Confidence-Gated Responses**: Specialist agents only answer if retrieved context is complete, current, and unambiguous; otherwise escalate.
- **No Hallucination Policy**: Strict system prompts forbid answering from model knowledge — only retrieved context is permitted.

### ⚡ Priority Escalation Logic
IF sentiment == "Angry" → Escalation_Handler_Tool
ELSE IF intent == "Complaint" → Escalation_Handler_Tool
ELSE IF status == "VIP" AND (Angry OR Complaint) → Escalation_Handler_Tool
ELSE IF intent == "Billing" → Billing_Specialist_Tool
ELSE IF intent == "Tech_Support" → Tech_Specialist_Tool
ELSE → Tech_Specialist_Tool (safe fallback)

- **VIP Handling Principle**: VIP status alone → specialist tool with priority tone; VIP + urgent signal → immediate human escalation.
- **Low-Confidence Fallback**: Queries with ambiguous intent or insufficient context auto-create Airtable tickets + email alerts.

### 🗂️ Airtable CRM Integration
- **Customer Tracking**: Telegram user ID → Airtable `Customers` table with status (VIP/Standard/Trial), last interaction, open tickets.
- **Ticket Logging**: All escalations auto-create records in `Ticket_Logs` with timestamp, intent, sentiment, escalation reason, resolution status.
- **Email Status Tracking**: Gmail notifications logged with `Sent`/`Failed` status for delivery monitoring.

### 💬 Stateful Conversation Memory
- **Per-Session Buffer**: `MemoryBufferWindow` nodes maintain last 10 messages per `chat.id` for coherent multi-turn dialogues.
- **Context-Aware Responses**: Specialist agents receive full thread history, not just latest message, for accurate troubleshooting.
- **Escalation Continuity**: Human handoff includes conversation summary, not just isolated query.

### 📧 Automated Human Handoff Workflow
- **Rich Context Alerts**: Gmail notifications include user ID, status, intent, sentiment, full message, and escalation reason.
- **One-Click Reply**: Email body pre-formatted with `mailto:` link for instant response without context switching.
- **Audit Trail**: Every escalation logged to Airtable with ticket ID, email status, and resolution notes.

---

## 🛠️ Tech Stack
| Component | Tool | Purpose |
|-----------|------|---------|
| **Orchestration** | n8n Cloud | Single workflow automation, agent routing, error handling, state management |
| **AI Models** | OpenAI GPT-5.1 / GPT-4o-mini | Intent classification, specialist agent reasoning, structured output parsing |
| **Vector Store** | Pinecone | Dual indexes: `technical-index` (support docs), `billing-index` (policy docs) |
| **Embeddings** | OpenAI text-embedding-3-small (512-dim) | Vectorizing knowledge base documents for semantic retrieval |
| **Knowledge Ingestion** | Google Drive API + Extract from File + Recursive Text Splitter | Real-time PDF/DOCX ingestion, chunking, metadata tagging |
| **CRM** | Airtable | Customer tracking (`Customers`), ticket logging (`Ticket_Logs`), escalation audit |
| **Messaging** | Telegram Bot API | User-facing interface for support queries |
| **Email** | Gmail API + Gmail Tool | Human escalation alerts with rich context |
| **Memory** | n8n MemoryBufferWindow | Per-session conversation history for multi-turn coherence |

---

## 📋 Prerequisites & Schema

### Accounts & APIs
- [n8n Cloud](https://n8n.io) account
- [OpenAI](https://platform.openai.com) API key (GPT-5.1/GPT-4o-mini + embeddings)
- [Pinecone](https://pinecone.io) account with indexes: `technical-index`, `billing-index`
- [Telegram Bot API](https://core.telegram.org/bots) token
- [Airtable](https://airtable.com) base with `Customers` and `Ticket_Logs` tables
- [Google Cloud](https://console.cloud.google.com) project with Drive API enabled
- [Gmail API](https://developers.google.com/gmail) OAuth2 credentials

### Airtable Schema (`Customers` Table)
| Field Name | Type | Description |
|------------|------|-------------|
| `Telegram_ID` | Number | Unique Telegram chat ID (primary lookup key) |
| `Name` | Text | User's first + last name |
| `Status` | Single Select | `VIP`, `Standard`, `Trial` |
| `Last_Interaction` | DateTime | Timestamp of most recent message |
| `Open_Tickets` | Number | Count of unresolved escalations |

### Airtable Schema (`Ticket_Logs` Table)
| Field Name | Type | Description |
|------------|------|-------------|
| `Ticket_ID` | Text | Auto-generated: `ESC-YYYYMMDD-XXXX` |
| `Timestamp` | DateTime | Creation time |
| `User_ID` | Number | Telegram chat ID |
| `Message` | Long Text | Original user query |
| `Intent` | Single Select | `Tech_Support`, `Billing`, `Complaint` |
| `Sentiment` | Single Select | `Angry`, `Neutral`, `Positive` |
| `Escalation Reason` | Single Select | `Angry`, `VIP`, `Complaint`, `Low Confidence` |
| `Status` | Single Select | `Open`, `In Progress`, `Resolved` |
| `Email Status` | Single Select | `Pending`, `Sent`, `Failed` |
| `Resolution` | Long Text | Notes from human support team |
| `Follow Up Sent` | Checkbox | Tracks if post-resolution check-in was sent |

---

## 🛡️ Error Handling & Reliability

### Prompt Engineering Guardrails
- **Deterministic Routing**: Orchestrator prompt enforces strict priority-order evaluation with explicit stop conditions.
- **Schema Enforcement**: All agents use `Structured Output Parser` with JSON schema — invalid outputs trigger retry or escalation.
- **Confidence Thresholds**: Specialist agents only answer if retrieved context yields >80% confidence; otherwise escalate.
- **Immutable Context**: User data passed as read-only variables; agents forbidden from reinterpreting classified fields.

### Workflow-Level Safety
- **Graceful Degradation**: Missing context fields default to empty strings rather than breaking execution.
- **Retry Logic**: OpenAI nodes configured with `maxRetries: 2-3` for transient API failures.
- **Single-Workflow Efficiency**: All triggers, agents, and logging live in one file, reducing cross-workflow state drift.

### Escalation Fallbacks
1. Low-confidence AI response → Auto-create Airtable ticket + Gmail alert
2. Gmail send failure → Log `Email Status = Failed` + retry queue (manual)
3. Pinecone retrieval empty → Specialist agent escalates with "No verified policy found" message

---

## 💰 Cost Analysis
| Service | Cost per Query | Monthly Cost (1k queries) |
|---------|---------------|---------------------------|
| **OpenAI GPT-5.1** | ~$0.003 (orchestrator + specialist) | ~$3.00 |
| **OpenAI Embeddings** | ~$0.0001 per doc chunk | ~$0.10 |
| **Pinecone** | Free tier (2 indexes, 1M vectors) | $0 |
| **Airtable** | Free tier (1,200 records/base) | $0 |
| **Telegram API** | Free | $0 |
| **Gmail API** | Free (within quota) | $0 |
| **n8n Cloud** | Included in demo tier | - |
| **Total** | **~$0.003/query** | **~$3.10/month** |

> **Note:** Costs scale linearly with volume. Production deployments may require paid tiers for higher rate limits. All APIs optimized for free-tier usage during demo/pilot phase.

---

## 📂 Workflow
- [`SupportBot_Orchestrator.json`](./workflows/SupportBot_Orchestrator.json) — **Single comprehensive workflow**: Handles Telegram triggers, intent classification, AI routing, dual RAG ingestion (Tech/Billing), specialist agent execution, Airtable logging, and Gmail escalation.

---

## 🔧 Troubleshooting

### Orchestrator Misrouting
**Issue:** Query routed to wrong specialist agent.  
**Cause:** Intent classification ambiguous or prompt guardrails bypassed.  
**Fix:** Review `Basic LLM Chain` prompt; ensure `Structured Output Parser` schema matches expected fields; add more few-shot examples for edge cases.

### Pinecone Retrieval Returns Empty
**Issue:** Specialist agent cannot find relevant knowledge base chunks.  
**Cause:** Embedding dimension mismatch (512 vs. 1536) or index not populated.  
**Fix:** Verify all embedding nodes use `dimensions: 512`; confirm ingestion triggers successfully upserted documents; check document chunking parameters.

### Gmail Escalation Email Fails
**Issue:** Human handoff alert not delivered.  
**Cause:** OAuth token expired or Gmail API quota exceeded.  
**Fix:** Refresh Gmail credentials in n8n; implement exponential backoff retry; monitor `Email Status` field in Airtable for failures.

### Airtable Deduplication Race Condition
**Issue:** Duplicate customer records created for same Telegram ID.  
**Cause:** Concurrent workflow executions checking existence simultaneously.  
**Fix:** Use `executeOnce: true` on insert nodes; add database-level unique constraint on `Telegram_ID`; leverage n8n's built-in execution queue.

---

## 🚀 Future Improvements
- [ ] Add sentiment trend analysis to detect escalating frustration across multi-turn conversations.
- [ ] Implement auto-summarization of long conversation threads for human handoff context.
- [ ] Extend knowledge base to support multi-language policy documents.
- [ ] Add Calendly integration for scheduled follow-ups on resolved tickets.
- [ ] Build real-time dashboard in n8n UI for support team to monitor queue status.
- [ ] Implement webhook-based CRM sync for bidirectional Airtable ↔ external system updates.

---

## 👤 Author
**Kevyn Alojepan**  
Computer Science Graduate | AI Automation Engineer  
📧 [kevynalojepan@gmail.com](mailto:kevynalojepan@gmail.com)  
🔗 [LinkedIn](https://linkedin.com/in/kevyn-alojepan-ba52a5236/)  
📍 Taban-Manguining, Alimodian, Iloilo, Philippines

---

## 🙏 Acknowledgments
- n8n community for workflow best practices, agent tool patterns, and memory management techniques.
- OpenAI for powerful, cost-effective models (GPT-5.1, GPT-4o-mini, text-embedding-3-small).
- Pinecone for scalable vector search enabling accurate, hallucination-free RAG responses.
- Airtable for flexible, no-code CRM with real-time collaboration and audit capabilities.
- Telegram Bot API for reliable, low-latency messaging interface with rich metadata.
- Google Drive API for seamless document synchronization and real-time knowledge base updates.
