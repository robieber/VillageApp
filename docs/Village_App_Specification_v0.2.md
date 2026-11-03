# Village App

Business, Functional and Technical Specification

Draft v0.2 • 03 November 2026

| Control | Value |
| --- | --- |
| Status | DRAFT — for business review and preliminary estimation; not an approved implementation baseline |
| Revision | Adds explicit consumer portal theme configuration, testing, implementation guidance and documentation coverage. |
| Reconstruction note | Recreated from the v0.1 specification content in the conversation because the prior binary file was unavailable in this turn. Formatting and wording have been consolidated; this is not a byte-preserving edit. |
| Approvers | Product owner, HOA process owner/treasurer, technical lead, designated security/privacy reviewer — names TBD |
| Technology direction | Evaluate RESCo reuse and an Odoo backend/resident portal; version, edition, licensing and capabilities require verification. |
| Source | Project conversation through 03 November 2026; no code, installed platform, legal instruments or integration contracts inspected. |

## Document interpretation

Confirmed means explicitly supplied by the product owner in discussion, not formal board or legal approval. Proposed requirements require validation. TBD is an unresolved decision. Stable IDs shall be retained across revisions. The document includes a seed acceptance pack and traceability examples, not completed QA evidence.

## 1. Objectives and scope

Village App provides resident self-service and HOA administration for property/member records, occupancy-based membership charges, collections, optional services, facility reservations, announcements/events, concerns and village operations. It must preserve a reliable record from policy and application through billing, payment and fulfillment.

| Release | Proposed scope |
| --- | --- |
| R0 — foundation | Policy decisions; Odoo/RESCo fit-gap; data/access architecture; prototype and migration rehearsal. |
| R1 — MVP | Property, dwelling and occupancy registry; assessments/statements; manual payment verification; configurable service catalog; stickers/seasonal requests; facilities; announcements/events; documents; concerns; dashboards; controlled portal theme configuration. |
| R2 — operations | Online payment integration if not in R1; automated reconciliation; assets/work orders; expenditure approvals; approved financial summaries and governance. |
| R3 — expansion | Construction/bonds, violations/appeals, committee workflows, visitor access/hardware and advanced analytics under separately approved scope. |

R1 exclusions unless approved: native mobile apps, binding electronic elections, general offline transaction synchronization, social marketplace, AI financial/enforcement decisions, gate/RFID/plate-recognition integration and full ERP replacement. Online payments are optional in R1; verified proof-of-payment processing is the minimum baseline.

Success measures: assessment exceptions, reconciliation completeness, adoption, staff processing time, concern response/resolution, payment review backlog and verified facility utilization. Baselines and numerical targets are TBD.

## 2. Roles, access and tenancy

| Role | Authorized responsibilities |
| --- | --- |
| Resident/designated occupant member | Own authorized household, dwelling and account; requests, bookings, payments and concerns. |
| Owner | Owned property and authorized obligations, including all-vacant fallback; no automatic access to all occupant personal records. |
| Administrator | Registry, occupancy verification, catalog, communications and requests within scope. |
| Treasurer/collection staff | Assessment review, verification, allocation, reconciliation, adjustments within delegated authority. |
| Board/approver | Approved reports and assigned approvals, not unrestricted access by default. |
| Facility/service staff | Assigned bookings, schedules and fulfillment information. |
| Contractor/maintenance | Assigned work and necessary contacts, not resident financial ledgers. |
| Security — later | Approved access information, not household balances. |
| Theme administrator | HOA-scoped theme settings and publication; distinct permission from business/finance approvals. |
| Platform administrator | Technical operations and expressly authorized, audited support access. |

Enforce role, HOA scope, property relationship, dates and sensitivity server-side across pages, APIs, attachments, searches, exports and background jobs. Ended relationships revoke associated access without erasing transaction history. One person may have different roles across properties.

Proposed platform tenant = one HOA organization, containing villages/phases/blocks. Platform tenant is distinct from resident tenant/renter. Deployment isolation and cross-HOA identity are TBD; a single-HOA pilot does not waive isolation design for subsequent multi-HOA deployment.

## 3. Domain model

