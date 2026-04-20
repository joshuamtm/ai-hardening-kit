# ChatGPT (OpenAI)

## Scope

This guide covers:

- **ChatGPT** (web and mobile apps, Free/Plus/Team/Enterprise)
- **Custom GPTs** (GPTs you build for yourself or distribute)
- **GPT Actions** (the connector/tool mechanism for Custom GPTs)
- **OpenAI Connectors** (Gmail, Drive, SharePoint, and others)
- **OpenAI API** (direct programmatic use)
- **ChatGPT Desktop app**

---

## What OpenAI does for you

OpenAI runs content filtering, refusal training, and moderation endpoints. They publish a [usage policy](https://openai.com/policies/usage-policies/) and a [security portal](https://openai.com/security/). For enterprise customers, they offer SOC 2 Type 2 compliance, SAML SSO, data retention controls, and admin APIs.

What they do not provide out-of-the-box:

- Strong prompt-injection resistance equivalent to adversarial training (OpenAI's approach differs from Anthropic's; independent benchmarks show GPT-4 family models are injectable at higher rates than Claude Opus family under comparable attacks).
- Egress controls for Custom GPT Actions (you configure these yourself).
- Tool-level allowlists for which URLs Actions can call (partial — domain verification is offered, but not hostname-level allowlists).

---

## ChatGPT (end-user)

### Settings to turn on

1. **Data controls → Chat history & training**: if you don't want your conversations used for training, disable. Be aware that disabling history also disables memory and some features.
2. **Data controls → Memory**: review and clear memory periodically. Memory is an indirect-injection vector.
3. **Connected apps**: review under Settings. Remove anything you don't actively use.
4. **Team/Enterprise admin**: require SSO, set data retention policies (Enterprise defaults to no training data use, but verify).

### Connectors (Gmail, Drive, SharePoint, GitHub, etc.)

OpenAI's connectors follow OAuth scoping. The scoping controls are governed by the source platform (Google, Microsoft, etc.), not by OpenAI directly.

- Grant read-only scope wherever possible.
- If a connector asks for broader scope than you need, consider whether you really need the connector or can get by with manual paste-in.
- Review granted permissions quarterly and revoke unused ones.

### Shared projects / enterprise workspaces

Projects and workspaces are shared contexts. A document another user added can contain injection payloads that affect your queries if the workspace is shared. Treat shared workspaces with the same skepticism you'd apply to shared Google Drives.

---

## Custom GPTs

Custom GPTs are a significant attack surface. They combine:

- A system prompt you write (that you cannot keep private).
- External Actions that call arbitrary HTTPS endpoints.
- Knowledge files (RAG-ish content the GPT references).
- Public distribution potentially exposing your GPT to any ChatGPT user.

### System prompt

Your Custom GPT's system prompt is **extractable**. Users have demonstrated extraction of virtually every published Custom GPT's instructions. Never put API keys, credentials, private business logic, or anything you don't want public into the system prompt.

Instead:

- Put instructions in the system prompt.
- Put secrets in the Action's authentication configuration (header, OAuth, API key stored by OpenAI).
- Put business logic in the Action's backend code, not in the prompt.

### Actions

Actions let your GPT call external HTTPS endpoints. Treat your Action backend as the security boundary, not the GPT prompt.

- **Authenticate every request.** The Action forwards a user identifier; verify it on your side.
- **Authorize every request.** The GPT may attempt to call your action with arbitrary parameters. Validate against business rules server-side.
- **Rate-limit.** A prompt injection can get the GPT to hammer your action. Protect yourself.
- **Do not trust Action descriptions** your GPT sends. An attacker who can inject instructions can get the GPT to send custom arguments that a poorly-validated backend would accept.

### Knowledge files

Knowledge files you upload to your GPT are loaded into context on user queries. They are also extractable. Do not upload sensitive documents as GPT knowledge.

Also: an injection in the user's query can, in some cases, cause the GPT to emit portions of its knowledge back to the user. If your knowledge contains material you don't want disclosed verbatim, don't put it there.

### Public GPTs

If you publish your GPT:

- Assume users will try to extract the system prompt. They will succeed often enough that you cannot treat the prompt as private.
- Assume users will probe every Action endpoint. Treat them as hostile clients.
- Monitor usage. OpenAI provides some Action telemetry; use it.

### Private / workspace-only GPTs

Slightly lower risk than public, but still:

- Workspace members can still extract system prompts.
- An attacker who phishes a workspace member is inside your trust boundary.

---

## OpenAI API (direct programmatic use)

Apply all five defense recipes. OpenAI-specific considerations:

### Model choice

For security-sensitive applications, use GPT-4o or newer. Smaller/older models are easier to inject. GPT-4o-mini is tempting for cost, but expect to see higher injection success rates.

### Function calling / tool use

OpenAI's function calling requires you to define JSON schemas. Use them strictly:

- `additionalProperties: false` on all object schemas.
- Enumerate allowed values where possible.
- Validate outputs server-side even if the schema says they should be valid. The model does not always comply.

### Structured outputs

OpenAI supports `strict: true` on function schemas as of late 2024. Turn this on for any security-sensitive tool. It enforces schema compliance at the decoding layer, which is much stronger than post-hoc validation.

### Moderation endpoint

OpenAI's moderation endpoint is cheap and covers classes of abuse (violence, sexual content, hate speech). It does not cover prompt injection. Do not assume the moderation endpoint protects you from injection.

### Logging

Log all requests and responses. OpenAI's billing dashboard is not enough. Log structured data so you can search by user, session, tool call, and output content.

---

## ChatGPT Desktop

The desktop app has additional capabilities over the web (screen reading, voice mode, some automation on macOS).

- **Screen reading**: treat anything on screen as potentially injection-laden. If you use this feature while processing content from untrusted sources, expect injection attempts to sometimes succeed.
- **Voice mode**: audio transcription is an injection vector. An attacker who can play audio near your microphone (or provide audio content for transcription) can inject.
- **Autonomous features**: turn off any "click buttons for me" / "run commands for me" feature unless you explicitly need it.

---

## Enterprise-specific controls

If you're on ChatGPT Enterprise or Team, additional controls:

- **SAML SSO**: enforce it. No personal accounts in the workspace.
- **Data retention**: set to the minimum your compliance allows.
- **Admin API**: use for user lifecycle, monitoring, and off-boarding.
- **IP allowlist** (Enterprise): restrict access to corporate networks if your use case allows.
- **Data Processing Agreement**: if you handle regulated data (HIPAA, GDPR, FERPA), ensure the DPA covers your use case.

---

## Things OpenAI does not currently offer (you need to build)

- **Egress allowlists on Actions.** You enforce this in your Action backend.
- **Cross-workspace anomaly detection.** Not provided; build your own if you need it.
- **Strong prompt-extraction protection on Custom GPTs.** Don't rely on it.
- **Granular per-connector scoping beyond what the source platform offers.**

---

## If something goes wrong

1. **Revoke connector permissions** in Google, Microsoft, GitHub, Slack — wherever the connector was granted.
2. **Rotate API keys** if a Custom GPT Action backend was involved.
3. **Report to OpenAI's [Bug Bounty](https://bugcrowd.com/openai)** if it looks like a platform-level vulnerability.
4. **Review Action logs** (if you run the Action backend) for anomalous calls.
5. **Pull Custom GPT from publication** if it was public, until you understand the vector.

---

## References

- [OpenAI API documentation](https://platform.openai.com/docs)
- [OpenAI enterprise privacy](https://openai.com/enterprise-privacy/)
- [OpenAI Bug Bounty program](https://bugcrowd.com/openai)
- [GPT Actions overview](https://platform.openai.com/docs/actions/introduction)
