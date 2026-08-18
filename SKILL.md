---
name: lff-3pf-chatbot-optimization
description: Source-grounded, row-identity-locked two-stage workflow for LFF/3PF seller-service conversations: annotate complete sessions without Sheet row drift, preserve multiple intents in one source row, cluster every valid seller question before filtering, distinguish QA answers from SOP-based API/Taskflow and executed handling, optionally assess Chatbot coverage from full solution message_content including SEH links, and produce prioritized QA, API/Taskflow, and human-handoff actions with solution-specific issue-description deltas and explicit dependencies. Use for LFF/3PF Google Sheets annotation, continuation or recheck tasks, high-frequency issue analysis, coverage measurement, API-capability identification, QA/SOP mapping, issue-description maintenance, Optimization plan generation or restructuring, Smart-KB implementation planning, and DLA/TLA or self-service impact evaluation.
---

# LFF / 3PF Chatbot Optimization

## Load the required references

- Before annotating, reviewing, continuing, or auditing conversations, read [references/annotation-rules.md](references/annotation-rules.md) completely.
- Before clustering, assessing coverage, prioritizing actions, creating/restructuring an Optimization plan, or maintaining issue descriptions, read [references/optimization-plan.md](references/optimization-plan.md) completely.
- For an end-to-end request, read both references before acting.
- Before designing, implementing, or reviewing changes to Issue, Solution, QA Solution, SOP Solution, Function, Condition, Clarification, Taskbot, publishing, or testing, read [references/chatbot-kb-management.md](references/chatbot-kb-management.md) completely.
- Before defining or reporting DLA/TLA, Chatbot adoption, self-service, deflection, resolution, answer-quality metrics, or Optimization-plan impact, read [references/chatbot-metrics.md](references/chatbot-metrics.md) completely.
- Do not load the KB-management or metrics reference for ordinary row-by-row annotation unless the current request also includes the corresponding implementation or measurement work.
- When editing Google Sheets, also use the available Google Sheets skill and follow its metadata-first, bounded-read, batch-write, and re-read verification workflow.
- If a workspace rule file is supplied, read it first and treat it plus the user's current instructions as authoritative over bundled references.

## Establish the workbook contract

1. Never reuse or infer a conversation-record path, spreadsheet ID, URL, tab name, row range, annotation destination, solution source, issue source, or Optimization-plan destination from a previous task, bundled example, or local workspace file.
2. Before reading or writing task data, obtain or explicitly confirm the current task's locations from the user:
   - Conversation input: file path or spreadsheet link/ID, exact tab/sheet, and requested physical row range.
   - Annotation output: destination file/spreadsheet, exact tab/sheet, and the target annotation fields/columns; confirm whether this is the conversation source tab or a different destination.
   - Coverage input, when requested: the current Chatbot solution file/spreadsheet and exact tab/sheet.
   - Issue mapping input, when requested: the issue-and-solution file/spreadsheet and exact tab/sheet.
   - Optimization-plan output, when requested: destination file/spreadsheet and exact existing tab/sheet, or explicit permission plus the new tab/sheet name to create.
3. If any location required for the requested action is missing or ambiguous, ask the user for it before accessing or mutating data. Do not substitute a path used in an earlier conversation.
4. After the user supplies the contract, resolve the spreadsheet ID, exact tab names, `sheetId` values, visible headers, requested row range, and destination before writing.
5. Locate fields by header name, not historical column letter. Expect at minimum:
   - Conversation source: `live_agent_chat_session_id`, `msg_data`, L3 reason, and source row.
   - Chatbot source: `solution_id`, `solution_name`, `solution_type_name`, `node_id`, `node_name`, `answer_branch_name`, and `message_content`; use `message_content_plaintext` only as a reading aid. Resolve these by header name, not historical column letter.
   - Issue mapping source: the user-specified issue-and-solution tab, with `issue_id`, `issue_name`/`issue_local_name`, `issue_description`, and SOP/taskbot ID.
6. Preserve unrelated data, formulas, validation, formatting, filters, and merged ranges.

## Lock physical row identity before every Sheet write

Treat the Google Sheet 1-based physical row number as an immutable primary key. Never derive a write row from a filtered-array index, valid-session count, batch index, loop index, or visible filtered order.