| Entity | Definition and essential relationships |
| --- | --- |
| HOA / village / phase / block | Organizational hierarchy, timezone, currency, enabled features, approved policy references. |
| Parcel | Land identity, area/unit, block/lot, registered reference, effective dates and predecessor/successor parcels. |
| Dwelling | Independent residence under a parcel; unit label, classification, dates and approval reference. |
| Party / ownership | People/organizations and effective-dated owner relationships to parcels or dwellings; co-ownership supported. |
| Household / occupancy | Household members; dwelling occupancy episode; designated member/payer; verified dates and approval. |
| Membership | Explicit status and party/property relationship, separate from login, occupancy count and voting entitlement. |
| Billing account | Responsible party and authorized viewers; links to property/unit; not a duplicate ledger. |
| Assessment policy / charge | Versioned basis, rate, effectivity, fallback and cycle; posted charge retains inputs and policy snapshot. |
| Payment / allocation | Evidence, external reference, verified/posted amount, allocation, refund/reversal and reconciliation. |
| Catalog offer / request | Type, workflow template, price/version, dates, eligibility, capacity and form fields; request retains accepted scope. |
| Vehicle / sticker | Authorized vehicle relationship; serial, validity, issue/replacement/revocation history. |
| Facility / reservation | Resource, hours, closures, rates, entitlement group, hold interval, confirmation and actual usage. |
| Concern / incident / work order | Category/location, confidentiality, reporter, assignment, updates, linked reports and resolution. |
| Communication / document | Audience, publication/expiry, version, owner and attachment access. |
| Theme profile / version — NEW | HOA ID, preset/version, validated design tokens, asset references, state, creator/approver/publisher, timestamps and active-version link. |
| Audit / approval / notification | Actor, time, tenant, object, change/decision and delivery result; no secrets. |

Parcel 1:N dwellings; dwelling 1:N occupancy episodes; household 1:N people; party M:N properties through ownership; account 1:N charges; payment M:N charges through allocations. Do not infer extra membership from each person or portal login. Floor area, exclusive-use land and billing allocation are separate from parcel area.

Formal land subdivision creates successor parcel records and closes the predecessor for future assessment, preserving lineage. Avoid double billing parent and successors. More dwellings on an unchanged lot do not constitute a formal parcel split or establish construction approval.

## 4. Business-rule catalog

| ID | Status | Rule |
| --- | --- | --- |
| RULE-MEM-001 | Confirmed | One membership unit for each occupied dwelling; occupant means its designated representative, not every resident. |
| RULE-MEM-002 | Confirmed | When all dwellings on the lot/apartment property are vacant, owner pays exactly one minimum membership unit. |
| RULE-MEM-003 | Confirmed | No additional owner minimum or vacant-unit membership charge while at least one dwelling is occupied. |
| RULE-MEM-004 | Derived | Billable membership units = max(1, verified occupied units) under the approved period policy; mid-period basis TBD. |
| RULE-LAND-001 | Confirmed | Additional dwellings/occupants do not increase lot area. |
| RULE-HIST-001 | Proposed | Preserve posted policy, occupancy basis, prices and liable party; correct through authorized adjustments. |
| RULE-LIAB-001 | TBD | Owner guarantee of occupant arrears and debt transfers require explicit approval; no automatic transfer authorized. |
| RULE-CAT-001 | Confirmed/refined | Staff can create, change and retire services/billable items, including stickers and seasonal assistance. |
| RULE-CAT-002 | Proposed | Delete only unreferenced drafts; archive referenced offerings and retain transactions. |
| RULE-CAT-003 | Proposed | Optional service publication does not automatically charge households. |
| RULE-CAT-004 | Proposed | Close new seasonal requests without cancelling existing commitments. |
| RULE-BOOK-001 | Proposed | Backend transaction controls prevent overlapping exclusive resource confirmations. |
| RULE-PAY-001 | Proposed | Screenshot submission is not proof of settled funds; verification/posting/allocation/reconciliation remain distinct. |
| RULE-DEP-001 | Proposed | Refundable deposits are not ordinary service income; authorized deduction/refund required. |
| RULE-GOV-001 | Proposed | Billable membership counts do not automatically grant voting rights or establish legal eligibility. |
| RULE-TEN-001 | Proposed | All business data, files and actions are isolated by authorized HOA scope. |
| RULE-RATE-001 | Proposed | Effective-dated price changes do not silently reprice accepted orders or posted charges. |
| RULE-THEME-001 | Proposed — NEW | Theme configuration affects presentation only, never financial rules, access rights, enabled business features or approvals. |
| RULE-THEME-002 | Proposed — NEW | Only validated, authorized, published theme versions are served; incomplete drafts cannot change the live portal. |

| Three-unit property | Membership units | Payer |
| --- | --- | --- |
| 3 occupied / 0 vacant | 3 | Three designated occupied-unit members |
| 2 occupied / 1 vacant | 2 | Two designated members; no owner extra |
| 1 occupied / 2 vacant | 1 | One designated member; no owner extra |
| 0 occupied / 3 vacant | 1 | Owner fallback only |

