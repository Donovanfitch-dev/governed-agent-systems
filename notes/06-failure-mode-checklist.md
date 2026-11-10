# 6. A failure-mode checklist

A review list for a system in which an AI agent can take actions. Each row is a way such systems
fail, the question that exposes it, and the control that addresses it. None of this is specific to
any model.

## Deciding

| Failure mode | Ask | Control |
| --- | --- | --- |
| The agent acts on its own description of the action | Is the reviewer shown the exact payload that will execute? | Proposal as a durable record; execution bound to it ([note 2](02-approval-is-a-state.md)) |
| The proposer approves itself | Can one identity both propose and approve? | Separation of duties checked on the record |
| Every action needs review, so review becomes reflex | What share of approvals are instant? | Risk-tiered policy; review only what is consequential |
| An old approval is used for a changed situation | Do pending proposals expire? | Expiry; re-evaluate policy on edit |
| Instructions arrive inside retrieved content | Can text from a document or web page change what the agent is permitted to do? | Permissions are decided outside the agent; retrieved content is data |

## Executing

| Failure mode | Ask | Control |
| --- | --- | --- |
| A dead worker holds a task forever | What releases a task whose worker vanished? | Leases with expiry ([note 3](03-leases-and-late-workers.md)) |
| A slow worker and its replacement both finish | What stops the first worker's late write? | Fencing tokens |
| A retry repeats a side effect | What happens if this exact request is sent twice? | Idempotency keys on every effect |
| The result is recorded but never reported, or reported twice | Is the notification in the same transaction as the state change? | Transactional outbox; idempotent receiver |
| A retried job redoes expensive work | Where does the second attempt start? | Checkpoints |
| A task fails forever and blocks the queue | Where does a task go after its last attempt? | Bounded retries; dead-letter with a reason |

## Knowing

| Failure mode | Ask | Control |
| --- | --- | --- |
| "Done" means "no error was raised" | What positive artifact proves the outcome? | Success defined by validated evidence ([note 1](01-three-answers-not-two.md)) |
| A check that did not run counts as passing | Can this status be `unknown`? | Three-valued status |
| Evidence is about the wrong version | Does each observation name its subject? | Bind evidence to revision or deployment |
| Evidence is old | Who decides how old is too old? | Freshness limits in policy |
| Nobody can reconstruct why something happened | Can I list every state change for this item, in order? | Append-only event log |
| The log could have been edited | Would a changed entry be detectable? | Hash chaining; external anchor for stronger guarantees |

## Changing

| Failure mode | Ask | Control |
| --- | --- | --- |
| A model swap changes behavior elsewhere | Which requests would route differently after this change? | Routing as policy; replay diff ([note 4](04-model-choice-is-policy.md)) |
| A provider outage becomes an application outage | What happens on the third consecutive timeout? | Fallback chain; circuit breaker |
| Spend grows without anyone choosing it | What rejects a request that exceeds its budget? | Per-tenant budgets checked before execution |
| A new capability goes live untested | Was it observed in shadow before it could act? | Gated phases; enabling is a separate, recorded change ([note 5](05-gated-phases.md)) |
| A waiver becomes permanent | Does every exception have an owner and an end date? | Time-boxed, revision-bound waivers |

## Using the list

Go through it for one concrete action the system can take, not for the system in general. "What
happens if the refund request is sent twice" has an answer that can be tested. "Is the system
idempotent" does not.
