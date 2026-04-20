# Security Policy

## Reporting a security issue in this kit

If you find a security issue in this kit itself — for example, a defense that is actively harmful, test code that leaks credentials, or a config template that recommends an insecure practice — please report it before opening a public issue.

**Preferred:** email `joshua@mtm.now` with the subject line `[pentest-kit security]` and a short description.

Please do not:
- Open a public GitHub issue for security-sensitive findings.
- Demonstrate the issue against third-party AI providers or production systems that aren't yours.

You will get a reply within five business days, usually faster.

## What is not in scope

This kit is guidance. It is not a production security product. The following are explicitly out of scope for security reports:

- Vulnerabilities in Claude, ChatGPT, Gemini, or other AI providers themselves. Report those to the vendor's official security program (e.g., [Anthropic's Responsible Disclosure Program](https://www.anthropic.com/responsible-disclosure-policy)).
- Vulnerabilities in third-party MCP servers, plugins, or connectors. Report those to the server's maintainer.
- Disagreements with the advice in a defense recipe. Those should go in a regular GitHub issue so they can be discussed publicly.

## Responsible use

The self-test runner in [`test/`](test/) is designed to probe AI assistants you own or have explicit authorization to test. It uses synthetic canary values — no real credentials are embedded — and targets a system prompt you construct yourself. Do not use it to test systems you do not own or have authorization to test.
