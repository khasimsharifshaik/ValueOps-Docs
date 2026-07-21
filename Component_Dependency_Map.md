# ValueOps 3D Component Architecture — Dependency & Interaction Map

**Generated:** 2026-07-21 | **Scope:** MVP + R1  
**Purpose:** Detailed component-level reference for understanding data flows, dependencies, and integration patterns

---

## 🎯 Architecture Layers Overview

```
┌─────────────────────────────────────────────────────────────────┐
│ 🎨 FRONTEND LAYER (React)                                       │
│ SpecAuthoringPanel │ SpecViewer │ Persona Surfaces │ API Client │
└─────────────────┬───────────────────────────────────────────────┘
                  │ HTTP/REST + JWT
┌─────────────────▼───────────────────────────────────────────────┐
│ 🔐 AUTH LAYER                                                   │
│ Keycloak (helio realm) → JWT token → Persona RBAC              │
│ Projection Layer (server-side surfacing enforcement)            │
└─────────────────┬───────────────────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────────────────┐
│ ⚙️  BACKEND LAYER (NestJS Modules)                             │
│                                                                  │
│ ┌─────────────┐ ┌──────────┐ ┌────────┐ ┌──────────┐           │
│ │ D1: Arch    │ │ D2: Spec │ │ D3: R&E│ │ Platform │           │
│ │ Authoring   │ │ Capture  │ │ Risk   │ │ Services │           │
│ │ Policies    │ │ (stub)   │ │ Ethics │ │          │           │
│ │ Governance  │ │          │ │ Privacy│ │          │           │
│ │ Conformance │ │          │ │        │ │          │           │
│ │ Metadata    │ │          │ │        │ │          │           │
│ │ Tooling     │ │          │ │        │ │          │           │
│ └─────────────┘ └──────────┘ └────────┘ └──────────┘           │
└─────────────────┬────────────┬───────────┬───────────────────────┘
                  │ Events    │ Events    │ Events
┌─────────────────▼────────────▼───────────▼───────────────────────┐
│ 🔄 INTEGRATION LAYER                                             │
│ Redpanda (Event Bus) │ OPA (Policy Eval) │ GitHub Actions │      │
│ Conformance Engine (reads STD, governs TGT)                       │
└─────────────────┬────────────────────────────────────────────────┘
                  │
┌─────────────────▼────────────────────────────────────────────────┐
│ 💾 DATA & INFRASTRUCTURE LAYER                                  │
│                                                                   │
│ PostgreSQL (d1, d2, d3, audit) │ Prisma ORM                      │
│ Keycloak (RBAC) │ MinIO (S3)                                     │
│ Weaviate (vector) │ Ollama (AI) │ OPA CLI │ tfsec/ArchUnit      │
└────────────────────────────────────────────────────────────────────┘
```

---

## 📦 Component Inventory

### Frontend Components (React)

| Component | Purpose | Dependencies | Outputs |
|-----------|---------|--------------|---------|
| **React App Shell** | Entry point, routing, state | Keycloak JS adapter | App bootstrap, user context |
| **SpecAuthoringPanel (D1A)** | Intent spec editor | Backend D1:Authoring API, spec linter | Spec state (drafted → valid) |
| **SpecViewer (D1A)** | Sign-off UI (LE approval) | Backend D1:Authoring API, approval endpoint | Spec status: signed_off + audit event |
| **Persona Surfaces** | PM/EA/LA/SA/LE/Risk/Ethics/Compliance UIs | Projection layer API (filtered DTOs) | Persona-scoped actions (approve/recommend/waive) |
| **API Client Library** | HTTP service, token refresh, error handling | Keycloak tokens, Backend API | Typed API calls, interceptors |
| **Component Library** | Design system, WCAG 2.1 AA | Storybook, design tokens | Reusable React components |

### Backend Modules (NestJS)

#### **D1: Architecture as Code & Continuous Alignment**

