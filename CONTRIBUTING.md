# Contributing to the AI Hardening Kit

Thanks for considering a contribution. This kit improves when practitioners push back on it — with new attacks, new platforms, new controls, or corrections to guidance that has aged.

---

## Ways to contribute

**Report a problem.** If guidance is wrong, outdated, or leads people into a false sense of security, open an issue. Include the file and section, what the current guidance says, and what you think it should say.

**Suggest a new control.** If you have a pattern that works in production and isn't covered here, open an issue or a PR. Include: the threat it addresses, the pattern, a concrete example, and where it should live in the `defenses/` structure.

**Add a platform guide.** `platforms/` currently covers Claude, ChatGPT, Gemini, and local models. If you want to add another (Cohere, Mistral, enterprise deployments of open-weight models), follow the pattern of the existing guides: what the platform does for you, what it doesn't, specific controls to enable, common pitfalls.

**Contribute an attack to the test suite.** See [`test/attacks/`](test/attacks/) (added in a later sprint) for the attack library format. Every attack should map to an OWASP LLM / MCP / ATLAS technique ID.

**Translate.** This kit is English-only. If you'd like to translate, open an issue first so we can coordinate file structure.

---

## Before opening a PR

1. **Scope fit.** This kit is about reducing an organization's exposure to prompt injection, connector abuse, and egress. It is explicitly not about AI governance frameworks, content safety, model-level adversarial robustness, or regulatory compliance. If your contribution is in one of those areas, consider whether a pointer to external guidance might serve better than adding it here.

2. **Vendor-agnostic first.** Place generic guidance in `defenses/`, platform-specific guidance in `platforms/`. If a control exists on multiple platforms, the generic principle belongs in `defenses/` with a note in each platform file about how to configure it.

3. **Map to frameworks.** Every new control or defense section should be added to [`mappings/crosswalk.md`](mappings/crosswalk.md) and [`mappings/controls.yaml`](mappings/controls.yaml) with OWASP LLM / OWASP MCP / MITRE ATLAS / NIST AI RMF tags. Unmapped controls won't be merged — the crosswalk is load-bearing for the web app and for audit use.

4. **Write for a nonprofit IT lead, not a CISO.** Plain English. Examples before abstractions. "Do this / not this" framing where possible. A concrete snippet beats a principle.

5. **Update [`CHANGELOG.md`](CHANGELOG.md).** Every user-visible change needs a changelog entry under `Unreleased`.

---

## Testing changes

- **Pentest changes:** run `python3 test/test_check_leaks.py` before opening a PR. All tests must pass.
- **Content changes:** spell-check and verify every external link resolves.
- **YAML changes to `controls.yaml`:** `python3 -c "import yaml; yaml.safe_load(open('mappings/controls.yaml'))"` (or equivalent) to confirm valid YAML.

---

## Reporting security issues

Security issues *in this kit itself* (for example, test code that leaks real credentials, a config template that recommends an insecure practice, or a defense that is actively harmful) go to [`SECURITY.md`](SECURITY.md) — **not** a public issue.

Security issues in Claude, ChatGPT, Gemini, MCP servers, or other third-party systems should be reported to the vendor directly. This repository is not the right channel.

---

## Style notes

- Markdown, not HTML.
- Headings use `#` / `##` / `###`. No `####` — that's a signal a section should be its own file.
- External links go to authoritative sources (OWASP, NIST, MITRE, vendor blogs). Avoid news summaries or paywalled content.
- Don't introduce emojis unless the user-facing content warrants them (it usually doesn't).
- Short sentences. Declarative voice. Avoid hedging.

---

## License

By contributing, you agree that your contribution is licensed under the [MIT License](LICENSE) that covers this repository.
