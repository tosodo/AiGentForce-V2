# Integrations Guide for AiGentForce-V2

This document describes how to configure and use key integrations in the AiGentForce-V2 ecosystem, including LangChain, n8n, Custom GPT, branded onboarding, and the AIgentForce.io platform.

---

## Table of Contents
1. [LangChain](#langchain)
2. [n8n Workflow Automation](#n8n-workflow-automation)
3. [Custom GPT](#custom-gpt)
4. [Branded Onboarding](#branded-onboarding)
5. [AIgentForce.io Platform](#aigentforceio-platform)

---

## 1. LangChain

LangChain powers advanced language model orchestration, chain-of-thought reasoning, and complex AI workflows.

### **Installation**
Install LangChain via npm or pip:
```bash
npm install langchain
# OR
pip install langchain
```

### **Environment Variables**
Add to your `.env.local` or deployment environment:
```
LANGCHAIN_API_KEY=your-langchain-key
```

### **Integration Points**
- Chain composition (sequential, branching, conditional)
- Memory/context management for conversations
- Tool/agent composition
- Vector database integration for semantic search

### **Sample Usage**
```typescript
import { LLMChain } from "langchain/chains";
const chain = new LLMChain({
  ...
});
const response = await chain.run("What is our current action plan?");
```

**See:** [LangChain docs](https://langchain.readthedocs.io/)

---

## 2. n8n Workflow Automation

n8n enables no-code/low-code automation of workflows across SaaS and custom APIs.

### **Setup**
- Cloud-hosted or self-hosted (
See [n8n.io](https://n8n.io))
- Create a new workflow
- Add Webhook or Schedule Trigger
- Connect to integrated services (MySQL, OpenAI, etc.)

### **Integration Points**
- Trigger GPT actions via HTTP Webhooks
- Automate document ingestion and parsing
- Orchestrate notifications, approvals, and user onboarding
- Pipe data between external APIs and AiGentForce modules

### **Sample Usage**
```bash
POST /webhook/your-n8n-trigger
{
  "action": "run_analysis",
  "documentId": "DOC123"
}
```

---

## 3. Custom GPT

Deploy branded ChatGPT or OpenAI GPT models with:
- Custom training data
- Company or client-specific prompt templates
- File uploads for RAG (Retrieval-Augmented Generation)
- Conversation memory for user context

### **Integration Steps**
- Obtain API key from OpenAI or Azure OpenAI
- Configure your GPT endpoint and model params in `.env`
- Optionally, fine-tune using company documents and FAQs
- Specify branded instructions as system prompts

### **Sample Usage**
```typescript
import { OpenAI } from "openai";
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});
const result = await openai.chat.completions.create({
  model: "gpt-4-turbo",
  messages: [{ role: "system", content: "You are an onboarding assistant." }],
  ...
});
```

---

## 4. Branded Onboarding

Deliver a custom onboarding experience:
- Upload company logos, set brand colors, define onboarding steps
- Dynamic UI paths based on role (admin, consultant, trainee)
- Progressive disclosure—show features as users progress

### **Customization Points**
- Edit `branding.json` or use admin UI panel
- Configure with company theme (colors, fonts, slogans)
- Enable/disable onboarding modules in admin dashboard

---

## 5. AIgentForce.io Platform

Connects your deployment to:
- Multi-tenant license management
- Centralized analytics and reporting
- SSO authentication
- Marketplace for workflow and GPT templates

### **Integration Steps**
1. Register organization on [AIgentForce.io](https://aigentforce.io)
2. Obtain API credentials
3. Add credentials to `.env.local`:
```
AIGENTFORCE_IO_API_KEY=your-key-here
AIGENTFORCE_LICENSE_ID=your-license-id
```
4. Enable integration in settings or via admin panel

### **Example API Call**
```bash
POST https://api.aigentforce.io/v2/licenses/validate
Headers: { Authorization: Bearer $AIGENTFORCE_IO_API_KEY }
Body: { "license_id": "$AIGENTFORCE_LICENSE_ID" }
```

---

## Support
- [LangChain Quickstart](https://langchain.readthedocs.io/)
- [n8n Docs](https://docs.n8n.io/)
- [OpenAI Platform](https://platform.openai.com/docs/)
- [AIgentForce.io Support](https://aigentforce.io/support)

---

For system architecture, see [ARCHITECTURE.md](ARCHITECTURE.md)
For deployment procedures, see [DEPLOYMENT.md](DEPLOYMENT.md)
