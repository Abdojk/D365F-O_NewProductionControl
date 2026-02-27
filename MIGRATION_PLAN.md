# Migration Plan: Claude Code Repo → Claude.ai Project

## Overview

Move all resources from this repository into a **Claude.ai Project** so that every new conversation automatically inherits governance rules, reference knowledge, and requirement templates without needing a local repo or Claude Code CLI.

---

## Claude.ai Project has two slots for content

| Slot | Purpose | Limit | Format |
|------|---------|-------|--------|
| **Custom Instructions** | System prompt applied to every conversation | ~7,500 words | Plain text / markdown |
| **Knowledge Files** | Uploaded documents Claude can search and cite | ~200k tokens across all files combined | PDF, DOCX, TXT, MD, CSV, etc. |

---

## Resource Inventory & Target Mapping

| # | Current File | Target Slot | Action |
|---|-------------|-------------|--------|
| 1 | `Agents.md` (root) | **Custom Instructions** | Consolidate into system prompt |
| 2 | `README.md` | **Custom Instructions** | Merge (it's a condensed subset of Agents.md) |
| 3 | `Docs/Agents.md` | **Skip** | Duplicate of root Agents.md — not needed separately |
| 4 | `Docs/INDEX.md` | **Knowledge File** | Upload as-is (navigation guide for knowledge files) |
| 5 | `Docs/Source/D365__FO__Production__Control__Master__Reference.pdf` | **Knowledge File** | Upload as-is |
| 6 | `Docs/Source/D365__FO__Production__Control__Master__Reference.docx` | **Skip** | Duplicate of #5 in different format — PDF is preferred for Claude |
| 7 | `Requirements/Requirements.md` | **Knowledge File** | Upload as-is (master prompt template) |
| 8 | `Requirements/TECH001_Requirement.md` | **Knowledge File** | Upload as-is (specific requirement) |
| 9 | `Docs/Source/a` | **Skip** | Empty placeholder file — no value |

---

## Migration Sequence (5 Steps)

### Step 1 — Create the Claude.ai Project

1. Go to [claude.ai](https://claude.ai)
2. In the left sidebar, click **Projects**
3. Click **Create Project**
4. Name it: `D365 F&O Production Control — Solution Design`
5. Optionally add a description: *"Governance-driven solution designs for D365 Finance & Operations Production module"*

---

### Step 2 — Set Custom Instructions (System Prompt)

This is the **most critical step** — it controls the behavior of every conversation.

**Source files to consolidate:** `Agents.md` (root) + `README.md`

Copy the following merged content into the **Custom Instructions** box in the project settings. The content below consolidates both files, removes repo-specific references (like file paths), and adapts the instructions for a Claude.ai Project context:

```
You are a senior Microsoft Dynamics 365 Finance & Operations (Production module) Solution Architect.

PURPOSE
Produce functional Solution Designs with strict governance and repeatable output quality.

MANDATORY OPERATING RULES
1. Zero hallucination
   - Do not invent D365 F&O behavior, navigation, field names, parameters, defaults, or process steps.
2. Missing data handling
   - If any required detail cannot be verified from uploaded knowledge files or the current user prompt, output exactly:
     "Insufficient data to verify."
3. No assumptions
   - If a decision depends on unknowns, ask only the minimum clarifying questions needed.
4. OOTB-first
   - Prefer out-of-the-box / configuration first.
   - Propose extension/customization only when OOTB cannot meet the requirement and the gap is explicitly identified.
5. Risk transparency
   - Explicitly call out risks: performance, data integrity, compliance, security, maintainability, regression.

KNOWLEDGE USAGE RULES
Before drafting any solution:
1. Search the uploaded knowledge files for relevant content (INDEX, policies, process docs, config refs, requirements).
2. If knowledge files conflict, apply this precedence:
   1) These custom instructions
   2) Policies / governance knowledge files
   3) Domain docs (process, config, data)
   4) Examples
3. Keep evidence anchored to knowledge file content and user-provided facts only.

REQUIRED OUTPUT STRUCTURE (STRICT)
For requirement-response tasks, output exactly in this section order:

1. Requirement Restatement
   - One paragraph describing business success criteria.
2. Fit Assessment (OOTB vs Configuration vs Custom)
   - Option A: OOTB/Configuration approach (if feasible)
   - Option B: Extension/custom approach (only if required)
   - For each: why it works/fails + tradeoffs
3. Recommended Solution Design (Chosen Option)
   - Process flow (step-by-step)
   - Data entities involved (only if verifiable)
   - Controls and validations needed
4. Step-by-Step Setup Guide (Mandatory)
   - Setup area / configuration item
   - Purpose
   - Exact value / decision needed (or "Insufficient data to verify.")
   - Dependencies / prerequisites
   - Validation step + expected result
5. Test Scenarios (Minimum)
   - Happy path
   - Negative / edge case
   - Volume / performance consideration (if applicable)
6. Risks + Mitigations
   - Risk → Impact → Mitigation
7. Clarifications Required (Only if resolution is impossible)

RESPONSE QUALITY GATE (Self-check before finalizing)
- Did I include any unverified behavior/name/path/default? If yes → replace with "Insufficient data to verify."
- Did I provide the complete setup checklist?
- Did every setup item include a validation step + expected result?
- Did I avoid assumptions and keep customizations justified?
- Did I separate Data / Logic / Assumptions / Risks?

DEFAULT RULE
If no requirement statement is provided, respond with exactly:
"Insufficient data to verify."
```

**Why this goes first:** Custom Instructions shape every conversation. All subsequent knowledge files are interpreted through this lens.

---

### Step 3 — Upload the Master Reference Document

1. In the project, click **Add knowledge** (or the paperclip/upload icon in project settings)
2. Upload: `Docs/Source/D365__FO__Production__Control__Master__Reference.pdf`

**Why this goes second:** This is the foundational domain knowledge. All requirements and prompts depend on the D365 Production Control concepts in this document. Claude will use this to verify OOTB behavior before answering.

---

### Step 4 — Upload the Documentation Index

1. Upload: `Docs/INDEX.md`

**Why this goes third:** The index acts as a navigation guide. It tells Claude which categories of knowledge exist and the precedence rules for conflicts. Even though some referenced docs don't exist yet (see Step 5 notes), the index establishes the framework for when you add them later.

---

### Step 5 — Upload Requirement Files

Upload these two files in order:

1. `Requirements/Requirements.md` — the **master prompt template** (overhead absorption scenario)
2. `Requirements/TECH001_Requirement.md` — the **specific requirement** (pre-sales cost capitalization)

**Why these go last:** Requirements are the "work items" that consume governance rules + reference knowledge. They should be loaded after the foundation is in place.

---

## Post-Migration: Filling the Gaps

The `INDEX.md` references **10 documentation modules that don't exist yet**. Once migrated, you can progressively build and upload these as knowledge files:

| Priority | File to Create | Purpose |
|----------|---------------|---------|
| High | `solution-design-governance.md` | Expand on the governance rules beyond custom instructions |
| High | `company-constraints.md` | Legal entities, sites, warehouses, WMS, compliance boundaries |
| High | `production-order-lifecycle.md` | End-to-end production flow and status progression |
| Medium | `production-parameters.md` | Verified parameter decisions and rationale |
| Medium | `costing-and-journals.md` | Costing model rules, journal handling, variance posting |
| Medium | `master-planning-touchpoints.md` | Planning interactions and supply proposal dependencies |
| Low | `entities-and-mappings.md` | Data entities, field mappings, ownership |
| Low | `integration-contracts.md` | External interface definitions and error handling |
| Low | `approved-sd-example-01.md` | Gold-standard example output for formatting guidance |
| Low | `open-questions.md` | Unresolved decisions log |

---

## What NOT to Migrate

| File | Reason |
|------|--------|
| `Docs/Agents.md` | Duplicate of root `Agents.md` — already covered by custom instructions |
| `README.md` | Already merged into custom instructions |
| `Docs/Source/a` | Empty placeholder — no content |
| `Docs/Source/...docx` | Duplicate of the PDF — PDF works better as a knowledge file |
| `.git/` | Not applicable to Claude.ai Projects |

---

## Verification Checklist

After completing all 5 steps, open a new conversation in the project and test:

- [ ] Send an empty message → should get `"Insufficient data to verify."`
- [ ] Ask about a production parameter → should reference the master PDF
- [ ] Paste TECH001 requirement and ask for a solution → should produce all 7 output sections in order
- [ ] Check that OOTB-first rule is respected (no unnecessary customizations proposed)
- [ ] Verify risks section appears in every solution response

---

## Quick Reference: Final Project Structure

```
Claude.ai Project: "D365 F&O Production Control — Solution Design"
│
├── Custom Instructions (system prompt)
│   └── Consolidated from: Agents.md + README.md
│       → Role, rules, output structure, quality gate
│
└── Knowledge Files (4 files)
    ├── D365__FO__Production__Control__Master__Reference.pdf  (domain reference)
    ├── INDEX.md                                               (navigation & precedence)
    ├── Requirements.md                                        (master prompt template)
    └── TECH001_Requirement.md                                 (specific requirement)
```

Total: **1 custom instruction + 4 knowledge files**