Membership total = rate R × max(1,N). Any land-area charge is separately authorized and must not be introduced implicitly. Mid-period changes, co-owner responsibility and assessment grouping remain decisions, not developer assumptions.

## 5. Functional requirements

M = proposed MVP requirement; S = should have, subject to capacity; L = later. Requirements require business approval. Additional detailed validation and permission tests shall supplement the acceptance pack.

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-REG-001 | M | Maintain parcel, dwelling, ownership and household independently; multiple dwellings retain original lot area. |
| FR-OCC-001 | M | Maintain verified, effective-dated occupancy and designated member/payer; validate dates and conflicts. |
| FR-ACC-001 | M | Verified invitations, recovery and relationship-scoped portal access; revoke ended privileges. |
| FR-MEM-001 | M | Calculate occupied-unit membership and owner fallback using approved policy and accountable parties. |
| FR-BILL-001 | M | Generate reviewable, idempotent periodic charges with retained occupancy and policy snapshots. |
| FR-BILL-002 | M | Statements, partial/advance payments, credits, unapplied funds and approved adjustments reconcile to the ledger. |
| FR-PAY-001 | M | Capture evidence and verify before posting; rejected submissions retain reasons/audit. |
| FR-PAY-002 | M | Allocate payments without exceeding funds; retain references and expose reconciliation exceptions. |
| FR-PAY-003 | S | Approved payment provider integration with authenticated, idempotent event processing. |
| FR-CAT-001 | M | Create supported offerings/workflows without code changes. |
| FR-CAT-002 | M | Configure publication, request/delivery windows, pricing and eligibility; validate server-side. |
| FR-CAT-003 | M | Configure supported fields, attachments, quantity limits and payment timing. |
| FR-CAT-004 | M | Separate request approval, payment and fulfillment tracking. |
| FR-CAT-005 | M | Delete unused drafts only; archive referenced offers with readable transaction history. |
| FR-CAT-006 | M | Retain accepted offer scope and price; prevent retrospective repricing. |
| FR-CAT-007 | M | Audit catalog creation, pricing, publication and retirement. |
| FR-CAT-008 | M | Closing offers stops new requests, not existing completion/refund obligations. |
| FR-STK-001 | M | Link vehicle, applicant, eligibility approval, payment, serial/validity and issuance history. |
| FR-SVC-001 | M | Schedule seasonal services with capacity, assignment and completion evidence. |
| FR-FAC-001 | M | Configure facilities, hours, closures, prices and approved booking/entitlement policies. |
| FR-FAC-002 | M | Atomic expiring holds and conflict-safe confirmation, expiry and cancellation. |
| FR-FAC-003 | M | Track actual usage/no-shows separately from confirmed booking hours. |
| FR-COM-001 | M | Targeted announcements/events with publication and expiry controls. |
| FR-DOC-001 | M | Versioned controlled documents with authorized attachment retrieval. |
| FR-CASE-001 | M | Concerns with categories, location, evidence, confidentiality and private internal notes. |
| FR-CASE-002 | M | Assignment, escalation, resolution/reopen and duplicate linkage without reporter data leakage. |
| FR-OPS-001 | L | Link incidents to asset maintenance, contractors and approved expenditure. |
| FR-RPT-001 | M | Authorized collection, aging, occupancy, booking, service and concern reports with reconcilable totals. |
| FR-NOT-001 | M | Transactional notices with retry/outcome; failures do not undo committed transactions. |
| FR-AUD-001 | M | Audited sensitive changes, approvals, exports and financial actions. |
| FR-GOV-001 | L | Construction/bonds, violations and appeals; complaints do not automatically become violations. |
| FR-ADM-001 | M | Scoped tenant and role administration; deny unauthorized cross-HOA access. |
| FR-MIG-001 | M | Validated, reconcilable imports and opening balances; safe reruns. |

## 6. Workflows and exception handling

### Occupancy / assessment

Draft → pending verification → verified → ended/rejected. Approved occupancy feeds draft assessment → review/approval → posting. Missing fallback owner, conflicting occupancy, missing rates and retrospective changes enter an exception queue, not invented liability. Posted corrections are linked adjustments.

### Payments

Evidence submitted → review → verified/rejected. Posting, allocation and reconciliation are separate dimensions. Under/overpayments, duplicates, payer mismatches and settlement failures remain visible. Never transfer unpaid occupant balances without policy.

