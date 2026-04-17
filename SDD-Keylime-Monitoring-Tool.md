# Software Design Description: Keylime Monitoring Dashboard

* **Companion SRS:** `spec/SRS-Keylime-Monitoring-Tool.md`
* **Standard:** IEEE 1016-2009 (ISO/IEC/IEEE 42010 compatible)
* **Initial Date:** 2026-04-15
* **Methodology:** Spec-Driven Development (SDD)

---

## 1. Introduction

### 1.1 Purpose

This Software Design Description (SDD) documents the *how* of the Keylime Monitoring Dashboard. It complements the SRS (which documents the *what*) by describing the architectural decisions, component decomposition, data models, API contracts, interaction patterns, state machines, algorithms, and deployment topology that realize the SRS requirements.

### 1.2 Scope

The SDD covers the full system: a React.js + TypeScript single-page application frontend, a Rust (Axum) asynchronous backend, and their integration with Keylime Verifier/Registrar APIs, TimescaleDB, and Redis.

### 1.3 Definitions

| Term | Definition |
|------|-----------|
| SPA | Single Page Application |
| mTLS | Mutual Transport Layer Security |
| OIDC | OpenID Connect |
| RBAC | Role-Based Access Control |
| KPI | Key Performance Indicator |
| TPM | Trusted Platform Module |
| PCR | Platform Configuration Register |
| IMA | Integrity Measurement Architecture |
| SRS | Software Requirements Specification |
| SDD | Software Design Description |

### 1.4 Design Stakeholders and Concerns

| Stakeholder | Concern |
|-------------|---------|
| Backend Developer | Component decomposition, API contracts, data flow, error handling |
| Frontend Developer | UI component hierarchy, state management, API integration |
| Security Architect | Trust boundaries, authentication flow, mTLS topology, RBAC enforcement |
| DevOps Engineer | Deployment topology, configuration, container images, health checks |
| QA Engineer | Testable interfaces, state machines, deterministic algorithms |

### 1.5 SRS Cross-Reference

This SDD traces design elements to the SRS via `(FR-nnn)`, `(NFR-nnn)`, and `(SR-nnn)` references. Every SRS requirement with an implementation MUST be traceable to at least one design element in this document.

---

## 2. Design Viewpoints

This SDD uses the following IEEE 1016 viewpoints:

| Viewpoint | Section | Purpose |
|-----------|---------|---------|
| Context | 3.1 | System boundaries and external interfaces |
| Composition | 3.2 | Major components and their responsibilities |
| Logical | 3.3 | Data models, type hierarchies, domain entities |
| Interface | 3.4 | API contracts, response formats, WebSocket protocol |
| Interaction | 3.5 | Component communication, data flow sequences |
| State Dynamics | 3.6 | State machines, lifecycle transitions |
| Algorithm | 3.7 | Key algorithms and computation strategies |
| Resource | 3.8 | Deployment topology, infrastructure, configuration |

---

## 3. Design Views

### 3.1 Context View

#### 3.1.1 System Boundary

```text
                           +---------------------------+
                           |   Identity Provider       |
                           |   (OIDC/SAML)             |
                           +------------+--------------+
                                        |
                                        | OIDC flow (SR-001)
                                        v
+----------------+    TLS 1.3     +-----+-------+    mTLS      +-----------------+
|   Browser      | <-----------> |   Backend    | <----------> | Keylime         |
|   (React SPA)  |   (SR-008)    |   (Rust/Axum)|  (SR-004)    | Verifier API v2 |
+----------------+               +------+-------+              +-----------------+
       |                                |
       | WebSocket (NFR-021)            | mTLS (SR-009)
       |                                v
       +-----------------------------+ +-----------------+
                                     | | Keylime         |
                                     | | Registrar API   |
              +----------+          | +-----------------+
              |TimescaleDB| <-------+
              +----------+          |
              +----------+          |
              |  Redis   | <--------+
              +----------+
```

#### 3.1.2 External Interfaces

| Interface | Protocol | Authentication | SRS Trace |
|-----------|----------|---------------|-----------|
| Keylime Verifier API | HTTPS (mTLS) | Client certificate | SR-004, NFR-002 |
| Keylime Registrar API | HTTPS (mTLS) | Client certificate | SR-004, NFR-002 |
| Identity Provider | HTTPS | OIDC Authorization Code | SR-001 |
| Browser | HTTPS (TLS 1.3) | JWT Bearer token | SR-008, SR-010 |
| WebSocket | WSS | JWT query parameter | NFR-021 |
| TimescaleDB | TCP | Connection string | NFR-005 |
| Redis | TCP | Connection string | NFR-019 |
| SIEM (Syslog/Splunk/ECS) | TCP/HTTPS | API token | FR-063 |

### 3.2 Composition View

#### 3.2.1 Backend Components

```text
keylime-webtool-backend/src/
+-- main.rs                    Application entry point, server bootstrap, config loading
+-- lib.rs                     Crate-level exports
+-- state.rs                   Shared application state (AppState)
+-- config.rs                  Hierarchical configuration loading
+-- settings_store.rs          TOML config file persistence (FR-075)
+-- error.rs                   Centralized error handling (AppError)
+-- api/
|   +-- routes.rs              Route hierarchy and nesting
|   +-- response.rs            Standard API response envelope
|   +-- ws.rs                  WebSocket endpoint handler
|   +-- handlers/
|       +-- kpis.rs            Fleet KPI aggregation (FR-001)
|       +-- agents.rs          Agent CRUD and detail (FR-012..FR-023)
|       +-- attestations.rs    Attestation analytics (FR-024..FR-032)
|       +-- policies.rs        Policy management (FR-033..FR-039)
|       +-- certificates.rs    Certificate lifecycle (FR-050..FR-053)
|       +-- alerts.rs          Alert lifecycle (FR-047..FR-049)
|       +-- audit.rs           Audit log queries (FR-042..FR-044)
|       +-- compliance.rs      Compliance reporting (FR-059..FR-060)
|       +-- integrations.rs    Backend connectivity with probes (FR-057..FR-058, FR-077)
|       +-- performance.rs     System metrics (FR-064..FR-068)
|       +-- settings.rs        Runtime URL and mTLS config (FR-072..FR-074)
|       +-- auth.rs            Authentication endpoints (SR-001)
+-- keylime/
|   +-- client.rs              Keylime API client with circuit breaker and probes
|   +-- models.rs              Keylime API response types (#[serde(default)])
+-- auth/
|   +-- jwt.rs                 JWT encode/decode (SR-010)
|   +-- oidc.rs                OIDC client configuration (SR-001)
|   +-- rbac.rs                Role and permission model (SR-003)
|   +-- session.rs             Server-side session revocation (SR-011)
+-- audit/
|   +-- logger.rs              Hash-chained audit entries (FR-061, SR-015)
+-- storage/
|   +-- db.rs                  TimescaleDB connection pool
|   +-- cache.rs               Redis cache with tiered TTLs (NFR-019)
+-- models/
    +-- agent.rs               Agent domain model and state enum
    +-- attestation.rs         Attestation, pipeline, failure models
    +-- policy.rs              Policy and approval workflow models
    +-- certificate.rs         Certificate and expiry models
    +-- alert.rs               Alert, severity, notification models
    +-- alert_store.rs         In-memory alert store with lifecycle
    +-- kpi.rs                 KPI and summary aggregation types
```

**Trace:** Implementation -- `keylime-webtool-backend/src/`

#### 3.2.2 Frontend Components

