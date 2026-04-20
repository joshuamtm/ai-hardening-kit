# 04 · Connector Permissions

## Why this matters

Every connector you give your AI — Gmail, Google Drive, Slack, a CRM, a database, a shell, an MCP server — is a new capability for the AI and a new capability for any successful prompt injection.

The single highest-leverage thing you can do to reduce the blast radius of an attack is to scope connectors tightly. A prompt injection that succeeds against an AI with **read-only access to one specific folder** has nowhere to go. The same injection against an AI with **full Drive access** can exfiltrate years of institutional history.

The principle: **least privilege, always.** Give the AI the minimum access necessary for the task you actually want it to do. Not the task you might eventually want it to do. Not the task a vendor's onboarding guide tells you to grant. The minimum for what you're doing right now.

---

## The three scopes that matter

Every connector has three orthogonal scope dimensions. Think about all three for every integration.

| Dimension | Question | Minimizing this limits... |
|---|---|---|
| **Read surface** | What data can the AI read? | What a successful injection can exfiltrate. |
| **Write surface** | What can the AI modify, create, delete, or send? | What a successful injection can use to cause damage or exfiltrate via side channels (email, calendar invites, public docs). |
| **Identity** | Whose credentials does the AI use? | Whose data is exposed when things go wrong. |

---

## Gmail — concrete patterns

Gmail is the highest-risk connector for most organizations because it is the richest source of attacker-controlled content. Anyone can email you. Every email the AI reads is a potential injection vector.

### Minimize read surface

**Bad:** Full mailbox access, all labels, all threads.

**Better:** A specific label (e.g., `ForAI` or `AI-Triage`) that the user manually applies to emails they want the AI to process. Everything else is invisible to the AI.

**Best:** User-initiated, per-email access. The AI only sees an email when the user explicitly hands it one (forwards it, attaches it, or pastes the content into the conversation).

### Minimize write surface

**Bad:** Send-as permission on the user's primary address. Every AI-composed email looks like it came from the user.

**Better:** Drafts-only permission. The AI can draft; a human must click send. This matches Joshua's ["always draft, never send"](https://github.com/joshuamtm/pentest-kit) personal rule. This one control prevents most exfiltration-via-email attacks.

**Best:** Drafts-only AND outbound-domain allowlist. Drafts that address a domain outside the allowlist require additional confirmation.

### Minimize identity

**Bad:** Shared Gmail account for "the AI" that has elevated permissions.

**Better:** Each user's AI acts under each user's identity, with scopes granted via OAuth that the user can revoke at any time.

---

## Google Drive — concrete patterns

Drive is dangerous because the natural default ("access all my Drive files") is enormous. A prompt-injection that tells the AI to search for sensitive keywords across your Drive will often succeed by default, because the AI was granted the access to do so.

### Minimize read surface

**Bad:** Full Drive access, including shared drives.

**Better:** One specific folder designated as "AI-safe," containing only files the user has consciously chosen to make available.

**Best:** Per-session file selection. The AI only sees a file when the user explicitly attaches or references it.

### Minimize write surface

**Bad:** Full create/edit/delete on Drive.

**Better:** No write access. The AI drafts content in its own scratch space; the user copies and pastes into Drive manually if they want to keep it.

**Best:** Write access limited to a single "AI outputs" folder that the user periodically reviews and files.

### Special concern: shared drives and Workspace-wide files

If your Drive contains files shared from others or shared with the whole Workspace, understand that attacker-controlled content may already be in your Drive today. A document someone shared with you last month could contain an injection payload that activates the first time your AI reads it.

**Mitigation:** Do not give AIs unrestricted access to Drive files shared with you by others. If you must, combine with strong egress controls (see [`05-egress-controls.md`](05-egress-controls.md)).

---

## Slack — concrete patterns

Slack is a prompt-injection paradise: any channel member can post text, any public channel accepts messages from anyone in the workspace, and DMs can be sent by anyone.

### Minimize read surface

**Bad:** Access to all public channels and all DMs.

**Better:** Specific channels only, excluded from general channels, excluded from DMs by default.

**Best:** User-initiated read. The AI only sees a Slack message when the user forwards it or asks about it specifically.

### Minimize write surface

**Bad:** Post-as-user in all channels.

**Better:** Post-as-bot (with a bot account clearly identified as AI-generated) in designated channels only.

**Best:** Draft-only, post manually.

---

## Calendar, CRM, Notion, Airtable, and similar

