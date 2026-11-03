# Village App

Business, Functional and Technical Specification

Draft v0.3 • 03 November 2026

| Control | Value |
| --- | --- |
| Status | DRAFT — for business review and preliminary estimation; not an approved implementation baseline |
| Revision | v0.3 adds tenant/village configuration, defaults/overrides, effective-dated rules, approvals, simulations, implementation guidance, acceptance tests and traceability; retains v0.2 themes. |
| Reconstruction note | v0.2 was reconstructed from v0.1 conversation content. v0.3 directly revises the uploaded v0.2 Word document; prior provenance is retained. |
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
| R1 — MVP | Property, dwelling and occupancy registry; assessments/statements; manual payment verification; configurable service catalog; stickers/seasonal requests; facilities; announcements/events; documents; concerns; dashboards; controlled portal theme configuration. Organization/village configuration framework, policy history, high-risk configuration approval and two-village validation are required foundation capabilities. |
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

Platform tenant = one independently governed HOA organization, containing one or more villages and subordinate phases/blocks. Business configuration is scoped to the organization or village as declared in the configuration dictionary. Unrelated HOAs remain separate tenants even when one management company administers them. Management-company access is an explicitly granted relationship, not a merged data boundary. A platform tenant is distinct from a resident tenant/renter. Physical deployment isolation and cross-HOA identity remain decisions; village policy configuration does not itself provide security isolation.

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
| RULE-MEM-001 | Confirmed — initial village policy | One membership unit for each occupied dwelling; occupant means its designated representative, not every resident. |
| RULE-MEM-002 | Confirmed — initial village policy | When all dwellings on the lot/apartment property are vacant, owner pays exactly one minimum membership unit. |
| RULE-MEM-003 | Confirmed — initial village policy | No additional owner minimum or vacant-unit membership charge while at least one dwelling is occupied. |
| RULE-MEM-004 | Derived — initial village policy | Initial village: billable units = max(1, verified occupied units) under approved period policy. This formula is not imposed on all villages; other supported approved policies are configurable. |
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

For the initial village policy, membership total = rate R × max(1,N). Other villages may select different approved, supported assessment strategies. Any land-area charge is separately authorized. Mid-period changes, co-owner responsibility and fallback grouping remain decisions for each applicable policy. No developer may assume a platform-wide occupancy formula.

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

Implement a validated theme profile and version model, scoped asset references and an explicit active-version pointer. Map tokens to shared components/CSS variables or a versioned equivalent. Validate that both standard and custom portal templates consume the theme; do not assume a website theme covers custom account pages. Tenant-specific cache keys and trusted hostname routing are mandatory where shared infrastructure is selected. Use safe fonts/assets and retain compatible previous versions. Odoo-specific storage, assets and portal inheritance require a version/edition fit-gap, not an assumption. Theme inheritance and overrides must follow the configuration scope and permission model in Section 17; display the effective source. The configuration framework does not remove theme-specific validation, atomic publication or cache safeguards.

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
| TC-MEM-001 | FR-MEM-001 / RULE-MEM-001 | Using the initial village policy: 3 occupied → exactly 3 member charges; no owner fallback; lot area constant. |
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
| DEC-003 | Fallback grouping for co-owners, multiple buildings and legally split parcels. Resolve per village/policy rather than as an unavoidable global constant. |
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

## 17. Multi-tenant rules and village configuration — v0.3

This section extends Sections 1–16. Where earlier generic wording might imply a universal village policy, this section governs scope and configurability. Existing confirmed occupancy rules remain the initial village’s policy. This does not declare a universal fee, membership, or legal entitlement across HOAs.

### 17.1 Configuration principles and hierarchy

One reusable codebase shall serve independently configured HOAs and villages. Adding a village or changing a supported parameter shall not require code forks, redeployment or village-name/ID conditionals. New calculation strategies, workflow behaviors or integration types may require separately versioned platform development. “Configurable” means controlled capabilities, not arbitrary script execution.