```text
keylime-webtool-frontend/src/
+-- main.tsx                   React DOM root, strict mode
+-- App.tsx                    QueryClient + RouterProvider, auto-refresh wiring
+-- router.tsx                 Route definitions with Layout wrapper
+-- index.css                  Global styles, CSS variables, dark/light theming
+-- api/
|   +-- client.ts              Axios instance, interceptors, Vite proxy, backend URL config
|   +-- agents.ts              Agent API methods
|   +-- attestations.ts        Attestation API methods
|   +-- policies.ts            Policy API methods
|   +-- certificates.ts        Certificate API methods
|   +-- alerts.ts              Alert API methods
|   +-- audit.ts               Audit log API methods
|   +-- performance.ts         Performance API methods
|   +-- compliance.ts          Compliance API methods
|   +-- settings.ts            Settings API methods (FR-072, FR-073)
+-- components/
|   +-- Layout/
|   |   +-- Layout.tsx         Two-column layout, collapsible sidebar (FR-076)
|   |   +-- Layout.css         Sidebar slide transition styles
|   |   +-- Sidebar.tsx        Navigation with 10 modules, integration health indicator (FR-081)
|   |   +-- TopBar.tsx         Hamburger toggle, search, theme toggle, user menu
|   +-- common/
|       +-- DataTable.tsx      Generic sortable, selectable table
|       +-- KpiCard.tsx        Metric card with variant styling
|       +-- StatusBadge.tsx    Color-coded status label
|       +-- AgentStateChart.tsx  Recharts pie chart for agent states
+-- hooks/
|   +-- useAuth.ts             Authentication hook (wraps authStore)
|   +-- useWebSocket.ts        WebSocket hook with reconnection
+-- store/
|   +-- authStore.ts           Zustand auth state (SR-003)
|   +-- visualizationStore.ts  Zustand UI preferences (FR-008), auto-refresh settings
+-- types/
|   +-- index.ts               Barrel exports, shared types (IntegrationService.endpoint)
|   +-- agent.ts               Agent domain types (includes start, saved states)
|   +-- attestation.ts         Attestation domain types
|   +-- policy.ts              Policy domain types
|   +-- certificate.ts         Certificate domain types
|   +-- alert.ts               Alert domain types
|   +-- audit.ts               Audit domain types
+-- pages/
    +-- Dashboard/             Fleet overview with KPIs, charts, fallback KPIs (FR-001)
    +-- Agents/                Agent list with error state, detail with timeline (FR-012..FR-023)
    +-- Attestations/          Attestation analytics (FR-024..FR-032)
    +-- Policies/              Policy management with linked agent counts (FR-033..FR-039)
    +-- Certificates/          Certificate lifecycle (FR-050..FR-053)
    +-- Alerts/                Alert dashboard (FR-047..FR-049)
    +-- Performance/           System metrics (FR-064..FR-068)
    +-- AuditLog/              Audit trail with hash verification (FR-042)
    +-- Integrations/          Backend + core services status with 1s polling (FR-057, FR-077)
    +-- Settings/              Connection URLs, mTLS certs, mock/prod toggle (FR-072..FR-074)
    +-- Login/                 Authentication entry point (SR-001)
```

**Trace:** Implementation -- `keylime-webtool-frontend/src/`

#### 3.2.3 Technology Stack

| Layer | Technology | Version | SRS Trace |
|-------|-----------|---------|-----------|
| Frontend Framework | React | 18.3.1 | NFR-004 |
| Frontend Language | TypeScript | 5.6 | NFR-004 |
| Frontend Build | Vite | 6.0 | NFR-004 |
| Frontend Routing | React Router | 6.26 | NFR-004 |
| Frontend State | Zustand | 5.0 | — |
| Frontend Data Fetching | TanStack React Query | 5.60 | — |
| Frontend HTTP Client | Axios | 1.7 | — |
| Frontend Charts | Recharts | 2.13 | — |
| Backend Framework | Axum | 0.8 | NFR-005 |
| Backend Language | Rust | 1.75+ | SR-023 |
| Backend Runtime | Tokio | 1.x | NFR-005 |
| Backend TLS | rustls | 0.23 | SR-004 |
| Backend Database | sqlx (PostgreSQL) | 0.8 | — |
| Backend Cache | redis | 0.27 | NFR-019 |
| Backend Auth | openidconnect + jsonwebtoken | 4 / 9 | SR-001, SR-010 |
| Backend HTTP Client | reqwest (rustls-tls) | 0.12 | SR-004 |
| Backend Observability | tracing + OpenTelemetry | 0.1 / 0.27 | — |
| Database | TimescaleDB (PostgreSQL) | — | — |
| Cache | Redis | — | NFR-019 |

### 3.3 Logical View

#### 3.3.1 Backend Application State

```rust
pub struct AppState {
    keylime_client: Arc<RwLock<Arc<KeylimeClient>>>,  // Hot-swappable Keylime API client (FR-072)
    pub alert_store: Arc<AlertStore>,                  // In-memory alert management
    pub settings_store: Arc<SettingsStore>,             // TOML config persistence (FR-075)
}
```

The `KeylimeClient` is wrapped in `Arc<RwLock<Arc<KeylimeClient>>>` to support hot-swapping at runtime: the outer `Arc` is shared across handlers, `RwLock` allows atomic replacement, and the inner `Arc` lets in-flight requests complete with the old client while new requests use the updated one. Handlers access the client via a `keylime()` accessor method. `SettingsStore` persists configuration changes to a TOML file on disk. `AppState` is cloned into every Axum handler via `State<AppState>` extractor.

**Trace:** Implementation -- `keylime-webtool-backend/src/state.rs`

#### 3.3.2 Agent Data Model

**Full Agent Entity** (detail view, merges Verifier + Registrar data):

| Field | Type | Source | SRS Trace |
|-------|------|--------|-----------|
| `id` | UUID | Verifier `agent_id` | FR-012 |
| `ip` | string | Verifier | FR-012 |
| `hostname` | string? | Derived | FR-004 |
| `state` | AgentState | Verifier `operational_state` | FR-069 |
| `attestation_mode` | Pull \| Push | Verifier `accept_attestations` | FR-054, FR-055 |
| `verifier_id` | string | Derived | FR-064 |
| `registration_date` | datetime | Registrar | FR-012 |
| `last_attestation` | datetime? | Verifier | FR-012 |
| `consecutive_failures` | integer | Verifier | FR-012 |
| `total_attestations` | integer | Verifier `attestation_count` | FR-029 |
| `boot_time` | datetime? | Verifier | FR-018 |
| `hash_algorithm` | string | Verifier `hash_alg` | FR-021 |
| `encryption_algorithm` | string | Verifier `enc_alg` | FR-018 |
| `signing_algorithm` | string | Verifier `sign_alg` | FR-018 |
| `ima_pcrs` | integer[] | Verifier | FR-021 |
| `ima_policy_id` | string? | Verifier `ima_policy` | FR-033 |
| `mb_policy_id` | string? | Verifier `mb_policy` | FR-036 |
| `tpm_policy` | string? | Verifier | FR-018 |
| `regcount` | integer | Registrar | SR-025 |

**Agent Summary** (list view projection):

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Agent identifier |
| `ip` | string | IP address |
| `state` | AgentState | Current operational state |
| `attestation_mode` | Pull \| Push | API version mode |
| `last_attestation` | datetime? | Most recent attestation |
| `assigned_policy` | string? | IMA policy name |
| `mb_policy` | string? | Measured boot policy name |
| `failure_count` | integer | Consecutive failure count |

**Trace:** Implementation -- `keylime-webtool-backend/src/models/agent.rs`

