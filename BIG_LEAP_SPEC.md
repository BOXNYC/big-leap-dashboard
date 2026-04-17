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
  - uid (patient)
  - data (EHR)

---

## System and Technical Design

### Architecture

A lightweight, task-driven web application that enables structured collaboration between Big Leap and provider practices to resolve data gaps and maintain accurate patient records.

- **Frontend & API Layer**
  - Next.js (App Router), TypeScript
  - API routes + AWS serverless functions for async workflows

- **Database, Auth & Storage**
  - Supabase (Postgres + Auth + Storage, [HIPAA add-on](https://supabase.com/docs/guides/security/soc-2-compliance))
    - *Supabase was chosen as it is SOC 2 compliant as well as HIPAA with a $7k add on. Being v1 needs to be completed in 6–8 weeks, and SOC 2 compliance can take weeks or months to achieve, Supabase can provide compliance without a delay.*
  - **Row-level security for strict practice-level data isolation**
  - **Kysely** A type-safe SQL query builder specifically designed for TypeScript. Works very well with Supabase PosgreSQL.  *Personal preference*

### Core System Model

The system is centered around **Tasks**, not records.

- Patients are hydrated from EHR data
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
- Email notifications (transactional task updates)
- OCR pipeline (image/PDF → structured data extraction)

### Security & Compliance

- HIPAA-aligned architecture using Supabase (DB, Auth, Storage)
- Row-level security (practice isolation)
- Encrypted storage and transport
- Full audit logs for all reads/writes
- No silent overwrites of PHI; all changes are versioned and reviewable

---

## Product Thinking

This is not a data viewer. It is a **workflow engine for resolving incomplete or unreliable healthcare data**.

### Principles

- **Tasks over data**
  - Surface what needs action, not raw records

- **Minimize effort for practices**
  - Only request what is missing or uncertain
  - Pre-fill from EHR wherever possible

- **Source-aware data**
  - Distinguish between:
    - EHR data
    - practice-submitted data
    - extracted (OCR) data
  - Require confirmation where confidence is low

- **Operational clarity**
  - Always show:
    - what is needed
    - why it matters
    - current status

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
- Email notifications

**Why:**
Targets highest-friction workflows and validates adoption quickly.

---

### V2 - Scale & Automation

**Goal:** Reduce manual intervention and increase throughput.

**Add:**

- OCR extraction + review queue
- Automated task generation (rules-based)
- Background job processing (async pipelines)
- Analytics (resolution time, bottlenecks)
- Admin tooling + bulk operations

**Result:**
System evolves from manual coordination tool → semi-automated data pipeline.

---

## Risks, Unknowns, and Questions

### Risks

- **Unreliable source data**
  → Mitigation: versioning + source attribution + review states

- **Low practice adoption**
  → Mitigation: ultra-simple UX, minimal required input

- **EHR variability**
  → Mitigation: normalize at ingestion layer, decouple UI from raw payloads

- **Ambiguous source of truth**
  → Mitigation: explicit states (system vs submitted vs approved)

---

### Unknowns

- Which EHR systems and data quality levels?
- Eligibility logic ownership and rules?
- Required turnaround times (SLAs)?
- Document review expectations (manual vs automated)?
- Preferred communication channels?

---

### Approach

- Start with 2–3 real workflows currently handled manually
- Define source-of-truth rules per field
- Ship V1 quickly and observe behavior
- Iterate based on real usage
