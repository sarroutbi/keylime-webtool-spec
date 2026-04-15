# Software Requirements Specification: Keylime Monitoring Dashboard

* **Source Document:** `slides/20260226-Keylime-Monitoring-Tool`
* **Initial Date:** 2026-04-13
* **Methodology:** Spec-Driven Development (SDD)
* **RFC 2119 Keywords:** MUST, MUST NOT, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, OPTIONAL

---

> ## Inline Review Summary
>
> **Reviewer:** Principal QA Architect & SDD Agile Coach | **Review Date:** 2026-04-09
>
> * **Non-deterministic assertions resolved (6 instances):** Gherkin scenarios using "MUST be disabled or hidden" — an ambiguous assertion preventing deterministic test automation — have been corrected. RBAC-denied actions now assert "MUST NOT be rendered" (feature absent for unauthorized role); state-based restrictions assert "MUST be disabled" (control present but precondition unmet).
> * **Multi-action scenarios decomposed (3 instances):** FR-020 (IMA Log), FR-035 (Policy Diff), and SR-003 (Operator RBAC) each contained multiple When/Then blocks in a single scenario. Each has been split into atomic scenarios per BDD single-outcome principle.
> * **RFC 2119 negation normalized:** "no X MUST be sent" rewritten to "the System MUST NOT send X" for consistency with RFC 2119 standard negation form (FR-004).
> * **Cross-references added:** SR-014 (PoP Token Privacy) now explicitly cross-references SR-013 (Data Minimization) to clarify their relationship and prevent redundant test coverage.

---

## 1. Core Objective

The Keylime Monitoring Dashboard (the "System") is a web-based security operations platform that provides centralized monitoring, management, and compliance capabilities for Keylime remote attestation infrastructure. It consumes Keylime's existing Verifier and Registrar REST APIs (v2 pull-mode and v3 push-mode) via mTLS without requiring any modification to Keylime components. The System targets Security Operations (SecOps) teams, System Administrators, Compliance Officers, and DevSecOps engineers.

The System transforms Keylime from a CLI-driven security tool into a visual operations platform, reducing mean time to detect (MTTD) attestation failures from hours to seconds, centralizing policy and certificate lifecycle management, and providing tamper-evident audit trails for compliance reporting.

**Technical Architecture:** React.js + TypeScript SPA frontend, Rust (Axum) async backend, TimescaleDB for time-series storage, Redis for caching, mTLS + rustls for Keylime API communication.

---

## 2. Requirements Inventory Table

### 2.1 Functional Requirements

| Req ID | Description | RFC Level | Traceability Source |
|--------|-------------|-----------|---------------------|
| FR-001 | Fleet overview KPI dashboard | MUST | Dashboard - Key Performance Indicators |
| FR-002 | KPI data auto-refresh with configurable interval | MUST | Dashboard - Key Performance Indicators |
| FR-003 | Sidebar navigation with core modules | MUST | Dashboard - Navigation Structure |
| FR-004 | Global agent search by UUID, IP, or hostname | MUST | Dashboard - Navigation Structure |
| FR-005 | Time range selector for data filtering | MUST | Dashboard - Navigation Structure |
| FR-006 | Auto-refresh toggle for live updates | MUST | Dashboard - Navigation Structure |
| FR-007 | CSV/JSON data export for compliance reporting | MUST | Dashboard - Navigation Structure |
| FR-008 | Dark/Light mode theme preference | MAY | Dashboard - Navigation Structure |
| FR-009 | In-app notification system with badge count | MUST | Dashboard - Navigation Structure |
| FR-010 | Email/Slack integration for critical alerts | SHOULD | Dashboard - Navigation Structure |
| FR-011 | Configurable alert severity thresholds | MUST | Dashboard - Key Performance Indicators |
| FR-012 | Agent fleet list view with sortable columns | MUST | Agent Fleet - List View |
| FR-013 | Agent list pagination | MUST | Agent Fleet - List View |
| FR-014 | Advanced multi-criteria agent filtering | MUST | Agent Fleet - Filtering and Search |
| FR-015 | IP address filtering with CIDR range support | MUST | Agent Fleet - Filtering and Search |
| FR-016 | Bulk operations on selected agents | MUST | Agent Fleet - Filtering and Search |
| FR-017 | Topology/map view for agent distribution | MAY | Agent Fleet - Map and Topology View |
| FR-018 | Agent detail view with info cards | MUST | Agent Detail - Overview Panel |
| FR-019 | Agent actions (Reactivate, Stop, Delete, Force Attest) | MUST | Agent Detail - Overview Panel |
| FR-020 | Agent detail six-tab deep-dive | MUST | Agent Detail - Tabs and Sub-Views |
| FR-021 | PCR values monitoring with diff and expected vs actual | MUST | Agent Detail - PCR Values View |
| FR-022 | PCR change detection with acknowledge/investigate | MUST | Agent Detail - PCR Values View |
| FR-023 | Cross-tab navigation (PCR→IMA, cert→Certificates) | SHOULD | Agent Detail - Tabs and Sub-Views (2/2) |
| FR-024 | Attestation analytics overview (volume, failures, latency) | MUST | Attestation Analytics - Overview Dashboard |
| FR-025 | Failure categorization by type and severity | MUST | Attestation Analytics - Failure Analysis |
| FR-026 | Automatic failure correlation across agents | MUST | Attestation Analytics - Automatic Correlation |
| FR-027 | Root cause suggestion and recommended actions (distinct from FR-026 grouping) | MUST | Attestation Analytics - Actionable Insights |
| FR-028 | One-click policy rollback from incident view | SHOULD | Attestation Analytics - Actionable Insights |
| FR-029 | Push mode (v3 API) attestation analytics | MUST | Attestation Analytics - Push Mode |
| FR-030 | Verification pipeline stage visualization | MUST | Verification Pipeline - Flow Visualization |
| FR-031 | Per-stage verification timing and metrics | MUST | Verification Pipeline - Stage Metrics |
| FR-032 | IMA quote progress tracking with gap detection | MUST | Verification Pipeline - Quote Progress Tracking |
| FR-033 | Unified policy list view (IMA & MB) with kind discriminator | MUST | Policy Management - IMA Policies |
| FR-034 | Policy CRUD with inline editor and syntax highlighting | MUST | Policy Management - Policy Editor |
| FR-035 | Policy versioning with change history and rollback | MUST | Policy Management - Policy Editor |
| FR-036 | Measured boot policy management | MUST | Policy Management - Policy Editor |
| FR-037 | Policy assignment matrix | MUST | Policy Management - Policy Editor |
| FR-038 | Pre-update policy impact analysis | MUST | Policy Management - Impact Analysis |
| FR-039 | Two-person rule for policy changes | MUST | Policy Management - Two-Person Rule |
| FR-040 | *(Merged into FR-039)* | — | — |
| FR-041 | Change management integration (ServiceNow, Jira) | MAY | Policy Management - Two-Person Rule |
| FR-042 | Security audit event log with filtering | MUST | Security Audit - Event Log Dashboard |
| FR-043 | Authorization tracking by action and identity type | MUST | Security Audit - Authorization Tracking |
| FR-044 | Identity verification event monitoring | MUST | Security Audit - Identity Verification Events |
| FR-045 | Anomaly detection on authorization patterns | SHOULD | Security Audit - Authorization Tracking |
| FR-046 | Revocation notification channel monitoring | MUST | Revocation - Notification Channels |
| FR-047 | Alert management dashboard with lifecycle workflow | MUST | Revocation - Alert Workflow |
| FR-048 | Alert auto-escalation after SLA timeout | SHOULD | Revocation - Alert Workflow |
| FR-049 | Alert auto-resolve on successful re-attestation | SHOULD | Revocation - Alert Workflow |
| FR-050 | Unified certificate view across all cert types | MUST | Certificate Management - Overview |
| FR-051 | Certificate expiry dashboard with tiered warnings | MUST | Certificate Management - Expiry Dashboard |
| FR-052 | Certificate detail inspection and chain visualization | MUST | Certificate Management - Operations |
| FR-053 | Automated certificate renewal workflow | SHOULD | Certificate Management - Operations |
| FR-054 | Pull mode attestation monitoring | MUST | Attestation Modes - Pull Mode Monitoring |
| FR-055 | Push mode attestation monitoring | MUST | Attestation Modes - Push Mode Monitoring |
| FR-056 | Mixed mode unified views | MUST | Attestation Modes - Comparative View |
| FR-057 | Backend connectivity status dashboard | MUST | Integration Status - Backend Connectivity |
| FR-058 | Durable attestation backend monitoring | MUST | Integration Status - Durable Attestation |
| FR-059 | Compliance framework mapping reports | MUST | Compliance - Framework Mapping |
| FR-060 | One-click compliance report export (PDF/CSV) | MUST | Compliance - Framework Mapping |
| FR-061 | Tamper-evident hash-chained audit logging | MUST | Compliance - Tamper-Evident Audit Logging |
| FR-062 | Incident response ticketing integration | SHOULD | Incident Response - Integration |
| FR-063 | SIEM integration (Syslog, Splunk HEC, ECS, Prometheus, OpenTelemetry) | MUST | Incident Response - Integration |
| FR-064 | Verifier cluster performance monitoring | MUST | System Performance - Verifier Metrics |
| FR-065 | Database connection pool monitoring | MUST | System Performance - Verifier Metrics |
| FR-066 | API response time tracking (p50/p95/p99) | MUST | System Performance - Verifier Metrics |
| FR-067 | Live configuration view with drift detection | MUST | System Performance - Configuration Monitoring |
| FR-068 | Capacity planning projections | SHOULD | System Performance - Key Metrics |
| FR-069 | Agent state machine visualization (pull + push) | MUST | Keylime - Agent State Machine / Push Mode |
| FR-070 | API version distribution visualization | MUST | Integration Status - Backend Connectivity |
| FR-071 | AI Assistant with Keylime MCP integration | SHOULD | AI Assistant - Conversational Interface |

### 2.2 Non-Functional Requirements

| Req ID | Description | RFC Level | Traceability Source |
|--------|-------------|-----------|---------------------|
| NFR-001 | KPI data refresh within 30 seconds | MUST | Dashboard - Key Performance Indicators |
| NFR-002 | Support Keylime API v2 and v3 simultaneously | MUST | Keylime - Data Model Overview |
| NFR-003 | API-first, non-invasive architecture (zero Keylime modifications) | MUST | Technical Architecture - System Design |
| NFR-004 | SPA frontend with client-side routing (<3s initial load on 10 Mbps) | MUST | Technical Architecture - Data Flow |
| NFR-005 | Async backend supporting 10K concurrent WebSocket connections (<100ms p99) | MUST | Technical Architecture - Why Rust |
| NFR-006 | Event-driven ingestion as primary data path | MUST | Technical Architecture - Event-Driven Ingestion |
| NFR-007 | Polling fallback with adaptive backpressure | MUST | Technical Architecture - Event-Driven Ingestion |
| NFR-008 | Scale to 100K+ agents under event-driven mode | SHOULD | Scalability - Ingestion Model Comparison |
| NFR-009 | Scale to ~1,000 agents under polling fallback | MUST | Scalability - Ingestion Model Comparison |
| NFR-010 | Active/Passive HA with <30s RTO and 0 RPO | MUST | High Availability - Architecture |
| NFR-011 | Active/Active HA for 5K+ agents | SHOULD | High Availability - Architecture |
| NFR-012 | Air-gapped deployment with no external dependencies | MUST | Deployment - Offline & Air-Gapped |
| NFR-013 | Self-contained packaging (no CDN, single binary) | MUST | Deployment - Offline & Air-Gapped |
| NFR-014 | WCAG 2.1 Level AA accessibility compliance | MUST | Deployment - Offline & Air-Gapped |
| NFR-015 | Container (OCI), Kubernetes (Helm), RPM, systemd deployment options | MUST | Technical Architecture - Deployment |
| NFR-016 | Graceful degradation when components are unavailable | MUST | High Availability - Architecture |
| NFR-017 | Circuit breaker on Verifier API latency | MUST | Technical Architecture - Event-Driven Ingestion |
| NFR-018 | Per-user and global request rate limiting | MUST | Technical Architecture - IMA Log & Data Decoupling |
| NFR-019 | Cache TTLs: agent list 10s, detail 30s, policies 60s, certs 300s | MUST | Scalability - Cache Invalidation |
| NFR-020 | Periodic reconciliation sweep every 5 minutes | MUST | Technical Architecture - Event-Driven Ingestion |
| NFR-021 | WebSocket real-time updates for UI | MUST | Technical Architecture - Data Flow |
| NFR-022 | Signed update packages with SBOM for offline updates | MUST | Deployment - Offline & Air-Gapped |
| NFR-023 | Maximum 5 parallel concurrent log fetches to Verifier | MUST | Technical Architecture - IMA Log & Data Decoupling |
| NFR-024 | AI Assistant query performance and rate limiting | SHOULD | AI Assistant - Conversational Interface |

### 2.3 Security Requirements

| Req ID | Description | RFC Level | Traceability Source |
|--------|-------------|-----------|---------------------|
| SR-001 | OIDC/SAML identity provider authentication | MUST | Dashboard Authentication - User Identity |
| SR-002 | MFA mandatory for Admin role | MUST | Dashboard Authentication - User Identity |
| SR-003 | Three-tier RBAC (Viewer, Operator, Admin) | MUST | Dashboard RBAC - Role Definitions |
| SR-004 | mTLS for all Keylime API communication | MUST | Threat Model - Trust Boundaries |
| SR-005 | mTLS private key MUST NEVER be stored on disk in cleartext | MUST | Secret Management - Credential Lifecycle |
| SR-006 | HSM or Vault-backed private key storage | MUST | Secret Management - Credential Lifecycle |
| SR-007 | TLS encryption on all network connections (no cleartext paths) | MUST | Transport Security - Encrypted Data Paths |
| SR-008 | Browser→API: TLS 1.3 minimum | MUST | Transport Security - Encrypted Data Paths |
| SR-009 | API→Keylime: TLS 1.2+ minimum | MUST | Transport Security - Encrypted Data Paths |
| SR-010 | Short-lived JWT session tokens (15 min) with refresh rotation | MUST | Dashboard Authentication - User Identity |
| SR-011 | Server-side session revocation | MUST | Dashboard Authentication - User Identity |
| SR-012 | CSP headers and input sanitization (XSS/injection prevention) | MUST | Threat Model - Threat Catalog |
| SR-013 | Never cache or store raw TPM quotes, IMA logs, boot logs | MUST | Threat Model - Data Classification |
| SR-014 | Never display or cache raw PoP tokens | MUST | Attestation Modes - Comparative View |
| SR-015 | Tamper-evident hash-chained audit log with RFC 3161 anchoring | MUST | Compliance - Tamper-Evident Audit Logging |
| SR-016 | SSRF protection on webhook URLs (allowlist, block RFC 1918) | MUST | Revocation - Alert Workflow |
| SR-017 | Two-person approval for policy changes (N-of-M quorum) | MUST | Policy Management - Two-Person Rule |
| SR-018 | Approver cannot be the same as drafter | MUST | Policy Management - Two-Person Rule |
| SR-019 | Multi-tenancy isolation (cross-tenant data never mixed) | MUST | Dashboard RBAC - Multi-Tenancy |
| SR-020 | Data classification enforcement (SECRET, CONFIDENTIAL, INTERNAL) | MUST | Threat Model - Data Classification |
| SR-021 | Write operations blocked at proxy for non-Admin roles | MUST | Dashboard RBAC - Role Definitions |
| SR-022 | mTLS sidecar option (Envoy/Ghostunnel) | MAY | Transport Security - mTLS Sidecar Option |
| SR-023 | `#![forbid(unsafe_code)]` on dashboard Rust crate | MUST | Technical Architecture - Why Rust |
| SR-024 | Signed cache entries with TTLs to mitigate cache poisoning | MUST | Threat Model - Threat Catalog |
| SR-025 | Identity alert on TPM key change during re-registration | MUST | Security Audit - Identity Verification Events |
| SR-026 | Audit log minimum retention of 1 year for compliance | MUST | Compliance - Tamper-Evident Audit Logging |
| SR-027 | Emergency bypass with break-glass audit trail | MUST | Policy Management - Two-Person Rule |
| SR-028 | Configurable idle session timeout | MUST | Dashboard Authentication - User Identity |
| SR-029 | Rate limiting on dashboard session creation endpoint | MUST | Attestation Modes - Comparative View |

---

## 3. Detailed Functional Requirements

### FR-001: Fleet Overview KPI Dashboard

**Description:** The System MUST display a fleet overview dashboard presenting computed KPIs derived from the Keylime Verifier and Registrar APIs. The dashboard MUST show: Total Active Agents, Failed Agents (states 7, 9, 10), Attestation Success Rate, Average Attestation Latency, Certificate Expiry Warnings, Active IMA Policies, Revocation Events (24h), Consecutive Failures per agent, and Registration Count.

**Trace:** Dashboard - Key Performance Indicators; Dashboard - Main Screen Layout

```gherkin
Feature: Fleet Overview KPI Dashboard

  Scenario: Display fleet health KPIs
    Given the dashboard backend is connected to the Verifier API via mTLS
    And the Verifier reports 247 agents in GET_QUOTE state
    And 3 agents are in FAILED state
    When the user navigates to the Fleet Overview Dashboard
    Then the dashboard MUST display "247" as Active Agents
    And the dashboard MUST display "3" as Failed Agents
    And the Attestation Success Rate MUST be computed from attestation history
    And Certificate Expiry Warnings MUST reflect certificates expiring within 30 days

  Scenario: Verifier API unreachable
    Given the dashboard backend cannot connect to the Verifier API
    When the user navigates to the Fleet Overview Dashboard
    Then the dashboard MUST display cached KPI data with a staleness indicator
    And a banner MUST warn "Verifier API unreachable — data may be stale"

  Scenario: Failed agent threshold alert
    Given the alert threshold for Failed Agents is configured to "any count > 0"
    When 1 or more agents enter state 7 (FAILED), 9 (INVALID_QUOTE), or 10 (TENANT_FAILED)
    Then the Failed Agents KPI MUST display in critical color
    And an alert MUST be raised in the notification system
```

### FR-002: KPI Data Auto-Refresh

**Description:** The System MUST refresh KPI data at a configurable interval. The default refresh interval MUST be 30 seconds. The System MUST support both HTTP polling and WebSocket push as refresh mechanisms.

**Trace:** Dashboard - Key Performance Indicators

```gherkin
Feature: KPI Data Auto-Refresh

  Scenario: Default KPI refresh interval
    Given the auto-refresh toggle is enabled
    And the refresh interval is set to the default of 30 seconds
    When 30 seconds elapse
    Then the System MUST fetch updated KPI data from the Verifier API
    And the dashboard MUST re-render all KPI values

  Scenario: KPI refresh via WebSocket
    Given the backend supports WebSocket push
    And a WebSocket connection is established
    When the backend receives an agent state change event
    Then the updated KPI data MUST be pushed to the browser
    And the dashboard MUST re-render affected KPI values

  Scenario: WebSocket connection lost
    Given a WebSocket connection was established
    When the connection drops unexpectedly
    Then the System MUST fall back to HTTP polling at the configured interval
    And a connection status indicator MUST show "reconnecting"
```

