# 🚀 Setup Guide

Follow this guide to configure and run the Multi-Agent Customer Support Orchestrator.

---

## 📋 Prerequisites

Ensure you have completed the prerequisites listed in the [README](README.md#-prerequisites) before proceeding.

---

## 🔧 Setup Steps

### 1. Import Workflow
1. Clone this repository.
2. In n8n, import the single workflow file:
   - `SupportBot_Orchestrator.json`
3. Activate the workflow.

### 2. Configure Credentials
In n8n, create credentials for:
- **OpenAI:** API key (ensure access to GPT-5.1/GPT-4o-mini and embeddings).
- **Pinecone:** API key + environment; create indexes `technical-index` and `billing-index`.
- **Telegram:** Bot token from [@BotFather](https://t.me/BotFather).
- **Airtable:** Personal Access Token with `data.records:write` scope; ensure base contains `Customers` and `Ticket_Logs` tables.
- **Google:** OAuth2 credentials for Drive API (for knowledge base ingestion).
- **Gmail:** OAuth2 credentials for sending escalation alerts.

### 3. Update Node References
- Replace Airtable Base/Table IDs in all Airtable nodes (`Customers`, `Ticket_Logs`).
- Update Pinecone index names in vector store nodes (`technical-index`, `billing-index`).
- Set your Telegram bot token and Gmail recipient (`kevynalojepan01@gmail.com`) in notification nodes.
- Configure Google Drive folder IDs for knowledge base ingestion (`Technical`, `Billing` folders).

### 4. Test the Orchestrator
#### Test Basic Routing
1. Send a test message to your Telegram bot: `"How do I reset my password?"`
2. Verify:
   - Orchestrator classifies intent as `Tech_Support`, sentiment as `Neutral`.
   - Query routed to `Tech_Specialist_Tool`.
   - Agent retrieves relevant chunk from `technical-index` and returns concise answer.
   - Response sent back to Telegram user.

#### Test Escalation Logic
1. Send an angry message: `"This is broken and I'm furious!!!"`
2. Verify:
   - Sentiment classified as `Angry` → routed to `Escalation_Handler_Tool`.
   - Airtable `Ticket_Logs` record created with `Escalation Reason = Angry`.
   - Gmail alert sent to support team with full context.
   - User receives empathetic acknowledgment message.

#### Test Knowledge Base Ingestion
1. Upload a new PDF policy doc to the watched Google Drive `Billing` folder.
2. Verify:
   - Ingestion trigger fires → PDF text extracted → chunked → embedded (512-dim) → upserted to `billing-index`.
   - Subsequent billing queries retrieve the new policy accurately.

#### Test VIP Priority Handling
1. Ensure a test user has `Status = VIP` in Airtable `Customers` table.
2. Send a billing question: `"What's the Pro plan price?"`
3. Verify:
   - Query routed to `Billing_Specialist_Tool` (not escalation, since sentiment is neutral).
   - Response includes VIP-aware tone: `"As a valued VIP member..."`.
   - No escalation triggered (correct behavior per routing logic).

---

## 🛡️ Error Handling & Recovery

### Prompt Engineering Debugging
| Issue | Diagnostic | Fix |
|-------|-----------|-----|
| **Orchestrator returns invalid JSON** | Check `Structured Output Parser` logs; review agent prompt for schema compliance | Add `maxRetries: 3` to OpenAI node; strengthen system prompt with "Return ONLY valid JSON" directive |
| **Specialist agent hallucinates** | Review retrieved Pinecone chunks; check confidence threshold logic | Ensure `confidence < 80%` triggers escalation; add "NEVER answer from model knowledge" to system prompt |
| **Routing loops or duplicate tool calls** | Inspect Orchestrator prompt stop conditions; verify `executeOnce` flags | Add explicit "STOP after returning tool call" directive; enable `executeOnce: true` on specialist nodes |

### Workflow Recovery Steps
1. Check Airtable `Ticket_Logs` for `Email Status = Failed` or `Status = Open` with no resolution.
2. Review n8n execution logs for specific node errors (Pinecone timeout, Gmail quota, etc.).
3. Fix root cause (refresh credentials, increase chunk size, adjust embedding dimensions).
4. Re-trigger workflow manually or wait for next scheduled run.

### Common Issues
| Issue | Cause | Fix |
|-------|-------|-----|
| **Pinecone retrieval empty** | Embedding dimension mismatch or index not populated | Verify all embedding nodes use `dimensions: 512`; confirm ingestion triggers succeeded |
| **Telegram message not received** | Bot token invalid or webhook not registered | Re-register webhook via Telegram API; verify bot is not blocked by user |
| **Airtable duplicate records** | Race condition in concurrent existence checks | Add unique constraint on `Telegram_ID`; use `executeOnce: true` on insert nodes |
| **Gmail OAuth token expired** | Credential refresh needed | Re-authenticate Gmail in n8n credentials; implement token refresh logic if self-hosting |

---

## 💡 Pro Tips

### Prompt Engineering Best Practices
- **Chain-of-Thought Enforcement**: Keep the Orchestrator's priority-order evaluation explicit and sequential — do not compress logic.
- **Few-Shot Coverage**: Include examples for all 8 routing conditions + edge cases (empty intent, VIP+Neutral, Angry+Standard).
- **Schema Rigor**: Use `Structured Output Parser` with strict JSON schema for every agent — never rely on free-form text.
- **Immutable Context**: Pass user data as read-only variables; forbid agents from reinterpreting classified fields.

### Operational Excellence
- **Sanitize Credentials**: Never commit API keys or tokens to version control. Use n8n's credential encryption.
- **Demo Video**: Record a Loom video showing end-to-end flow (Telegram query → AI routing → resolution/escalation) for portfolio showcase.
- **Highlight Reliability**: Emphasize the confidence-gated RAG responses and priority escalation logic in interviews.
- **Document Learnings**: Add a "Challenges & Solutions" section noting how you solved hallucination prevention, routing determinism, and stateful memory.

### Cost Monitoring
- Set up Pinecone usage dashboards to track vector storage and query volume.
- Monitor OpenAI token usage per agent type (Orchestrator vs. Specialist) to optimize prompt length.
- Use Airtable row count alerts to anticipate CRM tier upgrades.

---

## 📞 Support

For questions or issues:
- Open an issue in this repository with workflow name + execution ID + error logs.
- Contact the author: [kevynalojepan@gmail.com](mailto:kevynalojepan@gmail.com)

---

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

> ✨ **Built by Kevyn Alojepan** — Turning AI automation into reliable, human-centered support experiences.