### Catalog / services

Draft → published → paused/closed → archived. Request → validation/approval or quotation → accepted scope → payment per policy → assignment → fulfillment → closure. Independent financial and fulfillment states. New workflow types may require code even when new offers do not.

### Car stickers

Select vehicle → documents → eligibility review → payment per policy → assign serial/validity → release. Issuance pending/issued/expired/replaced/revoked. Record old/new sticker linkage and prevent unauthorized duplicate issuance.

### Seasonal decoration assistance

Request scope/preferred date → access/capacity review → quote acceptance if required → payment/deposit → crew assignment → installation → removal if included → closure. Separate request dates from service dates; retain labor/material scope, cancellation and weather/rescheduling policy.

### Facility reservations

Available → held → awaiting payment/approval → confirmed → used/no-show. Cancellation and expiry release availability according to approved rules. Late payment after an expired/rebooked hold becomes a tracked refund/credit/rebooking exception, never a second confirmation.

### Concerns / communications

Submit → triage → assign → investigate/work → resolve → resident confirmation/timed closure → reopen where allowed. Restricted internal notes never appear as resident updates. Announcements draft/approve/schedule/publish/expire. App is not guaranteed emergency dispatch without a staffed approved service.

## 7. Portal, catalog and administration screens

| Workspace | Minimum screens |
| --- | --- |
| Resident/owner | Dashboard; linked properties/dwellings; statements and payments; service catalog/my requests; vehicles/stickers; availability/my bookings; announcements/events; documents; concerns; profile/preferences. |
| Registry | Parcels/dwellings; owners; household/occupancy verification; member/payer assignments; import exceptions. |
| Finance | Assessment preview/approval; statements; evidence review; allocation; reconciliation; refund/credit approvals; aging. |
| Operations | Catalog; service request queues and calendars; sticker issuance; facility schedules; concern routing. |
| Management | Approved dashboards; policy/documents; roles; audit; theme management and preview. |

Mobile web first, explicit payable-action confirmation, understandable status/errors, downloadable statements and consistent local dates. Staff-assisted activity uses the same validations and records the staff actor.

Supported offering types: physical items, scheduled services, facility use, administrative services, recurring assessments and refundable deposits. Configuration includes code/name/category, description/assets, workflow template, effective-dated price/quantity basis, quote requirement, availability windows, eligibility, forms, limits, payment timing, responsible team, cancellation/refund rules and approved financial mapping.

## 8. Consumer portal branding and theme configuration — NEW

Objective: enable authorized HOA personnel to change supported consumer portal colors and styles without code deployment, using controlled presets and design tokens. The shared application shell and business behavior remain unchanged. Theme support is part of the proposed R1 baseline, subject to approval and fit-gap.

| Area | Supported configuration |
| --- | --- |
| Identity | HOA display name, approved logo, favicon, portal title, login image and dashboard banner. |
| Color tokens | Primary, secondary, accent, page background, surface/card, text, link, header/navigation and border; supported hover/focus variants derived or validated. |
| Typography | Approved bundled/licensed font choices; heading/body presets and supported size scale. No arbitrary remote font/script injection. |
| Component styles | Controlled button style, border radius, card/border appearance and spacing/density presets. |
| Navigation presentation | Approved header/sidebar color and light/dark navigation presets; logo fit/placement within supported layouts. |
| Presets | Named approved combinations with default reset; full dark mode is not implied by a dark navigation preset and needs separate validation/scope. |
| Assets | Safe image types, limits/dimensions and alternative text; approved file handling. SVG, if allowed, requires sanitization. |
| Responsive behavior | Same theme adapts to supported desktop/mobile layouts without obscuring controls or altering workflow. |

R1 excludes arbitrary CSS/JavaScript, unrestricted HTML, visual page-builder functionality, arbitrary structural navigation changes, per-user themes, native app theming and automated theming of email/PDF documents. These require separately approved scope. Existing financial documents retain their stored business data regardless of later portal styling changes.

