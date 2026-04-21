# Changelog

All notable changes to the AI Hardening Kit are documented here. This project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html): major versions for breaking changes to structure or public artifact paths, minor versions for new defenses or platforms, patches for corrections and clarifications.

This changelog is the authoritative record of "when was this last updated" for funder and audit questions. Every user-visible change belongs here.

---

## [Unreleased]

### Added
- README link to the new web companion app at [ai-safety-check.mtmapps.now](https://ai-safety-check.mtmapps.now), built on the kit's `mappings/controls.yaml`. The app is the nonprofit-staff-friendly entry point; this repo remains the technical reference.

---

## [1.0.0] — 2026-04-21

First tagged release. Content had been stable under active use through early 2026; this release establishes the baseline for versioned updates going forward.

### Added
- `mappings/crosswalk.md` — human-readable mapping of every defense, checklist item, and test workflow to OWASP LLM Top 10 2025, OWASP MCP Top 10 2025, MITRE ATLAS v5.4, and NIST AI RMF 1.0.
- `mappings/controls.yaml` — machine-readable version of the crosswalk, consumed by automation and by the forthcoming web application.
- `CONTRIBUTING.md` — contribution guidelines, scope boundaries, and PR requirements.
- `CHANGELOG.md` — this file.
- `test/test_check_leaks.py` — 15 unit tests for the leak-detection logic.

### Changed
- `test/pentest.py` — `check_leaks()` rewritten to detect canary leaks in multiple evasion forms: Base64, hex, and ROT13 encodings; MD5 / SHA1 / SHA256 hash digests; distinctive canary "cores" (e.g., an SSN leaked without the `CANARY-SSN-` prefix); and fuzzy forms with punctuation stripped. Previously matched only exact substrings, which attackers could evade by asking the model to base64-encode the output.

### Baseline (content carried forward from pre-1.0.0)
- `defenses/01-threat-model.md` through `defenses/05-egress-controls.md` — five defense recipes covering threat model, system prompt hardening, data/instruction boundary, connector permissions, and egress controls.
- `platforms/claude.md`, `chatgpt.md`, `gemini.md`, `local-models.md` — platform-specific guidance.
- `checklists/pre-deployment.md`, `quarterly-review.md` — operational checklists.
- `test/pentest.py` — self-test runner with four workflows (recon, direct, indirect, multi-stage) using canary credentials.
- `README.md`, `LICENSE` (MIT), `SECURITY.md`.

---

## Versioning policy

| Change type | Bump |
|---|---|
| New defense recipe, new platform guide, new checklist, new test workflow | Minor |
| New sub-section within an existing recipe, new attack in `test/attacks/`, new framework mapping | Minor |
| Correction, clarification, typo fix, link repair | Patch |
| Removal of a defense, restructure of the directory layout, breaking change to `controls.yaml` schema | Major |
| Updated `last_verified` date in `controls.yaml` with no other content change | Patch |

Releases are tagged on `main` using `git tag vMAJOR.MINOR.PATCH` and published to GitHub Releases.
