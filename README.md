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
- Claude API
- OpenAI API
- LangGraph
- Ollama
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

### PMS AI Search Assistant
An intelligent enterprise assistant capable of understanding natural language queries and routing them to database retrieval, application navigation, or AI-generated responses. Built using LangGraph, Ollama, Django, and Redis.

### Intelligent Document Extraction Platform
OCR + LLM-powered document processing system that extracts structured data from PDFs and images with a human-in-the-loop validation interface.

### SOL AI Platform
A healthcare-focused AI platform delivering conversational assistance and digital support experiences through LLM-powered interactions.

### Data Lake Analytics Platform
Kafka and PySpark-based large-scale data processing architecture supporting scalable analytics and operational intelligence.

---

# 📘 Case Study 1: PMS AI Search Assistant

## Overview

The PMS AI Search Assistant was developed to simplify information retrieval inside an Enterprise Project Management System (PMS). Instead of manually navigating modules or searching through menus, users can interact with the system using natural language.

---

## Business Problem

Enterprise users frequently struggle to:

- Locate information across multiple PMS modules
- Identify the correct workflow or screen
- Retrieve project information quickly
- Understand module relationships

These challenges reduce productivity and increase onboarding efforts.

---

## Solution

The PMS AI Search Assistant uses a multi-stage AI orchestration workflow to analyse user intent and determine the most appropriate action.

### Workflow

```text
User Query
    │
    ▼
Intent Detection
    │
    ├── Database Search
    │
    ├── Module Navigation
    │
    ├── Knowledge Retrieval
    │
    └── LLM Response
    │
    ▼
Content Moderation
    │
    ▼
Final Response
```

The assistant determines whether a query requires:

- Database retrieval
- Module navigation guidance
- Knowledge-based answers
- General AI assistance

and automatically routes requests accordingly.

---

## Key Features

### Intelligent Query Routing
Supports multiple execution paths based on detected user intent.

### Enterprise Search Experience
Provides users with contextual responses without requiring knowledge of system structure.

### Safety & Moderation
Includes vulgarity and content-validation checks for both user inputs and generated outputs.

### Hybrid Retrieval Architecture
Combines:

- Structured database lookup
- Business logic execution
- AI reasoning

to improve response accuracy.

---

## Technology Stack

```text
Python
Django
LangGraph
Ollama
Redis
PostgreSQL
Docker
```

---

## Business Impact

- Faster information discovery
- Reduced navigation effort
- Improved user productivity
- Enhanced adoption of enterprise applications

---

# 📘 Case Study 2: Intelligent Document Extraction Platform

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
OCR Extraction
        │
        ▼
LLM Field Recognition
        │
        ▼
Confidence Validation
        │
        ▼
Manual Review UI
        │
        ▼
Structured JSON Output
```

---

## Core Features

### OCR Processing

Extracts raw text from:

- PDFs
- Scanned files
- Mobile images

before passing information to the AI extraction layer.

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
OCR Engine
LLM APIs
PostgreSQL
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

Provides context-aware interactions using LLM APIs.

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
LLM APIs
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