1. Create one manifest entry for every physical row in the requested batch with `source_sheet_id`, `source_row`, `live_agent_chat_session_id`, `msg_data_present`, `annotation_fields`, and `row_status` (`ANNOTATE`, `SKIP_EMPTY`, or `SKIP_DUPLICATE`).
2. Preserve empty and duplicate-session rows as SKIP placeholders; never compress, delete, or reorder them. Their annotation cells must remain blank.
3. Immediately before writing an `ANNOTATE` row, re-read its physical row and require the current session ID to equal the manifest session ID. Stop the batch on any mismatch. Reconfirm that SKIP-empty rows remain empty.
4. For `updateCells`, calculate `startRowIndex = source_row - 1` directly. Update only the user-specified annotation columns; never sort, insert, delete, or move rows while annotating.
5. Process at most 10 consecutive physical rows per batch. After writing, re-read `source_row`, session ID, `msg_data`, and every annotation column for the same batch.
6. Treat connector/API `success` as provisional. Accept a batch only when expected and actual annotated rows match and all of these are empty: missing rows, unexpected rows, session-ID mismatches, blank-`msg_data` rows with annotations, and duplicate written rows.
7. On any validation failure, stop before the next batch, re-read only the current batch, repair only that batch, and repeat full validation.
8. Before building an Optimization plan from a completed range, require the valid-source-row set to equal the annotated-row set, all annotated session IDs to match their source rows, all empty/duplicate SKIP rows to remain blank, and all multi-intent content to remain in its original physical row.

## Follow the two-stage workflow

### Stage 1: minimally annotate complete sessions

1. Treat one complete `live_agent_chat_session_id` as one analysis unit; retain repeated sessions and empty `msg_data` as manifest SKIP placeholders even though they are not annotated or counted.
2. Read one full `msg_data` at a time and record only:
   - the seller's concrete question;
   - answer-changing applicability conditions;
   - the human agent's explicit answer, link, query result, and executed action;
   - human-intervention type;
   - query/execution input, action, and result when applicable;
   - knowledge reusability.
3. If a session contains independently answerable questions, different applicability conditions, or different required capabilities, number every intent in the same source row and use matching numbers for conditions, agent solutions, and query/execution evidence. Never move a later intent to the next physical row. Keep supplementary descriptions and follow-ups to the same question as one intent.
4. Label human intervention once per session using the highest actual action: executed handling > object-level backend query > explanation/link/fulfillment-type-only confirmation. State which intent each query or execution supports.
5. Label knowledge reusability once per session in this order: any incomplete intent → `客服回复不完整`; otherwise any explicit reusable rule/step/link with all intents complete → `可形成通用答案`; otherwise all case-only results → `仅个案查询结果`; otherwise all executed handling → `仅人工处理`.
6. Keep L3 reason, physical source row, and session ID for traceability.
7. By default, do not search SKB, judge coverage, or draft SOP changes per conversation. Do so only when the user explicitly requests a legacy row-level coverage audit.
8. Process at most 10 consecutive physical rows, write and validate them under the Row Identity Lock, then continue automatically to the requested end row. A batch boundary is never a stopping point.

### Stage 2: cluster all valid seller problems

1. Put every valid session into first-level seller-question clustering before any capability filter. Human-intervention type and knowledge reusability are descriptive dimensions, not admission filters.
2. Cluster first by the concrete seller question and conditions that change the answer.
3. Within each problem cluster, retain second-level solution variants for:
   - reusable agent rules, steps, and links;
   - object-level data queries;
   - executed handling;
   - incomplete replies;
   - conflicting answers or time thresholds.
4. Preserve problem-cluster ID, total sessions, intervention/reusability distributions, solution-variant counts, source rows, session IDs, answer consistency, and owner questions.
5. A multi-intent session may enter more than one problem cluster; preserve the same source session for audit and do not sum overlapping cluster counts as a unique-session denominator.
6. Retain low-frequency long-tail sessions in the unresolved/long-tail pool; record `样本不足`、`低频长尾`或`暂待累计` in the dependency/notes field, and never invent a fourth action type merely because they do not yet produce an action.

## Route each cluster into capabilities

- **QA:** Use when a stable explanation, rule, step, or link can answer the seller without object-level data, variables, selectors, or complex routing. Supplement an existing QA solution when possible; otherwise propose a new QA.
- **API/Taskflow (SOP):** Use when the answer requires seller-specific or object-level backend data, variables, order/shop selectors, complex conditions, or advanced self-service. Supplement an existing SOP when possible; otherwise propose a new SOP.
- **Human handoff/process automation plan:** Use when agents actually registered, created a case, reported to backend, expedited logistics, escalated, submitted to a specialist, added materials, or filed a claim.
- **Dependencies, not an action type:** Keep incomplete, conflicting, condition-missing, owner-unclear, sample-insufficient, or long-tail variants in frequency analysis. Put the unresolved item in the dependency/notes field when a route is known; otherwise retain it in the unresolved pool without fabricating an action.

