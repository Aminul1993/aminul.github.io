## 👋 Hi, I'm Aminul Islam


Senior Software Engineer with 9+ years of experience building enterprise applications, AI-powered solutions, and data engineering platforms. Passionate about AI integration, scalable architectures, enterprise software, and intelligent automation.

---

## 🛠 Tech Stack

### Backend
- Python
- Django
- FastAPI
- REST APIs
- Node.js

### AI & LLM
- Ollama
- LangGraph
- MCP (Model Context Protocol)
- AI Agents

### Data Engineering
- Kafka
- PySpark
- Spark SQL
- Elasticsearch
- Data Lakes

### DevOps & Cloud
- Docker
- GitLab CI/CD
- AWS S3
- Redis

### Databases
- PostgreSQL
- MySQL
- MariaDB
- Oracle

---

## ⭐ Featured Projects

### AI Search Assistant (AIChatAPI)
A conversational assistant for a multi-module that classifies each question and routes it to a self-correcting SQL agent, permission-filtered navigation, or session-aware recall. Built using LangGraph, Ollama, Django REST Framework, and MySQL.

📄 [Technical README →](ai-search-assistant/README.md)

### Intelligent Document Extraction Platform (Extraction Workbench)
A single, schema-driven API that turns invoices, purchase orders, and other business documents into structured, serializer-validated JSON using OCR/LLM conversion (MarkItDown) and a human-in-the-loop review UI.

📄 [Technical README →](extraction-workbench/README.md)

### Pest Identification Platform
A multimodal AI diagnostic platform that lets field users photograph crop damage and receive a structured, expert-grade pest/disease diagnosis in seconds — combining color-agnostic background removal, content-aware multi-crop embedding, vision-LLM attribute extraction, and triple-signal confidence gating. Built on AWS Bedrock, OpenSearch, DynamoDB, and S3.

📄 [Technical README →](pest-identification-platform/README.md)

### HRMS Agent
A permission-aware conversational AI layer over an existing HRMS platform, letting employees check leave balances, apply for leave, and review team attendance in plain language instead of multi-step UI forms. Built with FastAPI, LangGraph, and Anthropic Claude, with fail-closed authorization and per-user permission-gated tools.

📄 [Technical README →](hrms-agent/README.md)

### SOL AI Platform
A healthcare-focused AI platform delivering conversational assistance and digital support experiences through LLM-powered interactions.

### Data Lake Analytics Platform
Kafka and PySpark-based large-scale data processing architecture supporting scalable analytics and operational intelligence.

---

# 📘 Case Study 1: AI Search Assistant

> Internally implemented as the `AIChatAPI` module (`ai_module.views.AIChatAPI`).
> 📄 Full technical write-up: [ai-search-assistant/README.md](ai-search-assistant/README.md)

## Overview

The AI Search Assistant was developed to simplify information retrieval inside an Enterprise Project Management System. Instead of manually navigating modules or searching through menus, users can interact with the system using natural language.

---

## Business Problem

Enterprise users frequently struggle to:

- Locate information across multiple modules
- Identify the correct workflow or screen
- Retrieve project information quickly
- Understand module relationships

These challenges reduce productivity and increase onboarding efforts.

---

## Solution

The AI Search Assistant uses a multi-stage AI orchestration workflow to analyse user intent and determine the most appropriate action.

### Workflow

```text
User Query
    │
    ▼
decide_flow (LangGraph router)
    │
    ├── A · Database Query        → generate → self-check → run SQL
    │
    ├── B · Navigation / Knowledge → permission-filtered route match
    │                                  or domain knowledge answer
    │
    └── C · History Recall        → answer from rolling session summary
    │
    ▼
Persist turn + refresh session summary
    │
    ▼
Final Response
```

A single classifier decides, per question, whether it needs:

- A self-correcting, read-only SQL query against live project data
- A permission-filtered link to the right screen, or a general knowledge answer
- Recall of what was already discussed earlier in the same session

and routes the request accordingly, carrying context across the conversation via a per-session rolling summary.

---

## Key Features

### Intelligent Query Routing
A LangGraph decision graph classifies every question and routes it to a dedicated agent instead of relying on one monolithic prompt.

### Permission-Aware Navigation
Candidate screens are filtered against the asking user's own module, sub-module, and settings permissions *before* the LLM ever sees them — the model can't recommend a page the user isn't entitled to.

### Self-Correcting SQL Agent
Natural-language reporting questions become a generated SQL query that is reviewed and re-planned before it ever runs against live data.

### Session-Aware Memory
A rolling per-session summary lets the assistant answer "what did we discuss earlier" without replaying the full transcript.

---

## Technology Stack

```text
Python
Django
Django REST Framework
LangGraph
Ollama
MySQL
Knox Authentication
```

---

## Business Impact

- Faster information discovery
- Reduced navigation effort
- Improved user productivity
- Enhanced adoption of enterprise applications

---