Hierarchy: platform-supported defaults → HOA organization defaults → permitted village overrides. Each setting declares its allowed scope, inheritance behavior and override permissions. Configuration may be village-specific even when the financial/legal organization remains the HOA. A village must have exactly one owning tenant for authorization.

Do not introduce phase/property/person overrides by accident. Such scopes require named requirements and explicit precedence. Missing values use declared defaults or block the dependent operation with an actionable error; never guess a rate, responsible party or policy. An explicit zero/false value is not a missing value.

Administrators shall see effective value, inherited/overridden status, source scope/version, effectivity, approval and reset behavior. Reset-to-inherited creates a draft consequential change. Parent default changes must not silently change binding village policy; resolve and approve affected policy bundles before activation. Ordinary cosmetic defaults may propagate only according to declared inheritance semantics.

### 17.2 Configuration inventory

| Domain | Configurable scope / limitations |
| --- | --- |
| Identity and localization | Names, contacts, phase/block labels, timezone/currency, office hours; changes affecting existing financial records require migration review. |
| Branding and portal | Theme defaults/overrides, approved presets, logos, assets, colors, fonts and banners; retain Section 8 safeguards. |
| Module enablement | Enable supported facilities, stickers, seasonal services, concerns and later modules; do not grant permissions merely by enabling a module. |
| Membership and occupancy | Approved assessment strategy, representative eligibility, verified occupancy treatment, minimum units and fallback payer logic. |
| Assessment and rates | Per occupied unit, per lot, per land area, fixed or supported combination; approved effective-dated rate and billing-group definition. |
| Billing periods | Frequency, assessment date/cutoff, due dates and supported proration strategy; not arbitrary formulas. |
| Collections | Supported methods, allocation priority, partial/advance handling, reminder schedule, approved penalties, waivers and thresholds. |
| Facility rules | Hours, closures, resource rates, guest eligibility, booking entitlement basis, advance windows, hold timeout, cancellations, deposits and no-shows. |
| Catalog/services | Village-scoped offerings, price versions, seasonal windows, form fields, eligibility, limits, capacity and supported fulfillment template. |
| Approvals | Supported stages, role/threshold routing, delegation and escalation; preserve financial segregation of duties. |
| Concerns/operations | Categories, teams, priority, response/closure targets, escalation, reopening and privacy classification. |
| Communications/documents | Audience groups, approved templates, channels, reminders, publication approval and document visibility. |
| Onboarding | Required supported fields/documents, verification and authorized account invitation/access processes. |
| Reporting | Supported dashboard selections, reporting periods, classifications and authorized publication views. |
| Privacy/retention | Approved purpose/visibility/retention selections within platform/legal safeguards; not an arbitrary disable switch. |
| Integration bindings | Explicit merchant/bank/sender and provider bindings. Secrets are stored separately with scoped references and are never copied or inherited accidentally. |

### 17.3 Configuration dictionary and data model

Every supported parameter requires a dictionary entry: stable key; description; domain; type/unit; allowed scope; default; allowed values/range; required/nullable state; inheritance and null/reset semantics; dependencies; edit/approve/publish roles; risk classification; effective-date behavior; validation; implementation reference; test IDs; documentation; secret classification and compatibility/schema version. Maintain this as a developer deliverable, not prose-only promises.

| Entity / artifact | Required design |
| --- | --- |
| Configuration definition | Supported key/type, validation, scope and inheritance metadata; platform-maintained schema. |
| Configuration bundle/version | Tenant, optional village, domain, schema version, settings, status, effective dates, parent/source references, actor/approval and previous version. |
| Effective configuration resolution | Deterministic resolution with effective timestamp; return source/version and use tenant/village-aware cache keys. |
| Policy strategy | Approved named calculation/workflow strategy plus validated parameters; no general-purpose resident-authored executable code. |
| Transaction policy snapshot | Record applied policy/version and material resolved inputs on charge/order/reservation/request; retain explanation after defaults change. |
| Simulation result | Proposed version, input dataset/as-of date, expected changes, exceptions, totals, reviewer and limitations. |
| Integration binding | Explicit scope and secret-store reference; validate merchant and callback/account mapping without exposing credentials. |

