# Local Models (Ollama, LM Studio, llama.cpp, vLLM)

## Scope

This guide covers LLMs running on your own hardware or in your own cloud — Ollama, LM Studio, llama.cpp, vLLM, text-generation-webui, and similar. The model providers listed are examples; the guidance applies to any self-hosted LLM.

---

## The good news

Local models have one major inherent security advantage: **no third-party data egress by default**. If your model runs entirely on a laptop with no external network access, an injection can't exfiltrate data to an attacker's server, because there's no network path.

You still have prompt injection risk. You just don't have automatic data-out.

---

## The bad news

Most people using local models add back the egress paths that local models would otherwise eliminate:

- They install MCP servers that make HTTP calls.
- They connect the model to a browser-automation tool.
- They plug it into Claude Code / Continue / Aider / Cursor with filesystem access.
- They give it access to email via IMAP.
- They run it inside a larger agent framework (LangChain, LlamaIndex, AutoGen) that reaches across the network.

Each of those re-introduces egress risk. If you're running a local model with none of these, you're in a very low-risk posture. If you're running a local model with all of these, you're in roughly the same risk posture as Claude or ChatGPT — just with less provider-level adversarial training protecting you.

---

## Model choice

Local models vary enormously in injection resistance. In general:

- **Larger, newer, instruction-tuned models resist injection better** than smaller, older, base models.
- **Models with explicit safety training** (Llama 3.1+ Instruct, Qwen 2.5 Instruct, Mistral Small 3) resist better than minimally-tuned models.
- **Uncensored / "abliterated" / jailbreak-stripped models resist almost nothing.** If you're using one of these, you have no prompt-level defense. You must compensate with architecture and egress controls.

For security-sensitive work, prefer recent instruction-tuned models from well-resourced labs. Do not use uncensored models for anything connected to real data.

---

## The threat surface

### Layer 1 · The runtime itself

Ollama, LM Studio, llama.cpp, vLLM — each is a piece of software running on your machine. They receive model inputs and produce outputs. Vulnerabilities in the runtime itself (CVEs in server code, buffer overflows in inference kernels) are rare but real.

**Mitigations:**

- Keep runtimes updated.
- Subscribe to their security advisory feeds.
- For Ollama specifically: the default exposes a local API at `127.0.0.1:11434`. If you expose this to the network, anyone on the network can use your model. Don't expose it unless you mean to.

### Layer 2 · The model

The model itself is the injection surface. Unlike Claude or ChatGPT, you don't have vendor-supplied adversarial training at the tier you'd want. Apply the defense recipes in [`../defenses/`](../defenses/) aggressively — you can't lean on the provider.

### Layer 3 · Tool layer

If you've bolted on tools (via MCP, LangChain, function calling, or an agent framework), this is where most of your real risk lives. Apply:

- Tool-level input validation.
- Egress allowlisting.
- Strict output schemas.
- Human-in-the-loop for any action that touches the outside world.

### Layer 4 · The surrounding agent framework

Agent frameworks like LangChain and AutoGen add their own attack surface:

- Prompt templates that concatenate user input into LLM calls without careful escaping.
- Tool descriptions that can be crafted to induce misuse.
- Memory/vector-store implementations that can be poisoned.

Audit your agent framework's security posture. Some are more mature than others. Prefer frameworks with documented security considerations over those without.

---

## Concrete recipes

### Recipe 1 · Air-gapped local model (highest security)

Use case: processing sensitive data (legal, medical, financial) that can't leave the machine.

Configuration:

- Ollama or llama.cpp running on a laptop.
- No MCP servers connected to external services.
- No network access for the model process (firewall deny-outbound for the Ollama binary).
- No tool integrations beyond strict filesystem (read-only for source data, write-only for a single output folder).
- Strong system prompt with all the hardening patterns from [`../defenses/02-system-prompt-hardening.md`](../defenses/02-system-prompt-hardening.md).

Residual risk: almost entirely limited to "the model produces bad output." Injection can't exfiltrate because nothing can leave.

### Recipe 2 · Local model with light connectors (medium security)

Use case: personal productivity assistant that reads your local files, maybe your calendar.

Configuration:

- Ollama with local filesystem MCP server, scoped to a specific folder.
- Local calendar MCP server, read-only.
- No email connector.
- No browser or web-fetch connector.
- Egress firewall rules allow Ollama to reach its own update server only.

Residual risk: moderate. An injection in a file you're processing could make the model misbehave, but can't exfiltrate beyond the local filesystem scope.

### Recipe 3 · Local model in an agent framework (comparable to cloud risk)

Use case: autonomous agent running tasks, possibly including web browsing, email sending, code execution.

Configuration: apply **every** defense recipe. This configuration is only safer than Claude/ChatGPT if you've actually hardened it — and is often less safe, because you don't have provider-level adversarial training.

- Strict system prompt (see `defenses/02`).
- Architecture-level content filtering on ingested data.
- Egress allowlist enforced at the OS firewall level.
- Human-in-the-loop on every external write action.
- Audit logging of all tool calls.
- Regular self-testing with the tool in [`../test/`](../test/).

Residual risk: comparable to a well-configured cloud deployment. The local model's lack of provider-trained injection defenses needs to be compensated with stronger architecture.

---

## Specific runtime notes

### Ollama

- Default bind: `127.0.0.1:11434`. Don't change this without thinking.
- Expose via `OLLAMA_HOST=0.0.0.0` only if you need network access AND you have authentication / network ACLs in place.
- `ollama run <model>` loads models from Ollama's registry. Models are not signed. Pin model hashes or download only from trusted model publishers.

### LM Studio

- Similar to Ollama, with a GUI.
- Local API endpoint; same local-bind considerations.
- Model library is community-curated; verify model provenance before loading for sensitive work.

### llama.cpp

- Raw inference engine. You control everything.
- No API exposure unless you build one yourself.
- Highest-flexibility, highest-effort option.

### vLLM

- Production inference server, typically used in team / server deployments.
- Has a proper HTTP API with authentication support — turn it on.
- Can be exposed to your organization's network with appropriate ACLs.

---

## Audit checklist for local deployments

- [ ] Model runtime is running on a named user account (not root).
- [ ] Model runtime's outbound network access is firewalled to an explicit allowlist.
- [ ] No API exposure beyond the interfaces you intend.
- [ ] Agent framework / tool layer has been audited for its own security considerations.
- [ ] Every tool the model can call enforces its own security contract in code.
- [ ] System prompt follows the hardening patterns.
- [ ] Self-test in this kit has been run against your specific system prompt.
- [ ] Audit logs exist for tool calls and are retained for at least 30 days.

---

## References

- [Ollama security documentation](https://github.com/ollama/ollama/blob/main/docs/faq.md)
- [OWASP LLM Top 10 applied to self-hosted models](https://genai.owasp.org/) — same risks apply, minus the provider-side protections.
- [NIST AI RMF — self-hosted AI considerations](https://www.nist.gov/itl/ai-risk-management-framework)