### FR-003: Sidebar Navigation

**Description:** The System MUST provide a persistent sidebar navigation with the following modules: Dashboard (Fleet overview), Agents (Fleet management), Attestations (Analytics), Policies (IMA & MB), Certificates (TLS/TPM certs), Alerts (Alert lifecycle), Performance (System metrics), Audit Log (Security events), Integrations (Backend status), Settings (Configuration), and AI Assistant (Keylime MCP conversational interface).

**Trace:** Dashboard - Navigation Structure

```gherkin
Feature: Sidebar Navigation

  Scenario: Navigate between core modules
    Given the user is authenticated and viewing the dashboard
    When the user clicks "Agents" in the sidebar
    Then the Agent Fleet Management view MUST be displayed
    And the sidebar MUST highlight the "Agents" entry as active

  Scenario: All core modules accessible
    Given the user is authenticated
    Then the sidebar MUST display navigation entries for all 11 core modules
    And each entry MUST route to its corresponding view

  Scenario: Unauthenticated user cannot access sidebar
    Given the user is not authenticated
    When the user attempts to access any dashboard view
    Then the System MUST redirect the user to the login page
    And no sidebar navigation MUST be rendered
```

### FR-004: Global Agent Search

**Description:** The System MUST provide a global search bar that allows searching agents by UUID (exact or partial match), IP address (including CIDR range support), or hostname.

**Trace:** Dashboard - Navigation Structure; Agent Fleet - Filtering and Search

```gherkin
Feature: Global Agent Search

  Scenario: Search agent by partial UUID
    Given the agent fleet contains an agent with UUID "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
    When the user types "a1b2c3" in the global search bar
    Then the search results MUST include agent "a1b2c3d4-e5f6-7890-abcd-ef1234567890"

  Scenario: Search agent by CIDR range
    Given the fleet contains agents at IPs 192.168.1.10, 192.168.1.11, 192.168.2.10
    When the user searches for "192.168.1.0/24"
    Then the search results MUST include agents at 192.168.1.10 and 192.168.1.11
    And the results MUST NOT include the agent at 192.168.2.10

  Scenario: Search returns no results
    Given the agent fleet contains 250 agents
    When the user searches for "nonexistent-uuid-xyz"
    Then the search results MUST display an empty state with message "No agents found"

  Scenario: Invalid CIDR notation
    Given the user enters "192.168.1.0/99" in the search bar
    When the search is submitted
    Then the System MUST display a validation error indicating invalid CIDR notation
    And the System MUST NOT send a search request to the backend

> **Reviewer Note:** Rephrased from "no search request MUST be sent" to standard RFC 2119 negation form "the System MUST NOT send." The original syntax is ambiguous — "no" could modify "search request" or "MUST be sent."
```

### FR-005: Time Range Selector

**Description:** The System MUST provide a time range selector allowing users to filter displayed data by predefined intervals: 1h, 6h, 24h, 7d, 30d, and a custom date range picker.

**Trace:** Dashboard - Navigation Structure

```gherkin
Feature: Time Range Selector

  Scenario: Filter attestation data by time range
    Given the user is viewing the Attestation Analytics view
    When the user selects the "24h" time range
    Then only attestation data from the last 24 hours MUST be displayed
    And charts and tables MUST update to reflect the selected range

  Scenario: Custom date range
    Given the user selects "Custom" in the time range selector
    When the user specifies a start date of "2026-02-20" and end date of "2026-02-25"
    Then only data within that date range MUST be displayed

  Scenario: Invalid date range rejected
    Given the user selects "Custom" in the time range selector
    When the user specifies a start date later than the end date
    Then the System MUST display a validation error "Start date must be before end date"
    And the previous time range MUST remain active
```

### FR-006: Auto-Refresh Toggle

**Description:** The System MUST provide a toggle control that enables or disables live automatic updates of dashboard data.

**Trace:** Dashboard - Navigation Structure

```gherkin
Feature: Auto-Refresh Toggle

  Scenario: Disable auto-refresh
    Given auto-refresh is currently enabled
    When the user toggles auto-refresh off
    Then dashboard data MUST stop refreshing automatically
    And the displayed data MUST remain static until manually refreshed or re-enabled

  Scenario: Enable auto-refresh
    Given auto-refresh is currently disabled
    When the user toggles auto-refresh on
    Then dashboard data MUST begin refreshing at the configured interval
```

### FR-007: Data Export

**Description:** The System MUST provide CSV and JSON export functionality for compliance reporting. Export MUST be available for agent fleet data, attestation analytics, audit logs, and certificate data.

**Trace:** Dashboard - Navigation Structure

```gherkin
Feature: Data Export

  Scenario: Export agent fleet data as CSV
    Given the user has the Operator or Admin role
    And the agent fleet list is displayed with active filters
    When the user clicks "Export" and selects "CSV"
    Then the System MUST generate a CSV file containing the filtered agent data
    And the file MUST be downloaded to the user's browser

  Scenario: Viewer cannot export
    Given the user has the Viewer role
    When the user views the agent fleet list
    Then the Export action MUST NOT be rendered for the Viewer role

> **Reviewer Note:** Changed "MUST be disabled or hidden" to "MUST NOT be rendered" — RBAC-denied features must be absent from the UI, not merely grayed out. A disabled control implies the feature exists but is temporarily unavailable; an absent control implies the user lacks the capability, which is the correct RBAC semantic. Also changed the When step from "looks for the Export button" (user intent, not a testable action) to "views the agent fleet list" (observable state).

  Scenario: Export with empty filter result
    Given the user has applied filters that match zero agents
    When the user views the Export action
    Then the Export action MUST be disabled
    And a message MUST indicate "No data to export"

> **Reviewer Note:** Changed "MUST be disabled or display" to two deterministic assertions: the action is disabled AND a message explains why. Each Then step is independently verifiable.
```

### FR-008: Dark/Light Mode

**Description:** The System MAY provide a theme preference toggle allowing users to switch between dark and light visual modes.

**Trace:** Dashboard - Navigation Structure

```gherkin
Feature: Dark/Light Mode

  Scenario: Switch to dark mode
    Given the user is viewing the dashboard in light mode
    When the user selects "Dark Mode" in the theme settings
    Then the entire dashboard UI SHOULD render with a dark color scheme
    And the preference SHOULD persist across sessions
```

### FR-009: In-App Notification System

**Description:** The System MUST provide an in-app notification bell displaying an unread notification badge count. Notifications MUST include attestation failures, certificate expiry warnings, policy updates, agent registration events, and revocation events. Severity thresholds for generating notifications MUST be configurable.

**Trace:** Dashboard - Navigation Structure

```gherkin
Feature: In-App Notification System

  Scenario: Display notification badge count
    Given there are 3 unread critical notifications and 2 unread warnings
    When the user views the dashboard header
    Then the notification bell MUST display a badge count of 5

  Scenario: Mark notification as read
    Given the user has unread notifications
    When the user opens the notification panel and clicks a notification
    Then the notification MUST be marked as read
    And the badge count MUST decrement by 1

  Scenario: No notifications available
    Given there are zero unread notifications
    When the user opens the notification panel
    Then the panel MUST display an empty state with message "No new notifications"
    And the notification bell MUST NOT display a badge count
```

### FR-010: External Alert Integration

**Description:** The System SHOULD support sending critical alert notifications to external channels including Email and Slack. Alert routing MUST be configurable by severity threshold.

**Trace:** Dashboard - Navigation Structure

```gherkin
Feature: External Alert Integration

  Scenario: Send critical alert to Slack
    Given Slack webhook integration is configured
    And the alert severity threshold for Slack is set to "CRITICAL"
    When a CRITICAL attestation failure is detected
    Then the System SHOULD send a notification to the configured Slack channel
    And the notification MUST include the agent ID, failure type, and timestamp

  Scenario: External integration endpoint unreachable
    Given Slack webhook integration is configured
    When the System attempts to send a notification and the webhook endpoint is unreachable
    Then the System MUST retry delivery according to the configured retry policy
    And the System MUST log the delivery failure in the audit log
```

### FR-011: Configurable Alert Thresholds

**Description:** The System MUST allow administrators to configure alert thresholds per deployment. Configurable thresholds MUST include: attestation success rate floor (default: 99%), average latency ceiling (default: 2x quote_interval), certificate expiry warning window (default: 30 days), consecutive failure count (default: 3), and revocation event count.

**Trace:** Dashboard - Key Performance Indicators

```gherkin
Feature: Configurable Alert Thresholds

  Scenario: Configure attestation rate threshold
    Given the user has the Admin role
    When the admin sets the attestation success rate threshold to 98%
    Then the System MUST raise an alert only when the rate falls below 98%
    And the previous threshold of 99% MUST no longer trigger alerts

  Scenario: Default thresholds applied
    Given no custom thresholds are configured
    When the attestation success rate falls below 99%
    Then the System MUST raise an alert using the default threshold

  Scenario: Non-Admin cannot configure thresholds
    Given the user has the Operator role
    When the user attempts to change the attestation success rate threshold
    Then the System MUST deny the action with an insufficient permissions error
    And the threshold MUST remain unchanged
```

### FR-012: Agent Fleet List View

**Description:** The System MUST display agents in a sortable, paginated table with columns: Agent ID, IP Address, Operational State, Last Attestation time, Assigned Policy, Failure Count, and Actions. Rows for agents in FAILED or INVALID_QUOTE states MUST be visually highlighted. Rows for agents in RETRY state MUST be highlighted with a warning indicator.

**Trace:** Agent Fleet - List View

```gherkin
Feature: Agent Fleet List View

  Scenario: Display agent list with state-based highlighting
    Given the Verifier reports agents in various states
    When the user navigates to the Agent Fleet view
    Then agents in GET_QUOTE state MUST display with a success indicator
    And agents in FAILED state MUST display with a critical/danger indicator
    And agents in RETRY state MUST display with a warning indicator
    And each row MUST show Agent ID, IP, State, Last Attest, Policy, Failures, and Actions

  Scenario: Verifier API unavailable for agent list
    Given the dashboard backend cannot connect to the Verifier API
    When the user navigates to the Agent Fleet view
    Then the System MUST display cached agent data with a staleness indicator
    And a banner MUST warn that the agent list may not reflect current state
```

### FR-013: Agent List Pagination

**Description:** The System MUST paginate the agent fleet list. The System MUST display the current page range, total agent count, and total page count. Page size SHOULD be configurable.

**Trace:** Agent Fleet - List View

```gherkin
Feature: Agent List Pagination

  Scenario: Navigate to next page
    Given the agent fleet contains 250 agents with a page size of 25
    And the user is viewing page 1
    When the user clicks "Next Page"
    Then agents 26-50 MUST be displayed
    And the footer MUST show "Showing 26-50 of 250 agents | Page 2 of 10"

  Scenario: Navigate beyond last page
    Given the agent fleet contains 250 agents with a page size of 25
    And the user is viewing page 10 (the last page)
    When the user clicks "Next Page"
    Then the "Next Page" button MUST be disabled
    And page 10 MUST remain displayed
```

### FR-014: Advanced Multi-Criteria Agent Filtering

**Description:** The System MUST allow filtering the agent fleet by multiple criteria simultaneously: Agent UUID (exact or partial), IP Address (CIDR range), Operational State (multi-select), Verifier Assignment, IMA Policy, MB Policy, Last Attestation time range, Failure Count (min/max), Registration Date range, and API Version.

**Trace:** Agent Fleet - Filtering and Search

```gherkin
Feature: Advanced Agent Filtering

  Scenario: Filter by state and policy simultaneously
    Given the fleet contains agents in various states with different policies
    When the user selects state filter "FAILED" and policy filter "production-v2"
    Then only agents in FAILED state assigned to "production-v2" MUST be displayed
    And agents in other states or with other policies MUST be hidden

  Scenario: Filter returns no matching agents
    Given the fleet contains 250 agents
    When the user selects state filter "TERMINATED" and no agents are in that state
    Then the agent list MUST display an empty state with message "No agents match the selected filters"
    And a "Clear Filters" button MUST be available

  Scenario: Filter by failure count threshold
    Given the fleet contains agents with failure counts 0, 1, 3, 5
    When the user sets the failure count minimum to 3
    Then only agents with 3 or more failures MUST be displayed
```

### FR-015: IP Address CIDR Filtering

**Description:** The System MUST support filtering agents by IP address using CIDR notation (e.g., 192.168.1.0/24) in addition to exact IP match.

**Trace:** Agent Fleet - Filtering and Search

```gherkin
Feature: CIDR IP Filtering

  Scenario: Filter agents by subnet
    Given the fleet contains agents at IPs 10.0.1.5, 10.0.1.20, 10.0.2.15
    When the user enters IP filter "10.0.1.0/24"
    Then agents at 10.0.1.5 and 10.0.1.20 MUST be displayed
    And the agent at 10.0.2.15 MUST NOT be displayed

  Scenario: Invalid CIDR in filter
    Given the user enters "10.0.1.0/33" in the IP filter
    When the filter is applied
    Then the System MUST display a validation error indicating invalid CIDR notation
    And the agent list MUST remain unfiltered
```

### FR-016: Bulk Operations

**Description:** The System MUST support selecting multiple agents via checkboxes and performing bulk operations: Reactivate (resume monitoring), Stop (pause monitoring), Delete (remove from verifier), Reassign Policy (change IMA/MB policy), and Export (download agent data as CSV).

**Trace:** Agent Fleet - Filtering and Search

```gherkin
Feature: Bulk Agent Operations

  Scenario: Bulk reactivate stopped agents
    Given the user has selected 5 agents that are in a stopped state
    And the user has the Operator or Admin role
    When the user clicks "Reactivate"
    Then the System MUST send reactivation requests for all 5 agents to the Verifier API
    And each agent's state MUST update upon successful reactivation

  Scenario: Bulk operations denied for Viewer role
    Given the user has the Viewer role
    When the user selects agents in the fleet list
    Then the Reactivate, Stop, Delete, and Reassign Policy buttons MUST be disabled

  Scenario: Partial failure in bulk reactivation
    Given the user has selected 5 agents for reactivation
    When the System sends reactivation requests and 2 agents fail to reactivate
    Then the System MUST display a summary: "3 succeeded, 2 failed"
    And each failed agent MUST show the failure reason
    And the 3 successful agents MUST update their state
```

### FR-017: Topology/Map View

**Description:** The System MAY provide an optional topology/map view visualizing agent distribution across infrastructure. Agents MUST be groupable by datacenter, rack, or subnet. The view MUST show verifier-to-agent assignment mapping and registrar connection status. Interactive features MUST include click-to-drill-down on datacenters, hover for agent summary, and color-coded health indicators.

**Trace:** Agent Fleet - Map and Topology View

```gherkin
Feature: Topology View

  Scenario: Drill down into datacenter
    Given the topology view shows 3 datacenters: DC-East (120 agents), DC-West (85 agents), DC-EU (45 agents)
    When the user clicks on "DC-East"
    Then the view MUST expand to show individual agents within DC-East
    And failed agents MUST be indicated with a danger color

  Scenario: Hover for agent summary
    Given the topology view is displayed
    When the user hovers over an agent node
    Then a tooltip MUST display the agent UUID, IP, state, and last attestation time
```

### FR-018: Agent Detail View

**Description:** The System MUST display an agent detail page containing: Agent Information card (UUID, IP, Verifier assignment, Registration date), Attestation Statistics card (total attestations, last successful, consecutive failures, boot time), Cryptographic Details card (hash algorithm, encryption, signing, IMA PCRs), and Policy Assignment card (IMA policy, MB policy, TPM policy with a "Change Policy" action).

**Trace:** Agent Detail - Overview Panel

```gherkin
Feature: Agent Detail View

  Scenario: Display agent information
    Given agent "a1b2c3d4-e5f6-7890" exists in the Verifier
    When the user navigates to the agent detail page
    Then the Agent Information card MUST display UUID, IP address, assigned Verifier, and registration date
    And the Attestation Statistics card MUST display total attestations, last successful time, and consecutive failures
    And the Cryptographic Details card MUST display hash algorithm, encryption, and signing algorithms
    And the Policy Assignment card MUST display assigned IMA and MB policies

  Scenario: Agent not found
    Given no agent with UUID "nonexistent-uuid" exists in the Verifier
    When the user navigates to the agent detail page for "nonexistent-uuid"
    Then the System MUST display a "404 — Agent Not Found" error page
    And a link to return to the Agent Fleet view MUST be available
```

### FR-019: Agent Actions

**Description:** The System MUST provide the following actions on an individual agent detail page: Reactivate (resume monitoring), Stop (pause monitoring), Delete (remove from verifier), Force Attest (trigger immediate attestation), View Quotes (display raw TPM quote data), and Export (download agent data). Actions MUST be restricted by RBAC role.

**Trace:** Agent Detail - Overview Panel

```gherkin
Feature: Agent Detail Actions

  Scenario: Force attestation on an agent
    Given the user has the Operator or Admin role
    And the user is viewing agent detail for "a1b2c3d4"
    When the user clicks "Force Attest"
    Then the System MUST trigger an immediate attestation cycle for agent "a1b2c3d4"
    And the attestation result MUST update on the Timeline tab

  Scenario: Delete agent requires Admin role
    Given the user has the Operator role
    When the user views the agent detail page
    Then the "Delete" action MUST NOT be rendered for the Operator role

> **Reviewer Note:** Changed "MUST be disabled or hidden" to "MUST NOT be rendered" — consistent with RBAC semantic: the Operator role lacks the delete capability entirely, so the control should be absent.

  Scenario: Force attestation fails due to agent unreachable
    Given the user has the Operator role
    And agent "a1b2c3d4" is unreachable by the Verifier
    When the user clicks "Force Attest"
    Then the System MUST display an error "Agent unreachable — attestation could not be initiated"
    And the agent state MUST remain unchanged
```

### FR-020: Agent Detail Six-Tab Deep-Dive

**Description:** The System MUST provide six specialized tabs on the agent detail page: (1) Timeline — attestation success/failure history with zoomable time range; (2) PCR Values — current PCR bank values with change history and diffs; (3) IMA Log — measurement list entries with policy match/mismatch indicators and search by file path or hash; (4) Boot Log — UEFI event log entries with measured boot validation; (5) Certificates — EK, AK, IAK, IDevID, mTLS certificate details and expiry countdown; (6) Raw Data — JSON view of full agent record with copy/export.

**IMA Log Entry Schema:** Each IMA log entry returned by the backend MUST include: `pcr` (PCR index, typically 10), `template_hash` (SHA-256 hash of the template data), `template_name` (IMA template type, e.g., `ima-ng`), `filedata_hash` (hash of the measured file content), and `filename` (absolute path of the measured file).

