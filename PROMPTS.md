# PROMPTS

> Personal Learning Copilot — Prompt Library (Plan A: Pure Open‑Source on GCP)
> Author: Abhinav | Use these prompts with M365 Copilot to design, build, deploy, and open‑source your bot.
>
> Stack: Cloud Run (serverless), FastAPI + LangChain + LangServe, ChromaDB (RAG), Terraform (IaC), Ansible (config), Docker, optional GKE.
>
> Notes:
> - All prompts are in English and designed to trigger contextual follow‑ups.
> - Replace placeholders like `<PROJECT_ID>`, `<REGION>`, `<BUCKET_NAME>` before use.
>
---

## A) Foundations: Project Setup & Architecture

### 1. Project blueprint & naming
```
Design an end-to-end architecture for a personal learning copilot on GCP using pure open-source:
- Cloud Run (serverless) for API and UI
- FastAPI + LangChain + LangServe for REST endpoints
- ChromaDB for RAG
- Terraform for IaC, Ansible for post-provision config
- Docker images for services; optional GKE path later
Please propose project and resource naming conventions, regions (prefer Asia-South1/Delhi if suitable), and a secure IAM layout with least-privilege service accounts.
```

### 2. Architecture diagram (text + Mermaid)
```
Generate a text + Mermaid diagram showing:
- User → UI (Streamlit) → API (LangServe) → ChromaDB (vector store)
- Ingestion pipeline (PDF/YouTube) → embeddings → Chroma
- Exports (.docx/.xlsx) to Cloud Storage
- CI/CD via GitHub Actions → Cloud Run
Include trust boundaries, IAM roles, secrets handling, and scale-to-zero behavior.
```

---

## B) Terraform on GCP (IaC)

### 3. Terraform project scaffolding
```
Create a Terraform scaffold for:
- Cloud Run services (api, ui) with min-instances=0 and HTTPS-only
- Cloud Storage bucket for exports
- Service accounts and IAM bindings (least privilege)
- Optional Secret Manager
Include variables, outputs, and remote state guidance (local for now; show how to migrate to GCS state later).
```

### 4. Reusable Terraform modules
```
Design reusable Terraform modules:
- module_cloud_run_service (image, env vars, concurrency, CPU/memory)
- module_gcs_bucket (lifecycle rules, uniform access, encryption)
- module_sa (service account + IAM roles)
Provide examples of composition in a root stack.
```

### 5. Terraform security & policy
```
Propose Terraform IAM policies and constraints:
- Least-privilege roles for Cloud Run deploy, storage write, and logs
- Deny public invocations except where explicitly allowed for UI
- Secrets in Secret Manager; bind only to api SA
Suggest org policies (if personal project allows) and audit logging.
```

### 6. Terraform + CI/CD
```
Write a GitHub Actions workflow that runs terraform fmt/validate/plan, and on approval applies:
- Safe handling of service account JSON via GitHub secrets
- Plan file artifact and manual approval gate
Introduce rollback guidance and tagging strategy.
```

---

## C) Ansible for Post‑Provision Configuration

### 7. Ansible roles for VMs (optional Ollama host)
```
Create Ansible roles to:
- Install Docker
- Install and configure Ollama with Gemma/Mistral models
- Harden SSH (fail2ban, ufw), configure audit logs
Provide inventory examples for GCP VMs and a playbook to run after Terraform provisioning.
```

### 8. Ansible + Terraform pipeline
```
Show how to pass Terraform outputs (VM IPs, SA emails) into an Ansible playbook:
- Use `terraform output -json` → parse in CI
- Run Ansible against new instances
Include idempotent checks and health verification tasks.
```

---

## D) Docker & Kubernetes

### 9. Optimized Dockerfiles
```
Provide two multi-stage Dockerfiles:
- api.Dockerfile (FastAPI + LangServe): smallest possible base, non-root user, poetry/pip caching, healthcheck
- ui.Dockerfile (Streamlit or Gradio): small base, static asset caching
Include notes on reducing image size and enabling HTTP/2 on Cloud Run.
```

### 10. GKE path (optional)
```
Draft minimal Kubernetes manifests for deploying api + ui:
- Deployment, Service, Ingress
- ConfigMaps for app settings, Secrets for credentials
Add a Helm chart skeleton and values.yaml with production defaults.
```

---

## E) LangServe + Chroma (RAG & Teaching)

### 11. LangServe chain design
```
Design LangChain + LangServe routes:
- /ingest: accept PDF/YouTube URL, store raw in GCS, produce embeddings, write to Chroma
- /teach: hybrid pedagogy output (prime → concepts → practice → active recall → reflection) using Chroma retrieval
- /notes: export Cornell/Outline/Zettelkasten as .docx/.md to GCS
- /quiz: MCQ + short-answer + one DevOps lab task
Provide FastAPI router examples and error handling.
```

