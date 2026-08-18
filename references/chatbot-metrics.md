# Chatbot Metrics Reference

Use this reference only when the task defines, analyzes, reports, or evaluates DLA/TLA, Chatbot adoption, self-service, deflection, resolution, answer quality, or Optimization-plan impact. Do not load it for ordinary row-by-row annotation.

## Contents

- [Service journey and metric boundaries](#service-journey-and-metric-boundaries)
- [Core definitions](#core-definitions)
- [Measurement contract](#measurement-contract)
- [Relating annotation results to metrics](#relating-annotation-results-to-metrics)
- [Action-specific evaluation](#action-specific-evaluation)
- [Reporting safeguards](#reporting-safeguards)

## Service journey and metric boundaries

The Chatbot serves cross-border sellers through seller-facing entry points and should:

- enable self-service where the Chatbot has sufficient knowledge or executable capability;
- accurately clarify, route, or transfer when human intervention is genuinely required;
- avoid presenting necessary human handling as a Chatbot failure by default.

Analyze the journey as:

`seller issue → entry point → Chatbot interaction/answer → self-service outcome or live-agent handling`

Keep traffic entry, intent recognition, answer quality, executable capability, human routing, and final outcome as separate stages. A single rate cannot diagnose all of them.

## Core definitions

### DLA

`DLA` means a live-agent contact with no effective Chatbot interaction before human handling.

Do not assume every DLA session could have been prevented. Separate at least:

- knowledge-answerable seller questions;
- object-level queries that may need API/Taskflow;
- inherently human or permission-bound execution;
- incomplete/conflicting cases requiring confirmation;
- entry, recognition, answer-quality, or product-experience problems.

### TLA

`TLA` means the seller interacted with the Chatbot and then reached a live agent.

Do not mix TLA with DLA. TLA may reflect a knowledge gap, failed recognition, poor answer, failed API flow, expected escalation, seller preference, or other causes that require separate evidence.

### Other metric families

Relevant dashboard or operational metrics may include:

- total chat sessions and Chatbot sessions;
- Chatbot adoption;
- deflection;
- bot resolution;
- service CSAT;
- solution-level bad rating;
- IDK or no-answer rate;
- intent/answer accuracy;
- dropped or abandoned sessions;
- API/Taskflow completion and failure outcomes;
- handoff quality and repeat contacts.

The documented explicit adoption formula is:

`Chatbot Adoption = Chatbot Sessions / Chat Sessions`

For every other metric, use the exact numerator, denominator, eligibility rules, exclusions, and attribution window defined by the current source or dashboard. Do not invent a formula from the metric name.

## Measurement contract

Before calculating or comparing any metric, state:

- data source and extraction/version date;
- analysis period and timezone;
- business, site, seller segment, language, channel, RC/L3 reason, and entry-point scope;
- unit of analysis: session, seller, intent, issue, solution, or action;
- numerator and denominator;
- eligibility and exclusions, including empty/duplicate sessions;
- DLA/TLA definition used by that source;
- how multi-intent sessions and repeated sellers are counted;
- whether the result is observed, inferred, or a potential opportunity estimate.

If the source does not provide the denominator or metric logic, label it `待确认`; do not backfill it from prior work.

## Relating annotation results to metrics

The conversation workflow produces capability evidence, not automatic metric outcomes:

- `QA` shows a static-answer self-service opportunity only after source-supported content and matching scope are confirmed.
- `API/Taskflow` shows an SOP-based executable self-service opportunity only after the required backend query/action, variables, and routing are feasible and implemented.
- `人工承接/流程自动化计划` may improve routing, preparation, or handling efficiency even when live-agent contact remains necessary.
- Pending confirmation is a dependency or unresolved-pool status, not an action type. It remains outside confirmed reducible opportunity until its missing rule, owner, mapping, or capability is resolved.

Human-intervention and knowledge-reusability labels are analysis dimensions. They must not be used alone to claim:

- a session will be deflected;
- DLA or TLA will fall;
- the Chatbot can resolve the case;
- an API exists;
- a human handoff is avoidable.

Similarly:

- `完全覆盖` measures configured knowledge coverage for the assessed variant, not adoption, recognition, seller acceptance, resolution, or deflection.
- a no-additional-human-action pool is an upper-bound opportunity pool, not a guaranteed reducible-DLA rate.
- cluster frequency is demand evidence, not proof of implementation value without feasibility and outcome checks.

## Action-specific evaluation

Choose metrics according to the action, and confirm exact definitions in the current source.

### Knowledge/SOP

Evaluate where available:

- matching accuracy for supported seller questions and nearby negative cases;
- IDK/no-answer and wrong-intent behavior;
- solution bad rating or answer helpfulness;
- bot resolution, TLA, repeat-contact, and link/step completion signals;
- coverage of the approved source variants.

### API/Taskflow

Evaluate where available:

- eligible entries and flow starts;
- valid-input, API success, empty-result, error, timeout, and fallback outcomes;
- end-to-end task completion and bot resolution;
- unnecessary transfer reduction without hiding failed executions;
- incorrect or unsafe completion claims.

### Human handoff/process automation

Evaluate where available:

- correct handoff destination and reason;
- required-material completeness before transfer;
- successful case creation, escalation, or submission when the flow supports it;
- handling time, repeat contacts, and seller notification completion;
- CSAT without assuming that retaining human handling is a failure.

### Pending confirmation

Track owner, open question, sample size, conflict type, and confirmation status. Do not include pending items in confirmed coverage or reducible-opportunity totals unless the reporting view labels them separately.

## Reporting safeguards

- Keep DLA and TLA separate in tables, denominators, conclusions, and actions.
- State whether counts are unique sessions, intent occurrences, cluster memberships, or unique sellers.
- A multi-intent session may enter multiple clusters, but unique-session totals must deduplicate the session ID.
- State coverage scope: all sessions, sampled sessions, only knowledge variants, or only evaluated high-frequency clusters.
- Distinguish `未评估` from `未覆盖`.
- Distinguish configured capability, tested capability, published capability, seller use, successful completion, and measured outcome.
- Separate facts, source-grounded inferences, hypotheses, and recommended experiments.
- Do not present an Optimization-plan count or percentage as realized DLA/TLA reduction.
- When comparing periods, keep metric definitions, exclusions, and scope constant or disclose every change.
- For management summaries, make the action definition explicit: what was changed, for whom, in which channel, and what outcome window will be evaluated.