| Module | Capability | Primary Dependencies | Publishes Events |
|--------|-----------|---------------------|------------------|
| **D1A: Authoring** | ADR, model, spec, artefact CRUD | Postgres (d1), Prisma, MinIO | adr-drafted, spec-created, model-versioned |
| **D1B: Policies** | Policy CRUD, linting, OPA binding | Postgres (d1), OPA service, Rego files | policy-created, policy-validated |
| **D1C: Governance** | Approval workflow (SA→LA→EA), handover gate | Postgres (d1), Audit emitter, Keycloak, event bus | adr-recommended, adr-approved, handover-accepted |
| **D1D: Conformance** | Drift detection, ADR→impl tracing, score calc | Postgres (d1), STD catalogue reader, TGT scanner | drift-detected, conformance-checked |
| **D1E: Metadata** | Taxonomy, ADR tagging, policy-selector mapper | Postgres (d1), tag definition store | artefact-tagged, policy-selector-mapped |
| **D1F: Tooling** | Scaffold generator, CI hook orchestrator | GitHub API, scaffold templates, Jira API | scaffold-generated, ci-hook-registered |

#### **D2: Strategy & Capability Intelligence (Stub)**

| Module | Capability | Primary Dependencies | Publishes Events |
|--------|-----------|---------------------|------------------|
| **D2: Spec Capture** | Problem capture → Epic/Story generation | Postgres (d2), Jira API, problem parser | problem-captured, epic-created, story-created |

#### **D3: Risk, Ethics & Trust Architecture**

| Module | Capability | Primary Dependencies | Publishes Events |
|--------|-----------|---------------------|------------------|
| **D3A–D3B: Risk as Code** | Risk taxonomy, propagation, scoring, waivers | Postgres (d3), risk taxonomy store, score engine | risk-flagged, risk-scored, risk-waived |
| **D3C: Ethics as Code** | Ethics taxonomy binding, flag generation | Postgres (d3), ethics policy store | ethics-flagged, ethics-reviewed |
| **D3H: Privacy-as-Code** | Privacy constraints, consent, DPIA | Postgres (d3), privacy policy store | privacy-constraint-applied, dpia-generated |

#### **Platform/Core Services**

| Module | Purpose | Dependencies | Publishes Events |
|--------|---------|--------------|------------------|
| **Auth Service** | Keycloak integration, JWT validation, persona RBAC | Keycloak, JWT library, persona store | user-authenticated, permission-checked |
| **Personas/Projection** | Server-side surfacing (DTO filtering for business personas) | Auth service, DTO mappers | persona-projection-applied |
| **Audit Emitter** | Dual-write handler (artefact state + audit event) | Postgres (audit), Redpanda, Weaviate | **all audit events** (adr-transition, spec-signoff, waiver, etc.) |
| **Event Bus Handler** | Subscribes to all domain events, cross-domain orchestration | Redpanda, domain modules | orchestration-triggered |
| **Conformance Engine** | Reads STD policies, applies to TGT via CI | STD catalogue reader, OPA CLI runner, GitHub Actions | conformance-report, control-result |

---

## 🔌 Integration Points & Data Flows

### 1️⃣ ADR Authoring & Approval (D1C) — Dual-Write Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│ FLOW: ADR Lifecycle (SA Draft → LA Recommend → EA Approve)       │
└─────────────────────────────────────────────────────────────────┘

[1] SA Drafts ADR
    Frontend: SpecAuthoringPanel → POST /d1/adrs
             │
             ▼
    Backend D1:Authoring → Prisma: INSERT into d1.adrs (status: draft)
             │
             ├─→ Event: adr-drafted → Redpanda: valueops.d1.adr-drafted
             │
             └─→ Response: { id, status: draft, ... } → Frontend

[2] LA Reviews & Recommends (UX Action)
    Frontend: Approval button (LA persona only) → POST /d1/adrs/{id}/recommend
             │
             ▼
    Backend: Auth service verifies LA persona has "recommend" right
             │
             ├─→ YES: Proceed to dual-write
             │        WRITE 1: Postgres d1.adrs UPDATE status → recommend_by_LA
             │        WRITE 2: Postgres audit INSERT (eventId, actor, action, rationale, prevHash)
             │        ✓ Both succeed → 200 OK
             │        ✗ Partial → ROLLBACK → 500 Error
             │
             ├─→ Redpanda: Publish valueops.d1.adr-recommended
             │
             └─→ Event listener: Cross-domain services update their state
                 (e.g., D3:Risk may flag related risks)