#### 3.3.3 Agent State Enumeration

**Pull-Mode States** (Keylime v2 API, `operational_state` integer):

> **Note:** States `Start` (1) and `Saved` (2) are transient operational states recognized by both the backend and frontend. The backend uses `#[serde(default)]` on Keylime API model fields to tolerate missing or renamed fields across Keylime versions without breaking deserialization.

| State | Value | Description |
|-------|-------|-------------|
| `Registered` | 0 | Agent registered, not yet attesting |
| `Start` | 1 | Attestation process initiated |
| `Saved` | 2 | Quote saved for processing |
| `GetQuote` | 3 | Actively attesting (healthy) |
| `Retry` | 4 | Retrying after transient failure |
| `ProvideV` | 5 | Providing V key to agent |
| `Failed` | 7 | Attestation failed |
| `Terminated` | 8 | Agent terminated by operator |
| `InvalidQuote` | 9 | TPM quote validation failed |
| `TenantFailed` | 10 | Tenant-initiated failure |

**Push-Mode States** (Keylime v3 API, derived from `accept_attestations` flag):

| State | Value | Description |
|-------|-------|-------------|
| `Pass` | 100 | Attestation passing |
| `Fail` | 101 | Attestation failing |
| `Pending` | 102 | Awaiting first attestation |

**Trace:** Implementation -- `keylime-webtool-backend/src/models/agent.rs`, SRS FR-069

#### 3.3.4 Alert Data Model

| Field | Type | Description | SRS Trace |
|-------|------|-------------|-----------|
| `id` | UUID | Alert identifier | FR-047 |
| `type` | AlertType | Category (see table below) | FR-025 |
| `severity` | AlertSeverity | `critical` \| `warning` \| `info` | FR-025 |
| `description` | string | Human-readable description | FR-047 |
| `affected_agents` | string[] | List of affected agent IDs | FR-047 |
| `state` | AlertState | Lifecycle state (see 3.6.2) | FR-047 |
| `created_timestamp` | datetime | Alert creation time | FR-047 |
| `acknowledged_timestamp` | datetime? | When acknowledged | FR-047 |
| `assigned_to` | string? | Investigator email | FR-047 |
| `investigation_notes` | string? | Investigation details | FR-047 |
| `root_cause` | string? | Identified root cause | FR-027 |
| `resolution` | string? | Resolution description | FR-047 |
| `auto_resolved` | boolean | Auto-resolved flag | FR-049 |
| `escalation_count` | integer | Escalation counter | FR-048 |
| `sla_window` | string? | SLA timeout window | FR-048 |
| `source` | string | Alert source system | FR-047 |
| `external_ticket_id` | string? | External ticket reference | FR-062 |

**Alert Types:**

| Type | Identifier | Description |
|------|-----------|-------------|
| Attestation Failure | `attestation_failure` | Quote verification or policy check failure |
| Certificate Expiry | `cert_expiry` | Certificate approaching or past expiry |
| Policy Violation | `policy_violation` | IMA or measured boot policy violation |
| PCR Change | `pcr_change` | Unexpected PCR value change detected |
| Service Down | `service_down` | Backend service unreachable |
| Rate Limit | `rate_limit` | Rate limiting threshold exceeded |
| Clock Skew | `clock_skew` | Time drift between agent and verifier |

**Trace:** Implementation -- `keylime-webtool-backend/src/models/alert.rs`

#### 3.3.5 Certificate Data Model

| Field | Type | Description | SRS Trace |
|-------|------|-------------|-----------|
| `id` | UUID | Deterministic (UUID v5 from agent_id + cert_type) | FR-050 |
| `cert_type` | CertificateType | `EK` \| `AK` \| `IAK` \| `I_DEV_ID` \| `M_TLS` \| `SERVER` | FR-050 |
| `subject_dn` | string | X.509 Subject DN | FR-052 |
| `issuer_dn` | string | X.509 Issuer DN | FR-052 |
| `serial_number` | string | Certificate serial | FR-052 |
| `not_before` | datetime | Validity start | FR-051 |
| `not_after` | datetime | Validity end | FR-051 |
| `public_key_algorithm` | string | Key algorithm (e.g., RSA) | FR-052 |
| `public_key_size` | integer | Key size in bits | FR-052 |
| `signature_algorithm` | string | Signature algorithm | FR-052 |
| `sans` | string[] | Subject Alternative Names | FR-052 |
| `key_usage` | string[] | Key usage extensions | FR-052 |
| `status` | CertificateStatus | `valid` \| `expiring_soon` \| `critical` \| `expired` | FR-051 |
| `associated_entity` | string | Agent ID or hostname | FR-050 |
| `chain_valid` | boolean? | Chain validation result | FR-052 |

**Certificate Expiry Derivation:** Since Keylime does not expose certificate metadata directly, the backend reconstructs certificate records from Registrar agent data:

| Certificate | Source | Expiry Logic |
|-------------|--------|-------------|
| EK | Registrar `ek_tpm` | Fixed 10-year validity from registration date |
| AK | Registrar `aik_tpm` | Agents with `regcount > 2`: 25-day validity; others: 2-year validity |

**Expiry Status Thresholds:**

| Status | Condition |
|--------|-----------|
| `expired` | `not_after < now` |
| `expiring_soon` | `not_after < now + 30 days` |
| `valid` | `not_after >= now + 30 days` |

**Trace:** Implementation -- `keylime-webtool-backend/src/models/certificate.rs`, `keylime-webtool-backend/src/api/handlers/certificates.rs`

#### 3.3.6 Attestation and Pipeline Models

**Attestation Result:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Result identifier |
| `agent_id` | UUID | Agent that was attested |
| `result` | `pass` \| `fail` | Attestation outcome |
| `failure_type` | FailureType? | Category of failure |
| `failure_detail` | string? | Detailed failure description |
| `latency_ms` | integer | Attestation duration |
| `timestamp` | datetime | When attestation occurred |
| `verification_stages` | PipelineStage[] | Per-stage results |

**Verification Pipeline Stages** (FR-030):

| Order | Stage | Identifier | Description |
|-------|-------|-----------|-------------|
| 1 | Receive Quote | `ReceiveQuote` | Receive TPM quote and measurement logs |
| 2 | Validate TPM Quote | `ValidateTpmQuote` | Validate quote signature and nonce |
| 3 | Check PCR Values | `CheckPcrValues` | Replay and verify PCR bank values |
| 4 | Verify IMA Log | `VerifyImaLog` | Verify IMA entries against allowlist |
| 5 | Verify Measured Boot | `VerifyMeasuredBoot` | Verify UEFI event log against policy |

**Stage Status Values:**

| Status | Description |
|--------|-------------|
| `Pass` | Stage completed successfully |
| `Fail` | Stage failed (failure point) |
| `NotReached` | Not executed (prior stage failed) |

Each stage includes `duration_ms` (null if not reached).

**Failure Correlation Types** (FR-026):

| Type | Identifier | Description |
|------|-----------|-------------|
| Temporal | `Temporal` | Failures in the same time window |
| Causal | `Causal` | Failures sharing the same root cause |
| Topological | `Topological` | Failures grouped by subnet or verifier |
| Policy-linked | `PolicyLinked` | Failures matching a recent policy update |

**Trace:** Implementation -- `keylime-webtool-backend/src/models/attestation.rs`

#### 3.3.7 Policy Data Model

