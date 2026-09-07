# 🎯 Smart Interview Orchestration Platform (Control Tower)

[![Node.js](https://img.shields.io/badge/Node.js-v20%2B-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Express](https://img.shields.io/badge/Express-4.x-000000?logo=express&logoColor=white)](https://expressjs.com)
[![React](https://img.shields.io/badge/React-18-61DAFB?logo=react&logoColor=black)](https://react.dev)
[![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110%2B-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Google OR-Tools](https://img.shields.io/badge/OR--Tools-CP--SAT-4285F4?logo=google&logoColor=white)](https://developers.google.com/optimization)
[![Prisma](https://img.shields.io/badge/Prisma-6.x-2D3748?logo=prisma&logoColor=white)](https://prisma.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> Most interview tools only answer: *"Is this calendar slot free?"*  
> **This platform answers:** *"Is this interview plan fair, resilient, and optimal—and what happens the moment reality breaks it?"*

---

## 📌 Table of Contents
- [What We Did](#-what-we-did)
- [🤖 AI Transparency & Engineering Boundaries](#-ai-transparency--engineering-boundaries)
- [Architecture & System Flow](#-architecture--system-flow)
- [Key Features by Persona](#-key-features-by-persona)
- [Production Triad: Scalability, Resilience & Security](#-production-triad-scalability-resilience--security)
- [Getting Started](#-getting-started)
- [Pre-Seeded Walkthrough Accounts](#-pre-seeded-walkthrough-accounts)
- [Database & Switchable Engines](#-database--switchable-engines)
- [Project Structure](#-project-structure)

---

## 🚀 What We Did

Modern interview coordination is plagued by **scheduling gridlock**, **interviewer burnout**, **last-minute cancellations**, and **cascading round delays**. We built a full-stack, enterprise-grade **Interview Orchestration Platform** that replaces manual coordination with a self-healing scheduler.

### Core Breakthroughs Delivered:
1. **Mathematical Constraint Optimization**: Powered by Google OR-Tools CP-SAT in Python to evaluate multi-factor soft objectives (skill coverage, candidate preferences, workload distribution, and timezone comfort) instead of naive first-free-slot heuristics.
2. **The Control Tower (Self-Healing Incident Engine)**: A background observer that monitors active interviews, detects disruptions (declines, no-shows, meeting link drops), evaluates cascade risk, and applies automated remediation under an **Autonomy Policy**.
3. **Double-Booking Prevention with Zero-Race Guarantees**: A critical section operating under `Serializable` database transactions, buffer collision re-verification, and hardware-level unique constraints.
4. **Offline-First / Zero-Cloud Demo Mode**: Everything runs locally out of the box with zero paid API keys needed (using SQLite, deterministic ontology extractors, mock calendar, and mock video rooms). Switching to production cloud providers (Google Calendar, Gemini, Zoom, PostgreSQL) requires just a single `.env` change.

---

## 🤖 AI Transparency & Engineering Boundaries

In high-stakes enterprise systems (like hiring and executive calendar reservation), **unconstrained AI hallucinations and non-deterministic behavior are unacceptable**. 

We operate under an explicit **Triad Contract**:

```
┌─────────────────┐     ┌───────────────────────┐     ┌──────────────────────┐
│  AI (LLMs)      │     │  Algorithms           │     │  Optimization        │
│  "Meaning"      │     │  "Guarantees"         │     │  "Decisions"         │
└────────┬────────┘     └──────────┬────────────┘     └──────────┬───────────┘
         │                         │                             │
  Extracts unstructured     Enforces hard boundaries,      OR-Tools CP-SAT
  intent, sentiment,        timezones, rest buffers,       maximizes objective
  and skill semantics       RBAC, and physical locks       over feasible space
```

### 🔍 AI Transparency Matrix

| Subsystem | Underlying Technology | Why This Tech Was Chosen | Fallback & Guardrails |
| :--- | :--- | :--- | :--- |
| **Schedule Selection** | **Google OR-Tools CP-SAT** *(Not an LLM)* | Mathematical optimization guarantees the best feasible assignment without hallucinations. | Falls back to internal **Deterministic JS Heuristic Engine** if solver times out. |
| **Double-Booking Defense** | **Prisma `Serializable` Transaction** *(Algorithm)* | Concurrency conflicts require strict ACID consistency and unique index locking. | HTTP `409 SLOT_TAKEN` instant rollback; zero partial state. |
| **Natural Language Availability** | **LLM (Gemini / Ollama)** | Candidates type natural phrases like *"Free Tuesdays after 2 PM IST"*. | Schema enforced via **Pydantic**. Retries once on malformed JSON; falls back to **Regex/Ontology Extractor**. |
| **Skill Taxonomy Matching** | **Embeddings & O\*NET Taxonomy** | Semantic matching between JD requirements and candidate resumes. | Falls back to exact lexical matching and alias dictionaries (`extractors.py`). |
| **Feedback Synthesis** | **LLM (Gemini / Ollama)** | Evaluates interviewer notes for strengths, gaps, and suggested probe topics. | Structured Pydantic extraction; results flagged with `aiProviderUsed` for complete auditability. |
| **Incident Recovery** | **Deterministic Rule Precedence Engine** | Autonomy decisions (replacing interviewers, shifting slots) require predictable execution. | **Autonomy Policy**: Only `LOW` risk actions auto-apply; `MEDIUM`/`HIGH` risk actions mandate human recruiter approval. |

> 🛡️ **Guaranteed Transparency Invariant**: Every proposal, score, and automated action records an **`AuditLog`** and returns an explainable breakdown (`breakdownJson`, `reasonsJson`, and `engineUsed: "ORTOOLS" | "JS_FALLBACK"`).

---

## 🏗️ Architecture & System Flow

The system is organized as a **Modular Monolith** (Node.js/Express) bridged to a **High-Performance Math/AI Engine** (Python/FastAPI).

```mermaid
flowchart TB
  subgraph Client["Client Layer (React 18 + Vite + Tailwind)"]
    UI1[Recruiter Dashboard & Control Tower]
    UI2[Candidate Availability & Join Portal]
    UI3[Interviewer Assignments & Feedback]
    UI4[Admin Policies & System Health]
  end

  subgraph API["Backend API Layer (Node.js 20 / Express Monolith)"]
    AUTH[Auth & RBAC: 15m JWT + Hashed Refresh Tokens]
    DOMAIN[Domain Services: Jobs, Applications, Requests]
    CRIT[Critical Section: Serializable Tx + Booking Guard]
    CT[Control Tower: Detector, Cascade Impact, Bounded Autonomy]
    FALLBACK[Deterministic JS Fallback Scheduler]
    PROV[Adapters: Google Calendar / Jitsi / Twilio / SMTP]
  end

  subgraph AISVC["AI & Optimization Service (Python 3.12 / FastAPI)"]
    OPT[Google OR-Tools CP-SAT Solver]
    MATCH[Skill Embeddings & Ontology Matcher]
    LLM[LLM Gateway: Gemini / Ollama / Mock Extractor]
    VAL[Pydantic Schema Validation & Sanitizers]
  end

  DB[(Database: SQLite / PostgreSQL via Prisma)]

  Client -->|REST API + JWT| API
  API --> DB
  API -->|HTTP + Circuit Breaker + 90s Timeout| AISVC
  API -.->|On Timeout or Error| FALLBACK
  AISVC --> OPT & MATCH & LLM
```

---

## 👥 Key Features by Persona

### 1. 👔 Recruiter Command Center
- **Job & Pipeline Management**: Define roles with weighted must-have/nice-to-have skills and interview windows.
- **Visual Schedule Builder**: View ranked slot proposals with multi-factor health scores (`0-100`) and explainable "WHY" breakdowns.
- **Control Tower Live Monitoring**: Real-time disruption alerts (e.g., interviewer decline, meeting link failure) with one-click approval for recommended recovery plans.
- **Audit & Analytics**: Complete compliance history, funnel conversion times, and interviewer load distribution.

### 2. 🎓 Candidate Portal
- **Natural Language Availability**: Type or speak availability (e.g., *"Monday morning or Wednesday after 3pm"*) converted to structured UTC intervals.
- **Timezone-Aware Interactive Picker**: Visual slot confirmation automatically aligned with local civil hours.
- **One-Click Meeting Join**: Access embedded Jitsi/Google Meet links with zero external friction.

### 3. 🧑‍💻 Interviewer Workspace
- **Smart Assignment Queue**: Review upcoming technical panels with candidate resume match insights and focus topics.
- **Workload & Burnout Guard**: Configure daily/weekly interview caps, preferred working hours, and rest buffers.
- **Structured Feedback Submission**: Grade skills on a 1–5 scale with instant AI parsing for strengths, growth areas, and recommended next-round probe topics.

### 4. ⚙️ Administrator & Governance
- **Autonomy Policy Tuner**: Set the risk ceiling (`LOW`, `MEDIUM`, `HIGH`) for automated schedule self-healing.
- **Engine Switcher**: Seamlessly toggle between Google OR-Tools and internal fallback algorithms.
- **Health Discovery**: Real-time status checks across the database, AI engine, and external integrations (`/api/health`).

---

## 🛡️ Production Triad: Scalability, Resilience & Security

### ⚡ Scalability
- **Node-to-Python Boundary**: CPU-intensive integer programming and vector computations are offloaded to Python, leaving Node.js free for rapid I/O.
- **Multi-Stage Pruning**: Ineligible interviewers (skills, daily load limits, time-off) are pruned before invoking the solver, preventing combinatorial explosion.
- **Database Portability Contract**: The schema avoids engine-specific extensions, allowing clean migrations from local SQLite to scaled PostgreSQL read replicas.

### 🔄 Resilience
- **The Critical Section**: Concurrent confirmations run inside a database transaction (`assertNoConflicts()`) backed by a composite unique index `@@unique([userId, startUtc, kind])`. Losers receive an instant HTTP `409 Conflict: SLOT_TAKEN`.
- **Decoupled Side Effects**: Google Calendar sync and email notifications run **post-commit**. If Google Calendar is down, the booking is preserved, and an incident is raised in the Control Tower.
- **Circuit Breaker**: If the Python service fails 3 times, the breaker enters `OPEN` state and routes requests to the internal **JavaScript Fallback Engine**.
- **Idempotency Safeguard**: Mutating endpoints require an `Idempotency-Key` header, preventing duplicate bookings from accidental double-clicks or network retries.

### 🔒 Security
- **Defense-in-Depth Identity**: 15-minute access JWTs coupled with SHA-256 hashed refresh tokens in the database, allowing instant session revocation on logout.
- **IDOR Prevention**: Granular route guards (`requireCandidateAccess`) ensure candidates cannot read or modify another candidate's profile or schedule.
- **AES-256-GCM Encryption at Rest**: Third-party Google Calendar OAuth tokens are stored in the database with authenticated symmetric encryption (`lib/crypto.js`).
- **File Upload Hardening**: Resumes are renamed to cryptographically random filenames, restricted by extension and MIME type allowlists, and served with `nosniff` headers to eliminate RCE and path traversal risks.

---

## 🏁 Getting Started

### Prerequisites
- **Node.js** >= 20.0.0
- **Python** >= 3.11 (Python 3.12 recommended)
- **npm** >= 10.0.0

### Quick Start (One Command)

1. **Clone the repository:**
   ```bash
   git clone https://github.com/poojakarank/interview-scheduler.git
   cd interview-scheduler
   ```

2. **Run the automated setup:**
   ```bash
   npm run setup
   ```
   *(This installs root, backend, and frontend dependencies, runs Prisma migration, and seeds the demo database.)*

3. **Set up the Python AI Service:**
   ```bash
   cd ai-service
   python -m venv .venv
   # Windows:
   .venv\Scripts\activate
   # macOS/Linux:
   source .venv/bin/activate

   pip install -r requirements.txt
   cd ..
   ```

4. **Launch all services concurrently:**
   ```bash
   npm run dev
   ```

The applications will start on:
- 🌐 **Frontend Web App:** [http://localhost:5173](http://localhost:5173)
- ⚙️ **Backend API:** [http://localhost:4000](http://localhost:4000)
- 🧠 **AI & Optimization Service:** [http://localhost:8000](http://localhost:8000)
- 📊 **Health Check:** [http://localhost:4000/api/health](http://localhost:4000/api/health)

---

## 🎭 Pre-Seeded Walkthrough Accounts

The database comes pre-seeded with specialized personas designed to demonstrate specific pipeline behaviors.

All accounts use the universal password: **`Password123`**

| Persona / Role | Email | Unique Demo Scenario |
| :--- | :--- | :--- |
| **Lead Recruiter** | `recruiter@scheduler.dev` | Command Center, Control Tower, Schedule Builder, Audit Logs |
| **Senior Staff Engineer** | `ananya.sharma@company.dev` | Wide-open availability; auto-accepts matching backend rounds |
| **Staff Data Engineer** | `priya.nair@company.dev` | Mornings-only availability; demonstrates interviewer load balancing |
| **Senior Data Engineer** | `leila.haddad@company.dev` | Limited Tuesday/Thursday availability; triggers alternative offers |
| **Candidate (Aisha)** | `aisha.khan@gmail.com` | Overlaps Ananya's declared window; instant auto-booking |
| **Candidate (Rahul)** | `rahul.verma@gmail.com` | Afternoons-only preference; triggers the multi-tier offer ladder |
| **Candidate (Grace)** | `grace.hopper@gmail.com` | Requests an unstaffable round; demonstrates graceful, honest failure |
| **Candidate (Nadia)** | `nadia.connor@gmail.com` | Pre-configured with active Control Tower incidents (auto-healed & awaiting approval) |

---

## 🗄️ Database & Switchable Engines

The platform runs on **SQLite** by default (requiring zero database installations). To switch to **PostgreSQL**:

1. Update your `.env` with your PostgreSQL connection string:
   ```env
   DATABASE_URL="postgresql://postgres:postgres@localhost:5432/interview_scheduler?schema=public"
   DATABASE_PROVIDER="postgresql"
   ```
2. Run the switch script:
   ```bash
   npm run db:use-postgres
   npm run db:push
   npm run db:seed
   ```
   *(To return to SQLite: `npm run db:use-sqlite`)*

---

## 📂 Project Structure

```text
interview-scheduler/
├── ai-service/              # Python 3.12 / FastAPI service
│   ├── app/
│   │   ├── routers/         # /schedule/solve, /match/skills, /extract/*
│   │   ├── services/        # OR-Tools CP-SAT optimizer, health scorer, extractors
│   │   ├── providers/       # Gemini, Ollama, and Mock LLM adapters
│   │   └── schemas.py       # Strict Pydantic models & validation
│   └── requirements.txt
├── backend/                 # Node.js 20 / Express modular monolith
│   ├── src/
│   │   ├── routes/          # REST endpoints (auth, scheduler, control tower)
│   │   ├── services/        # Orchestration, matching, availability, audit
│   │   ├── middleware/      # Auth (JWT), RBAC, Idempotency, File Upload
│   │   ├── providers/       # Calendar, Meeting, Notification, AI-client
│   │   ├── jobs/            # Control Tower monitor (60s interval observer)
│   │   └── lib/             # Crypto (AES-256-GCM), Prisma client, Time math
├── frontend/                # React 18 + Vite + Tailwind CSS SPA
│   ├── src/
│   │   ├── pages/           # Recruiter, Candidate, Interviewer & Admin portals
│   │   ├── components/      # UI widgets, Calendar views, Notification drawer
│   │   └── context/         # Auth, Notification, and Theme state
├── prisma/                  # Single Portable Schema & Deterministic Seeds
│   ├── schema.prisma        # 23 relational models (Portability Contract)
│   └── seed.js              # Scripted multi-persona test scenarios
├── shared/                  # Universal constants (Roles, Statuses, Risk levels)
└── docs/                    # Architecture diagrams and specifications
```

---

## 📄 License
This project is licensed under the **MIT License**.