One problem cluster may produce multiple independent action rows.

### Fulfillment-type exception

If the agent only uses an order/shop identifier to confirm `3PF`, `SLS`, `Local`, `Cross-border`, `Pick-up`, `Drop-off`, or another fulfillment/order type and then gives a general rule:

- classify it as `否：仅解释或指引`;
- do not create an API/Taskflow requirement for fulfillment-type lookup;
- retain the confirmed type as an applicability condition;
- state clearly which fulfillment type the solution applies to.

Only further object-level backend results qualify for the API route.

## Assess SKB coverage at the knowledge-variant level

1. Search only plausible candidate solutions after clustering.
2. Inspect each candidate's full `message_content`, including SEH links.
3. For SOP solutions, treat `Solution ID + node_id + answer_branch_name` as the smallest configured answer variant. Inspect every relevant branch under the same node separately because seller metrics may route to different answers. Use `node_name` only as a rough node description; never substitute it for the branch's full `message_content`.
4. Compare the seller question, answer-changing conditions, agent's explicit reusable solution, configured branch content, and the branch's applicable seller-metrics condition. If the source does not expose enough information to determine the branch-routing condition, mark it for owner confirmation rather than infer it.
5. Use `完全覆盖`, `部分覆盖`, `未覆盖`, or `未评估`:
   - `完全覆盖`: name `solution「solution_name」（solution_id）` and state what it covers; create no knowledge action.
   - `部分覆盖`: state what is covered and the exact missing branch, condition, step, entry, or link.
   - `未覆盖`: no applicable solution exists for the supported question and conditions.
   - `未评估`: not reviewed due to priority, time, or insufficient evidence; never treat it as `未覆盖`.
6. In SOP coverage evidence, identify `SOP ID + node_id + answer_branch_name` and state what that branch covers or misses. Do not combine different branches to create a false complete-coverage conclusion.
7. Do not label API or human-handling actions as QA `未覆盖` merely because a static answer cannot perform them. Pending items are dependencies or unresolved states, not actions.
8. If reporting coverage, state the evaluated scope and denominator. Map a cluster result only to the source sessions of the equivalent knowledge variant, not automatically to the whole first-level cluster.

## Build the Optimization plan

1. Prioritize after all valid sessions have contributed to frequency. Consider frequency, self-service feasibility, API feasibility, human-handoff improvement, business impact, answer consistency, and ownership clarity.
2. Use exactly three action types in the G/action-type field: `QA`, `API/Taskflow`, and `人工承接/流程自动化计划`. Put uncertainty, missing mappings, sample insufficiency, and long-tail status in the dependency/notes fields, not in G.
3. Put one independently executable action and one solution destination per row. Keep one base problem cluster and create separate `Q/A/H` action rows when the same seller problem needs QA, object-level query, or executed handling. Repeat cluster-level fields and use only action-specific evidence.
4. For QA rows, write the plan and Chatbot-facing reusable answer in O; map exactly one existing QA solution or proposed new QA in P–S: QA Solution ID, QA issue ID, QA issue name, and the conditional issue-description delta. Use `待新增`, `待创建`, or `待确认` only in the corresponding mapping fields.
5. For API/Taskflow rows, write the SOP plan in T and map exactly one existing or proposed SOP in U–X: SOP Solution ID, SOP issue ID, SOP issue name, and the conditional issue-description delta. The plan must state trigger conditions, inputs, query/execution, outputs, final seller-facing answer, and failure/no-result handling. Use current variable names for dynamic values; label proposed fields with explicit proposed variable names and dependencies.
6. If the source contains independently answerable seller intents, place them in separate problem clusters instead of treating them as multiple solutions to one problem.
7. Keep filterable action type, separate QA and SOP Solution ID/status fields, separate QA and SOP issue fields, source rows, and session IDs.
8. For every API/Taskflow action, record only source-supported trigger conditions, inputs, query action, returned fields/states, success answer, failure/handoff route, owner/dependency, and evidence sessions.
9. For every human-handoff/process action, record only source-supported trigger conditions, seller materials, destination/team, stated timing/notification method, automatable steps, and evidence sessions.
10. For every QA or SOP action, map exactly one solution to the user-specified issue source and output the corresponding `issue_id`, `issue_name`, and row-specific issue-description change that applies only if that row is implemented. Split rows whenever more than one QA/SOP destination is involved.
11. For an existing Tool-type SOP with no bound issue in the supplied source, keep the real SOP Solution ID, set the issue ID to `待确认`, explicitly state that the Tool SOP has no bound issue, and write only a conditional issue-description delta for whichever issue is later mapped. Never guess an issue ID.
12. When drafting seller-facing reusable answers, preserve source-grounded agent wording that improves seller acceptance, such as clear scenario confirmation, explanation, timing, next steps, and appropriately supported empathy. Do not add promises, outcomes, or reassurance that the agent did not state.
13. Do not pre-combine unimplemented QA or SOP actions into a final issue description. For a new QA/SOP, provide the proposed issue's complete initial two-section description only when source conditions are sufficient.
14. When the request includes KB implementation, route the approved action through the current Issue/Solution architecture, authoring controls, testing, approval, and publishing rules in the KB-management reference. Do not treat an Optimization-plan recommendation as already configured or published.
15. When the request includes impact evaluation, define every reported metric, scope, period, source, numerator, and denominator according to the metrics reference. Do not equate knowledge coverage, potential self-service, or a no-human-action pool with realized DLA/TLA reduction.

