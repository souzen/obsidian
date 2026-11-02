# Success Signals: Results, Not Effort — Reference

Sources:
- https://www.kpi.zone/understanding-input-and-output-metrics/ (KPI Zone, 2025)
- https://bharatsharma.pro/articles/activity-based-engineering-metrics-are-obsolete (Bharat Sharma, 2026)

Vault notes: `work/resources/product-mgmt/kpi-input-vs-output.md`, `inbox/ai-is-making-activity-based-engineering-metrics-obsolete.md`

`outcomes-over-outputs.md` tests the outcome statement. This file tests the **Success signal** under it: what does it actually measure?

---

## Input vs Output Metrics

- **Input metric** — resources and effort put in: money, hours, headcount, tasks done, features shipped, calls made. Shows how efficiently resources are used.
- **Output metric** — the result the process produces for the beneficiary: sales, incident rate, resolution time, satisfaction. Shows whether the goal is met.

Recipe analogy: inputs are the ingredients, the output is the finished dish. A Success signal should describe the dish.

Not the same as leading/lagging. Input/output is about *what* is measured; leading/lagging is about *timing*. Sales calls made = input and leading; revenue = output and lagging. A metric can be an output and still leading (e.g. onboarding completion rate predicting retention).

---

## Red Flags: the Signal Is Effort

- Counts delivery or activity: "20 training hours completed", "5 features shipped", "all services migrated"
- Counts spend: "€50k invested in tooling"
- Can be met while the beneficiary notices nothing
- Engineering activity proxies: lines of code, commit count, PR count, story points — these were always weak and become misleading when AI tools inflate them (AI-assisted PRs are larger, commit rates 3–4× higher, with more rework and security defects)

## Green Flags: the Signal Is a Result

- Measured on the beneficiary or the system's behaviour: time to detect, rework rate, tickets resolved without escalation, change failure rate
- Has a before/after or baseline to compare against
- Goes down or up only if the work actually helped

| Input / activity signal (❌) | Result signal (✅) |
|---|---|
| 20 training hours completed | Customer service ratings rise from 3.8 to 4.3 |
| 5 features shipped this quarter | New users complete setup without contacting support (≥ 80%) |
| 40 PRs merged per sprint | Lead time from first commit to production under 2 days |
| Migration of 12 services done | DB-related incidents drop to 0 per month |
| Weekly sync held with the vendor | Vendor tickets answered within SLA (≥ 95%) |

---

## Optional: Add a Driver (Leading Input Metric)

Outputs lag. A project can run for months before the result moves. One clearly labelled **driver** — an input or leading metric believed to cause the result — lets the owner see early whether the work is on track:

- Pick the output metric first, then name the input that drives it (not the other way round)
- Keep it to one, maybe two. Only add it when the user supplies or confirms it — do not invent one
- Review the driver regularly. If the driver is healthy but the output lags, the assumed link is wrong; change the approach (e.g. heavy training spend, low service ratings → fix the training method)
- Balance both: outputs alone can hide inefficiency; inputs alone hide whether anything improved

---

## Engineering Projects: Diagnostics Worth Suggesting

When the project is about engineering productivity, AI-assisted development or delivery, prefer system-level result signals over activity counts:

- Delivery outcomes (DORA-style): lead time, deployment frequency, change failure rate, time to restore
- Diagnostics: review queue time, 14-day rework as a share of lines introduced, validation / pipeline failure rate, post-deployment rework
- Trend against a pre-change baseline (six months of telemetry if available), not against an absolute target

## Goodhart's Law

When a measure becomes a target it stops being a good measure. Guardrails to mention when drafting signals:

- Keep engineering metrics diagnostic at team level or above — never as individual performance targets
- Pair a throughput metric with a quality metric (e.g. lead time + change failure rate) so neither can be gamed alone
- Tie the signal to a business outcome, not to the metric's own movement

---

## Interview Questions (add to Phase 2c)

- "If this number moved, would the beneficiary notice? What would change for them?"
- "Is this measuring what we put in, or what came out?"
- "Is there an early indicator we can watch before the result shows up?"
- "How could this metric be met while the result stays the same?" (Goodhart check)
