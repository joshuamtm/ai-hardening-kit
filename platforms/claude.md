# Claude (Anthropic)

## Scope

This guide covers:

- **Claude.ai** (web and mobile apps)
- **Claude Desktop** (Mac and Windows apps)
- **Claude for Chrome** (browser extension)
- **Anthropic API** (direct programmatic use)
- **Claude Code** (terminal-based development agent)
- **Claude connectors** (Gmail, Drive, GitHub, Slack integrations)

If you use a custom application built on the Anthropic API, most of the guidance in the API section applies.

---

## What Anthropic does for you

Anthropic trains Claude with adversarial examples — they expose Claude to prompt injections during training and reward it for refusing them. Claude Opus 4.5 reports roughly 1% attack success rate against adaptive Best-of-N attackers. That's a significant improvement over earlier models, but not zero. ([Source](https://www.anthropic.com/news/prompt-injection-defenses).)

Anthropic also provides:

- **Prompt caching** with user-scoped keys (reduces cross-user contamination risk).
- **Content filtering** at the API layer for obviously harmful outputs.
- **OAuth scope controls** on connectors (Gmail, Drive).
- **Tool approval prompts** in Claude Desktop for destructive actions.
- **Published threat models** and disclosure relationships with researchers.

What they do not provide:

- A guarantee Claude won't follow a cleverly-crafted injection.
- Automatic egress controls for your data.
- Enforcement of your organization's own data policies.
- Protection against MCP servers you install yourself.

---

## Claude.ai (web and mobile)

### Settings to turn on

