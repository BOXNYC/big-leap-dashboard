# Big Leap - Healthcare Provider Dashboard

[Thought Exercise Overview](https://reminiscent-aries-c5f.notion.site/Full-Stack-Engineer-Thought-Exercise-29be4ce57d97808db4b7c3bc6f2834eb)

## Structural Overview

- **Users** (Authenticated)
  - profile, account, preferences
  - role: admin | agent | user | patient
  - practices (membership)
  - patient-specific: dob, insurance (versioned), eligibility, records
- **Practice**
  - info (name, location, etc.)
- **HealthRecords**
  - uid (patient user id)
  - pid (practice id)
  - data (EHR)
  - versioned — each write creates a new record; no silent overwrites of PHI
  - source: ehr | practice-submitted | ocr-extracted
- **Tasks**
  - id, type (e.g. missing-dob | insurance-update)
  - uid (patient), pid (practice)
  - status: open | in-progress | resolved | rejected
  - assigned_to (user id)
  - created_by, created_at
  - resolved_by, resolved_at
  - audit log — append-only event trail (assignment, status changes, edits, resolutions); attributed to user + timestamp

---

## System and Technical Design

### Architecture

A lightweight, task-driven web application that enables structured collaboration between Big Leap and provider practices to resolve data gaps and maintain accurate patient records.

- **Frontend & API Layer**
  - Next.js (App Router), TypeScript
  - API routes + AWS serverless functions for async workflows

- **Database, Auth & Storage**
  - Supabase (Postgres + Auth + Storage + Realtime + Queues, [HIPAA add-on](https://supabase.com/docs/guides/security/soc-2-compliance))
    - *Supabase was chosen as it is SOC 2 compliant as well as HIPAA with a $7k add on. Being v1 needs to be completed in 6–8 weeks, and SOC 2 compliance can take weeks or months to achieve, Supabase can provide compliance without a delay.*
  - **Row-level security for strict practice-level data isolation**
  - **[Supabase Realtime](https://supabase.com/docs/guides/realtime)** — Postgres change listeners broadcast row-level inserts/updates to subscribed clients instantly; task lists, patient detail views, and dashboards update live without a page refresh
  - **[Supabase Queues](https://supabase.com/docs/guides/queues/api)** for durable, async notification delivery — task assignments, status changes, and resolution events are enqueued in Postgres and consumed by workers, preventing notification loss during transient failures
  - **Kysely** A type-safe SQL query builder specifically designed for TypeScript. Works very well with Supabase Postgres.  *Personal preference*

### Core System Model

The system is centered around **Tasks**, not records.

- Patients are hydrated from EHR data, either by request, queue workers, or cron.
- Data gaps or inconsistencies generate **Tasks**
- Practices resolve Tasks via:
  - confirm / correct structured data
  - upload supporting documents (insurance cards, PDFs)
- All updates are:
  - versioned
  - attributed
  - auditable

### Key Integrations

- Supabase Auth (user management, access control)
- Supabase Storage (secure document uploads and retrieval)
- **[Supabase Realtime](https://supabase.com/docs/guides/realtime)** — live task and record updates pushed to all subscribed clients; users see task status changes and new assignments instantly without polling or refreshing
- **[Supabase Queues](https://supabase.com/docs/guides/queues/api)** — durable message queue (pgmq) for all notification events; workers dequeue and dispatch via email or other channels
- Email notifications (transactional task updates, dispatched from queue workers)
- OCR pipeline *([Google OCR?](https://cloud.google.com/use-cases/ocr))* (image/PDF → structured data extraction)

### Security & Compliance

- HIPAA-aligned architecture using Supabase (DB, Auth, Storage)
- Row-level security (practice isolation)
- Encrypted storage and transport
- Full audit logs for all reads/writes
- No silent overwrites of PHI (Protected Health Information); all changes are versioned and reviewable

---

## Product Thinking

This is not a data viewer. It is a **workflow engine for resolving incomplete or unreliable healthcare data**.

---

## Implementation Plan

### V1 - Proof of Value

**Goal:** Replace manual back-and-forth with a single shared workflow.

**Scope:**

- Auth + practice-scoped access
- Patient list (hydrated from EHR)
- Task system (initial types):
  - missing/incorrect DOB
  - insurance updates
- Patient detail view
- File upload (insurance cards, PDFs via Supabase Storage)
- Task resolution + audit log
- Live UI updates via [Supabase Realtime](https://supabase.com/docs/guides/realtime) — task list and patient detail views reflect changes instantly for all active users
- Notification system via [Supabase Queues](https://supabase.com/docs/guides/queues/api) — task assignment and status changes enqueued in Postgres, dispatched as transactional emails by a worker

**Why:**
Targets highest-friction workflows and validates adoption quickly.

---

### V2 - Scale & Automation

**Goal:** Reduce manual intervention and increase throughput.

**Add:**

- OCR extraction + review queue (enqueued via [Supabase Queues](https://supabase.com/docs/guides/queues/api)). *Only if OCR is not already provided in the EHR data*
- Automated task generation (rules-based)
- Background job processing (async pipelines via Supabase Queues workers)
- Analytics (resolution time, bottlenecks)
- Admin tooling + bulk operations
- **Practice ↔ Big Leap Chat** — real-time messaging per patient/task context using [Supabase Realtime](https://supabase.com/docs/guides/realtime) for instant message delivery; new message events enqueued via [Supabase Queues](https://supabase.com/docs/guides/queues/api) to dispatch email digests, indicators, and/or push notifications when the recipient is online or offline

**Result:**
System evolves from manual coordination tool → semi-automated data pipeline with real-time collaboration.

---

## Risks, Unknowns, and Questions

### Risks

- **Unreliable source data**
  → Mitigation: versioning + source attribution + review states

- **Low practice adoption**
  → Mitigation: ultra-simple UX, minimal required input

- **EHR variability**
  → Mitigation: normalize at ingestion layer, decouple UI from raw payloads


---

### Unknowns

- Which EHR systems and data quality levels?
- Eligibility logic ownership and rules?
- Preferred communication channels?

---

### Approach

- Start with 2–3 real workflows currently handled manually
- Define source-of-truth rules per field
- Ship V1 quickly and observe behavior. Practices incentivized by early adopter discounts
- Iterate based on real usage