For every connector, ask the three scope questions:

- What can this read? Can I narrow it?
- What can this write? Can I remove write entirely?
- Whose identity is it using? Can it be a scoped, revocable credential?

Most SaaS platforms support scoped API tokens, service accounts, or row/object-level access controls. Use them.

---

## MCP servers

The [Model Context Protocol](https://modelcontextprotocol.io/) is becoming the de facto standard for AI-to-tool connections. It is also a new attack surface.

### Risks specific to MCP

- **Tool shadowing / rug pulls** (OWASP MCP07): an MCP server can change its tool descriptions over time. The description your AI sees on the 100th call is not necessarily the description it saw on the 1st call. A malicious or compromised server can evolve its descriptions to induce the AI to misuse tools.
- **Shadow MCP servers** (OWASP MCP09): MCP servers running on localhost that weren't explicitly installed by the user — perhaps bundled with a browser extension, an IDE plugin, or a rogue process.
- **Excessive scope** (OWASP MCP02): MCP servers often request broader scope than they need because it's easier to build.
- **Indirect injection via tool output** (OWASP MCP05): MCP tool responses can contain injected instructions.

### Mitigations

- **Vendor each MCP server you install.** Read the repo. Understand what it does. Prefer signed, official sources over random GitHub repos.
- **Baseline tool descriptions at install time.** Store a snapshot. Periodically diff against the current state. Any unexpected change is a signal to review before continuing to use.
- **Sandbox MCP servers that interact with the outside world** (web, external APIs). Run them in a container or with restricted network egress.
- **Audit the MCP servers you actually have.** Most Claude Desktop / Claude Code users accumulate MCPs over time and never audit the list.

This kit's self-test includes a baseline check for MCP tool descriptions. See [`test/`](../test/).

---

## Identity and credential hygiene

### Never embed long-lived credentials in skill files, system prompts, or config checked into git

Use a secrets manager (1Password, Doppler, AWS Secrets Manager, or at minimum environment variables with a tight `.gitignore`).

### Prefer short-lived OAuth tokens over long-lived API keys

OAuth tokens expire. API keys don't. When an OAuth token leaks, the damage window is hours. When an API key leaks, the damage window is until someone notices.

### Audit token freshness periodically

Any OAuth token older than 90 days should trigger a review. If you can't remember granting it, revoke it.

### Log token usage

At least at the Gmail / Drive / Slack level, turn on access logs. If an AI-associated token is used outside a normal hours window, that's a signal.

---

## A real example — scoping a Claude Desktop deployment

**Goal:** Claude Desktop helps a nonprofit director triage email and draft responses.

**Conservative scoping:**

| Connector | Read | Write | Identity |
|---|---|---|---|
| Gmail | Inbox label `AI-Triage` only | Drafts only | Director's OAuth, revocable |
| Drive | Single folder `AI-Working` | No write | Director's OAuth, revocable |
| Calendar | Read-only, primary calendar only | No write | Director's OAuth, revocable |
| Slack | None | None | — |

**What a successful prompt injection can do:**

- Read the emails the director explicitly triaged (already seen by the director).
- Draft an email (which the director has to click send on).
- Read files the director explicitly put in `AI-Working` (already seen by the director).
- Read calendar events (mostly meeting titles and times, low sensitivity).

**What it cannot do:**

- Exfiltrate arbitrary emails the director hasn't triaged.
- Send email without human confirmation.
- Touch Drive files outside the working folder.
- Access Slack.

This is not zero risk. It is manageable risk. The director can use the AI productively without one malicious email ending their week.

---

## The honest tradeoff

Least-privilege connectors make the AI less useful for autonomous, end-to-end workflows. If you want an AI that independently processes your inbox, scans your Drive for relevant context, and sends replies without your involvement, you are choosing a high-capability / high-risk configuration.

That configuration might be the right call for some use cases. It is almost never the right call for a first deployment. Start conservative, expand only after you've run the self-test and seen the model handle adversarial content correctly in your specific context.

---

## References

- [OWASP LLM06:2025 — Excessive Agency](https://genai.owasp.org/)
- [OWASP MCP Top 10 (2025)](https://genai.owasp.org/)
- [NIST AI RMF — MAP 3.5, MANAGE 2.4: scope and permission boundaries](https://www.nist.gov/itl/ai-risk-management-framework)
- [Model Context Protocol documentation](https://modelcontextprotocol.io/)