[3] EA Approves (UX Action)
    Frontend: Approval button (EA persona only) → POST /d1/adrs/{id}/approve
             │
             ▼
    Backend: Auth service verifies EA persona has "approve" right
             │
             ├─→ YES: Proceed to dual-write
             │        WRITE 1: Postgres d1.adrs UPDATE status → approved_by_EA
             │        WRITE 2: Postgres audit INSERT + prevHash chain verification
             │        ✓ Both succeed → 200 OK
             │
             ├─→ Event: adr-approved → Redpanda
             │
             ├─→ GitHub Actions triggered (webhook on STD repo ADR-merge event)
             │   - Resolve applicable policies (from D1E:Metadata)
             │   - OPA CLI: conftest eval against policies
             │   - Check conformance vs TGT (tfsec, ArchUnit)
             │   - Publish result → Redpanda: valueops.d1.conformance-checked
             │
             └─→ Audit event: adr-approved appended to audit schema (hash-chained)

[4] LE Handover & Build
    Frontend: Handover action (LE persona) → POST /d1/handovers/{id}/accept
             │
             ▼
    Backend: Validates ADR coverage (no gap)
             │
             ├─→ Dual-write: handover accepted + audit event
             │
             └─→ Triggers build phase (scaffold generation from D1F:Tooling)
```

**Key Enforcement:**
- ✅ **Auth check before write:** Persona right verified FIRST
- ✅ **Dual-write atomicity:** Both artefact + audit succeed or BOTH fail
- ✅ **Immutable audit trail:** No UPDATE/DELETE path on audit schema
- ✅ **Event propagation:** Each state transition publishes for cross-domain listeners

---

### 2️⃣ Intent Spec Workflow (SDD v0, D1A) — Linting & Sign-Off

```
┌─────────────────────────────────────────────────────────────────┐
│ FLOW: Spec Creation → Validation → LE Sign-Off → Build          │
└─────────────────────────────────────────────────────────────────┘

[1] SA Creates Intent Spec
    Frontend: SpecAuthoringPanel → POST /d1/specs
             │
             ▼
    Linter (Frontend + Backend):
      - Problem, actors, outcome non-empty?
      - Behaviour, constraints, acceptance outcome-shaped? (no mechanism)
      - storyId unique + 1:1 with story?
      - No orphans at handover?
    ✓ Valid → status: valid
    ✗ Invalid → status: drafted (errors shown in UI)

[2] Spec Reaches "valid" State
    Postgres d1.specs: { id, status: valid, ... }
    Event: spec-validated → Redpanda
    Handover gate: spec must be "valid" before LE can sign off

[3] LE Sign-Off (Approval Action)
    Frontend: SpecViewer sign-off button (LE persona) → POST /d1/specs/{id}/sign-off
             │
             ▼
    Backend Auth: Verify LE persona has "sign_off" right
    Dual-write:
      WRITE 1: Postgres d1.specs UPDATE
        - status → signed_off
        - baselinedAt → ISO 8601 timestamp
        - baselineVersion → 1
      WRITE 2: Postgres audit INSERT
        - action: spec-signoff
        - actor: LE persona
        - artefactRef: spec-id
        - rationale: (optional)
        - prevHash: sha256(previous audit row)

[4] Handover Gate Check
    Coverage check:
      - All in-scope stories have ONE valid spec? YES
      - No orphans? YES
      - ADR coverage resolved (no "gap" state)? YES
    ✓ Gate passes → Spec marked "handover-ready"
    ✗ Gate fails → Handover blocked + error detail

[5] Build Phase
    Scaffold generator (D1F) reads:
      - Spec (problem, actors, outcome, constraints)
      - Referenced ADRs
      - Policies linked to ADRs
    Generates:
      - Service scaffold (NestJS module structure)
      - Prisma schema stubs
      - Policy/control templates
      - GitHub Actions workflow scaffolds
    Output → TGT repo as PR (governance-gated)

Event: spec-signed-off → Redpanda (D3 listeners may flag new ethics/privacy concerns)
```

**Key Enforcement:**
- ✅ **Outcome-shaped acceptance:** Linter rejects mechanism language (endpoint, field names, etc.)
- ✅ **1:1 storyId:** spec-linter enforces no duplicates
- ✅ **Gap blocking:** Handover cannot proceed if ADR coverage = gap
- ✅ **Linter gate:** Spec cannot reach "valid" unless all rules pass

---

### 3️⃣ Conformance & Drift Detection (D1D) — Policy Evaluation in CI

```
┌─────────────────────────────────────────────────────────────────┐
│ FLOW: TGT Commit → GitHub Actions → Policy Check → Audit        │
└─────────────────────────────────────────────────────────────────┘

[1] TGT Repo Commit (e.g., new Java service in Banking-Target)
    Webhook → GitHub Actions rule-validation pipeline triggered