**Boot Log Entry Schema:** Each boot log entry returned by the backend MUST include: `pcr` (PCR index associated with the event), `event_type` (UEFI event type identifier, e.g., `EV_EFI_VARIABLE_DRIVER_CONFIG`), `digest` (hash digest of the event data), and `event_data` (human-readable description of the event).

**Trace:** Agent Detail - Tabs and Sub-Views

```gherkin
Feature: Agent Detail Tabs

  Scenario: View IMA log entries
    Given the user is viewing agent "a1b2c3d4" detail page
    When the user selects the "IMA Log" tab
    Then the IMA measurement list entries MUST be displayed
    And each entry MUST indicate policy match or mismatch

> **Reviewer Note:** Split from original "View IMA log with search" scenario which contained two When/Then blocks. Each BDD scenario MUST have exactly one When/Then pair to maintain test isolation and deterministic pass/fail.

  Scenario: Search IMA log by file path
    Given the user is viewing the IMA Log tab for agent "a1b2c3d4"
    When the user searches for "/usr/bin/bash"
    Then only IMA entries for that file path MUST be shown

  Scenario: View attestation timeline
    Given the user is viewing agent "a1b2c3d4" detail page
    When the user selects the "Timeline" tab
    Then the attestation success/failure history MUST be displayed chronologically
    And each entry MUST show timestamp, result (pass/fail), and failure reason if applicable
    And the timeline MUST support zoomable time range navigation

  Scenario: View PCR values with change history
    Given the user is viewing agent "a1b2c3d4" detail page
    When the user selects the "PCR Values" tab
    Then the current PCR bank values MUST be displayed with expected vs. actual comparison
    And changed PCR values MUST be highlighted with a "Changed" indicator

  Scenario: View boot log with measured boot validation
    Given the user is viewing agent "a1b2c3d4" detail page
    When the user selects the "Boot Log" tab
    Then the UEFI event log entries MUST be displayed
    And each entry MUST show measured boot validation status (compliant/non-compliant)

  Scenario: View agent certificates with expiry countdown
    Given the user is viewing agent "a1b2c3d4" detail page
    When the user selects the "Certificates" tab
    Then EK, AK, IAK, IDevID, and mTLS certificate details MUST be displayed
    And each certificate MUST show an expiry countdown (days remaining)

  Scenario: View raw JSON data
    Given the user is viewing agent "a1b2c3d4" detail page
    When the user selects the "Raw Data" tab
    Then the full agent JSON record from the Verifier API MUST be displayed
    And a "Copy" button MUST allow copying the JSON to clipboard

  Scenario: Tab data unavailable due to API error
    Given the user is viewing agent "a1b2c3d4" detail page
    And the Verifier API returns an error for IMA log data
    When the user selects the "IMA Log" tab
    Then the tab MUST display an error message "Unable to load IMA log data"
    And a "Retry" button MUST be available
```

### FR-021: PCR Values Monitoring

**Description:** The System MUST display current PCR bank values (SHA-256) for each agent with an expected vs. actual comparison. Each PCR MUST show its description, status (Match/Changed), and last changed date. The System MUST support PCR banks across SHA-1, SHA-256, SHA-384, and SHA-512.

**Trace:** Agent Detail - PCR Values View

```gherkin
Feature: PCR Values Monitoring

  Scenario: Display PCR comparison table
    Given agent "a1b2c3d4" has PCR values reported by the Verifier
    When the user views the PCR Values tab
    Then a table MUST display each PCR index, description, match status, and last changed date
    And PCR 10 (IMA measurements) MUST show "Match" with a recent timestamp
    And any PCR with a value differing from expected MUST show "Changed"

  Scenario: PCR data unavailable for agent
    Given agent "a1b2c3d4" has not yet completed an attestation cycle
    When the user views the PCR Values tab
    Then the tab MUST display "No PCR data available — awaiting first attestation"
```

### FR-022: PCR Change Detection

**Description:** The System MUST monitor PCR values for each agent and detect changes relative to expected baseline values. When a PCR value changes, the System MUST highlight the changed PCR, display the last changed timestamp, and allow the administrator to acknowledge the change or initiate an investigation. The System MUST alert if the IMA policy needs updating for legitimate changes.

**Trace:** Agent Detail - PCR Values View

```gherkin
Feature: PCR Change Detection

  Scenario: Detect PCR drift from baseline
    Given agent "a1b2c3d4" has a policy-defined expected value for PCR 8
    And the agent's current PCR 8 value differs from the expected value
    When the user views the PCR Values tab for agent "a1b2c3d4"
    Then PCR 8 MUST display with a "Changed" indicator
    And the last changed date MUST be displayed
    And "Acknowledge" and "Investigate" action buttons MUST be available

  Scenario: PCR change acknowledgement denied for Viewer
    Given the user has the Viewer role
    When the user views a PCR with a "Changed" indicator
    Then the "Acknowledge" and "Investigate" actions MUST NOT be rendered for the Viewer role

> **Reviewer Note:** Changed "MUST be disabled or hidden" to "MUST NOT be rendered" — Viewer role lacks the capability to acknowledge or investigate PCR changes, so the controls should be absent.
```

### FR-023: Cross-Tab Navigation

**Description:** The System SHOULD interconnect agent detail tabs so that clicking a PCR value in the PCR Values tab navigates to the corresponding IMA entries in the IMA Log tab, and certificate warnings link directly to the Certificates tab for renewal.

**Trace:** Agent Detail - Tabs and Sub-Views (2/2)

```gherkin
Feature: Cross-Tab Navigation

  Scenario: Navigate from PCR to IMA entries
    Given the user is viewing the PCR Values tab for agent "a1b2c3d4"
    And PCR 10 is highlighted with a "Changed" status
    When the user clicks on the PCR 10 value
    Then the view MUST switch to the IMA Log tab
    And the IMA entries corresponding to PCR 10 measurements MUST be displayed

  Scenario: Navigate from cert warning to Certificates tab
    Given the user sees a certificate expiry warning on the overview
    When the user clicks the warning link
    Then the view MUST switch to the Certificates tab
    And the expiring certificate MUST be highlighted
```

### FR-024: Attestation Analytics Overview

**Description:** The System MUST provide an attestation analytics overview displaying: total successful attestations, total failed attestations, average latency, and success rate as summary KPIs; an hourly attestation volume bar chart; a failure reason breakdown (donut chart); a latency distribution histogram; and a top failing agents ranked list.

**Trace:** Attestation Analytics - Overview Dashboard

```gherkin
Feature: Attestation Analytics Overview

  Scenario: Display attestation summary KPIs
    Given there were 12,450 successful and 38 failed attestations in the last 24 hours
    When the user navigates to the Attestation Analytics view
    Then the summary MUST show "12,450" successful, "38" failed, and the computed average latency
    And the success rate MUST display as "99.7%"

  Scenario: Display top failing agents
    Given multiple agents have different failure counts
    When the user views the Attestation Analytics
    Then a "Top Failing Agents" list MUST rank agents by failure count descending

  Scenario: No attestation data available
    Given no attestation data has been collected yet
    When the user navigates to the Attestation Analytics view
    Then the summary KPIs MUST display zeroes
    And charts MUST display an empty state with message "No attestation data for the selected period"
```

### FR-025: Failure Categorization

**Description:** The System MUST categorize attestation failures by type and severity: Quote Invalid (Critical), Policy Violation (Critical), Evidence Chain broken (Critical), Boot Violation (High), Timeout (Medium), PCR Mismatch (Medium), and Clock Skew (Low). Each failure type MUST include a description and common cause.

**Trace:** Attestation Analytics - Failure Analysis

```gherkin
Feature: Failure Categorization

  Scenario: Classify failure by type
    Given agent "agent-042" fails with a TPM quote signature mismatch
    When the failure is processed by the analytics engine
    Then the failure MUST be categorized as "Quote Invalid"
    And the severity MUST be set to "Critical"
    And the common cause MUST indicate "TPM hardware issue, key mismatch"

  Scenario: Failure with unknown type
    Given agent "agent-099" fails with an unrecognized error code from the Verifier
    When the failure is processed by the analytics engine
    Then the failure MUST be categorized as "Unknown"
    And the severity MUST default to "High"
    And the raw error details MUST be preserved for manual review
```

### FR-026: Automatic Failure Correlation

**Description:** The System MUST automatically correlate attestation failures across agents using four correlation dimensions: Temporal (failures within the same time window), Causal (same failure reason across agents), Topological (failures grouped by datacenter, subnet, or verifier), and Policy-linked (failures matching a recent policy update). Correlated failures MUST be grouped into a single incident.

**Trace:** Attestation Analytics - Automatic Correlation; Attestation Analytics - Actionable Insights

```gherkin
Feature: Automatic Failure Correlation

  Scenario: Correlate failures after policy update
    Given IMA policy "production-v2" was updated 3 minutes ago
    And 15 agents sharing policy "production-v2" report IMA policy violation within a 2-minute window
    When the System processes these failure events
    Then the System MUST group all 15 failures into a single correlated incident
    And the suggested root cause MUST reference the recent policy update
    And the incident view MUST provide a one-click rollback option

  Scenario: Distinguish targeted attack from mass failure
    Given a single agent fails with a unique IMA hash mismatch
    And no other agents share the same failure reason or timing
    When the System processes the failure
    Then the System MUST NOT correlate it with other incidents
    And the alert MUST indicate "warrants immediate investigation"
```

### FR-027: Root Cause Suggestion and Recommended Actions

**Description:** For each correlated incident created by FR-026, the System MUST generate a suggested root cause by analyzing the triggering event (e.g., recent policy change, certificate expiry, infrastructure outage). The System MUST provide a recommended action (e.g., "rollback policy", "renew certificate", "investigate agent"). The System MUST automatically link the incident to the triggering change in the audit log.

**Trace:** Attestation Analytics - Actionable Insights

```gherkin
Feature: Root Cause Suggestion

  Scenario: Suggest root cause from policy change
    Given a correlated incident exists grouping 15 IMA policy violation failures
    And IMA policy "production-v2" was updated 3 minutes before the first failure
    When the System analyzes the incident
    Then the suggested root cause MUST reference the policy update to "production-v2"
    And the recommended action MUST include "Rollback policy to previous version"
    And the incident MUST link to the policy change audit log entry

  Scenario: Suggest root cause from certificate expiry
    Given a correlated incident exists grouping 5 mTLS handshake failures
    And the Verifier's server certificate expired 10 minutes before the first failure
    When the System analyzes the incident
    Then the suggested root cause MUST reference the expired certificate
    And the recommended action MUST include "Renew Verifier server certificate"

  Scenario: No root cause identified
    Given a correlated incident exists but no recent policy, certificate, or infrastructure change is found
    When the System analyzes the incident
    Then the suggested root cause MUST display "Unknown — manual investigation required"
    And the recommended action MUST include "Escalate to security team"
```

### FR-028: One-Click Policy Rollback from Incident

**Description:** The System SHOULD provide a one-click rollback option from the incident view that reverts a policy to its previous version when a policy update is identified as the root cause of correlated failures.

**Trace:** Attestation Analytics - Actionable Insights

```gherkin
Feature: Policy Rollback from Incident

  Scenario: Rollback policy from incident view
    Given an incident is open with suggested root cause "policy update to production-v2"
    And the user has the Admin role
    When the user clicks "Rollback Policy" on the incident
    Then the System SHOULD revert "production-v2" to its previous version
    And the rollback MUST be recorded in the audit log

  Scenario: Rollback subject to two-person rule
    Given an incident suggests policy rollback
    And the two-person rule is enforced for policy changes
    When Admin A clicks "Rollback Policy"
    Then the rollback MUST be submitted as a draft requiring approval from Admin B
    And the System MUST NOT apply the rollback without a second approver
```

### FR-029: Push Mode (v3 API) Attestation Analytics

**Description:** The System MUST provide analytics specific to push-mode attestation including: attestation submission rate per agent, nonce expiry tracking, challenge lifetime compliance, evidence evaluation duration, rate limit violations per IP/agent. Push-mode alerts MUST include: agent not submitting attestations, nonce expired before evidence, repeated evaluation failures, rate limit threshold reached, and session token expiry warnings.

**Trace:** Attestation Analytics - Push Mode

```gherkin
Feature: Push Mode Analytics

  Scenario: Track nonce expiry rate
    Given the deployment operates in push mode (v3 API)
    And 10% of nonces expired before evidence was submitted
    When the user views push mode analytics
    Then the nonce expiry rate MUST be displayed as 10%
    And a warning alert MUST be raised for high nonce expiry

  Scenario: Alert on silent agent
    Given agent "agent-100" has not submitted attestation evidence for 10 minutes
    And the expected submission interval is 2 minutes
    When the System evaluates push mode agent activity
    Then the System MUST raise an alert indicating "agent not submitting attestations"

  Scenario: Push mode analytics unavailable in pull-only deployment
    Given the deployment operates exclusively in pull mode (v2 API)
    When the user navigates to the Push Mode Analytics view
    Then the System MUST display "Push mode is not enabled in this deployment"
    And no push-specific metrics MUST be shown
```

### FR-030: Verification Pipeline Visualization

**Description:** The System MUST visualize the multi-stage verification pipeline showing each stage: Receive Quote + Logs, Validate TPM Quote, Check PCR Values, Verify IMA Log, and Verify Measured Boot. Each stage MUST show pass/fail status and timing. The visualization MUST indicate where in the pipeline a failure occurred.

**Trace:** Verification Pipeline - Flow Visualization

```gherkin
Feature: Verification Pipeline Visualization

  Scenario: Display pipeline with failure point
    Given agent "agent-042" failed at the "Verify IMA Log" stage
    When the user views the verification pipeline for the latest attestation
    Then stages "Receive", "Validate TPM Quote", and "Check PCR Values" MUST show pass indicators
    And stage "Verify IMA Log" MUST show a fail indicator
    And subsequent stages MUST be shown as not reached

  Scenario: Pipeline data unavailable for agent
    Given agent "agent-042" has not yet completed any attestation cycle
    When the user views the verification pipeline
    Then the System MUST display "No pipeline data available — awaiting first attestation"
```

### FR-031: Per-Stage Verification Metrics

**Description:** The System MUST track timing and success rates for each verification stage independently: Quote Validation (signature, nonce freshness), PCR Check (replay time, bank coverage), IMA Verification (entries processed, allowlist hit/miss ratio, excludelist matches, TOMTOU errors), and Measured Boot (event log entries, compliance rate, firmware update detections).

**Trace:** Verification Pipeline - Stage Metrics

```gherkin
Feature: Per-Stage Verification Metrics

  Scenario: Display IMA verification depth metrics
    Given the Verifier has processed attestations for agent "a1b2c3d4"
    When the user views verification stage metrics
    Then IMA stage MUST show entries processed count
    And IMA stage MUST show allowlist hit/miss ratio
    And IMA stage MUST show average processing time

  Scenario: Metrics unavailable for new deployment
    Given the System has just been deployed and no attestations have been processed
    When the user views verification stage metrics
    Then all metrics MUST display "N/A" or zero values
    And a message MUST indicate "Insufficient data — metrics will populate after attestation activity"
```

### FR-032: IMA Quote Progress Tracking

**Description:** The System MUST track the gap between total IMA measurement entries and verified entries per agent. If the gap exceeds a configurable threshold (default: 1000 entries or 5 minutes of growth), the System MUST raise an alert indicating potential network issues, agent overload, or intentional evasion.

**Trace:** Verification Pipeline - Quote Progress Tracking

```gherkin
Feature: IMA Quote Progress Tracking

  Scenario: Alert on IMA verification lag
    Given agent "agent-042" has 50,000 total IMA entries
    And only 48,500 entries have been verified
    And the gap threshold is configured at 1,000 entries
    When the System evaluates IMA progress for "agent-042"
    Then the System MUST raise an alert for IMA verification lag
    And the alert MUST display the gap size (1,500 entries)

  Scenario: IMA progress tracking disabled for push-mode agent
    Given agent "agent-042" operates in push mode (v3 API)
    And push mode does not expose incremental IMA progress
    When the user views IMA quote progress for "agent-042"
    Then the System MUST display "IMA progress tracking not available for push mode agents"
```

### FR-033: Policy List View and Management

**Description:** The System MUST display a unified list of all runtime policies — both IMA and Measured Boot — in a single view. Each policy row MUST show: Policy Name, Kind (IMA or Measured Boot), number of assigned Agents, Entry count, Checksum, Last Updated date, and Edit/View actions. A search bar MUST allow filtering policies by name. New Policy and Import buttons MUST be available.

**Trace:** Policy Management - IMA Policies

```gherkin
Feature: Unified Policy List View

  Scenario: Display unified policy list with kind column
    Given the Verifier has 3 IMA policies and 2 Measured Boot policies configured
    When the user navigates to the Policy Management view
    Then all 5 policies MUST be listed with name, kind, agent count, entry count, checksum, and last updated date
    And each policy MUST display its kind as "IMA" or "Measured Boot"
    And each policy MUST have "Edit" and "View" action links

  Scenario: Search policies by name
    Given policies named "production-v2", "staging-v3", "minimal" exist
    When the user types "prod" in the policy search bar
    Then only "production-v2" MUST be displayed

  Scenario: No policies configured
    Given the Verifier has zero policies configured
    When the user navigates to the Policy Management view
    Then the list MUST display an empty state with message "No policies configured"
    And a "New Policy" button MUST be prominently available
```

### FR-034: Policy CRUD with Editor

**Description:** The System MUST provide full CRUD operations for IMA runtime policies via the dashboard. The editor MUST support: file upload for allowlist, inline editing with syntax highlighting, hash algorithm selection (SHA-256, SHA-384), exclude list configuration, IMA signature key management, policy validation before save, and automatic checksum computation.

**Trace:** Policy Management - Policy Editor

```gherkin
Feature: Policy Editor

  Scenario: Create new IMA policy via upload
    Given the user has the Admin role
    When the user clicks "New Policy" and uploads an allowlist file
    Then the System MUST parse and validate the file
    And the System MUST compute and display the checksum
    And the user MUST be able to save the policy

  Scenario: Validate policy before save
    Given the user is editing a policy with an invalid hash format
    When the user clicks "Save"
    Then the System MUST display a validation error
    And the policy MUST NOT be saved until errors are corrected

  Scenario: Concurrent edit conflict
    Given Admin A is editing policy "production-v2" at version 3
    And Admin B saves a change to "production-v2" creating version 4
    When Admin A attempts to save their changes
    Then the System MUST reject the save with a conflict error
    And Admin A MUST be prompted to reload the latest version before editing
```

### FR-035: Policy Versioning

**Description:** The System MUST maintain a version history for each policy with change diffs. The System MUST support rollback to any previous version. An audit trail MUST record who changed what and when. A side-by-side comparison view MUST be available.

**Trace:** Policy Management - Policy Editor

