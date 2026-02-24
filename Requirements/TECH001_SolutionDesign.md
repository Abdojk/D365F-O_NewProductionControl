# Solution Design — TECH001

**Title:** D365 F&O — Capitalise Pre-Sales Engineering Timesheets into Production Order Cost
**Date:** 2026-02-24
**Status:** Draft — Pending Business Clarification on Open Questions

---

## 1. Requirement Summary

An engineer-to-order manufacturer records pre-sales engineering effort (AutoCAD design timesheets) against a project in D365 F&O during the tendering phase — before a production order exists. If the tender is won, the accumulated engineering labour cost (currently held in project WIP) must flow into the finished-goods inventory cost via the project production order. If the tender is lost, the cost must be expensed. The solution must provide a full audit trail, prevent double-counting, and minimise manual rework.

**Key Facts (confirmed by requirement inputs):**

- Pre-sales time is already recorded via **Project Management & Accounting timesheets** (Projects module)
- Timesheets currently post to **WIP** (balance sheet)
- Production is via **project production order** (ETO scenario)
- **Cost groups** are used for labour absorption
- Costing method: **Hour cost**
- CRM/tender linking: **Not relevant**
- Financial policy: **Capitalise into inventory only if tender is won; expense if lost**

---

## 2. Assumptions & Open Questions

### Assumptions

- **A1:** A pre-sales / internal project (project type = Internal, Investment, or Time) already exists for each tender and timesheets are posted against it. The requirement statement confirms "currently the project is created and Projects timesheets are recorded."
- **A2:** When the tender is won, a separate delivery/production project is created (project type that supports item requirements and project production orders — typically Fixed-price or Investment type). This is the "winning project."
- **A3:** The project production order is created from item requirements on the winning project, and master planning firms planned orders into production orders linked to that project.
- **A4:** The company uses project groups and posting profiles that post hour transactions to WIP (balance sheet) accounts.
- **A5:** The D365 F&O version supports standard Project Management & Accounting functionality including project adjustments, estimate/elimination, and project production orders.
- **A6:** Financial dimensions are used to track cost by project/tender for reporting purposes.

### Open Questions (require business/functional team clarification)

| ID | Question | Impact |
|----|----------|--------|
| OQ-1 | What project type is used for the pre-sales project? (Internal, Investment, Time & Material, or Fixed-price?) | Affects posting behavior and WIP handling |
| OQ-2 | When the tender is won, is the pre-sales project closed/completed, or does it continue as the delivery project? Or is a completely new project created? | Determines whether Option A or C is the right fit |
| OQ-3 | Is there a one-to-one relationship between pre-sales project and winning project, or can one pre-sales project feed multiple production projects (or vice versa)? | Affects transfer/split logic |
| OQ-4 | What is the inventory costing model for the finished goods item? (Standard cost, FIFO, Weighted average, Moving average?) | Standard cost would create variances rather than absorbing transferred cost into inventory |
| OQ-5 | Are project cost categories aligned with production cost groups? If not, mapping will be needed. | Configuration complexity |
| OQ-6 | Is the estimate/elimination process already operational for existing projects? | If not, represents significant additional implementation effort |
| OQ-7 | What approval workflow exists for project adjustments/transfers? | Governance and controls |
| OQ-8 | Does the company use project budgets or forecasts that would need to account for transferred pre-sales costs? | Budget management impact |

---

## 3. Solution Design by Requirement

### Option Matrix

| Option | Tier | Core Mechanism | Fit | Setup Complexity | Audit Strength | Accounting Risk | Effort |
|--------|------|---------------|-----|-----------------|---------------|----------------|--------|
| **A — Project Adjustment (Transfer hours to winning project)** | B | Project adjustment transactions to reclassify hours from pre-sales project to winning project; project production order absorbs project costs via estimate/elimination | ETO with project production orders | Medium | High — full project transaction audit trail | Low — stays within project costing framework | Medium |
| **B — Two-project model with manual cost journal on production order** | C | Pre-sales WIP reversed via project elimination; production adjustment journal adds cost to production order | When project-to-production linkage is weak | High | Medium — requires reconciliation across project and production journals | Medium — manual journal risks error | High |
| **C — Single project lifecycle (pre-sales becomes delivery project)** | A/B | Pre-sales project continues as the delivery project; item requirements and project production order created on same project; pre-sales hours naturally included in project cost pool | When same project can span pre-sales to delivery | Low | High — single project, no transfer needed | Low — seamless cost flow | Low |

