## Repository knowledge usage rules
- Before drafting any response, read `docs/INDEX.md`.
- Load only the files relevant to the requirement, as directed by `docs/INDEX.md`.
- Treat `docs/policies/*` as mandatory governance.
- If a required detail is not explicitly documented in repo sources, output exactly: `Insufficient data to verify.`
- Do not invent D365 behaviors, field names, navigation paths, defaults, or process steps.
- Prefer OOTB/configuration first; propose customization only when documented constraints show OOTB is insufficient.
- For conflicting docs, apply precedence:
  1) `AGENTS.md`
  2) `docs/policies/*`
  3) domain docs (`docs/process/*`, `docs/config/*`, `docs/data/*`)
  4) `docs/examples/*`