```gherkin
Feature: Policy Versioning

  Scenario: View policy version history
    Given policy "production-v2" has been updated 3 times
    When the user views the change history
    Then the System MUST list all 3 versions with timestamps and authors

> **Reviewer Note:** Split from original "View policy change diff" scenario which contained two When/Then blocks. Version listing and version comparison are distinct user actions that MUST be independently testable.

  Scenario: Compare policy versions
    Given the user is viewing the change history for policy "production-v2"
    When the user selects two versions for comparison
    Then a side-by-side diff MUST highlight added, removed, and modified entries

  Scenario: Rollback to previous version
    Given the user has the Admin role
    And policy "production-v2" is at version 3
    When the user selects "Rollback to version 2"
    Then the policy MUST revert to version 2 content
    And a new version 4 MUST be created reflecting the rollback

  Scenario: Rollback denied for non-Admin
    Given the user has the Operator role
    And policy "production-v2" is at version 3
    When the user attempts to select "Rollback to version 2"
    Then the rollback action MUST NOT be rendered for the Operator role

> **Reviewer Note:** Changed "MUST be disabled or hidden" to "MUST NOT be rendered" — rollback is an Admin-only capability, so Operators should not see the control at all.
```

### FR-036: Measured Boot Policy Management

**Description:** The System MUST support managing measured boot (MB) policies: list all policies, create from reference boot log, edit rules and constraints, associate with agent groups, and validate against known event logs.

**Trace:** Policy Management - Policy Editor

```gherkin
Feature: Measured Boot Policy Management

  Scenario: Create MB policy from reference boot log
    Given the user has the Admin role
    When the user uploads a reference UEFI event log
    Then the System MUST generate a measured boot policy from the log
    And the user MUST be able to edit and save the generated policy

  Scenario: Invalid boot log upload rejected
    Given the user has the Admin role
    When the user uploads a file that is not a valid UEFI event log
    Then the System MUST display a validation error "Invalid boot log format"
    And no policy MUST be generated
```

### FR-037: Policy Assignment Matrix

**Description:** The System MUST display a matrix view showing which agents are assigned to which policies. The System MUST support batch reassignment, orphan policy detection (policies not assigned to any agent), impact analysis before changes, and preview of policy effect on agents.

**Trace:** Policy Management - Policy Editor

```gherkin
Feature: Policy Assignment Matrix

  Scenario: Detect orphan policies
    Given policy "dev-old" is not assigned to any agent
    When the user views the policy assignment matrix
    Then "dev-old" MUST be flagged as an orphan policy

  Scenario: Batch reassign agents to new policy
    Given 15 agents are assigned to policy "staging-v2"
    When the admin selects all 15 agents and assigns them to "staging-v3"
    Then all 15 agents MUST be reassigned to "staging-v3"
    And the assignment change MUST be recorded in the audit log

  Scenario: Batch reassignment partial failure
    Given 15 agents are assigned to policy "staging-v2"
    When the admin reassigns all 15 agents to "staging-v3" and 3 agents fail to update
    Then the System MUST display a summary: "12 succeeded, 3 failed"
    And each failed agent MUST show the failure reason
```

### FR-038: Pre-Update Policy Impact Analysis

**Description:** The System MUST perform an impact analysis before applying policy changes. The analysis MUST categorize affected agents into three groups: Unaffected (no files match changes), Affected (have modified files), and Will Fail (removed hash currently in use). The System MUST display a recommendation and provide Submit for Approval, Staged Rollout, and Cancel actions.

**Trace:** Policy Management - Impact Analysis

```gherkin
Feature: Policy Impact Analysis

  Scenario: Analyze impact of IMA policy update
    Given IMA policy "production-v2" is assigned to 185 agents
    And the proposed update adds 45 hashes, removes 12 hashes, and modifies 8 hashes
    When the administrator requests impact analysis for the proposed change
    Then the System MUST display the number of unaffected agents
    And the System MUST display the number of affected agents
    And the System MUST display the number of agents that will fail
    And a recommendation MUST be provided based on the analysis

  Scenario: Impact analysis with Verifier API unavailable
    Given the Verifier API is unreachable
    When the administrator requests impact analysis for a proposed change
    Then the System MUST display an error "Unable to perform impact analysis — Verifier API unavailable"
    And the "Submit for Approval" action MUST be disabled
```

### FR-039: Two-Person Rule for Policy Changes

**Description:** The System MUST enforce a two-person approval workflow for policy changes. Admin A drafts the policy change, the System runs impact analysis automatically, and Admin B (a different user) reviews and approves. The System MUST support configurable quorum (e.g., 2-of-3, 3-of-5). The approver MUST NOT be the same user as the drafter. Approval requests MUST have a configurable time-limited window (default: 24 hours) after which they expire and revert to Draft state.

**Trace:** Policy Management - Two-Person Rule

```gherkin
Feature: Two-Person Policy Approval

  Scenario: Policy change requires different approver
    Given Admin A drafts a change to IMA policy "production-v2"
    And the System runs impact analysis automatically
    When Admin A attempts to approve their own policy change
    Then the System MUST reject the approval
    And the System MUST display an error indicating the approver cannot be the drafter

  Scenario: Approval window expiry
    Given Admin A submitted a policy change for review
    And 24 hours have elapsed without approval
    When Admin B attempts to approve the change
    Then the System MUST reject the approval as expired
    And the policy change MUST revert to Draft state

  Scenario: Successful two-person approval
    Given Admin A drafts a policy change
    And Admin B reviews and approves the change within 24 hours
    When the approval is recorded
    Then the System MUST automatically push the policy to the Verifier
    And the audit log MUST record both the drafter and approver identities

  Scenario: Configure approval window duration
    Given the Admin configures the approval window to 48 hours
    When Admin A submits a policy change for review
    Then the approval window MUST expire after 48 hours instead of the default 24 hours
    And the pending change MUST display the configured expiry time

  Scenario: Single admin cannot approve policy changes
    Given there is only one Admin user in the system
    When Admin A drafts a policy change
    Then the System MUST allow the draft to be submitted for approval
    And the System MUST display a warning that no other approver is available
    And the policy change MUST remain in "Pending Approval" until a second Admin is created
```

### FR-041: Change Management Integration

**Description:** The System MAY integrate with external change management systems. Integration options MUST include: optionally gating policy push on ServiceNow Change Request approval, auto-creating Jira tickets for policy review, linking CR numbers in the audit log, and supporting emergency bypass with break-glass audit.

**Trace:** Policy Management - Two-Person Rule

```gherkin
Feature: Change Management Integration

  Scenario: Gate policy push on ServiceNow CR
    Given ServiceNow integration is enabled
    And Admin A drafts a policy change
    When the change is submitted for approval
    Then a ServiceNow Change Request MUST be auto-created
    And the policy push MUST be blocked until the CR is approved in ServiceNow

  Scenario: ServiceNow integration unavailable
    Given ServiceNow integration is enabled
    And the ServiceNow API endpoint is unreachable
    When the change is submitted for approval
    Then the System MUST display a warning "ServiceNow unreachable — CR not created"
    And the policy change MUST remain in draft state until the CR can be created
```

### FR-042: Security Audit Event Log

**Description:** The System MUST maintain a searchable security audit event log. Each event MUST include severity level (CRITICAL, WARNING, INFO), timestamp, source component, action performed, and acting user identity. The log MUST be filterable by severity, category, and date range. An Export function MUST be available.

**Trace:** Security Audit - Event Log Dashboard

```gherkin
Feature: Security Audit Event Log

  Scenario: Filter audit log by severity
    Given the audit log contains events of various severities
    When the user selects severity filter "CRITICAL"
    Then only events with CRITICAL severity MUST be displayed
    And WARNING and INFO events MUST be hidden

  Scenario: Audit log captures attestation failure
    Given agent "a1b2c3d4" fails attestation with an IMA policy violation
    When the verifier reports the failure
    Then a CRITICAL audit event MUST be logged
    And the event MUST include the source (verifier ID), action (VERIFY_EVIDENCE), agent ID, and failure reason

  Scenario: Audit log search returns no results
    Given the audit log contains events from the past 30 days
    When the user filters by severity "CRITICAL" and date range with no matching events
    Then the log view MUST display an empty state with message "No events match the selected filters"
```

### FR-043: Authorization Tracking

**Description:** The System MUST track all authorization decisions made by Keylime's authorization provider. Tracked actions MUST include: Agent Management (CREATE, READ, UPDATE, DELETE by admin mTLS), Attestation (SUBMIT, READ, LIST by agent PoP token), Policy Management (CREATE, READ, UPDATE, DELETE by admin mTLS), Sessions (CREATE, EXTEND by agent), Verification (VERIFY_IDENTITY, VERIFY_EVIDENCE by admin mTLS), and Registration (REGISTER, ACTIVATE, DELETE by agent/admin).

**Trace:** Security Audit - Authorization Tracking

```gherkin
Feature: Authorization Tracking

  Scenario: Track admin policy update
    Given admin "admin@example.com" updates IMA policy "production-v2"
    When the authorization event is recorded
    Then the audit log MUST contain action "UPDATE" in category "Policy Management"
    And the identity type MUST be "admin (mTLS)"
    And the actor MUST be "admin@example.com"

  Scenario: Authorization tracking for denied action
    Given user "viewer@example.com" with Viewer role attempts to delete an agent
    When the authorization decision is recorded
    Then the audit log MUST contain action "DELETE" with result "DENIED"
    And the identity MUST be "viewer@example.com"
```

### FR-044: Identity Verification Event Monitoring

**Description:** The System MUST monitor agent identity verification events including: new agent registration (with identity type: ek_cert, iak_idevid, default), agent re-registration (regcount tracking), EK certificate validation results, and TPM key changes between registrations.

**Trace:** Security Audit - Identity Verification Events

```gherkin
Feature: Identity Verification Events

  Scenario: Track new agent registration
    Given a new agent registers with identity type "iak_idevid"
    When the registration event is processed
    Then the audit log MUST record action "REGISTER_AGENT"
    And the identity type MUST be "iak_idevid"

  Scenario: Alert on re-registration from different IP
    Given agent "agent-042" was originally registered from IP 10.0.1.5
    When agent "agent-042" re-registers from IP 10.0.2.10
    Then the System MUST raise a WARNING alert
    And the alert MUST indicate the IP change from 10.0.1.5 to 10.0.2.10

  Scenario: Agent registration with invalid EK certificate
    Given a new agent attempts to register with an EK certificate not in the trusted CA list
    When the registration event is processed
    Then the audit log MUST record action "REGISTER_AGENT" with result "REJECTED"
    And a CRITICAL alert MUST be raised indicating "untrusted EK certificate"
```

### FR-045: Anomaly Detection

**Description:** The System SHOULD detect anomalous authorization patterns including: unusual authorization sequences, failed authorization attempts, privilege escalation attempts, and off-hours administrative actions.

**Trace:** Security Audit - Authorization Tracking

```gherkin
Feature: Anomaly Detection

  Scenario: Detect off-hours admin action
    Given the organization's business hours are 08:00-18:00
    When admin "admin@example.com" deletes an agent at 03:00
    Then the System SHOULD flag the action as an off-hours anomaly
    And a WARNING alert SHOULD be raised for security review

  Scenario: Multiple failed authorization attempts
    Given user "unknown@example.com" attempts 5 failed authorization requests in 1 minute
    When the System evaluates authorization patterns
    Then the System SHOULD flag the activity as a potential brute-force attempt
    And a CRITICAL alert SHOULD be raised for immediate investigation
```

### FR-046: Revocation Notification Channel Monitoring

**Description:** The System MUST monitor all revocation notification channels: Agent (REST) direct notification with delivery status and latency, ZeroMQ notification with connection state and queue depth, and Webhook notification with HTTP status and retry count.

**Trace:** Revocation - Notification Channels

```gherkin
Feature: Revocation Channel Monitoring

  Scenario: Monitor webhook delivery failure
    Given a revocation webhook is configured for "hooks.slack.com"
    And a revocation event triggers for agent "agent-042"
    When the webhook delivery fails with HTTP 503
    Then the System MUST display "failed (retry 1)" for the webhook channel
    And the System MUST retry delivery according to the configured retry policy

  Scenario: All revocation channels unavailable
    Given all configured revocation notification channels are in a failed state
    When a new revocation event occurs
    Then the System MUST raise a CRITICAL alert "All notification channels unavailable"
    And the revocation event MUST be queued for delivery once a channel recovers

  Scenario: Monitor ZeroMQ connection state
    Given ZeroMQ is configured on port 8992
    When the ZeroMQ connection is active
    Then the integration status MUST show "connected" with current queue depth
```

### FR-047: Alert Lifecycle Workflow

**Description:** The System MUST implement an alert lifecycle: New → Acknowledged → Under Investigation → Resolved or Dismissed. Operators MUST be able to acknowledge alerts, assign them to team members, add investigation notes, reactivate agents after fix, update policies for false positives, and escalate to security teams.

**Trace:** Revocation - Alert Workflow

```gherkin
Feature: Alert Lifecycle Workflow

  Scenario: Acknowledge a critical alert
    Given a new critical alert exists for agent "agent-042"
    And the user has the Operator role
    When the operator clicks "Acknowledge" on the alert
    Then the alert state MUST change to "Acknowledged"

  Scenario: Move acknowledged alert to investigation
    Given an acknowledged alert exists for agent "agent-042"
    And the user has the Operator role
    When the operator assigns the alert and clicks "Investigate"
    Then the alert state MUST change to "Under Investigation"

  Scenario: Auto-resolve on re-attestation
    Given agent "agent-042" has an active alert for attestation failure
    When agent "agent-042" passes a subsequent attestation cycle
    Then the System SHOULD automatically resolve the alert
    And the resolution reason MUST indicate "auto-resolved on successful re-attestation"
```

### FR-048: Alert Auto-Escalation

**Description:** The System SHOULD automatically escalate alerts that remain unacknowledged after a configurable SLA timeout. Escalation MUST follow a defined escalation chain.

**Trace:** Revocation - Alert Workflow

```gherkin
Feature: Alert Auto-Escalation

  Scenario: Escalate unacknowledged critical alert
    Given a CRITICAL alert has been in "New" state for 30 minutes
    And the SLA timeout for critical alerts is configured at 15 minutes
    When the SLA timeout is exceeded
    Then the System SHOULD escalate the alert to the next level in the escalation chain
    And a notification SHOULD be sent to the escalation recipient

  Scenario: Escalation chain exhausted
    Given all levels in the escalation chain have been notified
    And the alert remains unacknowledged
    When the System attempts further escalation
    Then the System SHOULD raise a CRITICAL meta-alert "Escalation chain exhausted — alert unresolved"
    And the audit log MUST record all escalation attempts
```

### FR-049: Alert Auto-Resolve

**Description:** The System SHOULD automatically resolve alerts when the underlying condition clears, specifically when an agent passes a subsequent attestation cycle. The System MUST implement notification deduplication and severity auto-adjustment.

**Trace:** Revocation - Alert Workflow

```gherkin
Feature: Alert Auto-Resolve

  Scenario: Auto-resolve on successful attestation
    Given agent "agent-067" has an active warning alert for boot violation
    When agent "agent-067" completes a successful attestation
    Then the System SHOULD change the alert state to "Resolved"
    And the resolution MUST be marked as "auto-resolved"

  Scenario: Auto-resolve suppressed for manually escalated alert
    Given an alert has been manually escalated to the security team
    When the underlying condition clears
    Then the System MUST NOT auto-resolve the alert
    And the alert MUST remain in its current state until manually resolved
```

### FR-050: Unified Certificate View

**Description:** The System MUST provide a unified view of all certificate types in the Keylime ecosystem: EK Certificate (TPM vendor), AK Certificate (Keylime CA), mTLS Certificate (Agent↔Verifier), IAK Certificate (manufacturer), IDevID Certificate (manufacturer), and Server Certificates (Verifier/Registrar from Org CA).

**Trace:** Certificate Management - Overview

```gherkin
Feature: Unified Certificate View

  Scenario: Display all certificate types
    Given agents have EK, AK, and mTLS certificates
    And the Verifier and Registrar have server certificates
    When the user navigates to the Certificate Management view
    Then all certificate types MUST be listed in a unified view
    And each certificate MUST show its type, associated entity, and validity status

  Scenario: Agent with missing certificate data
    Given agent "agent-099" has no EK certificate recorded in the Registrar
    When the user views the Certificate Management view
    Then the entry for "agent-099" MUST display "EK Certificate: Not Available"
```

### FR-051: Certificate Expiry Dashboard

**Description:** The System MUST display a certificate expiry dashboard showing summary counts (Expired, Expiring within 30 days, Valid, Total) and a detailed list of certificates requiring attention. The System MUST display a 90-day certificate expiry timeline. Alert thresholds MUST be tiered: 90-day informational, 30-day action required, 7-day critical, 1-day emergency, and expired.

**Trace:** Certificate Management - Expiry Dashboard; Certificate Management - Operations

```gherkin
Feature: Certificate Expiry Dashboard

  Scenario: Display certificate expiry summary
    Given the fleet has 1 expired certificate, 5 certificates expiring within 30 days, and 244 valid certificates
    When the user navigates to the Certificate Expiry Dashboard
    Then the summary MUST show "Expired: 1", "Expiring <30d: 5", "Valid: 244", "Total: 250"

  Scenario: Certificate expiry alert at 7-day threshold
    Given agent "agent-015" has an mTLS certificate expiring in 7 days
    When the System evaluates certificate validity
    Then a critical alert MUST be raised for agent "agent-015"
    And the alert MUST indicate the certificate type and expiry date

  Scenario: No certificates expiring
    Given all certificates in the fleet are valid with more than 90 days remaining
    When the user navigates to the Certificate Expiry Dashboard
    Then the summary MUST show "Expired: 0", "Expiring <30d: 0"
    And the timeline MUST display no upcoming expirations
```

### FR-052: Certificate Detail Inspection

**Description:** The System MUST provide a detailed certificate inspection view showing: Subject and Issuer DN, serial number, validity period (Not Before/After), public key algorithm and size, signature algorithm, Subject Alternative Names (SANs), Key Usage / Extended Key Usage, certificate chain visualization, and PEM/DER export options. For EK certificates, the System MUST verify the TPM vendor and validate the chain against known CAs.

**Trace:** Certificate Management - Operations

```gherkin
Feature: Certificate Detail Inspection

  Scenario: Inspect EK certificate
    Given agent "a1b2c3d4" has an EK certificate issued by a TPM vendor
    When the user views the certificate detail for the EK cert
    Then the Subject DN, Issuer DN, serial number, and validity period MUST be displayed
    And the certificate chain visualization MUST show the chain to the root CA
    And PEM export MUST be available

  Scenario: Certificate chain validation failure
    Given agent "a1b2c3d4" has an EK certificate issued by an unknown CA
    When the System validates the EK certificate chain
    Then the chain validation MUST display "INVALID — issuer not in trusted CA list"
    And a WARNING MUST be displayed on the certificate detail view

  Scenario: Validate EK certificate chain
    Given the TPM vendor CA certificates are pre-loaded
    When the System validates agent "a1b2c3d4"'s EK certificate
    Then the chain validation result MUST be displayed (valid/invalid)
```