### 12. Chroma configuration & retrieval tuning
```
Recommend chunk size, overlap, Top-K, and metadata filters in Chroma for mixed PDFs and transcripts.
Add code to:
- Create collections
- Upsert docs with metadata
- Query with semantic + keyword filters
Optimize for low-cost personal use.
```

### 13. Exporters (.docx/.xlsx)
```
Write functions to export:
- Notes as .docx in Cornell/Outline/Zettelkasten templates
- Flashcards and spaced repetition schedule as .xlsx (SM-2 fields)
Explain where to store in GCS and download flow.
```

---

## F) Spaced Repetition & Quiz

### 14. SM‑2 scheduler (lightweight)
```
Implement a lightweight SM-2 scheduler:
- Inputs: question ID, last review date, ease, success (0/1 or 0-5)
- Outputs: next review date, updated ease
Provide pseudocode and a Python function, plus export to .xlsx format.
```

### 15. Quiz generation
```
Create a quiz generator that:
- Builds 10 MCQ, 5 short answers, 1 hands-on lab task (Terraform/Docker/K8s)
- Difficulty adaptively increases by prior score
Include a simple grading rubric and JSON schema for results.
```

---

## G) CI/CD (GitHub Actions) & Release

### 16. CI pipeline
```
Draft a CI workflow:
- Lint (flake8/black), unit tests (pytest), prompt/chain tests for determinism
- Build Docker images; push to Artifact Registry or GCR
Include caching and runtime matrix for Python 3.11/3.12.
```

### 17. CD to Cloud Run
```
Create a CD workflow that:
- On tag v*.*.* deploys to Cloud Run
- Uses traffic splitting (e.g., 10% canary) with quick rollback
- Posts deployment notes to the repo wiki (or releases)
```

### 18. Open‑source docs pack
```
Generate:
- docs/quickstart.md (5-min setup)
- docs/architecture.md (with diagram)
- docs/rag-design.md (chunking, Top-K, embeddings)
- docs/faq.md (local vs CI vs Cloud Run)
- docs/interview-cheatsheet.md
Use Apache-2.0 LICENSE and CONTRIBUTING.md templates.
```

---

## H) Security & Cost

### 19. Serverless security blueprint
```
Recommend hardening for Cloud Run:
- HTTPS-only, authenticated invocations where needed
- Dedicated service accounts, least privilege IAM
- Secret Manager for sensitive values
- Logging/monitoring setup and error budget policy
Provide Terraform snippets and operational checks.
```

### 20. Cost controls
```
Show concrete cost levers:
- min-instances=0, low memory (512–1024 MB), low concurrency
- No idle VMs; short timeouts
- Small Top-K and chunk sizes for RAG
Provide a monthly cost table for personal use and spend alerts.
```

---

## I) Interview‑Ready

### 21. 3‑minute architecture pitch
```
Write a concise 3-minute interview pitch explaining:
- Why Cloud Run for serverless containers (scale-to-zero, pay-per-use)
- Why LangServe (fast REST productionization of chains)
- Why Chroma (OSS vector DB; easy integration)
- CI/CD and Terraform for reproducible ops
Include trade-offs and future enterprise upgrades.
```

### 22. Trade-offs & upgrades
```
Explain trade-offs of OSS models vs managed (Gemini), and when to selectively add Agent Engine features (sessions/memory/eval/code execution) for enterprise-grade behavior.
```

---

## J) Daily Macros

### 23. Today’s plan
```
Given my current milestone (Week X), propose a 60–90 minute plan with:
- 3 actionable tasks (smallest possible steps)
- Expected outputs (files, endpoints, tests passing)
- Quick checks and fallback if something fails
```

### 24. Code review
```
Review my files for performance, security, and reliability:
- apps/api/server.py, apps/api/chains/*.py, docker/*.Dockerfile
- Suggest logging, error handling, retries, streaming, and test coverage
```

### 25. RAG tuning
```
Tune my RAG for the given resource set:
- Recommend chunk size/overlap/Top-K
- Show example queries and prompt templates
- Propose evaluation method (precision/recall, groundedness checks)
```

### 26. CI/CD guardrails
```
Advise guardrails for my GitHub Actions pipelines:
- Secret handling, approvals, rollback strategy
- Canary and traffic-split configuration for Cloud Run
```

### 27. Interview prep
```
Test me with 8 rapid-fire questions on:
- Cloud Run serverless patterns
- LangServe productionization
- Chroma RAG design
- Terraform/IaC and CI/CD best practices
```
