# Chatbot KB Management Reference

Use this reference only when the task moves from conversation analysis into Chatbot knowledge-base design, implementation, review, testing, approval, or publishing. It describes the management model and authoring controls; it is not evidence for seller-facing business rules.

## Contents

- [Authority and source boundaries](#authority-and-source-boundaries)
- [Knowledge-base models](#knowledge-base-models)
- [Issue design and intent detection](#issue-design-and-intent-detection)
- [Solution design and capability routing](#solution-design-and-capability-routing)
- [Authoring and presentation controls](#authoring-and-presentation-controls)
- [Testing, approval, publishing, and retirement](#testing-approval-publishing-and-retirement)
- [Auxiliary traffic and routing controls](#auxiliary-traffic-and-routing-controls)
- [Optimization-plan implementation checklist](#optimization-plan-implementation-checklist)

## Authority and source boundaries

- Treat the current user-provided KB export, admin view, issue table, and solution table as authoritative for what is configured now.
- Treat complete `msg_data` conversations as authoritative for what the human agent actually told or did for the seller.
- Treat solution `message_content`, including retained SEH links, as authoritative for the currently configured Chatbot answer.
- Use this reference to select the correct KB object and workflow only. Never use it to invent business rules, thresholds, materials, steps, API fields, or outcomes absent from the source evidence.
- Ask the user for the current KB source and output location. Do not embed or reuse historical file paths, spreadsheet IDs, tab names, or row ranges.

## Knowledge-base models

### Legacy Intent KB

The legacy model keeps recognition and answering mainly under one Intent. Typical objects and controls include:

- Intent ID and Intent Name;
- Main Question and Similar Questions;
- Description, Reason Code, related article/policy, category, and effective scope;
- Hot Question, Shortcut, Answer, and FAQ recommender;
- answer forms such as rich text, key answer, article, Taskbot, live-agent route, and conditional answer.

Use the legacy vocabulary only when the current source is still managed as Intent KB. Confirm the actual admin object before proposing changes.

### Smart KB

Smart KB separates recognition from answering:

- **Issue** describes the seller problem and supports intent matching.
- **Solution** contains the answer or executable flow shown after the Issue is matched.
- An Issue binds to one Solution in the documented model. Confirm the current export and admin state before editing mappings.

The same Optimization-plan action may therefore require coordinated changes to both objects:

1. expand or narrow Issue recognition only for the newly supported seller conditions;
2. update or create the corresponding Solution capability;
3. test the Issue-to-Solution mapping and every relevant branch;
4. publish only after approval.

## Issue design and intent detection

### Detection inputs

Issue matching can be affected by:

- English and local Issue names;
- Issue Description;
- Similar Questions;
- Solution content;
- configured definitions, business scope, key information, conditions, and clarification logic.

Do not update only the Solution when the newly added branch changes which seller questions should match the Issue. Add a row-specific Issue Description delta in the Optimization plan and implement it only if that action is approved.

### Issue Description structure

Use exactly two sections:

1. `核心定义（Core Definition）`: state the seller need covered by the Issue.
2. `适用范围 / 触发条件（Applicability / Conditions）`: list the concrete seller questions and conditions that should trigger it.

Rules:

- Describe recognition scope, not detailed answer steps.
- Include only conditions supported by the approved action and its evidence.
- Make each row's suggestion incremental. Do not pre-combine unimplemented actions into a final Issue Description.
- For a new Issue, propose a complete initial two-section description only when the evidence supports a stable scope.
- Keep unrelated intents separate when they can be answered independently, have different conditions, or need different capabilities.

### Similar Questions and Main Questions

- Write from the seller's natural phrasing and cover stable variants of the same intent.
- Keep questions concise, unambiguous, and free of order IDs, waybill IDs, shop IDs, personal data, or other case-specific identifiers.
- Do not turn different answers or different capability routes into superficial paraphrases of one question.
- Use historical source questions for regression tests, but anonymize them before reuse.

## Solution design and capability routing

### QA Solution

Use a QA Solution for a direct, stable answer that does not require a complex multi-step flow. Confirm the current fields in the KB source, including Solution Group, bound Issue, Solution name, internal description, answer type, configured content, and version.

Possible answer implementations include static text and supported function-based content. A function capability must not be implied merely because an Optimization-plan row recommends an API.

### SOP Solution

Use an SOP Solution when the seller journey needs conditions, clarification, variables, API/function calls, multi-step self-service, or an explicit handoff branch. Common canvas elements include:

- Chatbot or agent entry/role;
- Answer nodes;
- Condition nodes;
- Clarification nodes;
- Function/API, formula, variable, or code nodes;
- live-agent or fallback branches.

Map Optimization-plan action types as follows:

- `知识/SOP`: update an existing QA/SOP Solution or create a new one only after confirming the correct Issue and destination.
- `API/Taskflow`: design a supported SOP/function flow with source-grounded triggers, inputs, returned states, success response, failure response, and handoff. Treat the row as a requirement until the capability is confirmed and implemented.
- `人工承接/流程自动化`: configure a transparent handoff or process branch with only source-supported materials, destination, timing, and notification details.
- `待确认`: do not publish speculative content; resolve the named owner question first.

### Conditions and clarification

- Use a Condition when a known field deterministically changes the answer or route.
- Use Clarification when the seller must provide or select information before the system can choose the correct branch.
- Clarification may rely on supported identifiers, variables, or APIs, but only after the current capability is verified.
- Do not create an API requirement merely to identify the fulfillment type when the agent only confirms `3PF`, `SLS`, `Local`, `Cross-border`, `Pick-up`, or `Drop-off` and then gives a general rule. Keep that type as an applicability condition.

### Chatbot and human-agent roles

- Write Chatbot-facing answers directly to the seller. Do not narrate them as `客服回复`, `专员表示`, or another internal evidence voice.
- Keep human-agent scripts separate when the node is specifically for an agent.
- A Chatbot may claim a lookup, submission, escalation, or expedite only when the configured flow actually executed it and returned that result.
- If execution is unavailable, explain the supported preparation and handoff path without pretending completion.

## Authoring and presentation controls

Apply the current product limits shown in the admin interface. The documented writing guidance includes:

- Put UI button names in `【】`.
- Keep answer bubbles concise; the legacy guidance targets about 200–220 Chinese characters per bubble and no more than three bubbles.
- Confirm Smart-KB node limits in the current interface; documented examples include up to four QA answer bubbles and up to five SOP list items/nodes in relevant controls.
- Replace overly long raw links or titles with a concise clickable label such as `点击这里` when supported.
- For more than three cross-page steps, consider an image or GIF only when it is current and maintainable.
- Remove filler greetings when they reduce clarity; preserve a direct, seller-facing role.
- Never include real seller, order, shop, product, contact, address, account, or chat identifiers in reusable content.

These are presentation controls, not permission to add missing operational details.

## Testing, approval, publishing, and retirement

Treat an Optimization-plan row as a proposed change until the full lifecycle is completed:

1. confirm the target Issue, Solution, version, effective scope, and owner;
2. edit or create the draft;
3. test matching and answer behavior;
4. submit for approval when required;
5. publish or republish the approved version;
6. verify the live result and retain traceability to the source sessions.

Testing should cover:

- Issue Name, Similar Questions, Issue Description, and historical seller phrasings;
- positive matches and nearby negative cases;
- every condition and clarification branch;
- function/API success, empty result, invalid input, timeout, and fallback where supported;
- Chatbot and human-agent variants;
- links, effective scope, formatting, and version selection;
- regression against the evidence sessions used in the Optimization plan.

Editing a published object may create a new draft or require republishing. Do not report a change as live based only on a saved draft or connector success. Prefer retirement/deactivation over deletion when auditability or rollback matters, subject to the current admin controls.

## Auxiliary traffic and routing controls

Use these only when the task explicitly requires them:

- **Hot Questions / Shortcuts**: improve discoverability for frequent seller needs; they do not replace Issue matching quality.
- **Announcements**: communicate temporary or urgent information; confirm time scope and removal owner.
- **RBE or deterministic keyword rules**: reserve for strong, low-ambiguity patterns. Do not use broad keywords that may hijack unrelated intents.
- **Effective Scope**: constrain site, market, seller segment, business, language, or channel only when the current product supports and the source evidence requires it.
- **Knowledge operations / update workflows**: use for bulk maintenance, policy refreshes, and version control, while preserving approvals and regression checks.

## Optimization-plan implementation checklist

Before closing an implementation task, verify:

- the action row was approved and mapped to the correct current KB object;
- the Issue delta matches only the new Solution capability;
- the Solution contains no facts beyond the approved source-grounded answer;
- independent intents are not collapsed into one Issue merely because they occurred in one session;
- QA versus SOP Solution choice reflects the actual needed capability;
- API/function and handoff claims match configured behavior;
- seller-facing and internal-agent wording are separated;
- positive, negative, branch, fallback, link, and role tests pass;
- approval, version, publish state, effective scope, and live verification are recorded;
- the evidence source rows and session IDs remain traceable.