### FR-053: Automated Certificate Renewal

**Description:** The System SHOULD support automated certificate renewal workflows for Keylime CA-issued certificates. Renewal MUST include: approval workflow, batch scheduling, pre-renewal validation, and rollback on failure.

**Trace:** Certificate Management - Operations

```gherkin
Feature: Automated Certificate Renewal

  Scenario: Auto-renew expiring mTLS certificate
    Given agent "agent-015" has an mTLS certificate expiring in 7 days
    And auto-renewal is enabled for Keylime CA certificates
    When the System initiates renewal
    Then the certificate SHOULD be renewed with approval workflow
    And the new certificate MUST be validated before activation

  Scenario: Rollback on renewal failure
    Given a certificate renewal fails validation
    When the System detects the failure
    Then the System SHOULD rollback to the previous certificate
    And an alert MUST be raised indicating renewal failure

  Scenario: Auto-renewal not available for vendor certificates
    Given agent "agent-015" has an EK certificate issued by a TPM vendor
    When the System evaluates the certificate for auto-renewal
    Then the System MUST skip auto-renewal for vendor-issued certificates
    And the certificate expiry alert MUST indicate "manual renewal required"
```

### FR-054: Pull Mode Attestation Monitoring

**Description:** The System MUST provide monitoring specific to pull-mode (v2 API) attestation including: quote polling frequency compliance, agent response time distribution, state transition timeline, retry count and backoff status, V-key delivery tracking, and concurrent agent polling load. Alerts MUST include: agent unresponsive, quote interval drift, exponential backoff triggered, max retries exceeded, verifier overloaded, and agent stuck in RETRY state.

**Trace:** Attestation Modes - Pull Mode Monitoring

```gherkin
Feature: Pull Mode Monitoring

  Scenario: Alert on max retries exceeded
    Given agent "agent-042" has exceeded the configured max_retries of 5
    When the System evaluates pull mode agent health
    Then a WARNING alert MUST be raised indicating "max retries exceeded"

  Scenario: Detect verifier overload
    Given the verifier's pending verification queue is growing
    And the queue depth exceeds the configured threshold
    When the System monitors pull mode metrics
    Then a WARNING alert MUST indicate "verifier overloaded (queue growing)"
```

### FR-055: Push Mode Attestation Monitoring

**Description:** The System MUST provide monitoring specific to push-mode (v3 API) attestation including: agent submission frequency, nonce expiry rate, evidence evaluation time, session token lifecycle, challenge lifetime utilization, and capabilities negotiation outcomes. Alerts MUST include: agent silent, high nonce expiry rate, rate limit violations, session creation failures, verification timeout exceeded, and evidence chain broken.

**Trace:** Attestation Modes - Push Mode Monitoring

```gherkin
Feature: Push Mode Monitoring

  Scenario: Alert on rate limit violations
    Given agent "agent-100" exceeds the per-agent rate limit for attestation submissions
    When the rate limit violation is detected
    Then a WARNING alert MUST be raised
    And the alert MUST indicate the agent ID and the limit threshold

  Scenario: Push mode monitoring in pull-only deployment
    Given the deployment operates exclusively in pull mode (v2 API)
    When the user navigates to push mode monitoring
    Then the System MUST display "Push mode is not enabled in this deployment"

  Scenario: Track session token lifecycle
    Given push mode sessions are active
    When the user views push mode monitoring
    Then each session MUST show creation time, expiry time, and associated agent UUID
```

### FR-056: Mixed Mode Unified Views

**Description:** In deployments with both pull-mode and push-mode agents, the System MUST provide unified views that normalize metrics across modes. The System MUST also offer mode-specific drill-down views. The dashboard MUST adapt its displays based on the configured attestation mode.

**Trace:** Attestation Modes - Comparative View

```gherkin
Feature: Mixed Mode Unified Views

  Scenario: Unified fleet view with mixed modes
    Given the deployment has 200 agents in pull mode (v2 API) and 50 agents in push mode (v3 API)
    When the user views the Fleet Overview Dashboard
    Then all 250 agents MUST appear in the unified agent list
    And each agent MUST indicate its attestation mode
    And KPIs MUST aggregate across both modes

  Scenario: Mode-specific drill-down
    Given the dashboard detects agents operating in push mode
    When the user navigates to the Attestation Analytics view
    Then a mode toggle MUST allow switching between unified, pull-only, and push-only views

  Scenario: Single-mode deployment hides mode toggle
    Given the deployment operates exclusively in pull mode (v2 API)
    When the user views the Fleet Overview Dashboard
    Then the mode toggle MUST NOT be displayed
    And all views MUST default to pull-mode data
```

### FR-057: Backend Connectivity Status Dashboard

**Description:** The System MUST display a real-time connectivity status dashboard for all Keylime backend services. Monitored services MUST include: Core Services (Verifier, Registrar) with endpoint address, UP/DOWN/HIGH LOAD status, and uptime; Database Backends (Verifier DB, Registrar DB) with average query time and migration revision; Durable Attestation Backends (Rekor, Redis, RFC 3161 TSA) with latency and connection state; and Notification Channels (ZeroMQ, Webhook, Agent notifications) with delivery status. Each service MUST display a health indicator (green/yellow/red).

**Trace:** Integration Status - Backend Connectivity

```gherkin
Feature: Backend Connectivity Status

  Scenario: Display core service connectivity
    Given the Verifier is running at 10.0.0.1:8881 with 45 days uptime
    And a second Verifier at 10.0.0.3:8881 is under HIGH LOAD at 72%
    When the user navigates to the Integration Status view
    Then the Verifier (10.0.0.1:8881) MUST show status "UP" with uptime "45d"
    And the Verifier-02 (10.0.0.3:8881) MUST show status "HIGH LOAD" with a yellow indicator

  Scenario: Alert on backend service failure
    Given the RFC 3161 TSA at tsa.example.com is configured
    When the TSA connection times out
    Then the TSA entry MUST show status "TIMEOUT" with a red indicator
    And a WARNING alert MUST be raised indicating TSA unavailability

  Scenario: All backend services healthy
    Given all configured backend services are operational
    When the user navigates to the Integration Status view
    Then all services MUST show green health indicators
    And no alerts MUST be displayed for backend connectivity
```

### FR-058: Durable Attestation Backend Monitoring

**Description:** The System MUST monitor all durable attestation backends: Rekor (transparency log) with upload latency and inclusion proofs, Redis (time-series data) with latency, memory usage, and key count, SQL DB (persistent storage) with query time and storage growth, File-based (local audit trail) with disk usage and write speed, and RFC 3161 TSA (timestamping authority) with response time and token validity. Alerts MUST fire on: upload failures, log unavailable, connection lost, high latency, storage full, slow queries, disk full, permission errors, TSA unreachable, and certificate expired.

**Trace:** Integration Status - Durable Attestation

```gherkin
Feature: Durable Attestation Backend Monitoring

  Scenario: Monitor Rekor transparency log health
    Given Rekor is configured at transparency-log.example.com
    When the System polls Rekor status
    Then the dashboard MUST display upload latency and inclusion proof success rate
    And an alert MUST fire if Rekor uploads fail for more than 5 consecutive attempts

  Scenario: Alert on Redis connection loss
    Given Redis is configured at redis.internal:6379
    When the Redis connection is lost
    Then the System MUST display "disconnected" for the Redis backend
    And a CRITICAL alert MUST be raised indicating "attestation data backend unavailable"

  Scenario: Durable backend not configured
    Given no Rekor transparency log is configured for the deployment
    When the user views the Durable Attestation Backend panel
    Then the Rekor entry MUST display "Not Configured"
    And no alerts MUST be raised for Rekor status
```

### FR-059: Compliance Framework Mapping Reports

**Description:** The System MUST map attestation capabilities to specific compliance framework controls. Supported frameworks MUST include: NIST SP 800-155 (BIOS integrity measurement), NIST SP 800-193 (platform firmware resilience), PCI DSS 4.0 (Req 11.5 file integrity monitoring, Req 10.2 audit trail), SOC 2 Type II (CC7.1 monitoring activities, CC6.1 logical access controls), FedRAMP (CA-7 continuous monitoring, SI-7 software integrity), and CIS Controls v8 (2.5 allowlisted software). Each mapping MUST identify the relevant dashboard evidence.

**Trace:** Compliance - Framework Mapping

```gherkin
Feature: Compliance Framework Mapping

  Scenario: View PCI DSS compliance mapping
    Given the System has IMA attestation data for the fleet
    When the user selects the PCI DSS 4.0 compliance report
    Then the report MUST map Req 11.5 to IMA attestation history per agent
    And the report MUST map Req 10.2 to the tamper-evident audit log export
    And each control MUST indicate its coverage status (Covered / Partial / Gap)

  Scenario: View FedRAMP compliance mapping
    Given the System has real-time attestation metrics
    When the user selects the FedRAMP compliance report
    Then control CA-7 MUST map to real-time attestation success rate
    And control SI-7 MUST map to IMA policy enforcement reports

  Scenario: Compliance gap identified
    Given IMA attestation is not enabled for 10 agents in the fleet
    When the user views the PCI DSS 4.0 compliance report
    Then Req 11.5 coverage status MUST display "Partial" or "Gap"
    And the gap detail MUST list the 10 agents without IMA attestation
```

### FR-060: One-Click Compliance Report Export

**Description:** The System MUST support one-click export of compliance reports in PDF and CSV formats. Each report MUST include: attestation coverage summary, policy compliance matrix, exception list with justifications, and time-bounded audit evidence packages. Reports MUST be filterable by compliance framework.

**Trace:** Compliance - Framework Mapping

```gherkin
Feature: One-Click Compliance Report Export

  Scenario: Export PDF compliance report
    Given the user is viewing the SOC 2 Type II compliance mapping
    And the user has the Operator or Admin role
    When the user clicks "Export PDF"
    Then a PDF report MUST be generated containing the compliance mapping, attestation coverage summary, and exception list
    And the PDF MUST include a timestamp and the generating user's identity

  Scenario: Export time-bounded audit evidence package
    Given the user selects date range "2026-01-01" to "2026-03-31"
    When the user exports the compliance evidence package
    Then the export MUST include all audit log entries within the date range
    And the export MUST include attestation pass/fail summaries per agent for the period

  Scenario: Compliance report export denied for Viewer
    Given the user has the Viewer role
    When the user views a compliance report
    Then the Export action MUST NOT be rendered for the Viewer role

> **Reviewer Note:** Changed "MUST be disabled or hidden" to "MUST NOT be rendered" — RBAC-denied export capability should be absent for the Viewer role. Also changed When step from "attempts to export" (presupposes the control exists) to "views a compliance report" (observable state).
```

### FR-061: Tamper-Evident Hash-Chained Audit Logging

**Description:** The System MUST implement tamper-evident audit logging where each log entry includes a SHA-256 hash of the previous entry. The chain root MUST be anchored to an external RFC 3161 timestamp. Periodic chain checkpoints MUST be submitted to a Rekor transparency log. Tamper detection MUST run on startup and periodically. Each entry MUST include: timestamp (UTC, millisecond precision), actor (user identity from OIDC), action (CRUD + target), resource, source IP, user agent, result, and previous entry hash.

**Trace:** Compliance - Tamper-Evident Audit Logging

```gherkin
Feature: Tamper-Evident Audit Logging

  Scenario: Detect hash chain integrity violation
    Given the audit log contains 1,000 hash-chained entries
    And entry #500 has been modified such that its hash no longer matches entry #501's previous-hash field
    When the System runs periodic hash chain verification
    Then the verification MUST report a chain break at entry #501
    And the System MUST raise a CRITICAL alert indicating audit log integrity violation

  Scenario: Audit log entry structure
    Given user "admin@example.com" updates IMA policy "production-v2"
    When the audit log entry is created
    Then the entry MUST include timestamp in UTC with millisecond precision
    And the entry MUST include actor "admin@example.com"
    And the entry MUST include action "UPDATE_RUNTIME_POLICY"
    And the entry MUST include the SHA-256 hash of the previous entry
```

### FR-062: Incident Response Ticketing Integration

**Description:** The System SHOULD integrate with enterprise incident response and ticketing systems: ServiceNow (incident + change request), Jira (issue auto-creation), PagerDuty (alert routing), and OpsGenie (on-call escalation). Integration MUST support bidirectional status sync. Change management (ITSM) SHOULD gate policy changes on an approved change request. Emergency bypass MUST include a break-glass audit trail. Automated remediation SHOULD support: network quarantine via NAC integration, agent isolation, re-provisioning workflow triggers, configurable playbooks per failure type, and manual approval gate for destructive actions.

**Trace:** Incident Response - Integration

```gherkin
Feature: Incident Response Ticketing Integration

  Scenario: Auto-create Jira issue on attestation failure
    Given the Jira integration is configured with project "SECOPS"
    And agent "agent-042" fails attestation with a CRITICAL severity
    When the incident response workflow triggers
    Then a Jira issue SHOULD be created in project "SECOPS"
    And the issue SHOULD include agent UUID, failure type, timestamp, and link to dashboard

  Scenario: Bidirectional status sync with ServiceNow
    Given a ServiceNow incident INC0012345 was created for agent "agent-042"
    When the ServiceNow incident is resolved externally
    Then the dashboard alert status SHOULD sync to "Resolved"
    And the resolution source MUST be recorded as "ServiceNow INC0012345"

  Scenario: Ticketing integration not configured
    Given no incident response integration is configured
    When an attestation failure triggers the incident response workflow
    Then the System MUST create a local incident record only
    And the integration panel MUST display "No ticketing integrations configured"
```

### FR-063: SIEM Integration

**Description:** The System MUST integrate with Security Information and Event Management (SIEM) systems. Supported formats and protocols MUST include: Syslog (CEF/LEEF format), Splunk HEC (HTTP Event Collector), Elastic Common Schema (ECS), Prometheus metrics endpoint, and OpenTelemetry traces. SIEM integration MUST be available from Day 1 of deployment.

**Trace:** Incident Response - Integration

```gherkin
Feature: SIEM Integration

  Scenario: Export events via Syslog CEF format
    Given Syslog integration is configured with endpoint "siem.example.com:514"
    When an attestation failure event occurs for agent "agent-042"
    Then the System MUST emit a Syslog message in CEF format
    And the message MUST include severity, agent UUID, failure type, and timestamp

  Scenario: Expose Prometheus metrics endpoint
    Given the Prometheus integration is enabled
    When Prometheus scrapes the /metrics endpoint
    Then the endpoint MUST expose attestation success rate, alert counts, agent fleet size, and API response times
    And each metric MUST include appropriate labels (agent_id, severity, mode)

  Scenario: SIEM endpoint unreachable
    Given Syslog integration is configured with endpoint "siem.example.com:514"
    When the endpoint becomes unreachable
    Then the System MUST log the delivery failure and retry according to the configured policy
    And a WARNING alert MUST be raised indicating "SIEM endpoint unreachable"
```

### FR-064: Verifier Cluster Performance Monitoring

**Description:** The System MUST monitor verifier cluster performance including: CPU utilization per worker process, memory usage (RSS, heap), open file descriptors, thread pool utilization, network connections (active/idle), attestations per second, queue depth (pending verifications), worker split (web vs. verification), `dedicated_web_workers` utilization, exponential backoff events, and rate limit rejections.

**Trace:** System Performance - Verifier Metrics

```gherkin
Feature: Verifier Cluster Performance Monitoring

  Scenario: Display verifier node resource metrics
    Given the deployment has two verifier nodes and one registrar
    When the user navigates to System Performance
    Then each verifier node MUST display CPU utilization, memory usage, and connection counts
    And the registrar MUST display its own resource metrics independently

  Scenario: Alert on verifier overload
    Given Verifier-02 CPU utilization exceeds 70%
    When the System evaluates verifier health
    Then Verifier-02 MUST display a yellow "HIGH LOAD" indicator
    And a WARNING alert MUST be raised indicating the affected node

  Scenario: Verifier node unreachable
    Given the deployment has two verifier nodes
    And Verifier-02 is unreachable
    When the System polls verifier health
    Then Verifier-02 MUST display a red "DOWN" indicator
    And a CRITICAL alert MUST be raised indicating "Verifier-02 unreachable"
```

### FR-065: Database Connection Pool Monitoring

**Description:** The System MUST monitor the Keylime database connection pool: active/idle connections, pool size and overflow count, connection wait time, pool exhaustion events, and `database_pool_sz_ovfl` settings. Query performance metrics MUST include: slow query detection (>100ms), query count by type (SELECT/UPDATE), table row counts, index hit ratio, and migration status (Alembic revision).

**Trace:** System Performance - Verifier Metrics

```gherkin
Feature: Database Connection Pool Monitoring

  Scenario: Display connection pool status
    Given the database connection pool has 10 active and 4 idle connections out of 14 total
    And overflow is at 0
    When the user views the Database Connection Pool panel
    Then the panel MUST display "Active: 10/14 | Idle: 4 | Overflow: 0"
    And average query time and max query time MUST be displayed

  Scenario: Alert on pool exhaustion
    Given all 14 connections are active and overflow reaches the configured maximum
    When a new connection request is queued
    Then the System MUST raise a CRITICAL alert indicating "database pool exhausted"
    And the connection wait time MUST be displayed

  Scenario: Slow query detected
    Given a database query takes 250ms (threshold: 100ms)
    When the System monitors query performance
    Then the slow query MUST be logged with query type and duration
    And the slow query count metric MUST increment
```

### FR-066: API Response Time Tracking

**Description:** The System MUST track API response times at p50, p95, and p99 percentiles for all Keylime API endpoints. Tracked endpoints MUST include at minimum: `GET /agents/`, `GET /agents/:id`, `POST /attestations`, `PATCH /attestations/:idx`, `GET /policies/ima`, and `POST /sessions`. Response times MUST be color-coded: green (<threshold), yellow (warning), red (critical).

**Trace:** System Performance - Verifier Metrics

```gherkin
Feature: API Response Time Tracking

  Scenario: Display API response time percentiles
    Given the System has collected response time data for GET /agents/
    When the user views the API Response Times panel
    Then p50, p95, and p99 response times MUST be displayed for GET /agents/
    And each percentile MUST be color-coded based on configured thresholds

  Scenario: Alert on degraded API response times
    Given the p99 response time for PATCH /attestations/:idx exceeds 1200ms
    And the critical threshold is configured at 1000ms
    When the System evaluates API performance
    Then the PATCH /attestations/:idx p99 value MUST display in red
    And a WARNING alert MUST be raised indicating API latency degradation

  Scenario: API endpoint returns errors
    Given the POST /attestations endpoint returns HTTP 500 errors
    When the System tracks API response codes
    Then the error rate MUST be displayed alongside response times
    And a CRITICAL alert MUST be raised if the error rate exceeds the configured threshold
```