Overlapping active versions for the same setting scope/domain and time interval are prohibited unless explicit priority semantics are defined. Related changes activate as one coherent bundle; do not expose half-updated fee/eligibility rules. Use fixed precision for money and controlled units; identify timezone boundaries explicitly.

### 17.4 Policy examples and initial village setup

| Parameter | Initial village policy | Other supported example — illustrative only |
| --- | --- | --- |
| Membership charge basis | Verified occupied dwelling units | Fixed per-lot assessment |
| Minimum membership units | One only when all dwellings are vacant | Explicit approved policy-specific minimum |
| Fallback payer | Property owner when every dwelling is vacant | Explicitly configured authorized billing relationship |
| Vacant-unit extra | None while another dwelling is occupied | Separate charge only if independently approved |
| Rate and period basis | Rate/cycle and snapshot versus proration TBD | Village-specific approved rate/cycle |
| Facility allowance | Basis and limits TBD | Supported per-lot, per-dwelling, per-household or per-member grouping |

The alternative examples are not approved HOA policies. Initial setup must explicitly select the occupancy strategy and correct fallback rules, never assume max(1,N) for every tenant. The TC-MEM seed tests apply to that initial policy fixture. Dwellings still do not duplicate land area under any strategy. Voting remains separate from financial assessment configuration.

### 17.5 Approval, activation and historical integrity

Lifecycle: Draft → Validated → Simulated where required → Reviewed/Approved → Scheduled → Active → Superseded/Retired. Rejected/withdrawn outcomes retain appropriate audit. High-risk domains include rates, membership calculation, penalties, financial approvals, payment routing, eligibility and retention. Role separation and approval thresholds must be approved.

Before activation, show affected villages, inherited dependencies, changed values, target date, impacted future assessments/orders and outstanding obligations. Simulation must not post invoices, allocate payments or contact residents. Test and sandbox execution must not use production merchant credentials.

Transaction treatment must be explicit: posted financial records remain immutable except authorized linked adjustments; accepted orders and confirmed reservations retain accepted price/policy; new work uses the applicable active policy; unconfirmed drafts follow a documented revalidation rule with visible differences and reacceptance when material. Backdated changes cannot silently regenerate history.

Rollback/reactivation is a new audited policy event after validation and approval; it does not refund a payment, reverse an invoice or cancel a booking. Corrective business transactions use their own workflows. Concurrency controls must prevent activation conflicts and validate configuration again at transaction commitment where relevant.

### 17.6 Portability, safeguards and boundaries

Onboard from sanitized approved configuration templates. Copy only eligible schema-compatible settings; exclude residents, private files, transactions, credentials and identifiers that bind to another tenant’s roles/resources. Remap local roles, facilities and accounts explicitly. Preview and validate configuration imports before activation; reject unknown or disallowed settings.

Business administrators may not weaken tenant isolation, audit integrity, authentication/secret protections, safe upload validation, ledger consistency or required security controls. Regulatory/policy constraints require authorized review. Feature visibility is distinct from authorization; module deactivation must not conceal outstanding liabilities or block authorized settlement/refund/history access.

Users with management access to multiple HOAs receive explicit memberships/scopes; cross-tenant search/export is denied absent a separately approved capability. Scope background jobs, cache keys, storage, event handlers, email sender selection and merchant mapping. Logs shall identify scope without leaking private settings/secrets.

### 17.7 Functional requirements

