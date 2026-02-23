# Documentation Index (Source of Truth)

## How to use this index
1. Start here for every new requirement.
2. Read the “When to use” section to choose only relevant files.
3. If two files conflict, use this precedence:
   1) `AGENTS.md`
   2) `docs/policies/*`
   3) domain docs (`docs/process/*`, `docs/config/*`)
   4) examples (`docs/examples/*`)
4. If required detail is missing, return exactly: `Insufficient data to verify.`

---

## Policies / Governance

### `docs/policies/solution-design-governance.md`
- **Contains:** Non-hallucination rules, OOTB-first policy, risk requirements
- **When to use:** Every request (mandatory)

### `docs/policies/company-constraints.md`
- **Contains:** Legal entities, sites/warehouses, WMS, integration boundaries, compliance
- **When to use:** Any design impacted by org structure or constraints

---

## Process Documentation

### `docs/process/production-order-lifecycle.md`
- **Contains:** End-to-end production lifecycle and process decisions
- **When to use:** Production flow, status progression, execution design

### `docs/process/master-planning-touchpoints.md`
- **Contains:** Planning interactions and dependencies
- **When to use:** Any requirement touching planning/supply proposals

---

## Configuration References

### `docs/config/production-parameters.md`
- **Contains:** Approved parameter decisions and rationale
- **When to use:** Setup/checklist creation

### `docs/config/costing-and-journals.md`
- **Contains:** Costing model assumptions and journal handling rules
- **When to use:** Cost, variance, posting, and control scenarios

---

## Data / Integration

### `docs/data/entities-and-mappings.md`
- **Contains:** Verified entities, field mappings, ownership
- **When to use:** Reporting, migration, interfaces, integration design

### `docs/data/integration-contracts.md`
- **Contains:** Interface definitions, payload expectations, error handling
- **When to use:** External system touchpoints

---

## Examples (Reference only)

### `docs/examples/approved-sd-example-01.md`
- **Contains:** Approved sample output structure and quality bar
- **When to use:** Formatting and completeness guidance (not source of functional truth)

---

## Open Questions Log

### `docs/open-questions.md`
- **Contains:** Unresolved decisions blocking final design
- **When to use:** If requirement cannot be finalized without clarification