### FR-067: Live Configuration View with Drift Detection

**Description:** The System MUST display the current runtime configuration of the Verifier and Registrar, including key settings such as `quote_interval`, `retry_interval`, `max_retries`, `num_workers`, `database_pool_sz_ovfl`, `challenge_lifetime`, `mode`, and `enable_agent_mtls`. Each setting MUST indicate whether it is at its default value or modified. The System MUST alert on unplanned configuration changes that differ from the expected baseline.

**Trace:** System Performance - Configuration Monitoring

```gherkin
Feature: Configuration Drift Detection

  Scenario: Detect non-default configuration
    Given the verifier has "num_workers" set to 8 (default: 0/auto)
    And the verifier has "mode" set to "push" (default: "pull")
    When the user views the configuration monitoring panel
    Then "num_workers" MUST display with a "Modified" indicator
    And "mode" MUST display with a "Modified" indicator
    And settings at their default values MUST display with a "Default" indicator

  Scenario: Configuration drift detected
    Given the verifier's "quote_interval" was set to 30s in the baseline
    When the running configuration shows "quote_interval" changed to 60s
    Then the System MUST raise a WARNING alert "Configuration drift detected"
    And the changed setting MUST display both the baseline and current values
```

### FR-068: Capacity Planning Projections

**Description:** The System SHOULD provide capacity planning projections based on current attestation rate and agent growth trends. Projections SHOULD indicate when the verifier will need horizontal scaling, when database storage will reach capacity, and when connection pool limits will be approached.

**Trace:** System Performance - Key Metrics

```gherkin
Feature: Capacity Planning Projections

  Scenario: Project verifier scaling needs
    Given the current attestation rate is 500 attestations/second
    And the agent fleet is growing at 50 agents/week
    When the user views the Capacity Planning panel
    Then the System SHOULD display a projected date for when the verifier will reach capacity
    And the projection SHOULD include current utilization trend and estimated headroom

  Scenario: Project database storage growth
    Given TimescaleDB is currently using 120GB of 500GB available storage
    And storage is growing at 2GB/day
    When the user views storage projections
    Then the System SHOULD project the date when storage will reach 80% capacity
    And a WARNING SHOULD be raised if the projected date is within 30 days

  Scenario: Insufficient data for projections
    Given the System has been running for less than 24 hours
    When the user views the Capacity Planning panel
    Then the System SHOULD display "Insufficient historical data for projections"
    And no projected dates SHOULD be shown
```

### FR-069: Agent State Machine Visualization

**Description:** The System MUST provide visual state machine diagrams for agent lifecycle states. Pull mode (v2 API) MUST display all 10 states: REGISTERED (0), START (1), SAVED (2), GET_QUOTE (3), RETRY (4), PROVIDE_V (5), FAILED (7), TERMINATED (8), INVALID_QUOTE (9), and TENANT_FAILED (10) with transition arrows. Push mode (v3 API) MUST display three computed states: PASS (100), FAIL (101), and PENDING (102), derived from the agent's attestation results. The push-mode verification flow follows 3 stages: capabilities submission, challenge/nonce response, and evidence evaluation. Failed states MUST be visually distinguished (e.g., red). The current state distribution across the fleet MUST be overlaid on the diagram.

**Trace:** Keylime - Agent State Machine; Attestation Modes - Push Mode

```gherkin
Feature: Agent State Machine Visualization

  Scenario: Display pull mode state distribution
    Given 247 agents are in GET_QUOTE state, 3 in FAILED, and 1 in RETRY
    When the user views the Agent State Machine visualization for pull mode
    Then all 10 states MUST be displayed as a directed graph with transition arrows
    And GET_QUOTE MUST show count "247"
    And FAILED MUST show count "3" with a red visual indicator
    And RETRY MUST show count "1"

  Scenario: Display push mode computed states
    Given the deployment includes push mode (v3 API) agents
    When the user views the push mode state visualization
    Then three computed states MUST be displayed: PASS (100), FAIL (101), and PENDING (102)
    And each state MUST show the current count of agents in that state
    And the 3-stage verification flow (capabilities → challenge → evidence) MUST be shown alongside

  Scenario: No agents in a specific state
    Given no agents are in FAILED state
    When the user views the Agent State Machine visualization
    Then the FAILED state node MUST show count "0"
    And the FAILED state MUST still be visible in the diagram but with a neutral indicator
```

### FR-070: API Version Distribution Visualization

**Description:** The System MUST display the distribution of Keylime API versions in use across the agent fleet. The visualization MUST show agent counts per API version (e.g., v2.0, v2.1, …, v2.5, v3.0) and the server's supported API versions. This enables operators to track fleet API migration progress.

**Trace:** Integration Status - Backend Connectivity

```gherkin
Feature: API Version Distribution Visualization

  Scenario: Display agent API version distribution
    Given the fleet has agents using API versions v2.0 through v3.0
    When the user views the API Version Distribution panel
    Then a bar chart MUST display agent counts per API version
    And the server's supported API versions MUST be indicated

  Scenario: Identify outdated agents
    Given the server supports v2.5 and v3.0
    And 15 agents are still using v2.0
    When the user views the API Version Distribution
    Then agents on v2.0 MUST be highlighted as running an older API version
    And the System SHOULD display a recommendation to upgrade

  Scenario: Unknown API version reported
    Given an agent reports an API version not recognized by the System
    When the user views the API Version Distribution
    Then the unknown version MUST be displayed in a separate "Unknown" category
    And a WARNING MUST be raised indicating an unrecognized API version
```

### FR-071: AI Assistant with Keylime MCP Integration

**Description:** The System SHOULD provide an AI Assistant module accessible from the sidebar that offers a conversational interface for interacting with the Keylime infrastructure via the Model Context Protocol (MCP). The assistant MUST connect to a Keylime MCP server that exposes Keylime Verifier and Registrar operations as MCP tools. The assistant MUST allow users to query fleet status, investigate attestation failures, inspect agent details, review policies, and request recommended actions using natural language. All assistant interactions MUST respect the user's RBAC role — the MCP server MUST enforce the same permission boundaries as the REST API. The assistant MUST NOT execute write operations (policy changes, agent actions) without explicit user confirmation. All assistant queries and responses MUST be logged in the audit trail.

**Trace:** AI Assistant - Conversational Interface

```gherkin
Feature: AI Assistant with Keylime MCP

  Scenario: Query fleet status via natural language
    Given the user is authenticated with the Operator role
    And the AI Assistant view is displayed
    When the user types "How many agents are currently failing?"
    Then the assistant MUST query the Keylime MCP server for agent fleet status
    And the response MUST display the count of agents in FAILED, INVALID_QUOTE, and TENANT_FAILED states
    And the response MUST include agent identifiers for failed agents

  Scenario: Investigate attestation failure
    Given the user is viewing the AI Assistant
    When the user types "Why did agent d432fbb3 fail its last attestation?"
    Then the assistant MUST retrieve the agent's attestation history and failure details via MCP
    And the response MUST include the failure type, timestamp, and affected pipeline stage
    And the assistant SHOULD suggest a recommended action based on the failure type

  Scenario: Request policy information
    Given the user is viewing the AI Assistant
    When the user types "Which agents are assigned to policy production-v1?"
    Then the assistant MUST query the policy assignment matrix via MCP
    And the response MUST list the agents assigned to "production-v1"

  Scenario: Write operation requires explicit confirmation
    Given the user has the Admin role
    And the user is viewing the AI Assistant
    When the user types "Reactivate agent a1b2c3d4"
    Then the assistant MUST display a confirmation prompt: "This will reactivate agent a1b2c3d4. Confirm? [Yes/No]"
    And the assistant MUST NOT execute the action until the user confirms
    And the confirmation and outcome MUST be recorded in the audit log

  Scenario: RBAC enforcement on assistant queries
    Given the user has the Viewer role
    When the user types "Delete agent a1b2c3d4" in the AI Assistant
    Then the assistant MUST deny the request with "Insufficient permissions — Admin role required for agent deletion"
    And the denied attempt MUST be recorded in the audit log

  Scenario: MCP server unreachable
    Given the Keylime MCP server is not configured or unreachable
    When the user navigates to the AI Assistant view
    Then the view MUST display "AI Assistant unavailable — MCP server not connected"
    And a "Configure MCP" link SHOULD direct Admin users to the Settings view

  Scenario: Contextual follow-up queries
    Given the user previously asked "Show me all failing agents"
    And the assistant displayed 3 failing agents
    When the user types "Tell me more about the first one"
    Then the assistant MUST resolve the reference to the first agent from the previous response
    And the assistant MUST retrieve and display the agent's detail information via MCP

  Scenario: Assistant interaction logged in audit trail
    Given the user submits a query to the AI Assistant
    When the assistant processes the query and returns a response
    Then an audit log entry MUST be created with action "AI_ASSISTANT_QUERY"
    And the entry MUST include the actor, MCP tools invoked, resources accessed, and timestamp
    And raw query text MUST NOT be stored if it may contain sensitive data

  Scenario: AI Assistant not available for unauthenticated users
    Given the user is not authenticated
    When the user attempts to access the AI Assistant route
    Then the System MUST redirect the user to the login page
```

---

## 4. Non-Functional Requirements Detail

### NFR-001: KPI Data Refresh Latency

**Description:** The System MUST refresh KPI data within 30 seconds of a state change occurring in the Keylime backend. The default refresh interval MUST be configurable.

**Trace:** Dashboard - Key Performance Indicators

```gherkin
Feature: KPI Data Refresh Latency

  Scenario: KPI data refreshes within threshold
    Given auto-refresh is enabled with the default 30-second interval
    When an agent state change occurs in the Verifier
    Then the dashboard KPI data MUST reflect the change within 30 seconds

  Scenario: KPI refresh exceeds threshold
    Given auto-refresh is enabled
    When KPI data has not been updated for more than 30 seconds after a known state change
    Then the System MUST display a staleness warning on affected KPIs
```

### NFR-002: Dual API Version Support

**Description:** The System MUST support Keylime API v2 (pull mode) and v3 (push mode) simultaneously. The backend MUST detect the API version per agent and adapt its data collection strategy accordingly.

**Trace:** Keylime - Data Model Overview

```gherkin
Feature: Dual API Version Support

  Scenario: Simultaneous v2 and v3 agent handling
    Given the fleet contains agents using API v2 and agents using API v3
    When the System ingests data from the Verifier
    Then v2 agents MUST be polled using GET endpoints
    And v3 agents MUST receive push-mode events
    And both agent types MUST appear in the unified fleet view

  Scenario: Unsupported API version rejected
    Given an agent reports an API version not supported by the System
    When the System attempts to ingest data from that agent
    Then the System MUST log a WARNING indicating "unsupported API version"
    And the agent MUST be displayed with a "version incompatible" status
```

### NFR-003: Non-Invasive Architecture

**Description:** The System MUST operate as a standalone component that consumes Keylime's public REST APIs. The System MUST NOT require modifications to Keylime Verifier, Registrar, or Agent components. The System MUST NOT access Keylime databases directly.

**Trace:** Technical Architecture - System Design

```gherkin
Feature: Non-Invasive Architecture

  Scenario: System operates without Keylime modifications
    Given the Keylime Verifier and Registrar are running with default configuration
    When the dashboard backend starts and connects via mTLS
    Then the System MUST retrieve all data exclusively through public REST API endpoints
    And no direct database connections MUST be established to Keylime databases

  Scenario: System degrades when API is unavailable
    Given the Keylime Verifier API is unreachable
    When the System attempts to fetch data
    Then the System MUST display cached data with staleness indicators
    And the System MUST NOT fall back to direct database access
```

### NFR-004: Single Page Application Frontend

**Description:** The frontend MUST render as a Single Page Application with client-side routing. Initial page load MUST complete within 3 seconds on a 10 Mbps connection. The UI framework MUST support component-based architecture with type safety.

**Trace:** Technical Architecture - Data Flow

```gherkin
Feature: SPA Frontend Performance

  Scenario: Initial page load within performance budget
    Given the user accesses the dashboard for the first time
    When the browser loads the application on a 10 Mbps connection
    Then the initial page render MUST complete within 3 seconds
    And subsequent navigation between views MUST NOT trigger full page reloads

  Scenario: Client-side routing
    Given the user is viewing the Fleet Overview Dashboard
    When the user clicks "Agents" in the sidebar
    Then the browser URL MUST update without a full page reload
    And the Agent Fleet view MUST render via client-side routing
```

### NFR-005: Async Backend Performance

**Description:** The backend MUST support 10,000 concurrent WebSocket connections with less than 100ms p99 latency on a single node. The backend MUST use asynchronous I/O for all network operations.

**Trace:** Technical Architecture - Why Rust

```gherkin
Feature: Backend Performance

  Scenario: Handle 10,000 concurrent WebSocket connections
    Given the backend is running on a single node
    When 10,000 clients establish WebSocket connections simultaneously
    Then all connections MUST be maintained without connection drops
    And p99 message delivery latency MUST remain below 100ms

  Scenario: Backend handles API requests under load
    Given 500 concurrent HTTP requests are sent to the REST API
    When the backend processes the requests
    Then p99 response time MUST remain below 200ms
    And no requests MUST time out
```

### NFR-006: Event-Driven Ingestion

**Description:** The System MUST use event-driven ingestion as the primary data path. The System MUST consume ZeroMQ revocation events from the Verifier. The System SHOULD propose upstream state-change webhook/AMQP emit. A message broker (RabbitMQ/Kafka) SHOULD decouple load. The System MUST use event sequence numbers to detect gaps. A periodic reconciliation sweep MUST run every 5 minutes.

**Trace:** Technical Architecture - Event-Driven Ingestion; Scalability - Ingestion Model Comparison

```gherkin
Feature: Event-Driven Ingestion

  Scenario: Consume ZeroMQ revocation event
    Given the System is subscribed to the Verifier's ZeroMQ revocation channel
    When the Verifier publishes a revocation event for agent "agent-042"
    Then the System MUST process the event and update the agent's status
    And the event sequence number MUST be recorded

  Scenario: Detect event sequence gap
    Given the System has processed events up to sequence number 100
    When the next received event has sequence number 103
    Then the System MUST detect a gap of 2 missing events
    And the System MUST trigger an immediate reconciliation for the missing events
```

### NFR-007: Polling Fallback with Backpressure

**Description:** For environments without event support, the System MUST implement adaptive polling with backpressure: list poll at 30s intervals using ETag/If-Modified-Since, detail polling on-demand only (user clicks), double the interval if p95 latency exceeds 500ms, and pause detail polling if p95 latency exceeds 2s. The System MUST alert the operator when degraded to polling mode.

**Trace:** Scalability - Ingestion Model Comparison; Technical Architecture - Event-Driven Ingestion

```gherkin
Feature: Polling Fallback with Backpressure

  Scenario: Adaptive polling interval increase under load
    Given the System is operating in polling mode
    And the p95 API latency exceeds 500ms
    When the next polling cycle is scheduled
    Then the polling interval MUST be doubled from its current value
    And the System MUST alert the operator that backpressure is active

  Scenario: Pause detail polling under extreme latency
    Given the p95 API latency exceeds 2 seconds
    When the System evaluates polling strategy
    Then detail polling MUST be paused entirely
    And only list-level polling MUST continue
    And a WARNING alert MUST indicate "detail polling paused due to high latency"
```

### NFR-008: Event-Driven Scalability

**Description:** The System SHOULD scale to 100,000+ agents under event-driven ingestion mode. Performance MUST NOT degrade linearly with agent count when using event-driven ingestion.

**Trace:** Scalability - Ingestion Model Comparison

```gherkin
Feature: Event-Driven Scalability

  Scenario: Dashboard responsive at 100K agents
    Given the fleet contains 100,000 agents in event-driven mode
    When the user navigates to the Fleet Overview Dashboard
    Then the KPI data SHOULD render within 5 seconds
    And the agent list SHOULD paginate without blocking the UI
```

### NFR-009: Polling Fallback Scalability

**Description:** The System MUST support approximately 1,000 agents under polling fallback mode without exceeding acceptable API load on the Keylime Verifier.

**Trace:** Scalability - Ingestion Model Comparison

```gherkin
Feature: Polling Fallback Scalability

  Scenario: Polling mode at 1,000 agents
    Given the fleet contains 1,000 agents in polling mode
    When the System executes a polling cycle
    Then all agent states MUST be refreshed within 60 seconds
    And the API load on the Verifier MUST NOT exceed 50 requests per second
```

### NFR-010: Active/Passive High Availability

**Description:** The System MUST support Active/Passive HA deployment with less than 30 seconds Recovery Time Objective (RTO) and zero Recovery Point Objective (RPO) for committed transactions.

**Trace:** High Availability - Architecture

```gherkin
Feature: Active/Passive HA

  Scenario: Failover to standby node
    Given the System is deployed in Active/Passive HA mode
    When the active node becomes unavailable
    Then the standby node MUST assume the active role within 30 seconds
    And no committed data MUST be lost (0 RPO)
    And WebSocket clients MUST reconnect to the new active node
```

### NFR-011: Active/Active High Availability

**Description:** The System SHOULD support Active/Active HA deployment for environments with 5,000+ agents, distributing load across multiple backend nodes.

**Trace:** High Availability - Architecture

```gherkin
Feature: Active/Active HA

  Scenario: Load distribution across nodes
    Given the System is deployed in Active/Active mode with 2 backend nodes
    When 5,000 agents are being monitored
    Then each node SHOULD handle approximately half of the agent data ingestion
    And failure of one node SHOULD result in the surviving node handling the full load
```

### NFR-012: Air-Gapped Deployment

**Description:** The System MUST be fully self-contained with no runtime internet access required. All frontend assets MUST be bundled (no CDN). The Rust backend MUST compile to a single binary with no runtime dependencies. Fonts and icons MUST be embedded. Pre-built container images MUST include all layers. Offline EK certificate validation MUST be supported via pre-loaded TPM vendor CA certificates. Update packages MUST be signed (GPG) with integrity verification and SBOM included.

**Trace:** Deployment - Offline & Air-Gapped

```gherkin
Feature: Air-Gapped Deployment

  Scenario: No external network requests at runtime
    Given the System is deployed in an air-gapped environment
    When the System starts and operates normally
    Then the System MUST NOT make any outbound network requests to the internet
    And all UI assets (fonts, icons, scripts) MUST load from bundled resources

  Scenario: Offline EK certificate validation
    Given the TPM vendor CA certificates are pre-loaded in the System
    When an agent registers with an EK certificate
    Then the System MUST validate the EK certificate against the pre-loaded CA bundle
    And no network request to external CRLs or OCSP responders MUST be made
```

### NFR-013: Self-Contained Packaging

**Description:** The System MUST be packaged as a self-contained application. The backend MUST compile to a single binary. The frontend MUST be bundled with all assets. No CDN or external dependency downloads MUST be required at runtime.

**Trace:** Deployment - Offline & Air-Gapped