---

### REQ-001: Capture Pre-Sales Engineering Labour Cost in Project WIP

- **Tier:** A (Full OOB)
- **Approach:** This is already in place. Engineers record timesheets against a pre-sales project in the Projects module. The project group and posting profile are configured to post hour transactions to WIP (balance sheet). No change required.
- **D365 Components:**
  - Project Management & Accounting > Timesheets
  - Project groups (configured for WIP posting)
  - Project posting profiles (hour transactions → WIP ledger accounts)
  - Cost categories (linked to cost groups for labour)
- **Risks:** None — this is existing functionality already operational.

---

### REQ-002: Link Pre-Sales Cost to Winning Project / Production Order Upon Tender Win

#### Option A — Project Adjustment / Transfer (RECOMMENDED)

- **Tier:** B (Partial OOB + Workaround)
- **Why not Tier A:** Standard D365 F&O does not automatically transfer costs between projects upon a business event (tender win). However, the **project adjustment** functionality is standard OOB and allows reclassification of posted hour transactions from one project to another. This requires a documented manual/semi-manual process step (the "workaround"), not custom code.

**Approach — End-to-End Process Steps:**

1. **Pre-sales phase:** Engineers post timesheets against the pre-sales project (e.g., Project P-1001). Hours post to WIP per project posting profile.
2. **Tender won:** A delivery project (e.g., Project D-2001) is created. Item requirements are entered. Master planning generates planned production orders. Production orders are firmed — these are **project production orders** linked to Project D-2001.
3. **Cost transfer:** Using **Project Management & Accounting > Periodic > Adjustments**, create adjustment transactions that:
   - Credit (reverse) the hour cost on Project P-1001
   - Debit (post) the same hour cost on Project D-2001
   - This moves the cost from the pre-sales project to the delivery project while maintaining the original transaction reference.
4. **Estimate and elimination:** Run the **Estimate** process on Project D-2001. The estimate process calculates WIP based on all project costs (including the transferred hours). When the project production order is completed and ended, the **Elimination** process posts WIP to inventory/COGS per the project's completion method.
5. **Production order settlement:** The project production order's cost includes the project's cost pool. The production order is estimated, started, reported as finished, and ended. Upon ending, production order settlement posts variances.

**D365 Components:**

- Project Management & Accounting > Adjustments (transaction adjustment form)
- Project groups and line properties
- Project posting profiles (WIP accounts, cost accounts)
- Estimate and elimination periodic processes
- Production control > Project production orders
- Cost categories and cost groups
- Project parameters (adjustment settings)

**Configuration Required:**

- Project parameters: Enable adjustments (Project management and accounting parameters > General or Journal)
- Project posting profiles: Ensure both pre-sales and delivery projects use compatible posting profiles where WIP accounts are correctly defined for hour transactions
- Cost categories: Must be consistent between pre-sales timesheets and delivery project cost recognition
- Project group for delivery project: Must support estimate/elimination and be linked to a project type that allows item requirements and project production orders (Fixed-price or Investment)
- Line properties: Configure to control whether transferred hours post to WIP or accrued revenue

**Accounting Impact:**

| Event | Debit | Credit | Effect |
|-------|-------|--------|--------|
| Timesheet posted (pre-sales) | WIP — Project Labour (P-1001) | Payroll allocation / offset | Cost on balance sheet |
| Adjustment — transfer | WIP — Project Labour (D-2001) | WIP — Project Labour (P-1001) | Cost moves between projects; net zero P&L |
| Estimate (delivery project) | — | — | WIP calculated inclusive of transferred hours |
| Elimination (production complete) | Inventory — Finished Goods | WIP — Project Labour (D-2001) | Pre-sales labour in inventory value |

**Data Linkage:**

- Adjustment transaction references the original timesheet transaction ID
- Project D-2001 contains all costs (transferred + direct)
- Production order is linked to Project D-2001 via item requirement
- Financial dimensions carry through from project to production order

**Controls & Audit Trail:**

- Original timesheet on P-1001 remains visible (marked as adjusted)
- Adjustment transaction on D-2001 shows source project reference
- Project transaction inquiry shows full trail: original entry → adjustment → estimate → elimination
- Approval workflow can be configured for project adjustments
- Prevention of double counting: Original transaction on P-1001 is credited (reversed) when transferred, so it cannot be counted again