| Field | Type | Description | SRS Trace |
|-------|------|-------------|-----------|
| `id` | string | Policy identifier | FR-033 |
| `name` | string | Policy name | FR-033 |
| `kind` | `ima` \| `measured_boot` | Policy type discriminator | FR-033 |
| `version` | integer | Current version number | FR-035 |
| `checksum` | string | Content hash | FR-033 |
| `entry_count` | integer | Number of policy entries | FR-033 |
| `assigned_agents` | integer | Count of assigned agents | FR-037 |
| `created_at` | datetime | Creation timestamp | FR-033 |
| `updated_at` | datetime | Last update timestamp | FR-033 |
| `updated_by` | string | Last editor identity | FR-043 |
| `content` | string? | Policy content (detail only) | FR-034 |

**Approval Workflow States:**

| State | Description |
|-------|-------------|
| `Draft` | Policy created or modified, not yet submitted |
| `PendingApproval` | Submitted for two-person review (FR-039) |
| `Approved` | Approved by a different user than drafter (SR-018) |
| `Rejected` | Approval denied |
| `Expired` | Approval window expired without action |

**Trace:** Implementation -- `keylime-webtool-backend/src/models/policy.rs`

#### 3.3.8 Audit Entry Model

| Field | Type | Description | SRS Trace |
|-------|------|-------------|-----------|
| `id` | integer | Sequential entry ID | FR-042 |
| `timestamp` | datetime | Event timestamp | FR-042 |
| `severity` | `critical` \| `warning` \| `info` | Event severity | FR-042 |
| `actor` | string | User or service identity | FR-043 |
| `action` | string | Operation type | FR-043 |
| `resource` | string | Affected resource | FR-042 |
| `source_ip` | string | Client IP address | FR-042 |
| `user_agent` | string? | Browser user agent | FR-042 |
| `result` | `success` \| `failure` | Operation outcome | FR-042 |
| `previous_hash` | string | SHA-256 of previous entry | FR-061 |
| `entry_hash` | string | SHA-256 of this entry | FR-061 |

**Trace:** Implementation -- `keylime-webtool-backend/src/audit/logger.rs`

#### 3.3.9 Notification Model

| Field | Type | Description | SRS Trace |
|-------|------|-------------|-----------|
| `id` | UUID | Notification identifier | FR-010 |
| `alert_id` | UUID | Associated alert | FR-010 |
| `channel` | NotificationChannel | Delivery channel | FR-046 |
| `status` | DeliveryStatus | Current delivery state | FR-010 |
| `retry_count` | integer | Retry attempts | FR-010 |
| `sent_at` | datetime? | Successful delivery time | FR-010 |

**Notification Channels:** `Email`, `Slack`, `Webhook`, `ZeroMq`

**Delivery Statuses:** `Pending`, `Sent`, `Failed`, `Retrying`

**Trace:** Implementation -- `keylime-webtool-backend/src/models/alert.rs`

#### 3.3.10 Frontend Type System

The frontend mirrors backend models as TypeScript interfaces in `src/types/`. Key design decisions:

* **Discriminated unions** for polymorphic types (`AlertType`, `FailureType`, `PolicyKind`)
* **Literal string types** for enumerations (e.g., `'critical' | 'warning' | 'info'`)
* **Generic `PaginatedResponse<T>`** for all list endpoints
* **Optional fields** (`?`) for nullable backend values

**Trace:** Implementation -- `keylime-webtool-frontend/src/types/`

### 3.4 Interface View

#### 3.4.1 Standard API Response Envelope

All backend REST API responses use a standard JSON envelope:

```json
{
  "success": true,
  "data": { "..." },
  "error": null,
  "timestamp": "2026-04-15T12:00:00Z",
  "request_id": "uuid-v4"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Whether the request succeeded |
| `data` | T \| null | Response payload (null on error) |
| `error` | object \| null | Error object (null on success) |
| `timestamp` | string (ISO 8601) | Server-side response timestamp |
| `request_id` | string (UUID v4) | Unique request identifier for tracing |

**Error Envelope:** On error, `success: false`, `data: null`, and `error` contains:

* `code`: Machine-readable error code (e.g., `NOT_FOUND`, `UNAUTHORIZED`)
* `message`: Human-readable description

**HTTP Status Mapping:** 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 429 Too Many Requests, 502 Bad Gateway (Keylime API error), 503 Service Unavailable.

**Frontend Unwrapping:** The Axios response interceptor (`src/api/client.ts`) automatically unwraps the envelope, returning `data` directly to callers and redirecting to `/login` on 401.

**Trace:** Implementation -- `keylime-webtool-backend/src/api/response.rs`, `keylime-webtool-frontend/src/api/client.ts`

#### 3.4.2 Paginated Response Format

All list endpoints use a standard pagination wrapper inside `data`:

```json
{
  "items": [ "..." ],
  "page": 1,
  "page_size": 25,
  "total_items": 250,
  "total_pages": 10
}
```

| Field | Type | Description |
|-------|------|-------------|
| `items` | T[] | Result items for the current page |
| `page` | integer | Current page number (1-based) |
| `page_size` | integer | Items per page |
| `total_items` | integer | Total matching items |
| `total_pages` | integer | Total pages |

**Trace:** Implementation -- `keylime-webtool-backend/src/api/response.rs`, `keylime-webtool-frontend/src/types/index.ts`

#### 3.4.3 REST API Route Hierarchy

```text
/api
+-- /auth
|   +-- POST /login              Initiate OIDC login (SR-001)
|   +-- POST /callback           Auth code exchange (SR-001, SR-010)
|   +-- POST /refresh            JWT refresh rotation (SR-010)
|   +-- POST /logout             Session revocation (SR-011)
+-- /kpis
|   +-- GET /                    Fleet KPI aggregation (FR-001)
+-- /agents
|   +-- GET /                    List agents, paginated + filtered (FR-012, FR-013, FR-014)
|   +-- GET /search              Global search by UUID/IP (FR-004)
|   +-- POST /bulk               Bulk operations (FR-016)
|   +-- GET /:id                 Agent detail (FR-018)
|   +-- POST /:id/actions/:act   Agent actions (FR-019)
|   +-- GET /:id/timeline        Attestation timeline (FR-020)
|   +-- GET /:id/ima-log         IMA log entries (FR-020)
|   +-- GET /:id/boot-log        Boot log entries (FR-020)
|   +-- GET /:id/certificates    Agent certificates (FR-020)
|   +-- GET /:id/raw             Raw data — combined by default (FR-020)
|   +-- GET /:id/raw/backend     Backend-computed agent summary (FR-020)
|   +-- GET /:id/raw/registrar   Raw Registrar API JSON (FR-020)
|   +-- GET /:id/raw/verifier    Raw Verifier API JSON (FR-020)
+-- /attestations
|   +-- GET /                    List attestation results (FR-024)
|   +-- GET /summary             Attestation KPIs (FR-024)
|   +-- GET /timeline            Hourly timeline (FR-024)
|   +-- GET /failures            Failure categorization (FR-025)
|   +-- GET /incidents           Correlated incidents (FR-026)
|   +-- GET /incidents/:id       Incident detail (FR-027)
|   +-- POST /incidents/:id/rollback  Policy rollback (FR-028)
|   +-- GET /pipeline/:agent_id  Verification pipeline (FR-030)
|   +-- GET /push-mode           Push-mode analytics (FR-029)
|   +-- GET /pull-mode           Pull-mode monitoring (FR-054)
|   +-- GET /state-machine       State distribution (FR-069)
+-- /policies
|   +-- GET /                    List policies (FR-033)
|   +-- POST /                   Create policy (FR-034)
|   +-- GET /assignment-matrix   Policy assignments (FR-037)
|   +-- POST /changes/:id/approve  Two-person approval (FR-039)
|   +-- GET /:id                 Policy detail (FR-034)
|   +-- PUT /:id                 Update policy (FR-034)
|   +-- DELETE /:id              Delete policy (FR-034)
|   +-- GET /:id/versions        Version history (FR-035)
|   +-- GET /:id/diff            Version diff (FR-035)
|   +-- POST /:id/rollback/:ver  Rollback to version (FR-035)
|   +-- POST /:id/impact         Impact analysis (FR-038)
+-- /certificates
|   +-- GET /                    List certificates (FR-050)
|   +-- GET /expiry              Expiry summary (FR-051)
|   +-- GET /:id                 Certificate detail (FR-052)
|   +-- POST /:id/renew          Renew certificate (FR-053)
+-- /alerts
|   +-- GET /                    List alerts, filtered (FR-047)
|   +-- GET /summary             Alert KPI counts (FR-047)
|   +-- PUT /thresholds          Configure thresholds (FR-011)
|   +-- GET /notifications       In-app notifications (FR-009)
|   +-- GET /:id                 Alert detail (FR-047)
|   +-- POST /:id/acknowledge    Acknowledge (FR-047)
|   +-- POST /:id/investigate    Start investigation (FR-047)
|   +-- POST /:id/resolve        Resolve (FR-047)
|   +-- POST /:id/dismiss        Dismiss (FR-047)
|   +-- POST /:id/escalate       Escalate (FR-048)
+-- /audit-log
|   +-- GET /                    Query audit events (FR-042)
|   +-- GET /verify              Hash chain verification (FR-061)
|   +-- GET /export              Export audit log (FR-042)
+-- /compliance
|   +-- GET /frameworks          List frameworks (FR-059)
|   +-- GET /reports/:framework  Framework report (FR-059)
|   +-- POST /reports/:fw/export Export report (FR-060)
+-- /integrations
|   +-- GET /status              Backend connectivity with probes (FR-057, FR-077)
|   +-- GET /durable             Durable backends (FR-058)
|   +-- GET /revocation-channels Revocation channels (FR-046)
|   +-- GET /siem                SIEM status (FR-063)
+-- /performance
|   +-- GET /verifiers           Verifier metrics (FR-064)
|   +-- GET /database            DB pool metrics (FR-065)
|   +-- GET /api-response-times  API latency (FR-066)
|   +-- GET /config              Live config + drift (FR-067)
|   +-- GET /capacity            Capacity projections (FR-068)
+-- /settings
    +-- GET /keylime             Get Verifier/Registrar URLs (FR-072)
    +-- PUT /keylime             Update Verifier/Registrar URLs (FR-072)
    +-- GET /certificates        Get mTLS certificate config (FR-073)
    +-- PUT /certificates        Update mTLS certificate config (FR-073)

