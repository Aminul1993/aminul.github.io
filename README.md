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

# 📘 Case Study 3: SOL AI Platform

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