```gherkin
Feature: Self-Contained Packaging

  Scenario: Single binary backend deployment
    Given the backend binary is deployed to a server
    When the binary is executed
    Then the backend MUST start without requiring additional runtime downloads
    And all embedded assets MUST be served from the binary

  Scenario: Frontend loads without CDN
    Given the user accesses the dashboard in an environment with no internet
    When the browser loads the application
    Then all JavaScript, CSS, fonts, and icons MUST load from the backend server
    And no requests to external CDNs MUST be attempted
```

### NFR-014: Accessibility

**Description:** The dashboard UI MUST conform to WCAG 2.1 Level AA: keyboard navigation, screen reader support, color-contrast compliance, and ARIA labels on all interactive elements. This is REQUIRED for government procurement.

**Trace:** Deployment - Offline & Air-Gapped

```gherkin
Feature: WCAG 2.1 Level AA Accessibility

  Scenario: Keyboard navigation through dashboard
    Given the user navigates the dashboard using only the keyboard
    When the user presses Tab to move between interactive elements
    Then every interactive element MUST receive visible focus
    And the user MUST be able to activate any control using Enter or Space

  Scenario: Screen reader announces dashboard elements
    Given a screen reader is active
    When the user navigates to the Fleet Overview Dashboard
    Then all KPI values MUST have ARIA labels describing the metric name and value
    And all interactive elements MUST have descriptive ARIA labels
```

### NFR-015: Multiple Deployment Options

**Description:** The System MUST support deployment via OCI container images, Kubernetes Helm charts, RPM packages, and systemd services. Each deployment method MUST be documented and tested.

**Trace:** Technical Architecture - Deployment

```gherkin
Feature: Multiple Deployment Options

  Scenario: Deploy via Helm chart
    Given a Kubernetes cluster is available
    When the operator installs the Helm chart with default values
    Then the backend, frontend, and database components MUST deploy successfully
    And the dashboard MUST be accessible via the configured Ingress

  Scenario: Deploy via RPM package
    Given a Fedora/RHEL system is available
    When the operator installs the RPM package
    Then a systemd service MUST be created and enabled
    And the dashboard MUST start via systemctl
```

### NFR-016: Graceful Degradation

**Description:** The System MUST degrade gracefully when backend components are unavailable: if TimescaleDB is down, show live data only with no history; if Redis is down, make direct API calls with no cache; if apalis workers are down, support manual refresh only; if the Keylime API is down, show cached state with an alert.

**Trace:** High Availability - Architecture

```gherkin
Feature: Graceful Degradation

  Scenario: TimescaleDB unavailable
    Given TimescaleDB is down
    When the user views the dashboard
    Then the System MUST display live data from the Keylime API
    And historical charts MUST display "Historical data unavailable"
    And a banner MUST indicate the database is unreachable

  Scenario: Redis cache unavailable
    Given Redis is down
    When the System needs to fetch agent data
    Then the System MUST make direct API calls to the Keylime Verifier
    And a WARNING banner MUST indicate "Cache unavailable — increased API load"

  Scenario: Keylime API unavailable
    Given the Keylime Verifier API is unreachable
    When the user views the dashboard
    Then the System MUST display cached data with staleness timestamps
    And a CRITICAL banner MUST indicate "Keylime API unreachable — data may be stale"
```

### NFR-017: Circuit Breaker on Verifier API

**Description:** The System MUST implement a circuit breaker pattern on Verifier API calls. The circuit MUST open after a configurable number of consecutive failures (default: 5). While open, the System MUST use cached data. The circuit MUST transition to half-open after a configurable timeout to test recovery.

**Trace:** Technical Architecture - Event-Driven Ingestion

```gherkin
Feature: Circuit Breaker

  Scenario: Circuit opens after consecutive failures
    Given the Verifier API has returned errors for 5 consecutive requests
    When the System evaluates the circuit breaker
    Then the circuit MUST transition to the "open" state
    And subsequent requests MUST be served from cache without contacting the Verifier
    And a WARNING alert MUST indicate "Circuit breaker open — using cached data"

  Scenario: Circuit transitions to half-open
    Given the circuit breaker has been open for the configured timeout (default: 60s)
    When the timeout elapses
    Then the System MUST send a single probe request to the Verifier
    And if the probe succeeds, the circuit MUST close and resume normal operation
```

### NFR-018: Request Rate Limiting

**Description:** The System MUST enforce per-user and global request rate limiting to protect the Keylime Verifier and dashboard backend from overload. Rate limits MUST be configurable.

**Trace:** Technical Architecture - IMA Log & Data Decoupling

```gherkin
Feature: Request Rate Limiting

  Scenario: Per-user rate limit exceeded
    Given user "operator@example.com" is configured with a limit of 100 requests/minute
    When the user sends the 101st request within one minute
    Then the System MUST return HTTP 429 Too Many Requests
    And the response MUST include a Retry-After header

  Scenario: Global rate limit protects Verifier
    Given the global rate limit for Verifier API calls is 500 requests/second
    When aggregate dashboard requests exceed 500/second
    Then excess requests MUST be queued or rejected with HTTP 429
    And the System MUST NOT forward more than 500 requests/second to the Verifier
```

### NFR-019: Cache TTL Configuration

**Description:** The System MUST implement tiered cache TTLs: agent list 10 seconds, agent detail 30 seconds, policies 60 seconds, certificates 300 seconds. TTLs MUST be configurable.

**Trace:** Scalability - Cache Invalidation

```gherkin
Feature: Cache TTL Configuration

  Scenario: Agent list cache expires after TTL
    Given the agent list cache TTL is configured at 10 seconds
    When the user requests the agent list
    And the cache was last populated 11 seconds ago
    Then the System MUST fetch fresh data from the Verifier API
    And the cache MUST be updated with the new data

  Scenario: Certificate cache with longer TTL
    Given the certificate cache TTL is configured at 300 seconds
    When the user requests certificate data
    And the cache was populated 120 seconds ago
    Then the System MUST serve the cached certificate data
    And no request MUST be made to the Verifier API
```

### NFR-020: Periodic Reconciliation

**Description:** The System MUST run a periodic reconciliation sweep every 5 minutes to detect and correct any drift between the dashboard's cached state and the Verifier's actual state. The reconciliation interval MUST be configurable.

**Trace:** Technical Architecture - Event-Driven Ingestion

```gherkin
Feature: Periodic Reconciliation

  Scenario: Reconciliation detects state drift
    Given the dashboard shows agent "agent-042" in GET_QUOTE state
    And the Verifier reports agent "agent-042" in FAILED state
    When the 5-minute reconciliation sweep runs
    Then the dashboard MUST update agent "agent-042" to FAILED state
    And the drift MUST be logged as a reconciliation correction

  Scenario: Reconciliation finds no drift
    Given all cached agent states match the Verifier's reported states
    When the reconciliation sweep runs
    Then no state changes MUST occur
    And the reconciliation log MUST record "no drift detected"
```

### NFR-021: WebSocket Real-Time Updates

**Description:** The System MUST provide WebSocket connections for real-time UI updates. WebSocket connections MUST support automatic reconnection with exponential backoff. The System MUST indicate connection status to the user.

**Trace:** Technical Architecture - Data Flow

```gherkin
Feature: WebSocket Real-Time Updates

  Scenario: WebSocket delivers real-time updates
    Given a WebSocket connection is established between the browser and backend
    When an agent state change is ingested by the backend
    Then the change MUST be pushed to the browser via WebSocket
    And the UI MUST update without requiring a page refresh

  Scenario: WebSocket reconnection with backoff
    Given a WebSocket connection drops
    When the client attempts to reconnect
    Then the client MUST use exponential backoff (1s, 2s, 4s, 8s, ...)
    And a connection status indicator MUST show "Reconnecting..."
    And upon successful reconnection, the indicator MUST show "Connected"
```

### NFR-022: Signed Update Packages

**Description:** Offline update packages MUST be GPG-signed with integrity verification. Each update MUST include a Software Bill of Materials (SBOM). The System MUST verify the signature before applying any update.

**Trace:** Deployment - Offline & Air-Gapped

```gherkin
Feature: Signed Update Packages

  Scenario: Verify update package signature
    Given an update package is available with a GPG signature
    When the operator initiates the update
    Then the System MUST verify the GPG signature before applying changes
    And if the signature is invalid, the update MUST be rejected with error "invalid signature"

  Scenario: SBOM included in update
    Given an update package is available
    When the operator inspects the package contents
    Then an SBOM in SPDX or CycloneDX format MUST be included
```

### NFR-023: Concurrent Log Fetch Limit

**Description:** The System MUST limit concurrent IMA log fetch requests to the Verifier to a maximum of 5 parallel requests. This prevents overloading the Verifier when multiple users request agent detail views simultaneously.

**Trace:** Technical Architecture - IMA Log & Data Decoupling

```gherkin
Feature: Concurrent Log Fetch Limit

  Scenario: Enforce maximum concurrent log fetches
    Given 5 IMA log fetch requests are currently in progress
    When a 6th user requests an IMA log view for a different agent
    Then the 6th request MUST be queued until one of the active fetches completes
    And the user MUST see a "Loading — request queued" indicator

  Scenario: Concurrent fetches within limit
    Given 3 IMA log fetch requests are currently in progress
    When a 4th user requests an IMA log view
    Then the request MUST proceed immediately without queuing
```

### NFR-024: AI Assistant Query Performance and Rate Limiting

**Description:** The System SHOULD enforce rate limiting on AI Assistant queries to prevent abuse and ensure backend stability. Each user session SHOULD be limited to a configurable maximum number of queries per minute (default: 10). The assistant SHOULD respond to queries within 10 seconds; if the LLM or MCP server exceeds this threshold, the System SHOULD display a timeout message and allow the user to retry.

**Trace:** AI Assistant - Conversational Interface

```gherkin
Feature: AI Assistant Query Performance and Rate Limiting

  Scenario: Rate limit exceeded
    Given the user has submitted 10 AI Assistant queries in the last 60 seconds
    When the user submits an 11th query
    Then the System SHOULD reject the query with "Rate limit exceeded — please wait before submitting another query"
    And the rejection SHOULD be recorded in the audit log

  Scenario: Query timeout
    Given the user submits a query to the AI Assistant
    When the LLM or MCP server does not respond within 10 seconds
    Then the System SHOULD display "Query timed out — the AI service is not responding. Please try again."
    And the user SHOULD be able to retry the query
```

---

## 5. Security Requirements Detail

### SR-001: OIDC/SAML Authentication

**Description:** The System MUST authenticate all users via an external OIDC or SAML identity provider. No local username/password authentication MUST be supported. The System MUST support multiple IdP configurations for federated environments.

**Trace:** Dashboard Authentication - User Identity

```gherkin
Feature: OIDC/SAML Authentication

  Scenario: Authenticate via OIDC provider
    Given the dashboard is configured with an OIDC identity provider
    When a user navigates to the dashboard without a session
    Then the System MUST redirect the user to the OIDC provider login page
    And upon successful authentication, the user MUST be redirected back with a valid session

  Scenario: OIDC provider unreachable
    Given the OIDC identity provider is unreachable
    When a user attempts to log in
    Then the System MUST display an error "Authentication service unavailable"
    And no unauthenticated access to the dashboard MUST be permitted

  Scenario: Session token expired
    Given the user's session token has expired
    When the user attempts any dashboard action
    Then the System MUST redirect the user to re-authenticate via the IdP
    And the user MUST be returned to their original page after re-authentication
```

### SR-002: MFA for Admin Role

**Description:** The System MUST require Multi-Factor Authentication for users with the Admin role. MFA MUST be enforced at the identity provider level. The System MUST verify the MFA claim in the OIDC/SAML token before granting Admin privileges.

**Trace:** Dashboard Authentication - User Identity

```gherkin
Feature: MFA for Admin Role

  Scenario: Admin login requires MFA
    Given user "admin@example.com" has the Admin role
    When the user authenticates via the IdP without completing MFA
    Then the System MUST deny Admin-level access
    And the user MUST be granted the lowest applicable role (Viewer) until MFA is completed

  Scenario: Admin with valid MFA claim
    Given user "admin@example.com" has completed MFA at the IdP
    When the OIDC token includes an MFA claim
    Then the System MUST grant full Admin privileges
```

### SR-003: Three-Tier RBAC

**Description:** The System MUST enforce a three-tier RBAC model at the dashboard backend. Since Keylime's built-in authorization has no read-only admin role, the dashboard MUST proxy and restrict operations based on the user's dashboard role. The permissions matrix:

| Operation | Viewer | Operator | Admin |
|-----------|--------|----------|-------|
| View agent fleet, attestation analytics, policies, audit log | Yes | Yes | Yes |
| Export reports (CSV/PDF) | No | Yes | Yes |
| Acknowledge/manage alerts | No | Yes | Yes |
| Reactivate/stop agents | No | Yes | Yes |
| Create/edit/delete policies | No | No | Yes |
| Delete agents from verifier | No | No | Yes |
| Manage dashboard users/roles | No | No | Yes |
| Configure alert thresholds | No | No | Yes |
| View/export TPM key material | No | No | Yes |

All write operations to Keylime MUST be blocked at the dashboard proxy layer for non-Admin roles.

**Trace:** Dashboard RBAC - Role Definitions

```gherkin
Feature: Three-Tier RBAC Enforcement

  Scenario: Viewer cannot export reports
    Given the user has the Viewer role
    When the user attempts to export a CSV report
    Then the System MUST return HTTP 403 Forbidden
    And the audit log MUST record the denied access attempt

  Scenario: Operator can acknowledge alerts
    Given the user has the Operator role
    When the user acknowledges an alert
    Then the action MUST succeed

> **Reviewer Note:** Split from original "Operator can acknowledge alerts but not delete agents" which contained two When/Then blocks — a positive and negative test in one scenario. Each scenario MUST test a single outcome for deterministic pass/fail and clear traceability.

  Scenario: Operator cannot delete agents
    Given the user has the Operator role
    When the user attempts to delete an agent
    Then the System MUST return HTTP 403 Forbidden

  Scenario: Admin can manage dashboard users
    Given the user has the Admin role
    When the user creates a new dashboard user with Operator role
    Then the new user MUST be created successfully
```

### SR-004: mTLS for Keylime API Communication

**Description:** The System MUST use mutual TLS (mTLS) for all communication with Keylime Verifier and Registrar APIs. The System MUST present a valid client certificate signed by the Keylime CA. The System MUST verify the server certificate of Keylime components.

**Trace:** Threat Model - Trust Boundaries

```gherkin
Feature: mTLS API Communication

  Scenario: Backend presents client certificate to Verifier
    Given the backend is configured with a valid mTLS client certificate
    When the backend connects to the Verifier API
    Then the connection MUST use mutual TLS authentication
    And the Verifier MUST accept the dashboard's client certificate

  Scenario: Reject connection with invalid server certificate
    Given the Verifier presents a certificate not signed by the trusted CA
    When the backend attempts to connect
    Then the connection MUST be rejected
    And a CRITICAL alert MUST indicate "Verifier certificate validation failed"
```

### SR-005: mTLS Private Key Protection

**Description:** The mTLS private key grants full Keylime admin access. It MUST NEVER be stored on disk in cleartext. Supported storage backends: PKCS#11 HSM, HashiCorp Vault Transit, Kubernetes CSI Secret Store, or encrypted file with passphrase (development only). Rust's `rustls` MUST use a custom `SigningKey` trait for HSM/PKCS#11 offload.

**Trace:** Secret Management - Credential Lifecycle

```gherkin
Feature: mTLS Private Key Protection

  Scenario: Reject startup with cleartext key on disk
    Given the mTLS private key file is present on disk in cleartext (unencrypted PEM)
    When the System starts
    Then the System MUST refuse to start
    And an error MUST indicate "cleartext private key detected — use HSM or Vault"

  Scenario: Graceful handling of HSM unavailability
    Given the System is configured to use PKCS#11 HSM for key storage
    When the HSM device is unreachable at startup
    Then the System MUST refuse to start
    And the error MUST indicate "HSM unreachable — cannot load mTLS key"
```

### SR-006: HSM or Vault-Backed Key Storage