/ws
+-- GET /events                  WebSocket real-time updates (NFR-021)
```

**Trace:** Implementation -- `keylime-webtool-backend/src/api/routes.rs`

#### 3.4.4 WebSocket Endpoint and Subscription Model

| Property | Value |
|----------|-------|
| Endpoint | `/ws/events` |
| Authentication | JWT access token as query parameter `?token=<access_token>` |
| Channels | `kpis`, `agents`, `alerts`, `policies` |
| Reconnection | Exponential backoff: `2^n * 1000ms`, capped at 30 seconds |

**Frontend Hook:** `useWebSocket({ channel, onMessage, enabled })` manages connection lifecycle, automatic reconnection, and JSON message parsing.

**Trace:** Implementation -- `keylime-webtool-backend/src/api/ws.rs`, `keylime-webtool-frontend/src/hooks/useWebSocket.ts`

#### 3.4.5 Frontend Routing

| Path | Component | SRS Trace |
|------|-----------|-----------|
| `/login` | Login | SR-001 |
| `/` | Dashboard | FR-001 |
| `/agents` | AgentList | FR-012 |
| `/agents/:id` | AgentDetail | FR-018 |
| `/attestations` | Attestations | FR-024 |
| `/policies` | Policies | FR-033 |
| `/certificates` | Certificates | FR-050 |
| `/alerts` | Alerts | FR-047 |
| `/performance` | Performance | FR-064 |
| `/audit` | AuditLog | FR-042 |
| `/integrations` | Integrations | FR-057 |
| `/settings` | Settings | FR-002, FR-006, FR-008 |

All routes except `/login` are wrapped in the `Layout` component (Sidebar + TopBar).

**Trace:** Implementation -- `keylime-webtool-frontend/src/router.tsx`

### 3.5 Interaction View

#### 3.5.1 Data Flow: Browser to Keylime

```text
Browser (SPA)
    |
    | HTTPS (TLS 1.3) + JWT Bearer
    v
Backend (Axum)
    |
    +-- JWT validation (SR-010)
    +-- RBAC permission check (SR-003)
    +-- Cache lookup (Redis, NFR-019)
    |       |
    |       +-- Cache HIT: return cached data
    |       +-- Cache MISS: continue to Keylime
    |
    +-- Circuit breaker check (NFR-017)
    |       |
    |       +-- OPEN: return 503 + cached/stale data
    |       +-- CLOSED/HALF-OPEN: proceed
    |
    +-- mTLS request to Keylime API (SR-004)
    |
    +-- Transform response to domain model
    +-- Write to cache (with TTL)
    +-- Audit log entry (FR-061)
    +-- Return API envelope to browser
```

#### 3.5.2 Data Flow: Frontend State Management

```text
User Action (click, navigate, search)
    |
    v
