# Customer Deployment & Operational Readiness Review

**Customer:** Lindqvist Group — Nordic industrial manufacturer, €8.2B revenue,
42,000 employees.
**System:** AI workflow automation for purchase requisitions, supplier
onboarding and access requests.
**Deployment:** customer-owned on-premises Kubernetes, no outbound internet.

## → [**review.md**](review.md) ←

The deployment review document. Nine sections, as the brief requires:

| § | Section |
|---|---|
| 1 | Deployment architecture — the proposed model, with the options rejected |
| 2 | Customer constraints — infrastructure, security, operational |
| 3 | IAM strategy — roles, permissions, access controls |
| 4 | Secrets management |
| 5 | Operational readiness — monitoring, support, incident response |
| 6 | Runbook design — contents and the principles behind them |
| 7 | Knowledge transfer plan |
| 8 | Productization opportunity |
| 9 | Risk assessment — operational and security |

---

## The two decisions that define this review

**The model runs inside the customer's datacentre.** Lindqvist permits no
outbound internet from the application tier, which removes the usual
architecture for an AI system rather than merely constraining it. Self-hosting
resolves data residency, removes the need for a vendor DPA, and eliminates the
highest-value credential in a typical AI deployment — there is no model API key
because there is no third-party model.

It also hands the customer GPU capacity planning, model-serving availability
and the model upgrade lifecycle. That trade is made because the constraint
forces it, not because self-hosting is better, and both sides of it are stated.

**The agent cannot approve anything.** Lindqvist's audit policy forbids any
single identity from both initiating and approving a financial commitment. An
AI that routes *and* approves requisitions would breach that policy and create
a fraud path.

So segregation of duties is enforced three times independently: the agent's ERP
principal lacks the release authorisation, the policy engine can only emit an
approval *request*, and the ERP rejects any approval whose approver equals the
requester. **Three layers, because one is a configuration change away from
being none** — and the third still holds if someone widens the first two.

That control is also the productization candidate. The rule "the automation
prepares, a human decides, and the system can only emit a request" is what
makes AI acceptable in any approval workflow, and every enterprise has an
equivalent policy written in a document and enforced nowhere.

---

## What it deliberately does not claim

**Prompt injection is not solved.** Input sanitisation lowers the probability
of a crafted request reaching the model with bad instructions; it does not
eliminate it. What bounds the outcome is that the agent holds no authorisation
to approve, so the worst case is a misrouted request rather than an
unauthorised commitment.

**The agent itself is not reusable.** It encodes Lindqvist's ERP, cost-centre
model and approval hierarchy. The patterns travel; the implementation would
cost the next customer a week of deletion.

**The two highest risks are not technical** — GPU capacity contention with an
existing programme, and whether the customer can actually operate the system on
day 61. The first is a written capacity agreement before build; the second is
why the handover is structured as exercises with a readiness gate rather than
training with a feedback form.
