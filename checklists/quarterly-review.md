# Quarterly Review Checklist

AI products, provider platforms, and attack techniques all move fast. A deployment that was secure in Q1 may not be secure in Q3, not because anyone made a mistake but because the landscape shifted.

This checklist is designed to take 30–90 minutes per deployment, run once a quarter. If it takes longer, your deployment has drifted and you're catching it late.

---

## 1 · Access review

- [ ] List every OAuth grant the AI currently has. Revoke any that are unused, dormant, or no longer needed.
- [ ] List every MCP server installed. Remove any you don't actively use.
- [ ] List every tool or skill enabled. Disable anything unused.
- [ ] Review the user list (for Team/Enterprise deployments). Off-board anyone who's left.

## 2 · Configuration drift

- [ ] Compare the current system prompt to the one you last tested. Has it changed? Why?
- [ ] Compare the current MCP tool descriptions to your baseline (run `test/pentest.py` infra check). Any drift is a signal — known as a "rug pull" if the change is malicious.
- [ ] Compare current connector permissions to what was last approved. Google, Microsoft, GitHub, etc. can quietly expand scopes on updates.

## 3 · New attacks

- [ ] Check [OWASP Gen AI](https://genai.owasp.org/) for new entries or material revisions to the Top 10.
- [ ] Check Anthropic's [news](https://www.anthropic.com/news), OpenAI's [security blog](https://openai.com/security/), and Google's [Vertex AI updates](https://cloud.google.com/vertex-ai/docs/release-notes) for disclosed vulnerabilities or new protection features.
- [ ] Check [Simon Willison's blog](https://simonwillison.net/) — he tracks real-world injection incidents better than most security press.
- [ ] Add any new relevant attack class to your next self-test run.

## 4 · Model drift

- [ ] Has the model version changed? Providers update models on their own cadence, sometimes silently. If the model changed, re-test.
- [ ] If you pinned a specific model version and the provider has deprecated it, plan the migration now, not when the deprecation hits.
- [ ] If a newer, meaningfully more injection-resistant model is available (e.g., Claude Opus 4.5+ over Claude Sonnet 3.7), evaluate whether to switch for security-sensitive deployments.

## 5 · Self-test

- [ ] Re-run the full self-test in [`../test/`](../test/).
- [ ] Compare results to last quarter's run.
- [ ] Any new failures? Investigate immediately.
- [ ] Any failures resolved? Document the fix.
- [ ] Save the report with the date.

## 6 · Log review

- [ ] Pull 30–90 days of logs for this deployment.
- [ ] Scan for: unexpected egress destinations, anomalous tool-call patterns, unusual output sizes, suspicious user inputs.
- [ ] Note any patterns for further investigation.
- [ ] Confirm retention is working — logs you expect to be present actually are.

## 7 · Incident-readiness

- [ ] Test your revocation process. Can you actually revoke connector tokens quickly? When did you last try?
- [ ] Confirm your incident contact list is current.
- [ ] If you had any near-misses or minor incidents this quarter, write them up briefly. The pattern matters more than any single incident.

## 8 · User behavior

- [ ] How are users actually using this AI? Match intended use vs actual use. Drift is common.
- [ ] Are users connecting new documents, inboxes, or data sources? Should they be?
- [ ] Has anyone reported odd behavior? Follow up on any that were dismissed at the time.

## 9 · Provider policy and platform changes

- [ ] Read the provider's most recent policy / terms updates.
- [ ] Note any new features you should enable.
- [ ] Note any features that were deprecated or removed.

## 10 · Documentation refresh

- [ ] Update your threat model if it changed.
- [ ] Update the pre-deployment checklist with anything you've learned.
- [ ] Share notable findings with anyone else in the organization who owns AI deployments.

---

## Output of the review

A short document, one page or less, that answers:

1. What changed this quarter?
2. What's the current risk posture?
3. What's on the list to fix before next quarter?
4. Anything that needs leadership attention?

Archive these documents year-over-year. The delta across quarters is more valuable than any single report.

---

## When to review more often than quarterly

- You deploy a significant new connector or MCP server. Do a mini-review, not a full quarterly.
- The model version changed. Re-run the self-test.
- You've had an incident. Do a full review within a week, not at the next quarter boundary.
- A major vulnerability is disclosed in your stack. Address immediately; don't wait for the schedule.