### 8.1 Theme functional requirements

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-THEME-001 | M | Configure supported branding, colors, typography and component styles without code deployment. |
| FR-THEME-002 | M | Scope settings/assets to the correct HOA; prevent cross-tenant editing, preview and publication. |
| FR-THEME-003 | M | Provide approved presets and reset-to-default; reset creates a reviewable draft rather than silently overwriting live settings. |
| FR-THEME-004 | M | Preview unpublished settings on representative desktop/mobile pages; limit preview access to authorized users. |
| FR-THEME-005 | M | Apply published themes consistently to supported login, dashboard, statements, services, booking, concern and account pages. Pre-login HOA resolution must use a trusted routing configuration. |
| FR-THEME-006 | M | Retain immutable published versions and support authorized rollback to a previous compatible version. |
| FR-THEME-007 | M | Validate readable text contrast, visible focus and status/error distinction; never communicate transaction meaning through color alone. |
| FR-THEME-008 | M | Separate edit/preview/publish permissions as configured; audit changes, publication, reset and rollback; reject executable scripts and unsupported tokens. |
| FR-THEME-009 | M | Validate image assets, ownership/scope, accepted types and size limits before publication; prevent unsafe uploads. |
| FR-THEME-010 | M | Publish a complete validated version atomically; invalidate/version caches so users receive a coherent theme, not mixed settings. |
| FR-THEME-011 | M | Preserve navigation actions, permissions, fees, entitlements and workflow behavior across theme changes. |
| FR-THEME-012 | M | Use a safe default or last-known-good theme when a referenced asset/configuration is unavailable; keep core transactions usable and log the fault. |

### 8.2 Administration and lifecycle

Select current theme/preset → create editable draft → configure tokens/assets → server-side validation → desktop/mobile preview → approve if required → publish immutable version → observe → rollback if needed. Lifecycle: Draft → Validated → Published → Superseded; retain withdrawn/rejected draft history according to policy. Editing published settings creates a new draft. An unsuccessful validation/publication must leave the active version unchanged.

One active published version per HOA. Store who created, validated, approved and published each version, timestamps, previous version and schema/preset version. Publication and rollback permission need not include role/finance administration. Scheduled publication and multi-party approval are optional extensions unless specifically approved.

### 8.3 Implementation guidance

Implement a validated theme profile and version model, scoped asset references and an explicit active-version pointer. Map tokens to shared components/CSS variables or a versioned equivalent. Validate that both standard and custom portal templates consume the theme; do not assume a website theme covers custom account pages. Tenant-specific cache keys and trusted hostname routing are mandatory where shared infrastructure is selected. Use safe fonts/assets and retain compatible previous versions. Odoo-specific storage, assets and portal inheritance require a version/edition fit-gap, not an assumption.

Acceptance objectives: proposed WCAG AA text contrast checks (4.5:1 normal text, 3:1 large text), tested keyboard focus and non-color status cues; exact accessibility scope to be approved. Publication propagation target and allowed asset formats/sizes are TBD before implementation acceptance. A preview must not leak draft images to unauthorized users.

## 9. Architecture and Odoo/RESCo fit-gap

Candidate flow: browser → TLS/reverse proxy → Odoo portal and role-specific backend → HOA domain modules → PostgreSQL/controlled attachments. Background jobs process assessments, notifications, expiry and retries. External payment/messaging adapters remain separate. RESCo identity, UI, upload and payment-review patterns are reusable candidates only after code/security/license review.

| Candidate component | Responsibility / validation |
| --- | --- |
| Shared portal | Identity, uploads, navigation, notifications and theme-aware components; inspect actual RESCo coupling. |
| hoa_core | Properties, ownership, households, occupancy, membership, tenancy and access rules. |
| hoa_billing | Policy versions, snapshots, fallback assessment and authoritative ledger linkage. |
| hoa_services | Catalog, forms, seasonal requests, capacity and sticker issuance. |
| hoa_facilities | Resources, policies, transaction-safe holds and utilization. |
| hoa_cases / communications | Concerns, documents, announcements and restricted updates. |
| hoa_portal_theme — NEW | Scoped presets, validated tokens/assets, previews, publication versions, rollback and cache coordination. |
| Integration adapters | Approved payment/bank, email/SMS and future hardware interfaces. |

Candidate names are not claims that addons exist. Produce a requirement-to-standard-Community/Enterprise/OCA-or-third-party/custom matrix with licenses, dependencies, maintenance and recurring costs. Verify selected version before assuming helpdesk, appointments, subscription, accounting, website or theme functionality.

Decide database-per-HOA versus shared database, multi-company versus custom tenancy, identity scope, attachment isolation, queues, hosting, accounting boundaries and theme asset delivery. Separate frontend is not required for MVP; keep rules backend-owned so another client can later use them.

## 10. Integration and financial controls

- Use one authoritative posted financial ledger; custom charge generation must not create a conflicting balance store.

- Define API/event schemas, auth, tenant derivation, rate/error behavior, retry limits, correlation IDs and responsible owner.

