# Outcomes over Outputs — Reference

Source: https://martinfowler.com/bliki/OutcomeOverOutput.html

---

## Core Principle

An **output** is something the team *delivers* — a feature, a dashboard, a migration, a refactor.

An **outcome** is a *change in behavior* of a user, employee, or customer that drives value for the organization.

> "Focusing on outcomes, rather than output, favors building features that do more to improve the effectiveness of the software's users and customers."

---

## The Seiden Definition

An outcome = **a change in behavior of a user, employee, or customer that drives a good thing for the organization.**

Two levels:
- **Outcome** — behavioral change that is observable (easier to measure)
- **Impact** — broader organizational effect (harder to apportion)

Customer value outcomes (e.g. reliability) > Business value outcomes (e.g. cost savings).

---

## Output vs Outcome — Pattern Recognition

### Red flags (these are OUTPUTS disguised as outcomes):
- Starts with a verb about shipping: "Deploy", "Build", "Release", "Migrate", "Implement", "Refactor"
- Describes what the team does, not what changes for the user
- No measurable signal
- Could be marked done even if nobody benefits

### Green flags (these are real OUTCOMES):
- Names a person or role who is affected: "Engineers", "Support team", "Customers", "Managers"
- Describes a behavioral change: "can do X without Y", "no longer need to", "resolve in N minutes instead of N hours"
- Has a measurable signal: a number, a rate, a before/after comparison
- Survives the "so what?" test: you can explain why this matters to the business

---

## Outcome Formula

```
[WHO] [can / will / no longer] [BEHAVIOR] [measurable signal or observable change]
```

### Examples

| Output (❌) | Outcome (✅) |
|---|---|
| Deploy monitoring dashboard | On-call engineers detect incidents 30% faster |
| Add export button to reports | Finance team exports monthly reports without filing a dev ticket |
| Migrate DB to Postgres | DB-related incidents drop to 0 per month |
| Refactor auth module | Users no longer get logged out during active sessions |
| Build CI/CD pipeline | Developers merge PRs in under 10 minutes with zero flaky builds |
| Implement AI ticket classification | Support team resolves L1 tickets without engineering escalation |
| Release new onboarding flow | New users complete setup without contacting support |

---

## Outcome Quality Checklist

When evaluating an outcome statement, ask:

1. **Who is the beneficiary?** (user, customer, employee, team)
2. **What do they do differently** after this project succeeds?
3. **How would we know it worked?** (observable signal, metric, before/after)
4. **Does it survive "so what"?** Can you explain why this matters to the business?
5. **Is it a behavioral change**, not just a feature being shipped?

---

## Common Mistakes

- Writing a task as an outcome: *"Deliver observability tooling"* — this is still a deliverable
- Writing an impact as an outcome: *"Improve company resilience"* — too abstract, not observable
- Writing an output metric as an outcome: *"Ship 5 features this quarter"* — this is measuring output
- Passive voice hiding the absence of a beneficiary: *"Incidents will be reduced"* — by whom? observed by whom?

---

## Interview Questions to Elicit Good Outcomes

Use these during Phase 2 (outcomes review) to help the user articulate real outcomes:

1. "Who is the primary user or stakeholder this project serves?"
2. "When this project is done and successful — what will they be able to do that they can't do today?"
3. "What will they stop doing, or stop experiencing, that's currently painful?"
4. "How would you know in 3 months that this project worked? What would you observe?"
5. "Is there a number, a rate, or a before/after comparison that would tell the story?"
6. "What's the cost to the business if this project doesn't happen?"
