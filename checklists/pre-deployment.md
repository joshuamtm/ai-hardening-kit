# Pre-Deployment Checklist

Before you turn on an AI assistant that will touch your data or your users' data, run through this list. If you cannot check an item, write down why — the exceptions are where your risk lives.

This is a practitioner's checklist. It assumes you've read the [`defenses/`](../defenses/) material and the relevant platform guide. It is not a substitute for a security review if you handle regulated data.

---

## 1 · Scope and purpose

- [ ] I can describe in one sentence what this AI is for, who will use it, and what data it will touch.
- [ ] I have identified the worst-case scenario if this AI is abused by a prompt injection.
- [ ] I have named the threat actors I'm defending against (see `defenses/01-threat-model.md`).
- [ ] The deployment is narrowly scoped to the stated purpose. I am not enabling capabilities "just in case."

## 2 · System prompt

- [ ] No credentials, API keys, or long-lived secrets are embedded in the system prompt.
- [ ] Role is anchored specifically (not "you are a helpful assistant").
- [ ] Explicit instructions for handling adversarial input are included.
- [ ] Explicit data-vs-instruction framing is included.
- [ ] Confirmation gating is specified for irreversible actions.
- [ ] I have assumed the system prompt will leak and am comfortable with what's in it.

## 3 · Connectors

For each connector (Gmail, Drive, Slack, CRM, MCP server, etc.):

- [ ] Read scope is the minimum needed for the stated purpose.
- [ ] Write scope is the minimum needed — ideally zero, or drafts-only.
- [ ] Identity is a revocable OAuth token, not a shared account or long-lived API key.
- [ ] OAuth token grants are documented — I know what each token can do.
- [ ] I can revoke any single connector in under 5 minutes if something goes wrong.

For MCP servers specifically:

- [ ] I have reviewed the source of every MCP server I've installed.
- [ ] I have baselined the tool descriptions (see `test/pentest.py` and the `mcp-tool-baseline.json` pattern).
- [ ] I know which MCP servers make external network calls and which do not.

## 4 · Egress controls

- [ ] I know what network egress paths exist for this AI (HTTP, email, shared docs, MCP).
- [ ] Egress to external destinations is allowlisted, not denylisted, where possible.
- [ ] Email send (if present) is drafts-only OR restricted to an allowlist of recipient domains.
- [ ] File sharing (if present) defaults to private; AI cannot create public shares.
- [ ] Outbound HTTP from the AI process is either disabled or proxied through an allowlist.

## 5 · Human-in-the-loop

- [ ] Every irreversible action requires explicit, in-the-moment user confirmation.
- [ ] Confirmation cannot be satisfied by in-conversation text (because that can be forged by injection).
- [ ] The user sees what they are confirming in plain language.

## 6 · Monitoring

- [ ] All AI requests and responses are logged.
- [ ] All tool calls are logged with arguments.
- [ ] Logs are retained for at least 30 days, ideally 90.
- [ ] I have alerts or weekly reviews for: new egress destinations, unusually large outputs, unusual tool-call patterns, new MCP tool descriptions.

## 7 · Credential hygiene

- [ ] No secrets in the system prompt.
- [ ] No secrets in skill files, config files, or code checked into git.
- [ ] All `.env` files are gitignored.
- [ ] OAuth tokens older than 90 days are reviewed or rotated.
- [ ] If a secret leaks, I know which secret and how to rotate it.

## 8 · Self-test

- [ ] I have run the self-test in [`../test/`](../test/) against this deployment's system prompt.
- [ ] Zero canary values leaked across all workflows.
- [ ] Any failures have remediation before go-live (not "we'll fix it later").
- [ ] I have saved the baseline report for comparison on future test runs.

## 9 · Incident response

- [ ] I know who to contact if a prompt injection incident is suspected.
- [ ] I know how to revoke all AI connector tokens quickly.
- [ ] I know how to rotate credentials that may have been exposed.
- [ ] I know where to report platform-side vulnerabilities (Anthropic / OpenAI / Google).
- [ ] Users have been told: "if Claude / ChatGPT / Gemini does something you didn't ask for, tell me immediately."

## 10 · Disclosure and consent

- [ ] Users of this AI know it's an AI.
- [ ] Users know what data it can access.
- [ ] Users have a way to opt out or to request review of AI-assisted decisions.
- [ ] If you're in a regulated space (health, finance, legal), required notices are in place.

---

## Scoring

- **8–10 sections fully checked:** ready to go live with routine monitoring.
- **5–7 sections fully checked:** go live only for low-sensitivity use cases while you close the gaps.
- **Below 5:** not ready. Most common gap is egress controls and monitoring. Both are unglamorous and both are where real incidents originate.

---

## What this checklist doesn't cover

- Regulatory compliance requirements (HIPAA, GDPR, FERPA, PCI, etc.). Those have their own control sets.
- Data classification and labeling policies — you need those before you can scope connectors meaningfully.
- Vendor risk management for the AI provider itself.
- Training and awareness for the humans using the AI.

If the deployment handles regulated data or is business-critical, use this checklist as a starting point and layer your compliance framework on top.