- Idempotency covers billing runs, imports, orders, callbacks and refunds; replay never duplicates posting.

- Verify provider events server-side; browser return and screenshots cannot independently confirm settlement.

- Retain amount/currency, allocations, fees/settlement references and approved accounting treatment. Refunds/reversals retain original links.

- Prefer provider-controlled payment collection; do not store card secrets or log credentials.

- Reconciliation exceptions, unmatched funds and unapplied credits remain visible; approvals follow delegated authority.

## 11. Nonfunctional requirements

| ID | Requirement / verification |
| --- | --- |
| NFR-SEC-001 | Server-side isolation across pages/APIs/files/search/reports/exports; automated negative tests. |
| NFR-SEC-002 | TLS, approved secret/storage practices, privileged MFA where supported; security review. |
| NFR-SEC-003 | Audited support access and financial permission separation; role tests. |
| NFR-PRV-001 | Approved inventory, visibility, notice and retention rules; no public delinquency list; privacy review. |
| NFR-PERF-001 | Proposed p95 ≤3 seconds for interactive actions at an agreed workload, excluding third-party latency; load model TBD. |
| NFR-CON-001 | No duplicate exclusive bookings/charges under concurrency and retries; failure-injection tests. |
| NFR-AVL-001 | Availability/support window and maintenance exclusions TBD; monitoring evidence required. |
| NFR-REC-001 | RPO/RTO TBD; consistent database/attachment backup and recorded restore rehearsal. |
| NFR-AUD-001 | Protected audit, retention TBD, no secret leakage; inspect access and retention. |
| NFR-UX-001 | Responsive supported browsers/devices, keyboard-operable core journeys and readable errors; test matrix TBD. |
| NFR-OBS-001 | Alerts for job failures, stuck holds, queue lag, payment and notification exceptions; simulate alerts. |
| NFR-TIME-001 | Unambiguous stored timestamps; approved local timezone and cycle-date tests. |
| NFR-MAINT-001 | Versioned code/dependencies/migrations, reproducible deployment and upgrade rehearsal. |
| NFR-THEME-001 | Theme validation, coherent cache/version behavior, safe fallback and accessibility regression across supported pages; tests required. |

These are proposed targets, not measured capabilities or compliance certifications. Approve load, service and recovery values before production readiness sign-off.

## 12. Migration, cutover and operations

- Inventory and normalize sources with stable IDs; distinguish owner, resident, member and payer before import.

- Dry-run for duplicates, orphan dwellings, missing fallback owners, overlapping occupancy and unmapped balances.

- Approve opening balances/as-of date and reconcile counts, balances, credits and sample statements; prevent overlap with generated charges.

- Pilot an agreed subset; define freeze/final import, verification, go/no-go, rollback and staff/resident training.

- Restore database and files; configure queues/alerts; verify access and support accounts; publish manuals.

- Runbooks cover provider outages, failed assessments, stuck holds, payment exceptions, rejected uploads, access incidents and staff turnover. Theme runbook includes broken assets, cache mismatch and rollback.

## 13. Seed acceptance/UAT pack