**Risks & Limitations:**

| Risk | Impact | Mitigation |
|------|--------|------------|
| Pre-sales project group posts hours to P&L (not WIP) | Adjustment creates P&L reversal/repost instead of WIP-to-WIP transfer | Confirm project group posts to WIP before go-live |
| Partial transfer complexity | Only some hours should transfer; must specify exact transactions/amounts | Document selection criteria; use transaction filters in adjustment form |
| Timing of adjustments vs. estimate/elimination | Costs excluded from estimate if adjustment posted late | Include adjustment posting as mandatory step before estimate in period-end checklist |
| Manual process risk | Adjustments are manual unless automated | Consider batch automation (elevates to Tier C) only if volume justifies it |

`Needs Verification:` The exact behavior of project adjustment transactions posting across different project types (e.g., from Internal project to Fixed-price project) and whether WIP posting profiles apply correctly in cross-project-type adjustments.

**Effort Level:** Medium

---

#### Option B — Production Order Cost Adjustment Journal

- **Tier:** C (Partial OOB + Customization)
- **Why Tier A/B ruled out:** Standard D365 F&O does not provide an OOB mechanism to post an arbitrary cost (from a project) directly as an additional cost line on a production order without customization. The production order cost is driven by BOM, route, and indirect costs — not project hour transfers. A production adjustment journal or production journal (route card/job card) could be used, but mapping project hours to production route operations requires manual intervention or custom logic.

**Approach:**

1. Pre-sales hours recorded on pre-sales project (same as Option A).
2. Tender won, project created, production order created.
3. **Reverse pre-sales WIP:** Run estimate/elimination on the pre-sales project to clear WIP to P&L (expense the cost from the pre-sales project perspective).
4. **Post to production order:** Create a production journal (route card journal or similar) on the production order with a cost category representing the pre-sales engineering labour. This adds the cost to the production order's actual cost.
5. Production order settlement absorbs total cost (including the added pre-sales cost) into finished goods.

**Accounting Impact:**

- Pre-sales project: WIP reversed → P&L (or transfer account)
- Production journal: Dr Production WIP / Cr offset (transfer account or P&L reversal)
- On production order end: Production WIP → Finished goods inventory

**Risks & Limitations:**

- Double-entry risk — must ensure pre-sales WIP reversal and production journal posting are tightly controlled
- Cost category mismatch between project and production cost categories
- If production order uses **standard cost**, additional cost posted via journal may create a **variance** rather than being absorbed into inventory
- Requires custom reporting or process documentation to reconcile project-to-production cost transfers

`Needs Verification:` Whether a route card journal can be posted on a project production order to add costs that are then included in the project estimate/elimination cycle.

**Effort Level:** High

---

#### Option C — Single Project Lifecycle (Pre-Sales Becomes Delivery)

- **Tier:** A or B (Full OOB or Partial OOB + Workaround)
- **Why this may qualify as Tier A:** If the business process allows the same project to span from pre-sales through delivery, then no cost transfer is needed at all. The pre-sales hours and the production order both sit on the same project. When estimate/elimination runs, all project costs (including pre-sales hours) flow into the production order cost / finished goods inventory.
- **Why it may not be feasible:** If business process or governance requires separate pre-sales and delivery projects, this option is not viable.

**Approach:**

1. Create the project at pre-sales stage with a project type that supports both timesheets AND item requirements / project production orders (e.g., Fixed-price, Investment, or `Needs Verification` on which types support both).
2. Engineers post timesheets — hours go to WIP.
3. **Tender won:** On the same project, enter item requirements. Master planning creates planned production orders. Firm into production orders — these become project production orders on the same project.
4. **No cost transfer needed.** All pre-sales hours are already on the project.
5. Run estimate → elimination → production order end. All costs (pre-sales + production) flow to inventory.

**Accounting Impact:**

- Seamless: Hours → WIP → Estimate includes all costs → Elimination to inventory on production completion
- No inter-project transfers or manual journals

**Risks & Limitations:**

- If tender is lost, the project must be written off (estimate/elimination posts WIP to P&L expense)
- Project type must support both timesheets and item requirements/production orders
- Not viable if governance requires separate project IDs for pre-sales vs. delivery
- If multiple tenders share design effort, hours cannot easily be split without adjustments
- **Decision Point:** Does the business accept using the same project ID from pre-sales through production?

**Effort Level:** Low

---

## 4. Architecture Overview