**Description:** The System MUST support HSM (PKCS#11) or HashiCorp Vault Transit as the primary storage backend for the mTLS private key. Kubernetes CSI Secret Store MUST be supported as an alternative. Encrypted file with passphrase MAY be supported for development environments only.

**Trace:** Secret Management - Credential Lifecycle

```gherkin
Feature: Vault-Backed Key Storage

  Scenario: Load private key from HashiCorp Vault
    Given the System is configured to use HashiCorp Vault Transit for key storage
    When the System starts
    Then the mTLS private key MUST be loaded from Vault
    And no key material MUST be written to disk

  Scenario: Vault token expired
    Given the Vault token has expired
    When the System attempts to load the private key
    Then the System MUST refuse to start
    And an error MUST indicate "Vault token expired — renew before starting"
```

### SR-007: TLS Encryption on All Connections

**Description:** The System MUST encrypt all network connections with TLS. No cleartext HTTP, database, or cache connections MUST be permitted. This applies to browser-to-dashboard, dashboard-to-Keylime, dashboard-to-database, and dashboard-to-cache connections.

**Trace:** Transport Security - Encrypted Data Paths

```gherkin
Feature: TLS on All Connections

  Scenario: Reject cleartext HTTP connection
    Given the dashboard is running with TLS enabled
    When a client attempts to connect via HTTP (port 80)
    Then the System MUST redirect to HTTPS or reject the connection
    And no data MUST be served over cleartext HTTP

  Scenario: Database connection uses TLS
    Given the TimescaleDB connection is configured
    When the backend connects to the database
    Then the connection MUST use TLS encryption
```

### SR-008: Browser-to-API TLS 1.3 Minimum

**Description:** The System MUST enforce TLS 1.3 as the minimum protocol version for browser-to-dashboard API connections. TLS 1.2 and earlier MUST be rejected.

**Trace:** Transport Security - Encrypted Data Paths

```gherkin
Feature: TLS 1.3 Minimum for Browser

  Scenario: Accept TLS 1.3 connection
    Given a browser supports TLS 1.3
    When the browser connects to the dashboard
    Then the TLS handshake MUST complete using TLS 1.3

  Scenario: Reject TLS 1.2 connection
    Given a client attempts to connect using TLS 1.2
    When the TLS handshake is initiated
    Then the System MUST reject the connection
```

### SR-009: API-to-Keylime TLS 1.2+ Minimum

**Description:** The System MUST enforce TLS 1.2 or higher for all connections from the dashboard backend to Keylime components. This allows compatibility with Keylime deployments that have not yet upgraded to TLS 1.3.

**Trace:** Transport Security - Encrypted Data Paths

```gherkin
Feature: TLS 1.2+ for Keylime API

  Scenario: Connect to Keylime Verifier with TLS 1.2
    Given the Keylime Verifier supports TLS 1.2 but not TLS 1.3
    When the backend connects to the Verifier
    Then the connection MUST complete using TLS 1.2

  Scenario: Reject SSLv3 or TLS 1.1
    Given a Keylime component offers only TLS 1.1
    When the backend attempts to connect
    Then the connection MUST be rejected
```

### SR-010: Short-Lived JWT Session Tokens

**Description:** The System MUST issue short-lived JWT session tokens with a maximum lifetime of 15 minutes. Refresh tokens MUST be rotated on each use. Stolen refresh tokens MUST be detectable via replay detection.

**Trace:** Dashboard Authentication - User Identity

```gherkin
Feature: Short-Lived Session Tokens

  Scenario: Session token expires after 15 minutes
    Given the user has an active session
    When 15 minutes elapse without token refresh
    Then the session token MUST expire
    And the user MUST be prompted to re-authenticate

  Scenario: Refresh token rotation
    Given the user's access token is about to expire
    When the client uses the refresh token to obtain a new access token
    Then a new refresh token MUST also be issued
    And the previous refresh token MUST be invalidated
```

### SR-011: Server-Side Session Revocation

**Description:** The System MUST support server-side session revocation. Administrators MUST be able to revoke any active user session. Revoked sessions MUST be immediately invalidated regardless of token expiry.

**Trace:** Dashboard Authentication - User Identity

```gherkin
Feature: Server-Side Session Revocation

  Scenario: Admin revokes user session
    Given admin "admin@example.com" views active sessions
    When the admin revokes the session for "operator@example.com"
    Then the operator's session MUST be immediately invalidated
    And the operator's next API request MUST return HTTP 401 Unauthorized

  Scenario: Revoked token cannot be reused
    Given a session token has been revoked
    When the revoked token is presented in an API request
    Then the System MUST reject the request with HTTP 401
```

### SR-012: XSS and Injection Prevention

**Description:** The System MUST implement Content Security Policy (CSP) headers and input sanitization to prevent XSS and injection attacks. All user input MUST be sanitized before rendering. CSP MUST restrict script sources to self only.

**Trace:** Threat Model - Threat Catalog

```gherkin
Feature: XSS and Injection Prevention

  Scenario: CSP headers present on all responses
    Given the dashboard serves an HTTP response
    Then the Content-Security-Policy header MUST be present
    And script-src MUST be restricted to 'self'

  Scenario: Sanitize user input in search
    Given the user enters "<script>alert('xss')</script>" in the search bar
    When the search is submitted
    Then the input MUST be sanitized before processing
    And no script execution MUST occur in the browser
```

### SR-013: Data Minimization

**Description:** The System MUST NEVER cache or store raw TPM quotes, IMA measurement logs, or raw boot event logs. These MUST be passed through from the Keylime API to the UI on demand and discarded. Only attestation results (pass/fail + timestamp) MUST be retained for historical analysis. PoP token hashes MUST NOT be displayed or cached; only session metadata (creation time, expiry, agent UUID) MAY be shown.

**Trace:** Threat Model - Data Classification; Attestation Modes - Comparative View

```gherkin
Feature: Data Minimization

  Scenario: Raw TPM quotes not persisted
    Given the user views the Raw Data tab for agent "a1b2c3d4"
    When the System fetches the TPM quote from the Verifier API
    Then the raw quote data MUST be served directly to the browser
    And the System MUST NOT write the raw quote data to any persistent store

  Scenario: PoP tokens not displayed
    Given a push-mode agent has an active session with a PoP token
    When the user views the agent detail page
    Then only session metadata (creation time, expiry, agent UUID) MUST be displayed
    And the PoP token hash MUST NOT be shown or cached
```

### SR-014: PoP Token Privacy

**Description:** The System MUST NEVER display or cache raw Proof-of-Possession (PoP) tokens. Only session metadata (creation time, expiry, agent UUID) MAY be shown in the UI. This requirement is a specific instance of SR-013 (Data Minimization) applied to push-mode PoP tokens.

> **Reviewer Note:** SR-014 overlaps with SR-013's PoP token scenario. SR-014 is retained as a distinct requirement because PoP token handling is a push-mode-specific security concern that warrants independent traceability. Cross-reference added to clarify the relationship.

**Trace:** Attestation Modes - Comparative View

```gherkin
Feature: PoP Token Privacy

  Scenario: PoP token excluded from API responses
    Given the dashboard API serves push-mode session data
    When the API response is generated
    Then the response MUST NOT include raw PoP token values
    And only session metadata MUST be included
```

### SR-015: Tamper-Evident Audit Log with RFC 3161

**Description:** The System MUST implement tamper-evident hash-chained audit logging with RFC 3161 timestamp anchoring. This is defined in detail in FR-061 and duplicated here as a security requirement to ensure traceability.

**Trace:** Compliance - Tamper-Evident Audit Logging

```gherkin
Feature: Tamper-Evident Audit Log

  Scenario: Audit log entries are hash-chained
    Given the audit log has existing entries
    When a new audit event occurs
    Then the new entry MUST include the SHA-256 hash of the previous entry
    And the chain MUST be verifiable from the root anchor

  Scenario: RFC 3161 timestamp anchoring
    Given the System is configured with an RFC 3161 TSA
    When the audit log chain root is created
    Then the root MUST be anchored with an RFC 3161 timestamp token
    And the timestamp token MUST be verifiable against the TSA certificate
```

### SR-016: SSRF Protection

**Description:** The System MUST protect against Server-Side Request Forgery (SSRF) on webhook URLs. Webhook destinations MUST be validated against an allowlist. RFC 1918 private addresses MUST be blocked unless explicitly allowed. DNS rebinding protection MUST be implemented.

**Trace:** Revocation - Alert Workflow

```gherkin
Feature: SSRF Protection

  Scenario: Block webhook to private IP address
    Given an admin configures a webhook URL pointing to 192.168.1.100
    When the webhook configuration is saved
    Then the System MUST reject the URL with error "RFC 1918 addresses blocked"

  Scenario: Allow webhook to whitelisted destination
    Given "hooks.slack.com" is in the webhook allowlist
    When an admin configures a webhook to "https://hooks.slack.com/services/T00/B00/xxxx"
    Then the webhook configuration MUST be accepted
```

### SR-017: Two-Person Approval for Policy Changes

**Description:** The System MUST enforce a two-person approval workflow for all policy changes. This is a security requirement that mirrors FR-039 to ensure policy modifications cannot be unilaterally applied by a single administrator.

**Trace:** Policy Management - Two-Person Rule

```gherkin
Feature: Two-Person Policy Approval Security

  Scenario: Policy change without approval blocked
    Given Admin A drafts and saves a policy change
    When Admin A attempts to push the policy to the Verifier without approval
    Then the System MUST block the push
    And the audit log MUST record the blocked attempt
```

### SR-018: Drafter Cannot Self-Approve

**Description:** The System MUST enforce that the approver of a policy change MUST NOT be the same user as the drafter. This separation of duties is a critical security control.

**Trace:** Policy Management - Two-Person Rule

```gherkin
Feature: Drafter Cannot Self-Approve

  Scenario: Same user attempts draft and approval
    Given Admin A drafted a policy change
    When Admin A attempts to approve the same change
    Then the System MUST reject the approval with "self-approval not permitted"
    And the audit log MUST record the rejected self-approval attempt
```

### SR-019: Multi-Tenancy Isolation

**Description:** The System MUST enforce strict multi-tenancy isolation. Cross-tenant data MUST NEVER be mixed or accessible. Each tenant MUST have isolated data stores, separate RBAC, and independent configuration.

**Trace:** Dashboard RBAC - Multi-Tenancy

```gherkin
Feature: Multi-Tenancy Isolation

  Scenario: Cross-tenant data access blocked
    Given Tenant A has agent "agent-001" and Tenant B has agent "agent-002"
    When a user authenticated to Tenant A requests agent "agent-002"
    Then the System MUST return HTTP 404 Not Found
    And no data from Tenant B MUST be included in the response

  Scenario: Tenant isolation in search results
    Given Tenant A has 100 agents and Tenant B has 50 agents
    When a Tenant A user performs a global search
    Then search results MUST only include Tenant A's 100 agents
```

### SR-020: Data Classification

**Description:** The System MUST enforce the following data classification and protection requirements:

| Data Type | Classification | Storage Policy | Protection |
|-----------|---------------|----------------|------------|
| mTLS private key | SECRET | Never on disk | HSM or Vault only |
| EK/AK public keys | CONFIDENTIAL | Cache only (TTL) | Encrypt at rest |
| TPM quotes | CONFIDENTIAL | Do not cache | Pass-through only |
| IMA policy content | CONFIDENTIAL | Cache with TTL | Encrypt at rest |
| Agent IPs/UUIDs | INTERNAL | TimeSeries DB | Encrypt at rest |
| Attestation results | INTERNAL | TimeSeries DB | Encrypt at rest |
| Dashboard audit log | CONFIDENTIAL | Append-only store | Hash-chained, signed |
| Dashboard credentials | SECRET | Never persisted | OIDC session tokens |

**Trace:** Threat Model - Data Classification

```gherkin
Feature: Data Classification Enforcement

  Scenario: SECRET data never written to disk
    Given the mTLS private key is classified as SECRET
    When the System operates normally
    Then the private key MUST NOT exist on disk in any form (cleartext or encrypted)
    And the key MUST be loaded exclusively from HSM or Vault

  Scenario: CONFIDENTIAL data encrypted at rest
    Given EK/AK public keys are classified as CONFIDENTIAL
    When the System caches public key data in Redis
    Then the cached data MUST be encrypted at rest
    And the cache entry MUST have a TTL configured
```

### SR-021: Write Operations Blocked for Non-Admin

**Description:** The System MUST block all write operations to Keylime APIs at the dashboard proxy layer for non-Admin roles. This provides defense-in-depth beyond RBAC role checks.

**Trace:** Dashboard RBAC - Role Definitions

```gherkin
Feature: Write Operation Blocking

  Scenario: Operator write request blocked at proxy
    Given the user has the Operator role
    When the user sends a POST request to create a policy (bypassing UI)
    Then the dashboard proxy MUST block the request before it reaches the Keylime API
    And the System MUST return HTTP 403 Forbidden
    And the blocked attempt MUST be logged in the audit log
```

### SR-022: mTLS Sidecar Option

**Description:** The System MAY support an mTLS sidecar (Envoy or Ghostunnel) as an alternative to application-level mTLS. This allows deployments where certificate management is handled by the service mesh.

**Trace:** Transport Security - mTLS Sidecar Option

```gherkin
Feature: mTLS Sidecar Support

  Scenario: Delegate mTLS to Envoy sidecar
    Given the System is deployed with an Envoy sidecar handling mTLS
    When the backend communicates with the Keylime Verifier
    Then the Envoy sidecar SHOULD terminate the mTLS connection
    And the backend SHOULD communicate with Envoy over localhost
```

### SR-023: No Unsafe Rust Code

**Description:** The dashboard Rust crate MUST use `#![forbid(unsafe_code)]` to prevent unsafe Rust code blocks. This eliminates entire classes of memory safety vulnerabilities.

**Trace:** Technical Architecture - Why Rust

```gherkin
Feature: No Unsafe Rust Code

  Scenario: Build fails on unsafe code
    Given the dashboard crate has #![forbid(unsafe_code)] at the crate root
    When a developer adds an unsafe block to the crate
    Then the Rust compiler MUST reject the build with error "unsafe code is forbidden"
```

### SR-024: Signed Cache Entries

**Description:** The System MUST sign cache entries and enforce TTLs to mitigate cache poisoning attacks. Cache entries MUST be validated before use.

**Trace:** Threat Model - Threat Catalog

```gherkin
Feature: Signed Cache Entries

  Scenario: Detect poisoned cache entry
    Given agent data is cached with an HMAC signature
    When a cache entry is modified externally (cache poisoning attempt)
    Then the System MUST detect the signature mismatch
    And the System MUST discard the poisoned entry and fetch fresh data from the API
    And a CRITICAL alert MUST be raised indicating "cache integrity violation"
```

### SR-025: TPM Identity Change Detection

**Description:** The System MUST alert on TPM identity changes during agent re-registration. CRITICAL alerts MUST fire when: TPM key changes on re-registration, or EK certificate mismatches on re-registration. WARNING alerts MUST fire when: regcount exceeds 3, or re-registration occurs from a different IP. TPM identity keys SHOULD be immutable; any change indicates potential compromise.

**Trace:** Security Audit - Identity Verification Events

```gherkin
Feature: TPM Identity Change Detection

  Scenario: Detect TPM key change on re-registration
    Given agent "agent-042" is registered with EK certificate "cert-A"
    When agent "agent-042" re-registers with a different EK certificate "cert-B"
    Then the System MUST raise a CRITICAL alert
    And the alert message MUST indicate "EK cert mismatch on re-registration"
    And the audit log MUST record both the previous and new certificate identifiers

  Scenario: Warn on high re-registration count
    Given agent "agent-042" has a registration count of 4
    When the System evaluates identity events
    Then the System MUST raise a WARNING alert indicating "high regcount (>3)"
```

### SR-026: Audit Log Retention

**Description:** The System MUST retain audit log entries for a minimum of 1 year to meet compliance requirements. Retention period MUST be configurable. Archived logs MUST remain tamper-evident and verifiable.

**Trace:** Compliance - Tamper-Evident Audit Logging

```gherkin
Feature: Audit Log Retention

  Scenario: Audit log retained for minimum period
    Given the retention policy is set to 1 year
    When audit log entries are older than 1 year
    Then the System MUST archive the entries but NOT delete them automatically
    And archived entries MUST remain verifiable via hash chain validation

  Scenario: Prevent premature log deletion
    Given an administrator attempts to delete audit log entries less than 1 year old
    When the deletion request is submitted
    Then the System MUST reject the request with "retention policy violation"
```

### SR-027: Emergency Bypass with Break-Glass Audit

**Description:** The System MUST support an emergency bypass mechanism that allows policy changes to skip the two-person approval workflow. Emergency bypass MUST require break-glass authentication and MUST produce a detailed audit trail including: who invoked the bypass, why (free-text justification), what action was taken, and timestamp.

**Trace:** Policy Management - Two-Person Rule

```gherkin
Feature: Emergency Bypass with Break-Glass Audit

  Scenario: Emergency bypass for critical policy rollback
    Given a critical incident requires immediate policy rollback
    And the two-person rule is enforced
    When Admin A invokes the emergency bypass with justification "critical production incident"
    Then the policy change MUST be applied without a second approver
    And the audit log MUST record the bypass with: actor, justification, action, and timestamp
    And a CRITICAL alert MUST be raised: "Break-glass bypass invoked"

  Scenario: Break-glass without justification rejected
    Given Admin A invokes the emergency bypass
    When no justification text is provided
    Then the System MUST reject the bypass with "justification required for emergency bypass"
```

### SR-028: Configurable Idle Session Timeout

**Description:** The System MUST enforce a configurable idle session timeout. Sessions inactive beyond the timeout period MUST be automatically terminated. The default idle timeout MUST be 30 minutes.

**Trace:** Dashboard Authentication - User Identity

```gherkin
Feature: Idle Session Timeout

  Scenario: Session terminated after idle timeout
    Given the idle session timeout is configured at 30 minutes
    When the user has been inactive for 31 minutes
    Then the session MUST be automatically terminated
    And the user MUST be redirected to the login page on their next action

  Scenario: Activity resets idle timer
    Given the idle session timeout is 30 minutes
    When the user performs an action at the 25-minute mark
    Then the idle timer MUST reset to 0
    And the session MUST remain active for another 30 minutes of inactivity
```

### SR-029: Rate Limiting on Session Creation

**Description:** The System MUST enforce rate limiting on the dashboard session creation endpoint to prevent brute-force attacks on authentication. Failed login attempts MUST be rate-limited per source IP.

**Trace:** Attestation Modes - Comparative View

```gherkin
Feature: Session Creation Rate Limiting

  Scenario: Rate limit exceeded on login attempts
    Given the rate limit for session creation is 10 attempts per minute per IP
    When a client sends the 11th login request within one minute from the same IP
    Then the System MUST return HTTP 429 Too Many Requests
    And the response MUST include a Retry-After header

  Scenario: Successful login after cooldown
    Given a client was rate-limited on login attempts
    When the Retry-After period elapses
    Then the client MUST be able to attempt login again
```

---

## 6. Implementation Phasing

As defined in the source presentation, the implementation follows three phases:

**Phase 1 — Secure Foundation:** Agent fleet list, agent state monitoring, OIDC/SAML authentication, three-tier RBAC, tamper-evident audit log, mTLS to Keylime APIs. *Exit criteria: all API access authenticated, RBAC enforced on every endpoint, audit log captures all actions, mTLS key never on disk in cleartext, threat model reviewed, penetration test passed.*

**Phase 2 — Operations:** Attestation analytics, policy management UI, certificate monitoring, alert notifications, push mode (v3 API) support, SIEM integration, WebSocket updates, SSRF-validated webhooks, Prometheus metrics endpoint.

**Phase 3 — Enterprise Scale:** Multi-tenancy, compliance reports (NIST, PCI DSS, SOC 2, FedRAMP), incident response integration (ServiceNow, Jira, PagerDuty), HA deployment, air-gapped packaging, multi-cluster support, WCAG 2.1 AA accessibility.

**Trace:** Implementation Roadmap

---

## 7. Software Design Description Cross-Reference

> **Migrated:** 2026-04-15 — Implementation Refinements (IR-001 through IR-013) have been migrated to the companion Software Design Description (SDD) document, following the IEEE 1016 standard separation between requirements (*what*) and design (*how*).

The design details that realize these requirements -- including component decomposition, data models, API contracts, state machines, algorithms, and deployment configuration -- are documented in the companion SDD:

**Document:** [`spec/SDD-Keylime-Monitoring-Tool.md`](SDD-Keylime-Monitoring-Tool.md)

**SDD Viewpoint Mapping:**

| Former IR | SDD Section | SDD Viewpoint |
|-----------|-------------|---------------|
| IR-001: API Response Envelope | 3.4.1 | Interface View |
| IR-002: Paginated Response Format | 3.4.2 | Interface View |
| IR-003: WebSocket Endpoint | 3.4.4 | Interface View |
| IR-004: Agent Data Model | 3.3.2 | Logical View |
| IR-005: Pipeline Stage Enumeration | 3.3.6 | Logical View |
| IR-006: Failure Correlation Types | 3.3.6 | Logical View |
| IR-007: Notification Channels | 3.3.9 | Logical View |
| IR-008: Alert Lifecycle State Machine | 3.6.2 | State Dynamics View |
| IR-009: Certificate Expiry Derivation | 3.3.5 | Logical View |
| IR-010: Timeline Distribution Algorithm | 3.7.1 | Algorithm View |
| IR-011: Frontend RBAC Enforcement | 5.1 | Security Overlay |
| IR-012: Visualization Settings | 3.8.2 | Resource View |
| IR-013: KPI Fallback Computation | 3.7.2 | Algorithm View |

The SDD also includes a full SRS traceability matrix (Section 6) mapping every implemented requirement to its corresponding design element.
