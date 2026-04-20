# AI Infrastructure Hardening Kit

*A practical starter kit for people building or adopting AI assistants who want to reduce the risk of prompt injection, data exfiltration, and connector abuse.*

Maintained by Joshua Peskay at [Meet the Moment](https://mtm.now). Kit is vendor-agnostic — applies to Claude, ChatGPT, Gemini, and locally-hosted models.

---

## The scenario this kit is built around

You ask Claude to read through your recent emails and draft replies to any meeting requests. One of those emails — ostensibly a vendor inquiry — contains hidden instructions in white text, invisible to you but processed by the agent. Those instructions tell Claude to search your Drive for sensitive documents and send them to an attacker-controlled URL.

This is called **indirect prompt injection**, and it is the single most important class of attack to understand when you connect an AI assistant to tools that read and write your data (Gmail, Drive, Slack, CRM, calendar, etc.). It is currently the [#1 risk](https://genai.owasp.org/llmrisk/llm01-prompt-injection/) on the OWASP Top 10 for LLM Applications, and Anthropic itself [explicitly names this scenario](https://www.anthropic.com/news/prompt-injection-defenses) as the canonical threat for browser- and email-connected agents.

No model is immune. Claude Opus 4.5 reports a ~1% attack success rate against adaptive Best-of-N attackers — a significant improvement, but not zero. Your defenses cannot rely on the model alone.

---

## What this kit gives you

| Layer | What you get |
|---|---|
| **Threat model** | One-page mental model of the attack surface for an AI with Gmail / Drive / Slack access. |
| **Defense recipes** | Five concrete, implementable defenses with examples and why-it-matters reasoning. |
| **Platform guides** | Specific guidance for Claude, ChatGPT, Gemini, and local models — each platform has different controls to turn on. |
| **Checklists** | Pre-deployment and quarterly-review checklists you can hand to a CTO, CISO, or IT lead. |
| **Self-test tool** | A Python script that runs 22+ prompt-injection probes against an Anthropic API endpoint, with synthetic canary credentials. Tells you what leaks. |

This kit does **not** make your AI invulnerable. It gives you the pattern of defenses that reduces your exposure from "one carefully crafted email ruins your week" to "an attacker has to combine five independent failures to get anything useful."

---

## Who this is for

- **Nonprofit and small-business leaders** who've been asked "is it safe to give Claude/ChatGPT access to our email?" and want a real answer.
- **Consultants and vCISOs** who need a structured starting point when a client adopts an AI assistant.
- **IT staff** at any organization implementing Claude Desktop, ChatGPT Enterprise, Gemini for Workspace, or a custom LLM agent.
- **Independent builders** using Claude Code, ChatGPT Custom GPTs, or local models with MCP servers.

If your AI assistant has **read access to data you care about** OR **write access to external systems** (email, Slack, webhooks, CRMs, code), this kit is for you.

---

## How to use this kit

Read in this order:

1. [`defenses/01-threat-model.md`](defenses/01-threat-model.md) — the mental model. 10 minutes. Skip at your peril.
2. [`defenses/02-system-prompt-hardening.md`](defenses/02-system-prompt-hardening.md) — what to put in your system prompt (and what not to).
3. [`defenses/03-data-instruction-boundary.md`](defenses/03-data-instruction-boundary.md) — the core architectural problem LLMs have, and how to work around it.
4. [`defenses/04-connector-permissions.md`](defenses/04-connector-permissions.md) — scoping what your AI can actually touch.
5. [`defenses/05-egress-controls.md`](defenses/05-egress-controls.md) — stopping data from leaving.
6. The [`platforms/`](platforms/) guide for whatever you're using (Claude, ChatGPT, Gemini, local).
7. [`checklists/pre-deployment.md`](checklists/pre-deployment.md) — go through this before you ship anything.
8. [`test/`](test/) — run the self-test. Review the report.
9. Re-run the self-test quarterly using [`checklists/quarterly-review.md`](checklists/quarterly-review.md).

Total time to read the core material: ~90 minutes.
Time to run the self-test: ~10 minutes.
Time to implement meaningful defenses: 2–8 hours depending on your stack.

---

## Structure

```
ai-hardening-kit/
├── README.md                          ← You are here
├── LICENSE                            ← MIT
├── SECURITY.md                        ← How to report issues in this kit
│
├── defenses/                          ← Platform-agnostic defense recipes
│   ├── 01-threat-model.md
│   ├── 02-system-prompt-hardening.md
│   ├── 03-data-instruction-boundary.md
│   ├── 04-connector-permissions.md
│   └── 05-egress-controls.md
│
├── platforms/                         ← Platform-specific controls
│   ├── claude.md
│   ├── chatgpt.md
│   ├── gemini.md
│   └── local-models.md
│
├── checklists/
│   ├── pre-deployment.md
│   └── quarterly-review.md
│
└── test/                              ← Self-test runner
    ├── README.md
    ├── pentest.py
    └── config.template.env
```

---

## What this kit will not give you

- **A guarantee of safety.** Prompt injection is an active area of research. The defenses here raise the cost of attack; they do not eliminate it.
- **Compliance certification.** This is not a SOC 2 control set, a HIPAA checklist, or a GDPR DPIA. It is a practitioner's starting point.
- **A product.** This is open content. Fork it, adapt it, improve it. If you're a consultant, you can build a service around it. If you're a user, you can just apply it.
- **A substitute for a security professional.** If you handle sensitive data — health records, financial data, legal client information — you need a real security review, not a GitHub repo.

---

## Contributing

If you find something that doesn't work, a defense that's missing, a new attack class that deserves coverage, or a platform that needs its own guide — open an issue or pull request. This kit improves when real practitioners push back on it.

See [`SECURITY.md`](SECURITY.md) for how to report security issues in the kit itself.

---

## Credits and sources

This kit draws on:

- [OWASP Top 10 for Large Language Model Applications (2025)](https://genai.owasp.org/) — the authoritative taxonomy of LLM risks.
- [OWASP Top 10 for MCP (2025)](https://genai.owasp.org/) — connector-specific risks.
- [Anthropic: Mitigating the risk of prompt injections in browser use](https://www.anthropic.com/news/prompt-injection-defenses) — the canonical threat model.
- [NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework) — the governance layer.
- Oasis Security's [disclosure of the Claude.ai exfiltration vulnerability](https://www.oasis.security/blog/claude-ai-prompt-injection-data-exfiltration-vulnerability) — a real-world example of this attack class.
- Meet the Moment's internal pentest skill and vCISO practice, where most of these patterns were first tested on real nonprofit infrastructure.

---

*Maintained by Joshua Peskay, CISSP / CISM. Last revised April 2026.*

*If you're new to AI security and this is overwhelming, start with just [`defenses/01-threat-model.md`](defenses/01-threat-model.md) and the [`checklists/pre-deployment.md`](checklists/pre-deployment.md). That alone will put you ahead of most deployments.*