### Cost Flow — Recommended Option A

```
PRE-SALES PHASE                    TENDER WIN                          PRODUCTION & DELIVERY
─────────────────                  ──────────                          ─────────────────────

Engineer posts     →  Project      Project         →  Delivery        Item Req → Planned PO →
timesheet hours       P-1001       Adjustment          Project            Firmed Production Order
                      (WIP)        (transfer            D-2001            (Project Production Order
                                   hours)               (WIP)             linked to D-2001)

                                                        ↓
                                                   Estimate
                                                   (calculates
                                                    total WIP
                                                    incl. transferred
                                                    hours)
                                                        ↓
                                                   Production Order
                                                   End + Elimination
                                                        ↓
                                                   Finished Goods
                                                   Inventory
                                                   (includes pre-
                                                    sales labour)
```

### Cost Flow — Tender Lost

```
Pre-sales Project P-1001 → Estimate → Elimination to P&L expense
(No transfer occurs; WIP is written off)
```

### Component Interaction

1. **Project Management & Accounting** is the master cost collector — both pre-sales and delivery costs live here.
2. **Project adjustment** is the bridge mechanism — it moves cost between the pre-sales and delivery projects within the Projects module.
3. **Estimate/Elimination** is the mechanism that pushes accumulated project costs (including transferred hours) to inventory via the production order link.
4. **Production Control** receives cost through the project production order linkage — when the production order is ended and the project elimination runs, costs flow to finished goods inventory.
5. **Cost groups** ensure the labour cost is categorized correctly throughout the flow for reporting and absorption analysis.

---

## 5. OVERWATCH — Design Challenges

### 1. Is there any OOB capability I may have overlooked that could eliminate a Tier C or D recommendation?

**Option C (Single Project Lifecycle)** could potentially eliminate the need for Option A entirely if the business accepts using a single project from pre-sales through delivery. This is purely an OOB solution with zero customization. The key decision point is whether governance allows the same project ID to span the pre-sales and delivery phases. If yes, Option C is Tier A and is the simplest path.

Additionally, **intercompany/interproject invoicing** or **project-to-project billing** exists in D365 F&O and could be another OOB mechanism for transferring costs between projects. `Needs Verification` on whether interproject invoicing can transfer hour costs between projects within the same legal entity without generating actual invoices — this may be overly complex for this scenario.

### 2. Are there any assumptions in this design that, if wrong, would break the solution?

- **Critical Assumption A2:** If the pre-sales project and the delivery project use different project types with incompatible posting profiles, the project adjustment may post to unexpected accounts. If the pre-sales project is an "Internal" type (which may post to P&L, not WIP), the accounting flow changes significantly.
- **Critical Assumption A3:** If item requirements on the delivery project do not create project production orders (e.g., if the project type or configuration does not support this), the link between project cost and production order is broken.
- **Critical Assumption:** If the inventory costing model is **Standard Cost**, the production order absorbs cost at the standard rate, and the transferred pre-sales labour would appear as a **production variance** rather than being included in inventory value. This would partially defeat the purpose. The solution works cleanly with actual costing models (Moving Average, FIFO, Weighted Average).

### 3. What is the highest-risk component of this design and why?

The **estimate/elimination process** is the highest-risk component because:
- It is the mechanism that pushes project WIP to inventory
- Incorrect configuration of the completion method, cost template, or elimination schedule can result in costs being posted to the wrong accounts or not being posted at all
- The timing of estimate/elimination relative to production order status is critical — if elimination runs before the production order is ended, or if the production order is ended before the estimate includes the transferred hours, the cost flow breaks
- This process is often poorly understood by implementation teams and is sensitive to project group/posting profile configuration

### 4. What gaps remain that require clarification from the business or functional team?

1. **Project type for pre-sales project** (OQ-1) — determines posting behavior and whether Option C is viable
2. **Single vs. separate project governance** (OQ-2) — determines whether Option A or C is the right fit
3. **Inventory costing model** (OQ-4) — Standard Cost would create variances rather than absorbing the transferred cost into inventory value; this fundamentally changes the accounting outcome
4. **Cost category alignment** (OQ-5) — misalignment between project and production cost categories would require mapping configuration
5. **Estimate/elimination maturity** (OQ-6) — if this process is not already operational, it represents significant additional implementation effort beyond the scope of this requirement
6. **Tender lost frequency** (new) — if tenders are frequently lost, the volume of write-offs and the process for identifying "lost" pre-sales projects needs to be defined