[2] Pipeline: Resolve Applicable Policies
    Conformance engine:
      - Reads TGT artefacts (Terraform, Java code, ADRs)
      - Extracts metadata tags (from D1E:Metadata)
      - Policy-selector mapper (D1E) → applicable policy set
    Example:
      - Tag: domain=D1, usecase=api-gateway
      - Selector → policies: ["api-design-policy", "security-policy-001"]

[3] OPA CLI: Evaluate Design Artefacts
    $ conftest test <artefacts> \
        --policy /policies/api-design-policy.rego \
        --policy /policies/security-policy-001.rego
    
    Results: PASS / FAIL / WAIVER
    Output → GitHub comment + Audit event

[4] Build-Time Controls: tfsec, ArchUnit, Conftest
    tfsec: IaC scanning (Terraform/CloudFormation)
    ArchUnit: Java code rules (e.g., no cross-domain imports)
    Conftest: Config validation
    
    Results: PASS / FAIL
    Output → GitHub check + Audit event

[5] Drift Detection
    Conformance engine compares:
      - TGT implementation vs ADR intent
      - TGT vs reference architecture (from approved ADR)
    
    If divergence found:
      - Drift flagged
      - Traced to governing ADR
      - Event: drift-detected → Redpanda
      - Audit: drift record with ADR ref

[6] Merge Decision
    If all controls PASS:
      - ✓ Merge allowed
      - Audit: control-check-passed
    
    If control FAIL:
      - ✗ Merge blocked
      - Audit: control-check-failed
      - Risk Officer / EA may waive (mandatory rationale)
      - Audit: waiver-issued (dual-write artefact state + audit)

[7] Emit Audit Event (CI Emitter)
    GitHub Actions writes to audit trail:
    {
      eventId: uuid(),
      timestamp: ISO 8601,
      actor: "github-actions",
      action: "control-check",
      artefactRef: "TGT repo PR #123",
      decision: "pass" | "fail" | "waiver",
      policyRefs: ["api-design-policy", "security-policy-001"],
      commitSha: "abc123...",
      runId: "GHA run ID",
      rationale: "Risk Officer approved..." (if waiver),
      prevHash: sha256(prior audit row)
    }
    → Postgres audit schema INSERT
    → Redpanda: valueops.audit.event-emitted
```

**Key Enforcement:**
- ✅ **Deterministic policy selection:** Tags drive policy resolution (no manual selection)
- ✅ **Fail-closed:** Unresolvable artefacts fail the pipeline
- ✅ **Audit trail:** Every evaluation recorded with actor, decision, and chain hash
- ✅ **Waiver with rationale:** Human override requires mandatory explanation

---

### 4️⃣ Risk & Ethics Governance (D3) — Waiver & Attestation

```
┌─────────────────────────────────────────────────────────────────┐
│ FLOW: Risk Flag → Waiver → Compliance Attestation               │
└─────────────────────────────────────────────────────────────────┘

[1] Risk Flag (D3A–D3B)
    Risk Officer UI → POST /d3/risks
    {
      title: "High criticality database",
      severity: "high",
      mitigation: "Multi-AZ + backup",
      artefactRef: "ADR-0042"
    }
    
    Backend:
      - Postgres d3.risks INSERT
      - Event: risk-flagged → Redpanda
      - Audit: risk-creation-event

[2] Risk Waiver Action (Only Risk Officer persona)
    Frontend: Risk Dashboard → Waiver button → POST /d3/risks/{id}/waive
             │
             ▼
    Backend Auth: Verify Risk Officer persona + right to waive
    Dual-write:
      WRITE 1: Postgres d3.risks UPDATE
        - status → waived
        - waivedBy → Risk Officer name
        - waiverTimestamp → ISO 8601
      WRITE 2: Postgres audit INSERT
        - action: risk-waived
        - actor: Risk Officer
        - rationale: MANDATORY (reason for waiver)
        - prevHash: chain verification

[3] Ethics Flag & Review (D3C)
    Ethics system auto-flags based on policies
    Ethics Reviewer UI → Review action → POST /d3/ethics/{id}/review
    
    Dual-write (same pattern):
      - Postgres d3.ethics status update
      - Audit trail + mandatory rationale for approve/reject

[4] Compliance Attestation (D3C + D3H)
    Compliance Lead → Attest action → POST /d3/compliance/attest
    Dual-write:
      - Postgres d3 attestation record
      - Audit trail

