# AGENTS.md — D365 F&O Production Solution Design Operating Rules

## Purpose
This repository is used to produce functional Solution Designs for Microsoft Dynamics 365 Finance & Operations (Production module) with strict governance and repeatable output quality.

---

## Scope & Precedence
- This `AGENTS.md` applies to the entire repository unless a deeper `AGENTS.md` exists in a subfolder.
- If multiple instruction sources exist, apply this order:
  1. Direct user prompt for the current request
  2. This `AGENTS.md`
  3. `docs/policies/*`
  4. Domain docs (`docs/process/*`, `docs/config/*`, `docs/data/*`)
  5. Examples (`docs/examples/*`)

---

## Mandatory Operating Rules
1. **Zero hallucination**
   - Do not invent D365 F&O behavior, navigation, field names, parameters, defaults, or process steps.
2. **Missing data handling**
   - If any required detail cannot be verified from repository sources or the current user prompt, output exactly:
   - `Insufficient data to verify.`
3. **No assumptions**
   - If a decision depends on unknowns, ask only the minimum clarifying questions needed.
4. **OOTB-first**
   - Prefer out-of-the-box/configuration first.
   - Propose extension/customization only when OOTB cannot meet requirement and the gap is explicitly identified.
5. **Risk transparency**
   - Explicitly call out risks: performance, data integrity, compliance, security, maintainability, regression.

---

## Required Knowledge-Loading Workflow
Before drafting any solution:
1. Read `docs/INDEX.md`.
2. Read all relevant files referenced by `docs/INDEX.md` for the requirement at hand.
3. Always include `docs/policies/*` as mandatory governance.
4. If docs conflict, follow precedence above.
5. Keep evidence anchored to repository content and user-provided facts only.

---

## Required Output Structure (Strict)
For requirement-response tasks, output **exactly** in this section order:

1. **Requirement Restatement**
   - One paragraph describing business success criteria.
2. **Fit Assessment (OOTB vs Configuration vs Custom)**
   - Option A: OOTB/Configuration approach (if feasible)
   - Option B: Extension/custom approach (only if required)
   - For each: why it works/fails + tradeoffs
3. **Recommended Solution Design (Chosen Option)**
   - Process flow (step-by-step)
   - Data entities involved (only if verifiable)
   - Controls and validations needed
4. **Step-by-Step Setup Guide (Mandatory)**
   - Setup area/configuration item
   - Purpose
   - Exact value/decision needed (or `Insufficient data to verify.`)
   - Dependencies/prerequisites
   - Validation step + expected result
5. **Test Scenarios (Minimum)**
   - Happy path
   - Negative/edge case
   - Volume/performance consideration (if applicable)
6. **Risks + Mitigations**
   - Risk
   - Impact
   - Mitigation
7. **Clarifications Required (Only if resolution is impossible)**

---

## Response Quality Gate (Self-check before finalizing)
- Did I include any unverified behavior/name/path/default?
  - If yes, replace with `Insufficient data to verify.`
- Did I provide the complete setup checklist?
- Did every setup item include validation step + expected result?
- Did I avoid assumptions and keep customizations justified?

---

## Repository Layout Convention
- `docs/INDEX.md` — entry point and document catalog
- `docs/policies/` — governance and constraints
- `docs/process/` — business and operational flows
- `docs/config/` — verified setup decisions
- `docs/data/` — entities, mappings, integration contracts
- `docs/examples/` — approved format examples (reference only)

---

## Standard Prompt Compatibility Rule
If the user provides no requirement statement, respond with exactly:

`Insufficient data to verify.`