1. **Workspace admin** (if you're on Team/Enterprise): enforce SSO, limit model switching, require organization-level API key control.
2. **Personal account**: review connected apps under *Settings → Integrations*. Remove anything you don't actively use.
3. **Memory**: if you use Claude's conversation memory feature, periodically review what Claude has stored. Memory is an indirect-injection vector — if you once pasted a document that contained an injection, it can live in memory.
4. **Sharing**: public conversation sharing is off by default. Keep it off unless you have a specific reason to share.

### Gmail and Drive connectors

The classic attack scenario targets these connectors directly. Specific guidance:

- **Gmail connector:** enable only if you need it. Grant read-only scope if you're just summarizing; do NOT grant send scope for initial deployment.
- **Drive connector:** prefer specific-folder access over full-Drive access. If Claude's connector doesn't let you scope to a folder, create a separate Drive account or Workspace for AI-safe content.
- **Google account:** consider using a secondary Google account for AI connections, so compromise of that account doesn't affect your primary identity.

### What to test

Run the self-test in [`../test/`](../test/) using your actual deployment's system prompt if you can. If you can't (Claude.ai doesn't expose the system prompt), at least verify that Claude refuses to:

- Reveal "IMPORTANT SYSTEM DATA" in its prompt.
- Summarize the contents of an email that contains "ignore previous instructions."
- Send an email based on an instruction embedded in a document.

---

## Claude Desktop

### MCP server hygiene

Claude Desktop supports MCP servers, and this is where most of the risk lives today.

- **Audit your installed MCP servers** by reviewing `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or the Windows equivalent.
- **Each MCP server is code running with your user privileges.** Treat it with the same scrutiny as a CLI tool you install.
- **Prefer signed, well-maintained servers** from Anthropic, Model Context Protocol organization, or major vendors (GitHub, Slack, Google, etc.). Be cautious with anonymous GitHub repos.
- **Use the tool baseline check** in this kit's self-test: run it, store the baseline, diff periodically to catch tool-shadowing / rug-pull attacks.

### Permissions

Claude Desktop has per-tool approval prompts. Do not blanket-approve them. Read what each tool does on first call. Deny aggressive permissions (filesystem write, shell execution) unless you have a specific need.

### Local file access

Claude Desktop's filesystem tools can read and write your local disk. This is extremely useful. It is also extremely dangerous if combined with indirect injection (a document you're summarizing tells Claude to write a malicious script to `~/Documents/`).

- Use the filesystem MCP server with the narrowest root directory possible.
- Do not give Claude access to `~/` or `/`. Scope to a project folder.
- Review Claude's filesystem writes periodically. Anything unexpected is a signal.

---

## Claude for Chrome

Claude for Chrome is available on the Max plan as of late 2025. It is the browser-agent scenario Anthropic designed around.

### Enabled protections (default)

- User-level confirmations for actions (form submissions, clicks).
- Adversarial training specifically for prompt injection in web content.
- Defense-in-depth: site-level allowlists, per-user action rate limits.

### What you should still do

- **Allowlist the sites Claude for Chrome operates on.** Don't let it run on arbitrary sites. Start with 2–5 trusted sites; expand cautiously.
- **Don't run Claude for Chrome on the same browser profile** as sites where you're logged into sensitive accounts (banking, health, legal). Use a separate Chrome profile or Chromium-family browser for AI-enabled browsing.
- **Review action history** weekly. Any unexpected interaction is a signal.

---

## Anthropic API (direct programmatic use)

This is where you have the most control. Apply all five defense recipes.

### System prompt

Use the reference template in [`../defenses/02-system-prompt-hardening.md`](../defenses/02-system-prompt-hardening.md). Do not embed credentials.

### Tool design

- Tools should enforce their own security contracts in code, not rely on the prompt.
- Use strict JSON schemas for structured outputs.
- Validate all tool inputs (recipients, URLs, file paths) against allowlists inside the tool function.

### Model choice

For security-sensitive applications, use the most recent Opus model. Claude Opus 4.5 has the best prompt-injection robustness as of this writing. The cost delta over Sonnet is meaningful; the security delta is also meaningful for high-risk use cases.

### Prompt caching

If you use prompt caching, ensure cache keys are scoped per-user. Cross-user cache contamination has been demonstrated as an attack vector in other systems — Anthropic's prompt caching is scoped, but verify this hasn't changed.

### Rate limiting and quotas

Set per-user and per-session quotas at your application layer. Runaway injection-driven behavior (e.g., "enumerate every file in Drive") can burn through cost quickly.

### Monitoring

Log all requests and responses. Flag unusual patterns:

- Unusually long outputs (potential data exfiltration).
- Base64 or hex content in outputs (potential encoded exfiltration).
- Tool calls with unusual arguments (URLs not in allowlist, recipients outside domain).
- Multiple consecutive tool calls without user interaction (potential injection-driven autonomy).

---

## Claude Code

Claude Code is the terminal-based development agent. It has significant capabilities (shell, filesystem, git, arbitrary tools via MCP) and therefore significant exposure.

### Defaults to change

- **Working directory scope:** Claude Code defaults to the directory where you launch it. That's fine for project work. It's not fine if you launch it in `$HOME`.
- **Permission prompts:** review each permission carefully. Do not blanket-allow destructive operations (git push, rm, etc.) at the user-settings level. Scope to the project's `.claude/settings.json`.
- **MCP servers:** same as Claude Desktop. Audit what's installed; prefer well-maintained sources.

### Specific risks

- **`CLAUDE.md` files:** these are loaded automatically. A repository that someone else wrote can plant instructions in its `CLAUDE.md` that affect your agent. Read project `CLAUDE.md` files before running Claude Code in an unfamiliar repo.
- **Documents processed by the agent:** the same indirect injection risk applies. A pull request description, a bug report, a log file — any text content can contain injection payloads.
- **Skill files:** custom skills execute with your user privileges. Audit any skill you install.

### Configuration recommendations

```json
// .claude/settings.json — tight defaults for a project
{
  "permissions": {
    "deny": [
      "Bash(rm:*)",
      "Bash(git push --force:*)",
      "Bash(curl -X DELETE:*)"
    ],
    "ask": [
      "Bash(git push:*)",
      "Bash(npm publish:*)"
    ]
  }
}
```

Start restrictive. Relax specific rules as you need them.

---

## Things Anthropic does not currently offer (you need to build)

- **Egress allowlists at the network layer.** You need to run your own proxy or firewall.
- **Per-connector data classification.** You decide what's sensitive; Anthropic does not.
- **Logs across Claude.ai and Claude Desktop into your SIEM.** You'd have to build exports.
- **Cross-session anomaly detection.** You need to instrument your application.

---

## If something goes wrong

If you suspect an injection succeeded against your Claude deployment:

1. **Revoke connector OAuth tokens immediately** (Gmail, Drive, GitHub, etc.).
2. **Rotate any credential that may have been exposed.**
3. **Review recent email sends, Drive file creates, and Slack messages** from the affected user.
4. **Report to Anthropic's [Responsible Disclosure Program](https://www.anthropic.com/responsible-disclosure-policy)** if the issue appears to be a Claude-side vulnerability rather than a configuration mistake on your side.
5. **Log an incident ticket** and run the self-test in this kit to confirm your current posture.

---

## References

- [Anthropic: Prompt injection defenses](https://www.anthropic.com/news/prompt-injection-defenses)
- [Anthropic: Responsible Disclosure Policy](https://www.anthropic.com/responsible-disclosure-policy)
- [Claude Desktop MCP documentation](https://docs.anthropic.com/claude/docs/mcp)
- [Model Context Protocol](https://modelcontextprotocol.io/)