## Non-negotiable safeguards

- Use `msg_data` as the conversation source and solution `message_content` only as the currently configured Chatbot-answer source.
- Never add facts, rules, steps, system capabilities, fields, thresholds, materials, or outcomes from another conversation, an SOP, experience, or common sense.
- Use only the human agent's explicit answer, advice, links, query result, or executed handling in reusable answers. Seller statements may establish applicability/evidence only.
- Keep evidence language and seller-facing language strictly separated. `客服回复/客服说明/客服表示/客服确认/客服查询/客服未查询到/客服提供/客服建议/根据客服/人工客服/转接后/专员回复` may appear in internal evidence fields, but must not appear after `可直接复用的答案：` in O or after `最终输出答案：` in T. Rewrite supported content as a direct, condition-specific Chatbot answer; if doing so would strengthen, generalize, or invent the source conclusion, leave the seller answer blank and record the missing condition or decision in Z/dependencies.
- Combine multiple sessions only when their conditions are compatible; retain conflicts for owner review.
- Never replace applicability conditions with `一般情况下`, `通常`, `原则上`, `大多数情况下`, or `视情况而定`.
- Anonymize order, waybill, shop, product, account, contact, address, and chat identifiers in reusable answers.
- Do not generalize a case-specific result. Conditional extraction is allowed only when the same source conversation explicitly supports both the condition and solution.
- If conditions, answer, ownership, API fields, variables, or issue mapping are missing, write the exact gap in the dependency/notes field. Do not create a `待确认` action type.
- Chatbot must not claim it completed a case, escalation, expedite, or other action unless the system actually executed it and returned the result.
- Before every Optimization-plan write and after re-reading it, lint both each nonblank O answer after `可直接复用的答案：` and each T seller answer after `最终输出答案：` for evidence-attribution phrases, unsupported completion claims, vague-condition wording, missing applicability conditions, and source drift. Any failure blocks the write or the next batch; move internal wording to L/M/Z or correct the answer before continuing.

## Completion criteria

- All requested rows or plan items are written and re-read.
- The final manifest proves valid source rows equal annotated rows; SKIP rows remain blank; missing, unexpected, mismatched, blank-source-annotated, and duplicate-written row sets are empty.
- Session IDs, immutable physical source rows, L3 reason, `msg_data`, and outputs remain aligned; `updateCells` rows were calculated only from `source_row - 1`.
- Every multi-intent session retains all numbered intents in its original physical row, while its intents may enter separate problem clusters without inflating the unique-session denominator.
- Every valid session contributes to a problem cluster or an explicit long-tail/pending pool.
- Knowledge reusability and human-intervention labels were not used as pre-clustering filters.
- Coverage evidence names matched solutions and distinguishes `未评估` from `未覆盖`.
- Only partial/no-coverage QA variants that require changes create QA actions; complete coverage creates no QA optimization.
- API actions exclude fulfillment-type-only checks and contain no invented fields or capabilities.
- Human-handoff actions contain only agent-requested materials and actually executed/stated processes.
- One action and one solution destination appear per row; QA and SOP have separate filterable Solution ID and issue fields.
- Issue-description suggestions are conditional, row-specific, and source-supported.
- Reusable answers are anonymized, condition-specific, source-grounded, and have bold `优化建议` and `可直接复用的答案` labels when written to Google Sheets.
- Every nonblank O-cell answer and T-cell final seller answer passes the Chatbot-role lint: no internal evidence attribution, no false completed action, no vague substitute for conditions, and no strengthening of a source that only says the agent could not find or confirm something.