[5] Export Evidence
    Compliance Lead → Export → /d3/compliance/export?format=iso31000
    Backend reads:
      - Audit trail (all risk/ethics events)
      - Policies applied + evidence
      - Waivers + rationale
    Output: PDF/JSON aligned to ISO 31000, NIST RMF, FCA, ICO, EU AI Act
```

**Key Enforcement:**
- ✅ **Persona-scoped actions:** Only Risk Officer can waive risks; only Compliance Lead can attest
- ✅ **Mandatory rationale:** Waiver/override without explanation rejected at API level
- ✅ **Audit immutability:** All decisions recorded hash-chained; no revision history loss
- ✅ **Plain business language:** Risk/Ethics/Compliance surfaces never surface policy code or metadata

---

### 5️⃣ Persona Surfacing (Projection Layer) — Server-Side Enforcement

```
┌─────────────────────────────────────────────────────────────────┐
│ FLOW: User Login → Projection → Filtered DTO → Frontend         │
└─────────────────────────────────────────────────────────────────┘

[1] User Login
    Frontend: POST /auth/login (username, password)
             │
             ▼
    Backend: Keycloak authentication
      - realm: helio
      - Validate credentials via LDAP/OAuth/etc.
      - Return JWT token with persona role

[2] Persona Detection
    JWT payload: { sub, email, realm_access: { roles: ["risk-officer"] } }
    Auth service maps role → persona (e.g., role "risk-officer" → persona "Risk Officer")

[3] Persona-Scoped Request
    Frontend: GET /d1/adrs (with Authorization: Bearer <JWT>)
             │
             ▼
    Backend: Request interceptor
      1. Verify JWT signature + expiry
      2. Introspect with Keycloak (optional re-validation)
      3. Extract persona from token
      4. Route through personas/projection layer

[4] Projection Layer DTO Filtering
    Raw response (internal DTO):
    {
      id: "ADR-001",
      title: "API Gateway Design",
      decision: "Use Kong",
      policies: [
        { id: "policy-1", body: "if apiGateway then requireAAAA..." }
      ],
      metadata: {
        createdBy: "alice",
        tags: ["D1", "api", "critical"],
        policySelector: "api-design-selector"
      }
    }

    Persona: "Product Manager"
    Projection applies DTO mapper → includes only:
    {
      id: "ADR-001",
      title: "API Gateway Design",
      decision: "Use Kong",
      readiness: "approved"
      // Excluded: policies, metadata, policySelector
    }

    Persona: "Enterprise Architect"
    Projection applies DTO mapper → includes:
    {
      id: "ADR-001",
      title: "API Gateway Design",
      decision: "Use Kong",
      decision: "Use Kong",
      policies: [ { id, body } ],  // ← FULL POLICY INCLUDED
      metadata: { ... },            // ← FULL METADATA
      readiness: "approved"
    }