---

## 6. Recommended Option & Justification

**Recommended: Option A — Project Adjustment / Transfer (Tier B)**

| Criterion | Option A (Recommended) | Option B | Option C |
|-----------|----------------------|----------|----------|
| Governance | Full project audit trail | Requires cross-module reconciliation | Simplest but requires same project ID for pre-sales and delivery |
| Traceability | Adjustment references original transactions | Manual journal with external reference | Natural — single project |
| Minimal customization | OOB project adjustment functionality | Requires custom mapping logic (Tier C) | Fully OOB (Tier A) |
| Costing correctness | Costs flow through project estimate/elimination to inventory | Risks double counting or omission | Seamless |
| Flexibility | Handles partial transfers, multi-tender splits | Complex for splits | Limited for cross-project scenarios |

**Why Option A over Option C:** While Option C is technically simpler (Tier A), it requires the business to accept using the same project ID from pre-sales through delivery. This is often not acceptable in organizations with separate governance for pre-sales and delivery phases, different project managers, different approval workflows, or different financial reporting requirements. Option A provides the flexibility to maintain separate projects while still achieving the cost transfer goal using OOB functionality.

**If the business confirms they can use a single project (OQ-2 resolved as "same project"), Option C becomes the preferred recommendation and should be used instead.**

---

## 7. Implementation Checklist (Preferred Option A)

### Phase 1: Configuration & Validation

- [ ] Confirm pre-sales project type and group configuration (OQ-1)
- [ ] Confirm delivery project type supports item requirements and project production orders
- [ ] Validate project posting profiles — hour transactions post to WIP for both project types
- [ ] Align cost categories between pre-sales timesheets and delivery project
- [ ] Configure project adjustment parameters in Project management and accounting parameters
- [ ] Configure approval workflow for project adjustments (if required)
- [ ] Confirm inventory costing model for finished goods items (OQ-4)

### Phase 2: Process Documentation

- [ ] Document end-to-end process: pre-sales timesheet → tender decision → adjustment → estimate → elimination → production
- [ ] Document tender-lost write-off process
- [ ] Document partial transfer procedure
- [ ] Create period-end checklist including timing of adjustments, estimates, and eliminations
- [ ] Define reconciliation report: pre-sales hours logged vs. hours transferred vs. hours written off

### Phase 3: Sandbox Testing

- [ ] Execute TC-001 through TC-007 in sandbox environment
- [ ] Verify WIP account balances at each step
- [ ] Verify finished goods inventory value includes transferred labour cost
- [ ] Verify P&L impact for tender-lost scenario
- [ ] Verify audit trail from original timesheet to inventory value
- [ ] Test estimate/elimination process with project production order

### Phase 4: User Training & Go-Live

- [ ] Train project accountants on adjustment process
- [ ] Train production planners on project production order linkage
- [ ] Train finance team on period-end estimate/elimination sequence
- [ ] Go-live with parallel run (manual reconciliation for first 2 periods)

---

## 8. Test Plan

### TC-001: Happy Path — Tender Won, Full Cost Transfer

- **Steps:** Post 100 hours at $50/hr to P-1001 → Transfer all hours to D-2001 → Create production order → Complete production → Run estimate/elimination
- **Expected:** Finished goods inventory includes $5,000 of pre-sales labour. P-1001 WIP = $0. D-2001 shows transferred transactions.
- **Validation:** Project transaction inquiry on D-2001 shows hour transactions with adjustment reference. Inventory value report shows labour component in finished goods cost.

### TC-002: Tender Lost — Write-Off to P&L

- **Steps:** Post 80 hours at $50/hr to P-1001 → Tender lost → Run elimination on P-1001
- **Expected:** WIP reversed. P&L expense account shows $4,000 charge. No impact on any production order or inventory.
- **Validation:** Trial balance shows WIP account for P-1001 = $0. P&L expense account increased by $4,000.

### TC-003: Partial Transfer — Scope Change

- **Steps:** Post 100 hours to P-1001 → Tender won but only 60 hours relevant → Transfer 60 hours to D-2001 → Write off remaining 40 hours
- **Expected:** D-2001 receives $3,000. P-1001 remaining $2,000 written off to P&L. Finished goods reflect $3,000 of pre-sales labour.
- **Validation:** Both project WIP balances reconcile to expected amounts.