| ID | Priority | Requirement |
| --- | --- | --- |
| FR-CONFIG-001 | M | Maintain independently scoped organization and village configuration with an explicit owning tenant. |
| FR-CONFIG-002 | M | Provide typed validated settings with defaults, allowed values, dependencies and override permissions. |
| FR-CONFIG-003 | M | Resolve declared defaults/overrides deterministically; show effective value and source/version. |
| FR-CONFIG-004 | M | Configure supported membership, assessment, collection, facility, service, communication and approval policies without code forks. |
| FR-CONFIG-005 | M | Version consequential policies and apply approved effective dates without silently changing historical transactions. |
| FR-CONFIG-006 | M | Preview/simulate assessment and designated high-risk changes without posting or external side effects. |
| FR-CONFIG-007 | M | Permission-control and audit configuration changes, approvals, activation, resets and retirement. |
| FR-CONFIG-008 | M | Onboard villages using sanitized approved templates, without copying operational/personal data or secrets. |
| FR-CONFIG-009 | M | Validate dependencies and outstanding obligations before disabling modules or retiring policies. |
| FR-CONFIG-010 | M | Prevent configuration from weakening isolation, access, audit, financial integrity or platform security safeguards. |
| FR-CONFIG-011 | M | Deliver and maintain a configuration dictionary covering every supported parameter and its implementation/test/manual links. |
| FR-CONFIG-012 | M | Operate at least two differently configured test villages simultaneously without code forks or cross-village effects. |
| FR-CONFIG-013 | M | Validate configuration exports/imports for authorization, scope, schema compatibility, secret exclusion and local reference remapping. |
| FR-CONFIG-014 | M | Activate related settings coherently and prevent conflicting effective versions; invalidate caches consistently. |
| FR-CONFIG-015 | M | Persist and expose applied policy/version and material basis for consequential transactions. |
| FR-CONFIG-016 | M | Require explicit authorized integration bindings; do not inherit or copy another village’s merchant/secret configuration. |

M denotes proposed MVP foundation requirement, requiring baseline approval. The framework is MVP scope; later functional domains become configurable when those domains are delivered. This does not bring R3 construction, elections or gate hardware into R1.

### 17.8 Additional business and nonfunctional rules

| ID | Requirement / rule |
| --- | --- |
| RULE-CONFIG-001 | Initial occupancy rules are village-policy fixtures, not universal platform behavior. |
| RULE-CONFIG-002 | Inheritance occurs only for declared settings/scopes; explicit zero/false is preserved. |
| RULE-CONFIG-003 | Approved consequential policy snapshots protect transactions from future parent/default changes. |
| RULE-CONFIG-004 | Configuration imports/templates exclude secrets, transactions and personal data. |
| RULE-CONFIG-005 | No configuration change bypasses authorization, isolation, financial integrity or audit controls. |
| NFR-CONFIG-001 | Test policy resolution, cache and job isolation for multiple villages/tenants; reject unsupported config and ensure deterministic results. |
| NFR-CONFIG-002 | Version schema/config migrations and rehearse upgrades without policy loss or cross-tenant leakage. |
| NFR-CONFIG-003 | Approve activation/propagation performance targets under an agreed workload; do not assert instantaneous propagation without evidence. |

### 17.9 Multi-village acceptance tests

| Test ID | FR-CONFIG suffix coverage | Expected outcome |
| --- | --- | --- |
| TC-CONFIG-001 | 001/004/012 | Configure two villages with different membership strategies/rates, booking limits, catalogs, approval routes and themes; both run correctly without code changes. |
| TC-CONFIG-002 | 002/003 | Validate defaults, allowed overrides, explicit zero/false, invalid ranges, required settings and source display. |
| TC-CONFIG-003 | 005/015 | Change future rate/policy; old posted charges and accepted bookings/orders retain policy/version and material inputs. |
| TC-CONFIG-004 | 006 | Simulate initial-village occupancy scenarios and an alternative strategy; totals/exceptions reconcile; no posting, payment or notifications occur. |
| TC-CONFIG-005 | 007/010 | Unauthorized edit/approval/activation and attempts to weaken isolation/audit controls are rejected and audited. |
| TC-CONFIG-006 | 008/013 | Create village from sanitized template/import; no residents, balances, secret values or foreign scoped resource bindings are copied. |
| TC-CONFIG-007 | 009 | Disable an offering/module with outstanding work/payments; prevent unsafe deactivation or preserve approved completion/refund/history access. |
| TC-CONFIG-008 | 014 | Concurrent activations and publication failure leave one coherent active version; never mix rate and policy components. |
| TC-CONFIG-009 | 003/014 | Parent setting changes and cache refresh do not silently alter approved child financial policy or another village/tenant. |
| TC-CONFIG-010 | 005/007 | Reactivate old policy through a new approved version; historical transactions remain unchanged. |
| TC-CONFIG-011 | 016 | Village A payment/callback uses A merchant binding; B cannot see/use A secret; missing binding blocks unsafe payment initiation. |
| TC-CONFIG-012 | 001/010/012 | Probe cross-tenant/village pages, files, exports, jobs, search and cache with permitted/forbidden roles; no unauthorized leakage. |
| TC-CONFIG-013 | 005/014 | Test effective-date boundary, timezone, missing strategy, invalid overlapping periods and retroactive update review. |
| TC-CONFIG-014 | 011/013 | Dictionary covers every implemented setting; export/import rejects unsupported schema/keys and validates local mappings. |