[5] Frontend Receives Filtered Response
    PM frontend: { id, title, decision, readiness }
    Cannot display policy or metadata fields (they don't exist in response)
    
    EA frontend: Full details + policy + metadata

[6] Governance Test
    Test: PM token cannot retrieve ADR.policies or ADR.metadata
    if (pm_token && response.includes(policies || metadata)) {
      FAIL = Surfacing leak detected
    }
```

**Key Enforcement:**
- ✅ **Server-side only:** Filtering at API boundary, not in React
- ✅ **DTO construction:** PM, Risk Officer, Ethics Reviewer, Compliance Lead DTOs exclude policy/metadata
- ✅ **Test gate:** Red test blocks merge if surfacing boundary violated
- ✅ **Impossible to bypass:** PM token physically cannot request policy fields (not in DTO)

---

## 🔄 Event Schema & Topics

### Topic Convention
```
valueops.<domain>.<event>
  valueops.d1.adr-drafted
  valueops.d1.adr-recommended
  valueops.d1.adr-approved
  valueops.d1.spec-created
  valueops.d1.spec-validated
  valueops.d1.spec-signed-off
  valueops.d1.handover-accepted
  valueops.d1.conformance-checked
  valueops.d1.drift-detected

  valueops.d3.risk-flagged
  valueops.d3.risk-scored
  valueops.d3.risk-waived
  valueops.d3.ethics-flagged
  valueops.d3.ethics-reviewed
  valueops.d3.compliance-attested

  valueops.audit.event-emitted
```

### Event Schema (Uniform)
```typescript
interface AuditEvent {
  eventId: string;                    // UUID
  timestamp: string;                  // ISO 8601
  actor: string;                      // User or "github-actions"
  persona?: string;                   // "Enterprise Architect", "Risk Officer", etc.
  action: string;                     // "adr-transition" | "spec-signoff" | "risk-waived" | "control-check" | ...
  artefactRef: string;                // ADR-001, spec-42, risk-12, etc.
  artefactType: string;               // "adr" | "spec" | "risk" | "control-result" | ...
  storyId?: string;                   // Jira story, if applicable
  adrRefs: string[];                  // ADRs this event touches
  decision: string;                   // "pass" | "fail" | "waiver" | "approve" | "recommend" | ...
  rationale?: string;                 // REQUIRED for waiver/override
  policyRefs: string[];               // Policies evaluated (if control-check)
  commitSha?: string;                 // Git commit SHA (if CI)
  runId?: string;                     // GitHub Actions run ID
  prevHash: string;                   // SHA256 of previous audit row (chain link)
}
```

---

## 🗄️ Database Schema Isolation

```
┌─────────────────────────────────────────────────────────────────┐
│ PostgreSQL (valueops DB)                                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│ SCHEMA: d1 (Architecture as Code)                               │
│ ├─ adrs (id PK, title, context, decision, alternatives, ...)   │
│ ├─ policies (id, body, status, adrRef, createdAt, ...)         │
│ ├─ controls (id, type, config, policyRef, ...)                 │
│ ├─ artefacts (id, type, path, version, minioRef, ...)          │
│ ├─ metadata_tags (id, artefactRef, tag, ...)                   │
│ └─ [other D1 tables]                                             │
│ ⚠️ NO FOREIGN KEYS to d2, d3, audit                              │
│                                                                   │
│ SCHEMA: d2 (Strategy & Capability — Stub)                       │
│ ├─ problems (id, statement, audience, value, ...)              │
│ ├─ epics (id, problemRef, title, ...)                           │
│ └─ [other D2 tables]                                             │
│ ⚠️ NO FOREIGN KEYS to d1, d3, audit                              │
│                                                                   │
│ SCHEMA: d3 (Risk, Ethics, Privacy)                              │
│ ├─ risks (id, title, severity, artefactRef, status, ...)       │
│ ├─ ethics_policies (id, taxonomy, binding, ...)                │
│ ├─ privacy_constraints (id, legalBasis, ...)                   │
│ └─ [other D3 tables]                                             │
│ ⚠️ NO FOREIGN KEYS to d1, d2, audit                              │
│                                                                   │
│ SCHEMA: audit (Immutable Append-Only)                           │
│ ├─ events (id PK, eventId, timestamp, action, actor,           │
│ │           artefactRef, decision, rationale, prevHash, ...)   │
│ └─ ⚠️ WRITER ROLE: INSERT ONLY (no UPDATE/DELETE)               │
│                                                                   │
│ SCHEMA: d4, d5 (Reserved, not populated in MVP+R1)              │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘

Cross-Domain Communication Pattern:
  d1 domain publishes event → Redpanda topic
  d3 domain subscribes to event
  d3 module receives event, applies logic, updates d3 schema
  d3 publishes response event
  
  NO direct foreign key across schemas
```

---

## 🔐 Security & Boundary Enforcement

| Boundary | Enforcement | Test |
|----------|------------|------|
| **Persona Surfacing** | DTO projection layer (server-side) | PM token cannot GET policy.body |
| **Approval Authority** | Auth service checks persona rights BEFORE write | Only EA can approve ADRs; only Risk Officer can waive |
| **Dual-Write Atomicity** | Transaction wrapping (both writes or both fail) | Audit record without artefact state change = FAIL |
| **Audit Immutability** | INSERT-only role on audit schema (no UPDATE/DELETE) | SELECT pg_roles; audit role lacks UPDATE/DELETE privs |
| **Cross-Schema Isolation** | No direct FKs; events bridge | D1 domain cannot directly query d3 tables |
| **Spec Linting** | Mechanism-language rejection | Acceptance criterion with "/endpoint" → linter fails |
| **ADR Coverage** | Handover gate check (no "gap" state) | Spec in "gap" → handover blocked |
| **Spec Uniqueness** | Unique constraint on (storyId) | Duplicate storyId INSERT → constraint violation |

---

## 🚀 Deployment Architecture (Docker Compose)

```yaml
services:
  # Frontend (React SPA)
  app-frontend:
    image: node:22-alpine
    build:
      context: ./apps/frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      REACT_APP_API_URL: http://localhost:3001
      REACT_APP_KEYCLOAK_URL: http://localhost:8080
    depends_on:
      - app-backend

  # Backend (NestJS)
  app-backend:
    image: node:22-alpine
    build:
      context: ./apps/backend
      dockerfile: Dockerfile
    ports:
      - "3001:3001"
    environment:
      DATABASE_URL: postgresql://user:password@postgres:5432/valueops
      KEYCLOAK_URL: http://keycloak:8080
      REDPANDA_BROKERS: redpanda:9092
      OPA_URL: http://opa:8181
      OLLAMA_BASE_URL: http://ollama:11434
      WEAVIATE_URL: http://weaviate:8080
    depends_on:
      - postgres
      - keycloak
      - redpanda
      - opa
      - ollama
      - weaviate
      - minio

  # Database
  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: valueops
      POSTGRES_USER: valueops_user
      POSTGRES_PASSWORD: ***
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-audit-schema.sql:/docker-entrypoint-initdb.d/001-init-audit-schema.sql

  # Auth
  keycloak:
    image: quay.io/keycloak/keycloak:latest
    ports:
      - "8080:8080"
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: ***
    volumes:
      - ./config/helio-realm-export.json:/opt/keycloak/helio-realm-export.json

  # Event Broker
  redpanda:
    image: docker.redpanda.com/redpanda:latest
    ports:
      - "9092:9092"
      - "8081:8081"
    volumes:
      - redpanda_data:/var/lib/redpanda/data

  # Policy Engine
  opa:
    image: openpolicyagent/opa:latest
    ports:
      - "8181:8181"
    volumes:
      - ./policies:/policies
    command: run --server --set system.authz=allow /policies

  # Vector Store
  weaviate:
    image: semitechnologies/weaviate:latest
    ports:
      - "8082:8080"
    environment:
      QUERY_DEFAULTS_LIMIT: 25

  # AI Runtime
  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    platform: linux/amd64  # Apple Silicon compatibility

  # Object Store
  minio:
    image: minio/minio:latest
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data:/minio
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: ***

volumes:
  postgres_data:
  redpanda_data:
  ollama_data:
  minio_data:
```

---

## 📊 Request Flow Example: Creating & Approving an ADR

```
USER (Solution Architect) → FRONTEND (React)
│
├─→ POST /d1/adrs (Create ADR)
│   {
│     title: "Microservices vs Monolith",
│     context: "Banking workload scaling needs",
│     decision: "Microservices with domain boundaries",
│     alternatives: ["Monolith (improved)", "Strangler Fig"],
│     consequences: "Team structure, deployment complexity, ..."
│   }
│
└─→ BACKEND (NestJS D1:Authoring)
    │
    ├─→ Auth: Verify JWT token + SA persona
    │
    ├─→ Validation: Context, decision non-empty? ✓
    │
    ├─→ Prisma ORM: INSERT into d1.adrs (status: draft)
    │   sql: INSERT INTO d1.adrs (...) VALUES (...) RETURNING id
    │   result: ADR-001
    │
    ├─→ MinIO: Store PlantUML/detailed content (versioned)
    │
    ├─→ Redpanda: Publish event
    │   topic: valueops.d1.adr-drafted
    │   payload: { id: ADR-001, actor: sa@valueops, ... }
    │
    └─→ Frontend: Response { id: ADR-001, status: draft }


[LATER] USER (Lead Architect) → FRONTEND (React)
│
├─→ GET /d1/adrs/ADR-001
│   (Fetches ADR detail)
│
└─→ BACKEND
    │
    ├─→ Prisma ORM: SELECT * FROM d1.adrs WHERE id = ADR-001
    │
    ├─→ Response: Full ADR detail (policies linked, metadata, ...)


[LATER] USER (Lead Architect) → Approval Action
│
├─→ POST /d1/adrs/ADR-001/recommend
│   {
│     rationale: "Aligns with our domain-driven strategy. Recommend for EA review."
│   }
│
└─→ BACKEND (NestJS D1:Governance)
    │
    ├─→ Auth: Verify JWT + LA persona + "recommend" right
    │
    ├─→ DUAL-WRITE:
    │   
    │   TRANSACTION BEGIN
    │   │
    │   ├─ WRITE 1: Prisma ORM
    │   │  UPDATE d1.adrs
    │   │  SET status = 'recommend_by_LA',
    │   │      recommendedBy = 'la@valueops',
    │   │      recommendedAt = now()
    │   │  WHERE id = ADR-001
    │   │
    │   ├─ WRITE 2: Prisma ORM
    │   │  INSERT INTO audit.events (
    │   │    eventId, timestamp, actor, action,
    │   │    artefactRef, artefactType, decision,
    │   │    rationale, prevHash, ...
    │   │  ) VALUES (
    │   │    uuid(), now(), 'la@valueops', 'adr-transition',
    │   │    'ADR-001', 'adr', 'recommend',
    │   │    'Aligns with domain-driven...', sha256(prior), ...
    │   │  )
    │   │
    │   ├─ TRANSACTION COMMIT ✓
    │   │
    │   └─ On PARTIAL failure: ROLLBACK both
    │
    ├─→ Redpanda: Publish event
    │   topic: valueops.d1.adr-recommended
    │   payload: { id: ADR-001, actor: la@valueops, ... }
    │
    └─→ Frontend: Response { status: recommend_by_LA }


[LATER] USER (Enterprise Architect) → Approval Action
│
├─→ POST /d1/adrs/ADR-001/approve
│   {
│     rationale: "Approved. Aligns with enterprise architecture principles."
│   }
│
└─→ BACKEND (NestJS D1:Governance)
    │
    ├─→ Auth: Verify JWT + EA persona + "approve" right
    │
    ├─→ DUAL-WRITE (same pattern)
    │
    ├─→ Redpanda: valueops.d1.adr-approved
    │
    ├─→ GitHub Actions triggered (webhook on STD repo)
    │   - Resolve applicable policies (from D1E)
    │   - OPA: conftest eval
    │   - tfsec/ArchUnit: Build controls
    │   - Emit: control-check event → Redpanda
    │
    └─→ Frontend: Response { status: approved_by_EA }
```

---

## ✅ Definition of Done (Components)

A component is production-ready when:

- [ ] Module created in NestJS (if backend) or React (if frontend)
- [ ] All dependencies documented in this map
- [ ] Integration tests pass (component + dependencies mocked or real)
- [ ] Governance gates pass (spec linter, persona surfacing, audit dual-write, etc.)
- [ ] Event publishing (if cross-domain) tested
- [ ] API contract versioned (OpenAPI if applicable)
- [ ] Error handling & logging implemented
- [ ] Observability (correlation IDs, structured logs) in place
- [ ] Security: Auth, RBAC, input validation, no PII in logs
- [ ] Documentation: README + API docs updated

---

## 📞 Cross-Component Communication Patterns

| Pattern | Use Case | Example |
|---------|----------|---------|
| **HTTP Sync** | Frontend ↔ Backend, Backend ↔ Keycloak | POST /d1/adrs/approve |
| **Event Async** | D1 → D3 (risk flagging), CI → Audit | adr-approved event triggers D3 risk eval |
| **Event + Sync** | CI gate → D1 drift detector → dashboard | GitHub Actions polls conformance API |
| **DB Read** | Shared reference data (e.g., policy selector) | D1:Metadata reads tag definitions from d1 schema |
| **OPA Eval** | Policy decision (in-app or CI) | Backend ↔ OPA service (HTTP) or CI runner ↔ OPA CLI |

---

## 🎯 Next Steps for Implementation

1. **Scaffold NestJS modules** for D1A, D1B, D1C, etc. (see `src/d1/`, `src/d2/`, `src/d3/`)
2. **Scaffold React surfaces** for each persona (PM, EA, LA, SA, LE, Risk, Ethics, Compliance)
3. **Implement Postgres schemas** (d1, d2, d3, audit) + Prisma migrations
4. **Wire up Keycloak** (helio realm, role mapping, JWT validation)
5. **Implement Redpanda** event topics + subscribers (cross-domain orchestration)
6. **Implement OPA** integration (CLI in CI, service for runtime)
7. **Implement audit emitter** (dual-write pattern, hash-chaining)
8. **Implement personas/projection** (server-side DTO filtering)
9. **Wire up GitHub Actions** (rule-validation pipeline, conformance checks)
10. **Add governance gate tests** (persona surfacing, approval authority, audit immutability, spec linting)

---

**End of Component Dependency Map**

Generated: 2026-07-21 | Scope: MVP + R1 | Architecture as Code Platform