# 📘 Case Study 2: Intelligent Document Extraction Platform

> Internally implemented as the `ExtractionWorkbenchAPI` view (`ai_module.views.ExtractionWorkbenchAPI`).
> 📄 Full technical write-up: [extraction-workbench/README.md](extraction-workbench/README.md)

## Overview

The Intelligent Document Extraction Platform automates information extraction from PDFs, scanned documents, and images using OCR and LLM technologies while maintaining human validation capabilities.

---

## Business Challenge

Organisations often receive:

- Scanned invoices
- Purchase orders
- Vendor documents
- Image-based forms

Traditional OCR solutions struggle with:

- Variable layouts
- Poor image quality
- Missing labels
- Changing field positions

leading to manual intervention and data entry efforts.

---

## Solution Architecture

```text
Document Upload
        │
        ▼
OCR / Text Conversion (MarkItDown)
        │
        ▼
Schema-Driven LLM Field Extraction
        │
        ▼
JSON-Shape Validation
        │
        ▼
Human Review & Correction
        │
        ▼
Structured, Schema-Matched Output
```

---

## Core Features

### Schema-Driven Extraction (One Endpoint, Any Document Type)

The API doesn't hard-code what a "purchase order" or "invoice" looks like — it reads the target Django REST Framework serializer's fields by reflection and asks the LLM to fill exactly those fields. Onboarding a new document type is a serializer change, not a new endpoint or parsing routine.

---

### OCR Processing

Extracts raw text from:

- PDFs
- Scanned files
- Mobile images

via MarkItDown, which normalizes each format into Markdown text before passing it to the AI extraction layer.

---

### LLM-Based Information Extraction

Transforms unstructured content into structured business data.

Example:

```json
{
  "invoice_number": "INV-1001",
  "vendor_name": "ABC Industries",
  "amount": 12000,
  "invoice_date": "2026-08-01"
}
```

---

### Human-in-the-Loop Validation

Allows users to:

- Verify extracted fields
- Correct values
- Adjust field mapping locations
- Improve document accuracy

through an intuitive review interface.

---

## Technology Stack

```text
Python
Django
Django REST Framework
Ollama
MarkItDown
Docker
```

---

## Business Impact

- Reduced manual data entry
- Improved extraction accuracy
- Accelerated document processing
- Created reusable extraction workflows

---

# 📘 Case Study 3: Pest Identification Platform

> 📄 Full technical write-up: [pest-identification-platform/README.md](pest-identification-platform/README.md)

## Overview

The Pest Identification Platform is a proof-of-concept AI-powered visual diagnostic system for tea and specialty-crop plantations. It lets anyone with a smartphone photograph a damaged leaf, stem, or fruit and receive, in seconds, a structured, expert-grade diagnosis — pest or disease identity, affected plant part, seasonal behavior, symptoms, and recommended treatment.

---

## Business Problem

Plantation crop-protection expertise is scarce and centralized in a handful of agronomists who cannot be present on every estate, so diagnosis latency is bound by expert availability rather than damage-detection speed. Real-world field photos — arbitrary angles, variable lighting, unmarked diagnostic regions — defeat naïve, whole-image similarity matching, and existing digital tools only work by exact filename or catalog lookup, which is useless to someone who doesn't already know what they're looking at.

---

## Solution

The platform layers four complementary AI techniques into a single pipeline rather than relying on any one in isolation.

### Workflow

```text
Photo Upload
    │
    ▼
Color-Agnostic Background Removal
    │
    ▼
Content-Aware 5-Crop Variant Generation
    │
    ▼
Titan Multimodal Embeddings → OpenSearch k-NN Search
    │
    ▼
Vision-LLM Structured Attribute Extraction
    │
    ▼
LLM Evaluator — Triple-Signal Confidence Gate
    │
    ├── Confident  → Structured diagnosis + matched region
    │
    └── Unconfident → Honest "unidentified" + queued for expert review
```

A result is only returned as a confident match when the evaluator's confidence, an explicit match against the admin's verified description, and the candidate's raw similarity all agree — otherwise the system honestly reports "unidentified" and queues the case for human review, pre-enriched with the AI's own best-effort guess.

---

## Key Features

### Color-Agnostic Background Removal
Strips soil, grass, hands, and surfaces from the frame before embedding — deliberately avoiding a hue-based mask, since crop damage is itself often a color deviation from healthy tissue.

### Content-Aware Multi-Crop Embedding
Generates five crop variants per photo via edge-density scoring, so a diagnostic region anywhere in an unposed field photo is captured tightly instead of diluted inside one large frame.

### Vision-LLM Attribute Extraction
Extracts structured, plant-part-aware attributes — texture, lesion shape, color, description — adding a language-based signal that raw pixel similarity cannot provide.

### Triple-Signal Confidence Gating
Requires image similarity, evaluator confidence, and a match against the admin's fact-checked description to all agree before returning a confident result, with a deterministic numeric fallback if the evaluator is unavailable.