Also rerun TC-MEM-001–005 against the initial village fixture and all prior theme/payment/booking tests. Detailed cases must cover both multiple villages under one tenant and unrelated tenants. A single tenant with two villages is not sufficient evidence of cross-tenant isolation.

### 17.10 Implementation, traceability and manuals

Add a candidate hoa_configuration component with schema registry, validated scope resolution, versioned policy bundles, strategies, approvals and simulation interfaces. Business modules consume a shared policy resolver rather than reading ad hoc settings. Bind transaction/tenant context server-side. No specific Odoo multi-company, parameter or record-rule mechanism is certified as sufficient without fit-gap and tests.

| Requirements | Candidate implementation | Acceptance | Manual/runbook |
| --- | --- | --- | --- |
| FR-CONFIG-001–003/011 | Scope model, dictionary and resolver | TC-CONFIG-001/002/009/014 | Admin: inherited values, overrides and dictionary |
| FR-CONFIG-004–007/015 | Policy strategies, snapshots, simulation and approval | TC-CONFIG-003/004/005/010/013 | Admin/treasurer: simulate, approve and activate |
| FR-CONFIG-008/013 | Sanitized templates/import/export | TC-CONFIG-006/014 | Admin: village onboarding and portability |
| FR-CONFIG-009/010/014 | Dependency controls and coherent publication | TC-CONFIG-007/008/012 | Operations: safe deactivation and config/cache recovery |
| FR-CONFIG-012/016 | Multi-village fixtures and scoped integration bindings | TC-CONFIG-001/011/012 | Operations: isolation checks and merchant routing |

Developer deliverables: configuration dictionary; strategy coverage/limitations matrix; schema/version migration plan; policy fixture packs; simulations; complete requirement-test links; access/approval matrix; and administrator/treasurer/runbook updates. Manuals must show preview, inherited-source inspection, override/reset, approvals, activation, impact review, templates and safe rollback. Full manuals remain separate release-specific deliverables.

### 17.11 Open decisions and release controls

| ID | Decision required |
| --- | --- |
| DEC-CONFIG-001 | Confirm one-HOA-per-tenant hierarchy and when multiple villages share organization-level settings. |
| DEC-CONFIG-002 | Approve MVP strategy inventory, editable parameter set, default values and allowed override scopes. |
| DEC-CONFIG-003 | Approve policy edit/review/activation roles, risk classes and delegation rules. |
| DEC-CONFIG-004 | Approve parent-change propagation, draft repricing/revalidation and effective-date/proration policies. |
| DEC-CONFIG-005 | Select deployment/identity/isolation and cross-HOA management access model. |
| DEC-CONFIG-006 | Approve template portability, credential binding, retention constraints and local resource remapping. |
| DEC-CONFIG-007 | Set simulation scope, data volumes, activation/caching performance and readiness evidence. |

v0.3 release note: adds 16 functional requirements, five business rules, three nonfunctional requirements and 14 seed acceptance scenarios. Clarifies initial-village membership scope and integrates tenancy, themes, implementation, deliverables and testing. All new framework requirements are proposed pending product/technical review. This revision updates the document only; it does not update the GitHub repository automatically.