### TC-004: Multiple Tenders Share Design Effort

- **Steps:** Post 100 hours to P-1001 → Tenders T1 and T2 both won → Transfer 60 hours to D-2001 (T1) and 40 hours to D-2002 (T2)
- **Expected:** D-2001 receives $3,000, D-2002 receives $2,000. P-1001 WIP = $0. Each production order reflects its share.
- **Validation:** Sum of transferred amounts equals original P-1001 total. No residual WIP on P-1001.

### TC-005: Late Timesheet Entry After Project Creation

- **Steps:** Create D-2001 and production order → Engineer posts additional 10 hours directly to D-2001 → Complete production
- **Expected:** The 10 hours are included in D-2001's cost pool directly (no transfer needed). Total project cost includes both transferred and direct hours.
- **Validation:** Project cost totals reconcile to transferred + direct hours.

### TC-006: Production Order Cancelled After Cost Transfer

- **Steps:** Transfer hours to D-2001 → Create production order → Cancel production order before completion
- **Expected:** Transferred costs remain on D-2001 as WIP. Project must be handled via estimate/elimination — either transferred to a new production order or written off.
- **Validation:** `Needs Verification` — exact behavior when a project production order is cancelled and whether WIP reversal is automatic.

### TC-007: Reversal of Incorrect Transfer

- **Steps:** Transfer hours to wrong project D-2002 → Discover error → Create reverse adjustment
- **Expected:** Adjustment reverses the original transfer on D-2002 and re-posts to correct project D-2001. Full audit trail of original transfer, reversal, and correction.
- **Validation:** Three adjustment transactions visible in project transaction inquiry. Net cost on D-2002 = $0. Correct cost on D-2001.

---

## 9. Risks & Mitigations

| # | Risk | Impact | Likelihood | Mitigation |
|---|------|--------|------------|------------|
| R1 | Standard cost model blocks absorption of transferred labour | Pre-sales cost appears as production variance, not in inventory value | Medium | Confirm costing model (OQ-4); if Standard Cost, evaluate whether the variance is acceptable or update standard cost to include engineering component |
| R2 | Project adjustment posted to wrong project or wrong amount | Incorrect cost allocation, potential double counting | Low | Implement approval workflow for project adjustments; reconciliation report |
| R3 | Estimate/elimination timing incorrect | Costs excluded from inventory or posted to wrong period | Medium | Document required sequence and build into period-end checklist |
| R4 | Pre-sales project type incompatible with WIP posting | Hours post to P&L instead of WIP | Low | Validate project type and group configuration before go-live (OQ-1) |
| R5 | Orphaned pre-sales projects (neither won nor lost) | WIP accumulates indefinitely on balance sheet | Medium | Implement periodic review; define aging policy for write-off |
| R6 | Cost category mismatch between projects and production | Costs do not map correctly to production cost groups | Low | Align cost categories used in pre-sales timesheets with production cost groups (OQ-5) |

---

## 10. "Insufficient Data to Verify" List

The following items could not be confirmed from available sources and require sandbox validation:

1. Whether **project adjustment transactions** correctly transfer WIP postings when moving hours between different project types (e.g., Internal → Fixed-price) — the posting profile behavior in cross-type adjustments needs to be verified.
2. Whether the **estimate/elimination** process on a delivery project automatically includes costs from project production orders in the WIP calculation when the production order is still in progress.
3. The exact **project type** required to support both timesheets (hour journals) AND item requirements with project production orders simultaneously.
4. Whether **interproject invoicing** within the same legal entity is a viable alternative mechanism for cost transfer.
5. The behavior when a **project production order is cancelled** — whether project WIP associated with the production order is automatically reversed.
6. Whether the route card journal can post additional costs to a **project production order** that are then included in the project estimate/elimination cycle (relevant to Option B).

---

## 11. Recommended Next Steps

1. **Resolve Open Questions OQ-1 through OQ-8** with the business and functional team — particularly OQ-2 (single vs. separate project) and OQ-4 (inventory costing model), as these determine which option is optimal.
2. **Sandbox proof-of-concept** for the recommended option (A or C based on OQ-2 resolution) — execute TC-001 and TC-002 to validate the end-to-end cost flow.
3. **Validate "Insufficient Data to Verify" items** in the sandbox environment.
4. **Business sign-off** on the recommended approach before proceeding to configuration and testing.
5. **Complete Phase 1 configuration** per the implementation checklist.