### Human-in-the-Loop Review Queue
Every unresolved case is queued for expert review and enriched in the background with an AI-suggested lead, turning every "we don't know" moment into a knowledge-base growth opportunity.

---

## Technology Stack

```text
Python
FastAPI
AWS Bedrock (Titan Multimodal Embeddings + vision LLM)
Amazon OpenSearch (k-NN)
DynamoDB
S3
rembg
```

---

## Business Impact

- First-pass diagnosis in seconds instead of days of expert-relay latency
- Targeted early-stage treatment instead of reactive, broad-spectrum spraying
- Expert time refocused on genuinely ambiguous cases
- A compounding, self-improving crop-protection knowledge asset

---

# 📘 Case Study 4: HRMS Agent

> 📄 Full technical write-up: [hrms-agent/README.md](hrms-agent/README.md)

## Overview

HRMS Agent replaces multi-step HRMS UI forms with a natural-language chat interface layered on top of the existing HRMS backend, letting employees check leave balances, apply for leave, and review team attendance by simply describing what they want — without HRMS Agent becoming a second source of truth for HR data.

---

## Business Problem

Routine HR tasks required navigating complex, multi-step UI forms, and HRMS exposed no conversational access path. Any new interface layer had to avoid duplicating HRMS's system of record, respect HRMS's individually-permissioned (not role-based) access model exactly, and integrate with the existing Angular frontend without a second login.

---

## Solution

HRMS Agent is a standalone FastAPI service that orchestrates a LangGraph tool-calling agent backed by Anthropic Claude, authenticating against HRMS and inheriting the user's exact permission set.

### Workflow

```text
User Login (HRMS credentials or SSO)
    │
    ▼
Fetch Permission Tree from HRMS (fail-closed)
    │
    ▼
Session Issued (HttpOnly token, 8h TTL)
    │
    ▼
LangGraph Agent Loop
    call_model → execute_tools → finish / force_final   (≤ 4 rounds)
    │
    ├── Read action        → Tool call → HRMS REST API → response
    │
    └── Mutating action     → Show payload → user confirms → execute → async audit log
    │
    ▼
Natural-Language Response
```

All HR data continues to live exclusively in HRMS's MySQL database; HRMS Agent stores only conversational and operational metadata — sessions, capabilities, chat history, and audit logs — in its own schema.

---

## Key Features

### LangGraph Tool-Calling Agent
An explicit `StateGraph` (`call_model → execute_tools → finish`), capped at 4 rounds, with a hallucination-detection module that retries when the model emits a tool call as plain text instead of a structured call.

### Permission-Based Tool Gating
Tools are assembled per user at login by exact alias matching against the user's live HRMS permission tree — the LLM can never see or call a tool the user isn't authorized to use.

### Fail-Closed Authorization
Login is blocked outright if the HRMS permission-list fetch fails, with no role fallback or approximation.

### Mutation Confirmation Gate
Every state-changing action (apply/cancel leave, approvals, regularization) shows the exact payload and waits for explicit user confirmation before executing.

### Async Audit Trail & Prompt Caching
Every mutation is logged asynchronously without adding latency, and the static system prompt is cached via Anthropic's prompt caching API, cutting input token cost by roughly 90% on cached turns.

---

## Technology Stack

```text
Python
FastAPI
LangChain / LangGraph
Anthropic Claude (Haiku / Sonnet)
httpx
PostgreSQL / MySQL
Pydantic v2
```

---

## Business Impact

- Reduced friction for routine HR self-service tasks
- No duplication of HRMS as the system of record
- Extensible foundation for future HR domains (payroll, appraisal)
- Lower, more predictable LLM inference cost via prompt caching
- Auditable, compliance-oriented operation with no added chat latency

---

# 📘 Case Study 5: SOL AI Platform

## Overview

SOL is a digital healthcare platform designed to provide AI-powered conversational support through an always-available intelligent assistant. The platform integrates LLM technologies within a healthcare-oriented workflow to enhance user engagement and accessibility.

---

## Objectives

- Deliver 24×7 AI-driven assistance
- Improve digital engagement
- Provide responsive conversational experiences
- Support users through intelligent interactions

---

## Core Components

### Conversational AI Engine

Provides context-aware interactions using Ollama-hosted LLMs.

### Secure Backend Services

Built using Django and REST APIs to support scalable integrations.

### Modular Architecture

Designed for future enhancement and additional AI capabilities.

---

## Technology Stack

```text
Python
Django
REST APIs
Ollama
MySQL
Docker
```

---

## Business Impact

- Improved accessibility
- Increased user engagement
- Enabled AI-assisted support workflows
- Demonstrated practical healthcare AI implementation

---

## 📫 Contact

- LinkedIn: linkedin.com/in/sk-aminul-islam-60300165
- Email: skaminulislam1993@gmail.com

> Building enterprise-grade AI solutions with Python, Django, LLMs, and modern cloud technologies.