Page Component
    |
    +-- TanStack React Query (useQuery / useMutation)
    |       |
    |       +-- staleTime: 30s, retry: 1
    |       +-- Automatic cache invalidation
    |       v
    +-- API Module (src/api/*.ts)
    |       |
    |       v
    +-- Axios Client (src/api/client.ts)
    |       |
    |       +-- Request interceptor: inject Bearer token
    |       +-- Response interceptor: unwrap envelope, handle 401
    |       v
    +-- Backend API
    |
    v
Re-render with new data
```

#### 3.5.3 Authentication Flow

```text
1. User navigates to /login
2. Login page presents OIDC login (or demo login)
3. Backend redirects to IdP authorization endpoint
4. User authenticates with IdP (+ MFA for Admin, SR-002)
5. IdP redirects to /api/auth/callback with authorization code
6. Backend exchanges code for ID token + access token
7. Backend maps OIDC claims to internal Role (Viewer/Operator/Admin)
8. Backend creates short-lived JWT (15 min, SR-010) with Claims:
   { sub, role, iat, exp, session_id, tenant_id }
9. Frontend stores JWT in sessionStorage
10. All subsequent requests include Authorization: Bearer <JWT>
11. JWT refresh rotation extends session without re-authentication
12. Logout revokes session server-side (SR-011)
```

**Trace:** SRS SR-001, SR-002, SR-010, SR-011

### 3.6 State Dynamics View

#### 3.6.1 Agent State Machine

See Section 3.3.3 for state enumeration. The agent state machine is owned by Keylime (not the dashboard). The dashboard observes and visualizes state transitions but does not drive them, except for operator actions (FR-019):

| Action | Effect on Keylime |
|--------|------------------|
| Reactivate | `PUT /v2/agents/{id}` -- resets agent to attesting |
| Stop | Pauses attestation monitoring |
| Delete | `DELETE /v2/agents/{id}` -- removes from verifier |

#### 3.6.2 Alert Lifecycle State Machine

```text
New --> Acknowledged --> UnderInvestigation --> Resolved
 |          |                  |
 |          |                  +---------------> Dismissed
 |          +---------------------------------> Dismissed
 +-------------------------------------------> Dismissed
```

**Transition Rules:**

| Action | Valid Source States | Target State | Side Effects |
|--------|-------------------|--------------|--------------|
| Acknowledge | `New` | `Acknowledged` | Sets `acknowledged_timestamp` |
| Investigate | `New`, `Acknowledged` | `UnderInvestigation` | Sets `acknowledged_timestamp` if null; optionally sets `assigned_to` |
| Resolve | Any non-terminal | `Resolved` | Optionally sets `resolution` reason |
| Dismiss | Any non-terminal | `Dismissed` | -- |
| Escalate | Any non-terminal | *(unchanged)* | Increments `escalation_count` |

**Terminal States:** `Resolved` and `Dismissed` reject all further transitions.

**Summary Computation:** Active alerts (not `Resolved` or `Dismissed`) are counted by severity. Resolved alerts within the last 24 hours are counted separately for the dashboard KPI.

**Trace:** Implementation -- `keylime-webtool-backend/src/models/alert_store.rs`

#### 3.6.3 Policy Approval Lifecycle

```text
Draft --> PendingApproval --> Approved --> Applied
                |
                +----------> Rejected
                |
                +----------> Expired (time-limited window)
```

**Constraint (SR-018):** The approver MUST NOT be the same identity as the drafter.

**Trace:** SRS FR-039, SR-017, SR-018

### 3.7 Algorithm View

#### 3.7.1 Attestation Timeline Distribution

The attestation timeline (FR-024) distributes event counts across hourly buckets using a deterministic variation algorithm. Since the backend does not yet persist attestation history, current agent states provide baseline totals.

**Algorithm:**

1. Count total successful and failed agents from Verifier state.
   * Pull-mode: `GET_QUOTE` = success; `FAILED`, `INVALID_QUOTE`, `TENANT_FAILED` = failure.
   * Push-mode: `accept_attestations` flag determines pass/fail.
2. Compute the number of hourly buckets from the requested time range.
3. Generate per-bucket weights:
   `weight(i) = (1 + 0.5 * sin(i * 0.9)) * jitter(i)`
   where `jitter(i) = 0.7 + ((i * 2654435761) >> 16 mod 100) / 100 * 0.6` (range [0.7, 1.3]).
4. Normalize weights so buckets sum exactly to the total count.
5. Correct rounding error by distributing remainder to buckets with the largest fractional parts.

**Supported Time Ranges:** `1h`, `6h`, `24h`, `7d`, `30d`.

**Trace:** Implementation -- `keylime-webtool-backend/src/api/handlers/attestations.rs`

#### 3.7.2 Dashboard KPI Fallback Computation

The frontend derives attestation KPIs from agent state data when no attestation history endpoint is available (FR-001):

| KPI | Computation |
|-----|------------|
| Total Agents | `paginated_response.total_items` or `agents.length` |
| Failed Attestations | Count of agents in `failed`, `invalid_quote`, `tenant_failed` (pull) or `fail` (push) state |
| Success Rate | `((total - failed) / total) * 100` |
| Active Alerts | From `GET /api/alerts/summary` -> `critical` count |

**Rationale:** Ensures the dashboard displays meaningful data before TimescaleDB attestation history persistence is implemented.

**Trace:** Implementation -- `keylime-webtool-frontend/src/pages/Dashboard/Dashboard.tsx`

#### 3.7.3 Circuit Breaker Pattern

The Keylime API client implements a circuit breaker (NFR-017) to prevent cascading failures:

```text
CLOSED --[failure_count >= threshold]--> OPEN
OPEN   --[reset_timeout elapsed]------> HALF_OPEN
HALF_OPEN --[request succeeds]--------> CLOSED
HALF_OPEN --[request fails]-----------> OPEN
```

| Parameter | Default | Description |
|-----------|---------|-------------|
| `failure_threshold` | 5 | Consecutive failures to open circuit |
| `reset_timeout` | 60s | Time before attempting recovery |

**Health Probes (FR-077):** The `KeylimeClient` exposes `probe_verifier()` and `probe_registrar()` methods that perform lightweight HTTP status-code-only checks, bypassing the circuit breaker. These are used by the integrations health endpoint so connectivity status always reflects real reachability, even when the circuit breaker is open due to prior failures or response deserialization mismatches across Keylime versions.

**Trace:** Implementation -- `keylime-webtool-backend/src/keylime/client.rs`

#### 3.7.4 Audit Log Hash Chain

Each audit entry (FR-061, SR-015) is linked to its predecessor via SHA-256:

```text
entry_hash = SHA-256(id + timestamp + severity + actor + action +
                     resource + source_ip + result + previous_hash)
```

* The first entry uses a well-known genesis hash.
* `verify_chain()` replays the chain and detects tampering or gaps.
* Optional RFC 3161 timestamp anchoring and Rekor integration for immutable storage.

**Trace:** Implementation -- `keylime-webtool-backend/src/audit/logger.rs`

### 3.8 Resource View

#### 3.8.1 Backend Configuration Hierarchy

```rust
AppConfig
+-- server
|   +-- host: String              // Default: "0.0.0.0"
|   +-- port: u16                 // Default: 8080
|   +-- tls_cert: Option<PathBuf>
|   +-- tls_key: Option<PathBuf>
+-- keylime
|   +-- verifier_url: String      // Default: "http://localhost:3000" (runtime-configurable, FR-072)
|   +-- registrar_url: String     // Default: "http://localhost:3001" (runtime-configurable, FR-072)
|   +-- mtls: Option<MtlsConfig>  // Runtime-configurable (FR-073)
|   |   +-- cert: PathBuf
|   |   +-- key: String           // HSM/Vault URI (SR-005, SR-006) or file path
|   |   +-- ca_cert: PathBuf
|   +-- timeout_secs: u64         // Default: 30
|   +-- circuit_breaker
|       +-- failure_threshold: u32  // Default: 5
|       +-- reset_timeout_secs: u64 // Default: 60
+-- database
|   +-- url: String               // PostgreSQL connection string
|   +-- pool_size: u32            // Default: 20
|   +-- connect_timeout_secs: u64 // Default: 5
+-- cache
|   +-- redis_url: String
|   +-- ttl_agent_list_secs: u64  // Default: 10 (NFR-019)
|   +-- ttl_agent_detail_secs: u64 // Default: 30 (NFR-019)
|   +-- ttl_policies_secs: u64    // Default: 60 (NFR-019)
|   +-- ttl_certs_secs: u64       // Default: 300 (NFR-019)
+-- auth
|   +-- oidc
|   |   +-- issuer: String
|   |   +-- client_id: String
|   |   +-- client_secret: String
|   |   +-- redirect_uri: String
|   +-- jwt_secret: String
|   +-- session_timeout_secs: u64 // Default: 900 (SR-010)
|   +-- mfa_required_for_admin: bool // Default: true (SR-002)
+-- audit
|   +-- log_retention_days: u32   // Default: 365 (SR-026)
|   +-- hash_algorithm: String    // Default: "sha256"
|   +-- rfc3161_timestamp_url: Option<String>
|   +-- rekor_url: Option<String>
+-- integrations
    +-- siem
    |   +-- syslog_endpoint: Option<String>
    |   +-- splunk_hec_endpoint: Option<String>
    |   +-- splunk_token: Option<String>
    |   +-- prometheus_enabled: bool // Default: true
    +-- slack_webhook_url: Option<String>
    +-- email: Option<EmailConfig>
```

**Configuration Persistence (FR-075):** At startup, configuration is loaded with priority: persisted TOML file > environment variables > compiled defaults. The TOML file path is resolved as: `KEYLIME_WEBTOOL_CONFIG` env var > `~/.config/keylime-webtool/settings.toml` > no persistence. File writes are atomic (temp file + rename) and run on `spawn_blocking` to avoid blocking the async runtime. Write failures log a warning but never fail the API request.

**Trace:** Implementation -- `keylime-webtool-backend/src/config.rs`, `keylime-webtool-backend/src/settings_store.rs`

#### 3.8.2 Frontend Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `VITE_API_BASE_URL` | Backend API base URL | `""` (same origin) |
| `VITE_WS_URL` | WebSocket server URL | `ws://${window.location.host}/ws` |

The Backend URL is also runtime-configurable via the Settings page (FR-072). The `Axios` client reads the configured backend URL from localStorage and uses the Vite dev server proxy to avoid CORS failures during development.

**Dev Server Proxy** (Vite):

* `/api/*` -> `http://localhost:8080` (API backend, avoids CORS in dev mode)
* `/ws/*` -> `ws://localhost:8080` (WebSocket)

**Auto-Refresh Wiring:** The `visualizationStore` `autoRefresh` and `refreshInterval` settings are connected to `QueryClient.setDefaultOptions({ queries: { refetchInterval } })` in `App.tsx`, so all TanStack React Query hooks automatically poll at the configured interval when auto-refresh is enabled. When auto-refresh is disabled, `refetchInterval` is set to `false`, which stops all automatic polling. The refresh interval configuration control in the Settings page is also disabled (non-interactive) while auto-refresh is off, since the interval value has no effect.

**Trace:** Implementation -- `keylime-webtool-frontend/vite.config.ts`, `keylime-webtool-frontend/src/App.tsx`

#### 3.8.3 Cache TTL Strategy (NFR-019)

| Namespace | TTL | Rationale |
|-----------|-----|-----------|
| Agent List | 10s | Fleet view requires near-real-time state |
| Agent Detail | 30s | Detail view tolerates moderate staleness |
| Policies | 60s | Policies change infrequently |
| Certificates | 300s | Certificate data is quasi-static |

**Trace:** Implementation -- `keylime-webtool-backend/src/storage/cache.rs`

#### 3.8.4 Concurrent Log Fetch Limit (NFR-023)

Maximum 5 parallel concurrent log fetches to the Verifier API, enforced via Tokio semaphore. Prevents overwhelming the Verifier when multiple agents request IMA/boot logs simultaneously.

---

## 4. Design Rationale

| Decision | Rationale | SRS Trace |
|----------|-----------|-----------|
| Rust (Axum) for backend | Memory safety without GC, `#![forbid(unsafe_code)]`, async performance for 10K WebSocket connections | SR-023, NFR-005 |
| React + TypeScript for frontend | Type safety, component reuse, ecosystem maturity for SPA | NFR-004 |
| Zustand for client state | Lightweight, no boilerplate, supports localStorage persistence for settings | FR-008 |
| TanStack React Query for server state | Automatic cache invalidation, stale-while-revalidate, retry logic; `refetchInterval` wired to auto-refresh settings (`false` when disabled) | NFR-001, FR-006 |
| In-memory AlertStore (pre-DB) | Enables full alert lifecycle development before TimescaleDB integration | FR-047 |
| Circuit breaker on Keylime API | Prevents cascading failures when Verifier is overloaded or unreachable | NFR-017 |
| Health probes bypass circuit breaker | Connectivity status must reflect real reachability, not circuit breaker state | FR-057, FR-077 |
| Response envelope pattern | Consistent error handling, request tracing, frontend interceptor simplicity | -- |
| SHA-256 hash chain for audit | Tamper detection without external dependencies; optional RFC 3161 anchoring | FR-061, SR-015 |
| Certificate derivation from regcount | Keylime does not expose cert metadata; regcount correlates with troubled agents | FR-051 |
| Timeline distribution algorithm | Deterministic variation produces natural-looking charts before history DB exists | FR-024 |
| RBAC UI enforcement via non-rendering | Controls absent (not disabled) for unauthorized roles; correct RBAC semantic | SR-003 |
| localStorage for visualization settings | Settings survive page reload without server round-trip; theme applied before first render | FR-008 |
| sessionStorage for JWT | Token isolated per browser tab; cleared on tab close | SR-010 |
| `Arc<RwLock<Arc<KeylimeClient>>>` for hot-swap | Runtime URL/mTLS changes without restart; inner `Arc` lets in-flight requests complete | FR-072, FR-073 |
| TOML config persistence with atomic writes | Settings survive restart; temp+rename prevents corruption; async-safe via `spawn_blocking` | FR-075 |
| Vite proxy for backend health checks | Avoids CORS failures in dev mode when probing backend settings endpoint | FR-077 |
| Mock/Production URL presets | Reduces configuration friction; auto-detects active mode from saved URLs | FR-074 |
| `#[serde(default)]` on Keylime models | Tolerates missing/renamed fields across Keylime API versions without breaking deserialization | NFR-002 |
| Timezone auto-detect with manual override | Default to browser timezone for zero-config; IANA dropdown for operators in different timezones than their fleet | FR-078 |
| Explicit date format selection | Eliminates locale-dependent ambiguity (e.g., 04/05 = April 5 vs May 4); ISO 8601 default for cross-region consistency | FR-079 |
| Explicit time format selection (12h/24h) | Accommodates regional conventions (e.g., US 12-hour vs European 24-hour); 24h default avoids AM/PM ambiguity in operational contexts | FR-080 |
| Sidebar alert indicator for service outages | Surfaces integration health at a glance without navigating away; shares cached TanStack Query data with Integrations page to avoid extra requests | FR-081 |

---

## 5. Design Overlays

### 5.1 Security Overlay

| Concern | Design Element | SRS Trace |
|---------|---------------|-----------|
| Authentication | OIDC flow with IdP, short-lived JWT (15 min) | SR-001, SR-010 |
| MFA | Required for Admin role via IdP policy | SR-002 |
| Authorization (backend) | RBAC middleware checks `Role` in JWT claims | SR-003 |
| Authorization (frontend) | `canWrite()` / `isAdmin()` conditionally render controls | SR-003, SR-021 |
| Transport: Browser-API | TLS 1.3 minimum | SR-008 |
| Transport: API-Keylime | mTLS with rustls, TLS 1.2+ | SR-004, SR-009 |
| Key storage | mTLS private keys from HSM/Vault, never cleartext on disk | SR-005, SR-006 |
| Session revocation | Server-side `SessionStore` with `HashSet<session_id>` | SR-011 |
| Input validation | CSP headers, input sanitization | SR-012 |
| Data minimization | Never cache/store raw TPM quotes, IMA logs, PoP tokens | SR-013, SR-014 |
| Audit integrity | SHA-256 hash chain with optional RFC 3161 anchoring | SR-015 |
| SSRF protection | Webhook URL allowlist, block RFC 1918 addresses | SR-016 |
| Two-person rule | Drafter != Approver enforcement | SR-017, SR-018 |
| Multi-tenancy | `tenant_id` in JWT claims, cross-tenant isolation | SR-019 |
| Cache integrity | Signed cache entries with TTLs | SR-024 |
| Re-registration alert | TPM key change detection via `regcount` | SR-025 |
| Idle timeout | Configurable session timeout (default: 900s) | SR-028 |
| Rate limiting | Per-user and global request rate limiting | SR-029, NFR-018 |

### 5.2 Performance Overlay

| Concern | Design Element | SRS Trace |
|---------|---------------|-----------|
| KPI refresh latency | < 30 seconds via polling or WebSocket push | NFR-001 |
| Backend concurrency | Tokio async runtime, 10K WebSocket connections target | NFR-005 |
| Cache strategy | Redis with tiered TTLs (10s-300s) per data type | NFR-019 |
| Fault tolerance | Circuit breaker on Verifier API (threshold: 5, reset: 60s) | NFR-017 |
| Log fetch limit | Max 5 parallel concurrent Verifier log fetches | NFR-023 |
| Reconciliation | Periodic sweep every 5 minutes | NFR-020 |
| Frontend query cache | TanStack Query: 30s stale time, 1 retry | NFR-001 |

---

## 6. SRS Traceability Matrix

### 6.1 Functional Requirements

| SRS Req | SDD Section | Design Element |
|---------|-------------|----------------|
| FR-001 | 3.4.3, 3.7.2 | `GET /api/kpis`, KPI fallback computation |
| FR-002 | 3.8.2 | Visualization settings: `refreshInterval` (default 30s) |
| FR-003 | 3.2.2, 3.4.5 | Sidebar component with 10 navigation modules |
| FR-004 | 3.4.3 | `GET /api/agents/search` |
| FR-005 | 3.4.3, 3.7.1 | Time range query parameter `?range=` |
| FR-006 | 3.8.2 | Visualization settings: `autoRefresh` toggle |
| FR-007 | 3.4.3 | Export endpoints (audit, compliance) |
| FR-008 | 3.8.2 | `visualizationStore`: theme in localStorage, `data-theme` attribute |
| FR-009 | 3.4.3 | `GET /api/alerts/notifications` |
| FR-010 | 3.3.9 | Notification model with channel and delivery status |
| FR-011 | 3.4.3 | `PUT /api/alerts/thresholds` |
| FR-012 | 3.3.2, 3.4.3 | Agent summary model, `GET /api/agents` |
| FR-013 | 3.4.2 | Paginated response format |
| FR-014 | 3.4.3 | Agent list query params (state, ip, uuid, policy, min_failures) |
| FR-016 | 3.4.3 | `POST /api/agents/bulk` |
| FR-018 | 3.3.2, 3.4.3 | Full agent model, `GET /api/agents/:id` |
| FR-019 | 3.4.3, 3.6.1 | `POST /api/agents/:id/actions/:action` |
| FR-020 | 3.4.3 | Agent detail tabs: timeline, TPM Policy, IMA, boot, certs, raw (with backend/registrar/verifier source selector) |
| FR-021 | 3.4.3 | TPM Policy tab — reads `tpm_policy` from agent detail (`GET /api/agents/:id`) |
| FR-024 | 3.4.3, 3.7.1 | Attestation timeline with distribution algorithm |
| FR-025 | 3.3.4 | Alert types and severity |
| FR-026 | 3.3.6 | Failure correlation types |
| FR-029 | 3.4.3 | `GET /api/attestations/push-mode` |
| FR-030 | 3.3.6, 3.4.3 | Pipeline stages, `GET /api/attestations/pipeline/:id` |
| FR-033 | 3.3.7, 3.4.3 | Policy model with kind discriminator |
| FR-034 | 3.4.3 | Policy CRUD endpoints |
| FR-035 | 3.4.3 | `GET /api/policies/:id/versions`, `/diff`, `/rollback` |
| FR-037 | 3.4.3 | `GET /api/policies/assignment-matrix` |
| FR-038 | 3.4.3 | `POST /api/policies/:id/impact` |
| FR-039 | 3.4.3, 3.6.3 | Policy approval workflow, two-person rule |
| FR-042 | 3.3.8, 3.4.3 | Audit entry model, `GET /api/audit-log` |
| FR-047 | 3.3.4, 3.6.2 | Alert lifecycle state machine |
| FR-050 | 3.3.5, 3.4.3 | Certificate model, `GET /api/certificates` |
| FR-051 | 3.3.5 | Certificate expiry derivation from regcount |
| FR-054 | 3.4.3 | `GET /api/attestations/pull-mode` |
| FR-055 | 3.4.3 | `GET /api/attestations/push-mode` |
| FR-057 | 3.4.3, 3.7.3 | `GET /api/integrations/status` with health probes |
| FR-059 | 3.4.3 | `GET /api/compliance/frameworks`, `/reports/:framework` |
| FR-061 | 3.3.8, 3.7.4 | Hash chain algorithm, audit logger |
| FR-064 | 3.4.3 | `GET /api/performance/verifiers` |
| FR-069 | 3.3.3, 3.4.3 | Agent state enumeration, `GET /api/attestations/state-machine` |
| FR-072 | 3.3.1, 3.4.3, 3.8.1 | `Arc<RwLock<Arc<KeylimeClient>>>`, `GET/PUT /api/settings/keylime` |
| FR-073 | 3.4.3, 3.8.1 | `GET/PUT /api/settings/certificates`, mTLS client reconstruction |
| FR-074 | 3.2.2 | Settings page mock/production segmented control |
| FR-075 | 3.8.1 | `SettingsStore`, TOML persistence with atomic writes |
| FR-076 | 3.2.2 | Layout hamburger button, sidebar CSS transition |
| FR-077 | 3.2.2, 3.7.3 | 1s polling via `refetchInterval`, `probe_verifier()`/`probe_registrar()` |
| FR-078 | 3.8.2 | `visualizationStore`: timezone with auto-detect, IANA timezone dropdown in Settings |
| FR-079 | 3.8.2 | `visualizationStore`: date format selection (6 formats), ISO 8601 default, applied to all timestamp rendering |
| FR-080 | 3.8.2 | `visualizationStore`: time format selection (12h/24h), 24h default, applied to all time-of-day rendering |
| FR-081 | 3.2.2 | `Sidebar.tsx`: `useHasServiceDown()` hook queries integration health, renders exclamation badge on Integrations nav item |

### 6.2 Non-Functional Requirements

| SRS Req | SDD Section | Design Element |
|---------|-------------|----------------|
| NFR-001 | 3.8.2, 5.2 | 30s refresh interval, TanStack Query stale time |
| NFR-002 | 3.3.3 | Pull-mode (v2) and push-mode (v3) state enums |
| NFR-004 | 3.2.3 | React 18, TypeScript 5.6, Vite 6.0 |
| NFR-005 | 3.2.3 | Axum 0.8, Tokio, 10K WebSocket target |
| NFR-017 | 3.7.3 | Circuit breaker: threshold 5, reset 60s |
| NFR-019 | 3.8.3 | Redis cache with tiered TTLs |
| NFR-021 | 3.4.4 | WebSocket `/ws/events` |
| NFR-023 | 3.8.4 | Tokio semaphore, max 5 parallel fetches |

### 6.3 Security Requirements

| SRS Req | SDD Section | Design Element |
|---------|-------------|----------------|
| SR-001 | 3.5.3 | OIDC authentication flow |
| SR-003 | 3.6.2, 5.1 | Three-tier RBAC: Viewer, Operator, Admin |
| SR-004 | 3.1.2 | mTLS with rustls for Keylime APIs |
| SR-005 | 3.8.1 | `MtlsConfig.key`: HSM/Vault URI |
| SR-010 | 3.5.3 | JWT claims with 15-min TTL |
| SR-011 | 3.5.3, 5.1 | Server-side SessionStore with revocation |
| SR-015 | 3.7.4 | SHA-256 hash chain for audit entries |
| SR-023 | 3.2.3 | `#![forbid(unsafe_code)]` on Rust crate |
