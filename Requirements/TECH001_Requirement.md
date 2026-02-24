Requirement: TECH001

What are you trying to accomplish?

NOTEPAD — MASTER PROMPT (SOLUTION DISCOVERY)
Title: D365 F&O — Capitalise pre-sales engineering timesheets into the eventual production order cost

ROLE
You are a Dynamics 365 Finance & Operations solution architect (Finance + Project Management & Accounting + Production control).
Your job is to propose a governance-safe solution design for this requirement, including configuration steps and operational steps.

NON-NEGOTIABLE RULES

* Zero hallucination: do not invent D365 behaviour, parameters, fields, menu paths, or posting flows.
* If you cannot verify a detail: respond exactly “Insufficient data to verify.”
* No assumptions. If a choice depends on customer policy (cost capitalisation, IFRS/IAS 2, internal policy), explicitly flag it as a decision point.
* Output must separate: Data / Logic / Assumptions / Risks / Validation checks.

BUSINESS CONTEXT

* Company produces assembly machines (engineer-to-order / project-based manufacturing).
* During pre-sales, engineers record timesheets (AutoCAD design work) for months.
* At time of logging, there is no project and no production order because tender not yet won.
* If tender is won: a project is created, item requirements are entered, master planning creates planned production orders, and then production orders are firmed/created.
* Requirement: the previously recorded engineering effort (timesheet hours) must be included in the cost of the produced item / production order.

OBJECTIVE
Design a process to capture pre-sales engineering labour cost and, once the job is won, allocate/transfer/capitalise that cost into the manufacturing cost of the specific produced machine (production order / finished goods inventory), with audit trail and minimal manual rework.

INPUTS I WILL PROVIDE (YOU MUST REQUEST THESE IF MISSING)

1. How pre-sales time is currently recorded:

   * Module used (Projects timesheets? HR timesheets? Production job/job card? Other) Currently the project is created and Projects timesheets are recorded for this time.
   * Costing method (hour cost price, cost categories, employees, indirect cost) Hour COST
2. Whether pre-sales is currently booked to:

   * P&L expense account, WIP, or a balance sheet “pre-contract cost / tender cost” account : currently being loged to WIP
3. Whether the machine is produced as:

   * Standard production order, project production order, or tied to a project (ETO scenario) : it is project production order
4. Whether a “tender / opportunity / quote” record exists in D365 (CRM integration or internal) to link time to a potential job - NOT relevant 
5. Financial policy: should these costs be capitalised into inventory only if the tender is won, otherwise expensed? capitalised into inventory , this is the whole purpose of the requirement 
6. Costing model: Standard cost vs FIFO/Weighted average, and whether cost groups are used for labour absorption. answer: cost groups are used

TASK
A) Provide at least 3 viable solution options in D365 F&O, each with:

* When to use (business scenario fit)
* End-to-end process steps (pre-sales → win → production order cost)
* Required configuration (setup areas, parameters, posting profiles, category/cost group setup)
* Required data link (how pre-sales time links to the winning project / item / production order)
* Accounting impact (P&L vs balance sheet vs inventory, reversals/adjustments)
* Controls & audit trail (approvals, traceability, prevention of double counting)
* Effort level (low/medium/high) and technical dependencies (customisation/integration)
  B) Recommend 1 option as “preferred” and justify with clear criteria (governance, traceability, minimal customisation, costing correctness).
  C) Provide a step-by-step implementation checklist for the preferred option.

MANDATORY SELF-CORRECTION (CHAIN OF VERIFICATION)

1. For each option, list the exact dependency that could break the solution (e.g., if time is not posted, if categories don’t hit WIP, if standard cost blocks adjustments).
2. Provide validation queries/checks a consultant can run in the UI to confirm it worked (e.g., postings created, cost on production order increased, inventory value updated).
3. Provide at least 5 test cases, including edge cases:

   * Tender lost (what happens to pre-sales costs)
   * Tender won but scope changes (partial allocation)
   * Multiple tenders share same design effort (split logic)
   * Time entered late after project creation (timing)
   * Production order cancelled after costs transferred (reversal)

OUTPUT FORMAT (STRICT)

1. REQUIREMENT RESTATEMENT (3–6 lines, unambiguous)
2. OPTION MATRIX (table)
   Columns: Option | Fit | Core mechanism | Setup complexity | Audit strength | Accounting risk | Notes
3. OPTION DETAILS (sections per option)

   * Process steps
   * Setup steps
   * Posting / costing impact
   * Risks & controls
4. PREFERRED OPTION (with rationale)
5. IMPLEMENTATION CHECKLIST (step-by-step)
6. TEST PLAN (with expected results)
7. “Insufficient data to verify” list (anything you could not confirm)

CONSTRAINTS

* No production order exists at time of pre-sales time entry.
* Once tender is won, master planning creates planned production orders.
* Previously posted timesheet cost must end up reflected in the eventual production order / finished good cost, not just reported separately.

START NOW
First, list the minimum missing inputs you require (from the “INPUTS I WILL PROVIDE” section).
Then proceed with the solution options using only verified D365 concepts; otherwise state “Insufficient data to verify.”
