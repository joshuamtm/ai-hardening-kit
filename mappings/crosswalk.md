# Framework Crosswalk

This document maps every defense, checklist item, and test workflow in this kit to three external frameworks:

- **OWASP Top 10 for LLM Applications 2025** ([genai.owasp.org](https://genai.owasp.org/)) — the authoritative taxonomy of LLM application risks. IDs: `LLM01` through `LLM10`.
- **OWASP Top 10 for MCP 2025** ([genai.owasp.org](https://genai.owasp.org/)) — connector-specific risks. IDs: `MCP01` through `MCP10`.
- **MITRE ATLAS** ([atlas.mitre.org](https://atlas.mitre.org/)) — adversarial tactics and techniques against AI systems. IDs: `AML.T####` (technique) or `AML.T####.###` (sub-technique).
- **NIST AI RMF 1.0** ([nist.gov/itl/ai-risk-management-framework](https://www.nist.gov/itl/ai-risk-management-framework)) — governance framework. IDs: `GOVERN N.N`, `MAP N.N`, `MEASURE N.N`, `MANAGE N.N`.

Use this crosswalk when:

- Completing a funder security questionnaire that asks "do you address OWASP LLM Top 10?"
- Mapping this kit into an existing NIST AI RMF program.
- Responding to a cyber-insurance questionnaire about AI controls.
- Tracing coverage gaps — the holes in this table are the holes in your program.

The machine-readable version of this crosswalk lives at [`controls.yaml`](controls.yaml) and is used by automation.

---

## How to read this table

Each row represents a control or piece of guidance in the kit. Framework columns contain one or more IDs that the control addresses. Empty cells mean the kit does not address that framework for that control — either because it's out of scope, or because it's a known gap to close in a future sprint.

Framework version pins:

| Framework | Version | Last verified |
|---|---|---|
| OWASP LLM Top 10 | 2025 | 2026-04-21 |
| OWASP MCP Top 10 | 2025 | 2026-04-21 |
| MITRE ATLAS | v5.4 (Feb 2026) | 2026-04-21 |
| NIST AI RMF | 1.0 | 2026-04-21 |

---

## Defenses

### `defenses/01-threat-model.md`

| Section | OWASP LLM | OWASP MCP | MITRE ATLAS | NIST AI RMF |
|---|---|---|---|---|
| Direct vs. indirect prompt injection | LLM01 | MCP01 | AML.T0051.000, AML.T0051.001 | MAP 2.1, MAP 5.1 |
| Threat actors (opportunistic, targeted, insider, supply-chain) | LLM01, LLM03 | MCP08 | AML.T0018 (Supply Chain) | MAP 1.1, MAP 3.1 |
| Canonical attack (hidden instructions in email) | LLM01, LLM02 | MCP01, MCP05 | AML.T0051.001 | MAP 5.1 |
| What a "win" looks like (reduce success rate, blast radius, data-out value, detect/recover) | LLM01, LLM02, LLM06 | — | AML.T0024, AML.T0025 | MAP 5.1, MEASURE 2.7, MANAGE 1.3 |

### `defenses/02-system-prompt-hardening.md`

| Section | OWASP LLM | OWASP MCP | MITRE ATLAS | NIST AI RMF |
|---|---|---|---|---|
| Anti-pattern: secrets in system prompt | LLM02, LLM07 | MCP10 | AML.T0056, AML.T0055 | MANAGE 2.3 |
| Anti-pattern: "confidential" / "never reveal" framing | LLM07 | — | AML.T0056 | MAP 5.1 |
| Anti-pattern: vague role anchoring | LLM01 | — | AML.T0054 | MAP 5.1 |
| Anti-pattern: no adversarial handling | LLM01 | — | AML.T0051 | MAP 5.1, MEASURE 2.7 |
| Anti-pattern: tool constraints only in prompt | LLM06 | MCP02 | AML.T0053 | MANAGE 2.4 |
| Pattern: role anchoring with refusal | LLM01, LLM09 | — | AML.T0054 | MAP 5.1 |
| Pattern: explicit data/instruction framing | LLM01 | MCP05 | AML.T0051.001 | MAP 5.1 |
| Pattern: naming attack classes | LLM01 | — | AML.T0051 | MEASURE 2.7 |
| Pattern: scope-tight tool guidance | LLM06 | MCP02 | AML.T0053 | MANAGE 2.4 |
| Pattern: user-confirmation gating | LLM06, LLM02 | MCP06 | AML.T0024, AML.T0025 | MAP 3.5, MANAGE 2.4 |

### `defenses/03-data-instruction-boundary.md`

| Section | OWASP LLM | OWASP MCP | MITRE ATLAS | NIST AI RMF |
|---|---|---|---|---|
| Layer 1: prompt-level delimiters | LLM01 | MCP05 | AML.T0051.001 | MAP 5.1 |
| Layer 1: double-confirmed intent | LLM06 | — | AML.T0024 | MAP 3.5, MANAGE 2.4 |
| Layer 1: instruction provenance | LLM01, LLM05 | MCP05 | AML.T0051.001 | MAP 5.1 |
| Layer 2: pre-processing / keyword filtering | LLM01, LLM05 | — | AML.T0051 | MANAGE 2.4 |
| Layer 2: content quarantine (per-item calls) | LLM01 | — | AML.T0051.001 | MAP 5.1, MANAGE 2.4 |
| Layer 2: structured output contracts | LLM05, LLM02 | — | AML.T0025 | MANAGE 2.4 |
| Layer 3: deny-by-default tool wrappers | LLM06, LLM02 | MCP02 | AML.T0053, AML.T0025 | MANAGE 2.4 |
| Layer 3: network egress allowlisting | LLM02 | MCP06 | AML.T0025 | MANAGE 2.4, MEASURE 2.7 |
| Layer 3: capability-scoped connectors | LLM06 | MCP02 | AML.T0053 | MAP 3.5, MANAGE 2.4 |
| Layer 3: human-in-the-loop | LLM06 | — | AML.T0024 | MAP 3.5, MANAGE 2.4 |

### `defenses/04-connector-permissions.md`

| Section | OWASP LLM | OWASP MCP | MITRE ATLAS | NIST AI RMF |
|---|---|---|---|---|
| Three scope dimensions (read/write/identity) | LLM06, LLM02 | MCP02, MCP04 | AML.T0025, AML.T0055 | MAP 3.5, MANAGE 2.4 |
| Gmail: minimize read surface (label-based) | LLM02, LLM06 | MCP02 | AML.T0025 | MANAGE 2.4 |
| Gmail: minimize write surface (drafts-only) | LLM06, LLM02 | — | AML.T0024, AML.T0025 | MAP 3.5, MANAGE 2.4 |
| Gmail: minimize identity (revocable OAuth) | LLM03 | MCP04, MCP10 | AML.T0055 | MANAGE 2.4 |
| Drive: minimize read surface | LLM02, LLM06 | — | AML.T0025 | MANAGE 2.4 |
| Drive: no write / AI-outputs folder only | LLM06 | — | AML.T0024 | MAP 3.5, MANAGE 2.4 |
| Drive: third-party-shared file risk | LLM01, LLM03 | MCP08 | AML.T0051.001, AML.T0018 | MAP 1.1 |
| Slack: scoped channels, post-as-bot, draft-only | LLM06, LLM02 | — | AML.T0024, AML.T0025 | MANAGE 2.4 |
| MCP: tool shadowing / rug pulls | LLM03 | MCP03, MCP07 | AML.T0053, AML.T0031 | MEASURE 2.7, MANAGE 2.4 |
| MCP: shadow servers | LLM03 | MCP09 | AML.T0018 | MEASURE 2.7 |
| MCP: excessive scope | LLM06 | MCP02 | AML.T0053 | MANAGE 2.4 |
| MCP: indirect injection via tool output | LLM01 | MCP05 | AML.T0051.001 | MAP 5.1 |
| MCP: vet + baseline + diff + sandbox | LLM03 | MCP03, MCP07, MCP08 | AML.T0053, AML.T0031 | MEASURE 2.7, MANAGE 2.4 |
| Credential hygiene (no secrets in prompts / git) | LLM07, LLM02 | MCP10 | AML.T0055, AML.T0056 | MANAGE 2.3 |
| Short-lived OAuth over long-lived API keys | LLM03 | MCP04, MCP10 | AML.T0055 | MANAGE 2.4 |
| Token freshness audit | LLM03 | MCP04, MCP10 | AML.T0055 | MANAGE 4.1 |
| Token usage logging | LLM03 | MCP10 | AML.T0055, AML.T0025 | MEASURE 2.7, MANAGE 4.1 |

### `defenses/05-egress-controls.md`

| Section | OWASP LLM | OWASP MCP | MITRE ATLAS | NIST AI RMF |
|---|---|---|---|---|
| Path 1: direct HTTP — no arbitrary HTTP / allowlist | LLM02, LLM06 | MCP06 | AML.T0025 | MANAGE 2.4 |
| Path 1: tool-level URL validation | LLM02 | MCP06 | AML.T0025 | MANAGE 2.4 |
| Path 1: Oasis Security case study | LLM02 | — | AML.T0025 | MEASURE 2.7 |
| Path 2: email send — drafts-only | LLM06, LLM02 | — | AML.T0024, AML.T0025 | MAP 3.5, MANAGE 2.4 |
| Path 2: recipient-domain allowlist | LLM02 | — | AML.T0025 | MANAGE 2.4 |
| Path 2: anomaly detection on send | LLM02 | — | AML.T0025 | MEASURE 2.7 |
| Path 3: shared documents — no public shares | LLM02 | — | AML.T0025 | MANAGE 2.4 |
| Path 3: monitor public-share creation | LLM02 | — | AML.T0025 | MEASURE 2.7, MANAGE 4.1 |
| Path 4: MCP tool output to attacker-controlled server | LLM02, LLM03 | MCP06, MCP08 | AML.T0025, AML.T0053 | MANAGE 2.4 |
| Path 4: zero-data-retention MCP providers | LLM02 | MCP06 | AML.T0025 | MANAGE 2.4 |
| Network-layer: egress proxy | LLM02 | MCP06 | AML.T0025 | MANAGE 2.4 |
| Network-layer: DNS filtering | LLM02 | — | AML.T0025 | MANAGE 2.4 |
| Network-layer: firewall rules to known-bad | LLM02 | — | AML.T0025 | MANAGE 2.4 |
| Home / small-office per-app network visibility | LLM02 | — | AML.T0025 | MEASURE 2.7, MANAGE 4.1 |
| Logging and alerting on new destinations / volumes | LLM02 | — | AML.T0025 | MEASURE 2.7, MANAGE 4.1 |

---

## Checklists

### `checklists/pre-deployment.md`

| Section | OWASP LLM | OWASP MCP | MITRE ATLAS | NIST AI RMF |
|---|---|---|---|---|
| 1. Scope and purpose | — | — | — | MAP 1.1, MAP 1.6, MAP 3.1 |
| 2. System prompt | LLM01, LLM07, LLM02 | — | AML.T0051, AML.T0056 | MAP 5.1, MANAGE 2.3 |
| 3. Connectors (read/write/identity + MCP review) | LLM06, LLM03 | MCP02, MCP03, MCP04, MCP08 | AML.T0053, AML.T0055 | MAP 3.5, MANAGE 2.4 |
| 4. Egress controls | LLM02 | MCP06 | AML.T0025 | MANAGE 2.4 |
| 5. Human-in-the-loop | LLM06 | — | AML.T0024 | MAP 3.5, MANAGE 2.4 |
| 6. Monitoring | LLM02, LLM06 | — | AML.T0025 | MEASURE 2.7, MANAGE 4.1 |
| 7. Credential hygiene | LLM07, LLM02 | MCP10 | AML.T0055 | MANAGE 2.3 |
| 8. Self-test | LLM01 | MCP01 | AML.T0051 | MEASURE 2.6, MEASURE 2.7 |
| 9. Incident response | — | — | — | MANAGE 1.3, MANAGE 4.1 |
| 10. Disclosure and consent | LLM09 | — | — | GOVERN 1.3, MAP 3.4 |

### `checklists/quarterly-review.md`

| Section | OWASP LLM | OWASP MCP | MITRE ATLAS | NIST AI RMF |
|---|---|---|---|---|
| 1. Access review | LLM06, LLM03 | MCP02, MCP04 | AML.T0055 | MANAGE 2.4, MANAGE 4.1 |
| 2. Configuration drift | LLM03 | MCP03, MCP07 | AML.T0031, AML.T0053 | MEASURE 2.7, MANAGE 4.1 |
| 3. New attacks (OWASP / vendor / community tracking) | LLM01, LLM03 | — | AML.T0051 | MEASURE 2.7, MANAGE 4.1 |
| 4. Model drift / version changes | LLM03 | — | AML.T0031 | MEASURE 2.7, MANAGE 4.1 |
| 5. Self-test re-run | LLM01 | MCP01 | AML.T0051 | MEASURE 2.6, MEASURE 2.7 |
| 6. Log review | LLM02, LLM06 | — | AML.T0025 | MEASURE 2.7, MANAGE 4.1 |
| 7. Incident-readiness | — | — | — | MANAGE 1.3, MANAGE 4.1 |
| 8. User behavior / use-case drift | — | — | — | MAP 3.1, MANAGE 4.1 |
| 9. Provider policy / platform changes | LLM03 | — | — | MEASURE 2.7, MANAGE 4.1 |
| 10. Documentation refresh | — | — | — | GOVERN 1.3, MANAGE 4.1 |

---

## Test workflows (`test/pentest.py`)

| Workflow | OWASP LLM | OWASP MCP | MITRE ATLAS | NIST AI RMF |
|---|---|---|---|---|
| `prompt_recon` — capability discovery, system prompt extraction | LLM07 | — | AML.T0056 | MEASURE 2.6, MEASURE 2.7 |
| `prompt_direct` — instruction override, role hijacking, delimiter escape, completion manipulation, encoding bypass | LLM01 | MCP01 | AML.T0051.000, AML.T0054 | MEASURE 2.6, MEASURE 2.7 |
| `prompt_indirect` — memory poisoning, tool-output injection, document injection, email injection | LLM01, LLM02 | MCP05 | AML.T0051.001 | MEASURE 2.6, MEASURE 2.7 |
| `prompt_multistage` — multi-turn social engineering and escalation | LLM01 | — | AML.T0051, AML.T0054 | MEASURE 2.6, MEASURE 2.7 |

---

## Known gaps (what this kit does not yet cover)

These map to OWASP / ATLAS / NIST areas that are in-scope for the AI security landscape but not addressed by this kit. They are candidates for future sprints.

| Gap | OWASP LLM | OWASP MCP | MITRE ATLAS | NIST AI RMF | Planned sprint |
|---|---|---|---|---|---|
| RAG / vector-store / embedding security | LLM08 | — | AML.T0048 (Backdoor), AML.T0031 | MAP 3.5, MANAGE 2.4 | Sprint 2 |
| MCP-specific hardening (tool poisoning, agent escape, CVE-2025-6514 class) | LLM03, LLM06 | MCP03, MCP07, MCP08 | AML.T0053, AML.T0068 | MAP 3.5, MEASURE 2.7 | Sprint 2 |
| Training-data poisoning / fine-tuning attacks | LLM04 | — | AML.T0018, AML.T0020 | MAP 4.1, MANAGE 2.4 | Out of scope — link to NIST adversarial ML guidance |
| Model inversion / membership inference | LLM02 | — | AML.T0057 | MEASURE 2.7 | Out of scope — vendor-model assumption |
| Evasion / adversarial examples | — | — | AML.T0043 | MEASURE 2.7 | Out of scope — LLM focus |
| Unbounded resource consumption / cost amplification | LLM10 | — | AML.T0034 | MANAGE 2.4 | Future candidate |
| Misinformation / confabulation | LLM09 | — | — | MEASURE 2.6 | Out of scope — content safety, not security |
| AI system inventory / governance documentation | — | — | — | GOVERN 1.1, GOVERN 1.3, MAP 1.1 | Minimal appendix only |

---

## Maintenance

This crosswalk is regenerated and verified whenever:

- A framework version changes (annual minimum).
- A defense, checklist, or test workflow is added or materially revised.
- A coverage gap is closed in a future sprint.

The [`CHANGELOG.md`](../CHANGELOG.md) records each revision.