| Test ID | Coverage | Expected outcome / scenario |
| --- | --- | --- |
| TC-MEM-001 | FR-MEM-001 / RULE-MEM-001 | 3 occupied → exactly 3 member charges; no owner fallback; lot area constant. |
| TC-MEM-002 | FR-MEM-001 / RULE-MEM-003 | 2 occupied/1 vacant → exactly 2, no owner extra. |
| TC-MEM-003 | FR-MEM-001 / RULE-MEM-003 | 1 occupied/2 vacant → exactly 1 occupied-unit charge. |
| TC-MEM-004 | FR-MEM-001 / RULE-MEM-002 | All vacant → exactly 1 owner fallback. |
| TC-MEM-005 | FR-BILL-001 / RULE-HIST-001 | Occupancy change after posting leaves historical bill unchanged. |
| TC-BILL-001 | FR-BILL-001 | Repeated assessment job does not duplicate charges. |
| TC-CAT-001 | FR-CAT-001–003 | Create/publish seasonal offer without code; enforce fields, windows, price and eligibility. |
| TC-CAT-002 | FR-CAT-005 | Delete unused draft; block referenced-item deletion; archive preserves orders. |
| TC-CAT-003 | FR-CAT-006 | Price change does not reprice accepted order. |
| TC-CAT-004 | FR-CAT-008 | Closed offer blocks new requests while pending work remains actionable. |
| TC-STK-001 | FR-STK-001 | Applicant/vehicle/payment/serial linked; payment does not imply issuance. |
| TC-SVC-001 | FR-SVC-001 | Concurrent requests cannot exceed service capacity. |
| TC-FAC-001 | FR-FAC-002 | Concurrent exclusive court bookings yield at most one confirmation. |
| TC-FAC-002 | FR-FAC-002 | Payment after expired/rebooked hold creates exception, not double booking. |
| TC-PAY-001 | FR-PAY-001 | Unverified/duplicate screenshot causes no automatic paid state or duplicate posting. |
| TC-PAY-002 | FR-PAY-002 | Partial/overpayment allocations reconcile and show outstanding/unapplied funds. |
| TC-SEC-001 | FR-ADM-001 / NFR-SEC-001 | Other HOA record, attachment and export attempts denied. |
| TC-SEC-002 | FR-ACC-001 | Ending occupancy/role revokes related access, preserves authorized history. |
| TC-CASE-001 | FR-CASE-001–002 | Confidential report/internal note visible only to intended roles. |
| TC-MIG-001 | FR-MIG-001 | Reconciled import rerun does not duplicate records. |
| TC-REC-001 | NFR-REC-001 | Restore validates consistent data/files and approved targets. |
| TC-INT-001 | FR-PAY-003 | Authentic replayed callback creates no duplicate payment, if integration is in scope. |
| TC-THEME-001 | FR-THEME-001/005 | Change approved colors/logo/style and publish; supported portal pages use the new version without deployment. |
| TC-THEME-002 | FR-THEME-002 | Change HOA A theme; HOA B and unauthorized preview requests remain unaffected/denied. |
| TC-THEME-003 | FR-THEME-004 | Draft previews on mobile/desktop while live portal stays unchanged. |
| TC-THEME-004 | FR-THEME-006 | Rollback restores prior compatible theme without modifying transactions. |
| TC-THEME-005 | FR-THEME-007 | Reject inaccessible configured text combinations; verify focus/status cues and non-color meaning. |
| TC-THEME-006 | FR-THEME-008/009 | Unauthorized publish, malicious input and invalid assets rejected and appropriately audited. |
| TC-THEME-007 | FR-THEME-010 | Publication failure leaves old version active; successful publish serves coherent versioned assets. |
| TC-THEME-008 | FR-THEME-011 | Theme changes preserve fees, permissions, menu actions and business validation. |
| TC-THEME-009 | FR-THEME-012 | Unavailable asset/configuration triggers safe fallback and logged fault; core journeys remain usable. |
| TC-THEME-010 | FR-THEME-003/008 | Reset produces draft; authorized publication and rollback record actor/time/version. |

QA must expand field/permission combinations, pricing, cancellations, negative cases, migrations, integration failures and regression tests. Every in-scope requirement must link to passing evidence, not merely a listed test. Use synthetic/protected data.

## 14. Traceability and change management

Required register fields: stable requirement ID; source/business owner; statement; approval status; related rules; priority/release; acceptance criteria; design/module; implementation issue/PR; test/evidence; manual/runbook section; change request. Many-to-many links are supported. Retire IDs with rationale rather than reusing them.

| Coverage | Design candidate | Acceptance pack | Documentation |
| --- | --- | --- | --- |
| Membership/assessments | hoa_core / hoa_billing | TC-MEM-001–005; TC-BILL-001 | Admin occupancy/assessment; treasurer |
| Catalog/services/stickers | hoa_services | TC-CAT-001–004; TC-STK-001; TC-SVC-001 | Resident requests; admin catalog and fulfillment |
| Facilities | hoa_facilities | TC-FAC-001–002 plus detailed cases | Resident booking; staff facilities |
| Payment | Ledger/provider adapters | TC-PAY-001–002; TC-INT-001 | Treasurer verification/reconciliation |
| Access/isolation | Access and storage scope | TC-SEC-001–002 | Role administration and runbook |
| FR-THEME-001–012; RULE-THEME-001–002 | hoa_portal_theme / shared UI | TC-THEME-001–010 | Admin theme configuration; operations rollback/cache |

Implementation references, test results and sign-offs are TBD. Change workflow: request → impact analysis (business/financial/privacy/design/test/manual) → approval → versioned baseline update → implementation → evidence → release. Policy/fee changes require appropriate HOA authority, not merely developer acceptance.

## 15. Developer deliverables and manuals

| Deliverable | Minimum acceptance |
| --- | --- |
| Technical design | Version/edition and addon/license fit-gap; RESCo inspection; ERD/access; concurrency; APIs; queues; theme token/schema and portal page coverage. |
| Implementation | Versioned source, deployment/configuration templates without secrets, migrations, demo data and automated tests. |
| QA/UAT | Complete requirement-test mapping, executed evidence, defects/limitations and approval. |
| Resident manual | Onboarding, linked properties, statements/payments, stickers/services, booking, concerns and preferences; actual released screens. |
| Administrator manual | Registry/occupancy, assessments, catalog/seasons, fulfillment, facilities, communications, concerns and roles; new theme preset/edit/preview/publish/reset/rollback tasks. |
| Treasurer manual | Opening balances, assessments, verification/allocation/reconciliation, adjustments/refunds and reporting. |
| Operations runbook | Deploy/restore/monitor, retry and job recovery, incidents/access revocation; theme failure, asset/cache diagnosis and rollback. |
| Release record | Version, requirement IDs, migrations/configuration, manual updates, rollback and approvals. |

Each manual task includes role, prerequisites, steps, expected outcome, error handling and escalation; screenshots follow stable UI. Manuals track released behavior and version. Definition of ready: owner, rule, priority, acceptance and dependencies clear. Definition of done: reviewed implementation, tests/security/financial validation, migration/configuration documentation, traceability/manual updates and product acceptance.

## 16. Open decisions and sign-off

| ID | Decision required |
| --- | --- |
| DEC-001 | Billing cycle and mid-month occupancy: snapshot versus proration; retroactive corrections. |
| DEC-002 | Membership representative/eligibility and voting treatment under governing policy. |
| DEC-003 | Fallback grouping for co-owners, multiple buildings and legally split parcels. |
| DEC-004 | Occupant arrears, owner guarantees and debt transfer. |
| DEC-005 | Rates, due dates, penalties, waivers and approval authority. |
| DEC-006 | Booking entitlement basis, prices, hold timeout, refunds, deposits and no-shows. |
| DEC-007 | Catalog approvers, supported templates/forms, stock/capacity and cancellation policy. |
| DEC-008 | Odoo version/edition, RESCo reuse, addon licensing, identity, deployment and isolation. |
| DEC-009 | R1 payment scope, provider, merchant/fee/settlement and bank integration. |
| DEC-010 | Privacy documents, visibility, retention, lawful processing and support access. |
| DEC-011 | Scale/load, availability/support hours and recovery objectives. |
| DEC-012 | Ledger/accounting mapping, tax treatment, migration source and cutover. |
| DEC-013 | Staffing, concern targets, escalation and emergency-channel wording. |
| DEC-THEME-001 | Approve default/preset themes, editable token set, fonts and assets. |
| DEC-THEME-002 | Theme editor/publisher roles and whether separate approval is required. |
| DEC-THEME-003 | Approved file formats/size/dimensions, accessibility scope and propagation target. |
| DEC-THEME-004 | Confirm emails/PDF styling, full dark mode and layout/page-builder are outside R1 unless separately approved. |

Risks: customer-only model reuse; unverified platform capabilities; confusing platform/resident tenancy; area/membership double counting; historical bill changes; screenshot-as-payment; deletion of referenced offers; booking/payment races; excess personal-data access; unscoped theme assets; unsafe styles; inaccessible color combinations; and partial theme cache updates.

| Gate | Evidence / owner |
| --- | --- |
| Business baseline | Approved rules/priorities and explicit decisions/deferments — product owner/HOA process owner |
| Technical baseline | Fit-gap/licenses, data/access/concurrency/theme architecture — technical lead |
| Financial readiness | Scenario results, balances and posting/reconciliation controls — treasurer |
| Security/privacy readiness | Negative tests, retention/access and reviewed notices — designated reviewer |
| Pilot/production | In-scope test evidence, training, restore rehearsal, manuals and known issues — sponsor/operations |

## Appendix. Source and revision notes

Source requirements: collections, members, facilities/utilization, announcements/events, concerns; RESCo reuse/Odoo proposal; multiple dwellings on constant-size lots; explicit occupied-unit membership/all-vacant owner fallback; handoff/traceability/manuals; configurable stickers and seasonal services; and requested portal color/style configuration. All are conversation inputs. Proposed controls and technical designs require review, not legal or vendor certification.

v0.1: initial business/functional/technical draft. v0.2: explicit theme rules, theme profile/version model, 12 theme functional requirements, one theme NFR, 10 acceptance scenarios, architecture/role/screen/manual/traceability updates and four theme decisions. Reconstructed from the conversational draft content; retain the original v0.1 separately for comparison.

