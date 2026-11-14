# System Architecture Document — Mortgage Servicing Customer Support

## Context & Instructions

This file records the three greenfield decisions in order: Work Ownership, Pattern Decision (with Reference Architecture), and Control Register (with Model and Context Strategy and Production Shape). It does not add product requirements. Sections 1–10 of `.cursor/templates/sad-template.md`, including the evaluation criteria table, wait for `*create-sad`.

The product receives one natural-language question and a library of articles the fixture marks approved. It searches that library. It does not receive the hidden expected result, and it does not receive a pre-chosen set of matching articles. A supported answer is delivered to the customer with source references, and no approval item is created. Any of the three escalation conditions creates one approval item that holds the question and the draft. The customer does not see that draft as the answer until the representative records approval. After approval, the customer receives that same draft and its source references.

`project-context/1.define/system-description.md` is not present. `ux.md` is not present. Channel, layout, wording, and a disability-access standard stay with `@ux.designer`.

## Input Requirements

**PRD Document**: `project-context/1.define/prd.md`  
**MRD**: `project-context/1.define/mrd.md`  
**System Description**: Not present at `project-context/1.define/system-description.md`  
**User Stories**: `project-context/1.define/user-stories/US-001.md`, `US-002.md`, `US-003.md`, `US-004.md`  
**MVP Scope**: PRD §4 Priority P0, FR-01 through FR-10. P1 is empty. P2 is future work.  
**Selected Runtime**: `crewai`

Resolution: `AAMAD_TARGET_RUNTIME` is unset. `aamad.config.yml` sets `runtime.target: crewai`. The adapter default is `crewai`. Resolved runtime is `crewai`. The PRD does not treat `runtime.target` as a product requirement. This SAD records the runtime for the generated application.

## System Architecture Specification

### Work Ownership

On this greenfield increment, “existing system” means the workflow code this document specifies. It does not mean a live mortgage servicer, CRM, Salesforce, or Zendesk. Those systems stay mocked and unconnected (MN-01, FR-09).

The four behavior properties used below are probabilistic output, knowledge cutoff, limited working memory, and steerability. A step whose output must be a closed set of reasons, a resolved citation, or a send-or-hold decision stays with code. A step that reads an open question and writes prose stays with the model, inside a schema the code checks.

| Step | Owner (model / existing system / human / collaborative) | Reversibility | Stakes | Accountable party | Rationale |
|------|----------------------------------------------------------|---------------|--------|-------------------|-----------|
| Customer submits one question (FR-01, US-001) | human | Reversible. A later question is a new trace. | Low for the act of submitting. | Customer | The customer is the primary user and starts the only happy path. The workflow has no representative-started entry. |
| Record the reason set (FR-02, US-002, US-004) | collaborative | Reversible until delivery. A wrong `supported` set is weakly reversible after the customer has the answer. | High. The set is the only input to the send decision. | Workflow code for the schema. The model proposes the set. | The judgment is probabilistic. The allowed values are a closed set: `supported`, or one or more of `unsupported`, `customer-specific loan information`, and `cannot be determined to be safe`. `supported` is exclusive. Code rejects any other shape. The customer is not required to see the set. The test reads it. |
| Search the approved library and the mock records (FR-03, FR-09, MN-05) | existing system | Reversible. A search does not send an answer. | High if a loan mock is treated as an approved source. | Workflow code | Search over fixtures is deterministic. The model does not choose the corpus from memory. A loan mock may be found. It is not an approved article, and it is not a live loan lookup. |
| Draft the answer and name source references (FR-04) | collaborative | Reversible until the customer receives it. | High. A sentence can state a fee, rate, account status, policy, or loan fact. | Workflow code for the citation check. The model writes the draft. | Prose is probabilistic. Every source reference must resolve to an article the fixture marks approved. The drafter does not receive loan-record field values. |
| Choose direct delivery or the one gate (FR-05, FR-06, FR-07) | existing system | Direct delivery is weakly reversible once the customer has the answer. Opening the gate is reversible until approval. | High. This is the consumer-harm point in MRD Risk. | Workflow code | The rule is fixed. Direct delivery is allowed only when the recorded set is exactly `supported` and every source reference resolves. Any escalation reason opens one approval item. The model does not own this choice. |
| Show the answer, the waiting state, or a non-answer (FR-08, US-003) | existing system | Reversible while the gate is open, because the draft is withheld. | High if the draft is shown early. | Workflow code and the customer view | The customer view follows the disposition. While the approval item is open, the view shows that the question is waiting on representative review and does not present the draft as the answer. Wording and layout are not specified here. |
| Representative records approval (FR-07, US-004) | human | The approval stands. The MVP has no second approver and no undo. | High. Approval is what releases an escalated draft. | Representative role | The representative is the secondary user and the only approver. The MVP action is to open the item and record approval. The item holds the question and the draft. No edit, reject, or sender role is in the PRD. |
| Deliver that same draft after approval (FR-07) | existing system | The customer then has the draft. | High if a new draft is generated instead. | Workflow code | The stored draft, including its source references, is what the customer receives. The model is not called again. |

**Feasibility verdict**: `feasible with constraints`

**Load-bearing constraint**: The product must record the reason set and draft the answer from the question and the fixture library, without the hidden answer key. The test observes that set before it observes the source references on the draft (MD-02, US-002). Two of the reasons, `unsupported` and `cannot be determined to be safe`, are judgments about the answer against the approved material, so the classifier may use the search tool before the set is stored. The set is not revised after the draft. A mock loan record may sit in the searchable data and still must not become a cited source, and it must not by itself clear the gate (MN-05, US-004). The general meanings of customer-specific, approved, safe, and accurate-beyond-a-citation are unresolved (PRD Open Questions, MP-03). A fixture pass shows the workflow. It does not define those terms for a live servicer.

### Pattern Decision

- **Primary shape**: workflow
- **Workflow sub-shape**: combination: chaining, then routing
- **Deciding factor**: Predictability rules out an agent. MD-02 and the MRD primary focus already fix the order: submit, classify, retrieve, draft, then one of two outcomes. The path can be written in advance. Error cost then rules out an augmented call. A wrong direct delivery of a fee, rate, account status, policy, or loan fact is the high risk in the MRD, and the customer would have that answer with no representative in between. The disposition has to be a deterministic rule on the recorded reason set and the citation check. An augmented call would leave that disposition inside the same model response as the draft.
- **Rejected alternatives and reversal cost**:
  - Agent. Rejected on predictability. Reversal cost is high. The observable step order, the boundary that keeps loan-record fields out of the draft, and the code-owned router would be replaced by a model-chosen loop.
  - Augmented call. Rejected on error cost. Reversal cost is moderate. One prompt could later be split into the two tasks below, and the MVP would already have shipped with the model owning the send decision.
  - Evaluator-optimizer. Not used. The PRD does not ask for a second model to grade the draft. Citation checks are deterministic. Sentence-level accuracy is MP-03 and has no threshold.
  - Parallelization. Not used. The five steps are ordered on one question.
  - There is no fourth pattern named multi-agent. Two model tasks in a sequence the crew config writes down remain a workflow.
- **Progression used**: prompt for the reason set and the draft, then retrieval over fixtures as a tool the workflow calls, then a code router. Evaluator-optimizer is not added. Fine-tuning is not recommended. No fine-tune set is in the inputs, and the cohort is a fixture demonstration.

This is not an agent. The sequence is written in crew configuration. Classify runs, then draft runs, then code routes. Neither task chooses the next role. `allow_delegation` stays false. Process mode is sequential. There is no model-chosen tool loop, no turn budget for exploration, and no stop condition of the agent kind.

The two model tasks are bounded:

- Classify receives the question and may call the fixture search before it proposes the reason set. It does not receive the hidden expected result.
- Draft receives the question, the stored reason set, and the approved passages the search already returned. It names source references from those passages.

Search, the citation check, the router, the customer view, and the copy of the stored draft after approval are code.

### Reference Architecture

| Shape | Use when | Known failure to prevent |
|-------|----------|--------------------------|
| Routing | The question ends in one of two outcomes | No human escalation, or an agent shipped before the routed workflow was measured |
| Retrieval | The draft cites a stable approved library | Retrieval used as a substitute for live state |

- **Selected reference architecture**: Routing
- **Known failure this design prevents**: A missing human gate, and an agent loop shipped in place of the routed steps. The router is code. Direct delivery happens only for an exactly `supported` set whose citations resolve. Every escalation reason opens the one representative gate.
- **Second pattern**: Retrieval. Approved articles are the only citable corpus. Loan mocks and customer records are searchable fixtures, not live servicing state, and not approved articles. Finding a loan mock is a tool result. It is not a citation, and it does not answer a customer-specific question (MN-05, US-004).

### Model and Context Strategy

- **Default model tier**: balanced default. No measured need in the PRD or the MRD requires another tier. No model name is selected here.
- **Context strategy**: progressive. Each step receives only the fields that step needs.
- **Swap-as-release rule**: A model change or a prompt change is a release. `@qa.eng` runs the evaluation criteria table, once `*create-sad` has filled it, before the swap. The rollback criterion is written before the swap: the previous model and prompt pair remains runnable, and the fixture set in FR-10 still matches on that pair. No latency or cost number is sourced (MC-01), so the rollback criterion is the fixture match.
- **Entry point**: direct API
- **Entry-point tradeoff named at decision time**: The crew runtime calls the model provider. The application keeps the route in code, so a supported answer is the only text the customer receives on the happy path. A hosted chat would put the customer in a vendor thread where the model could answer before the router runs.
- **Entry-point responsibility map**: One route. The customer surface and the representative surface talk to the workflow. The workflow is the only caller of the model. `AAMAD_TARGET_RUNTIME=crewai` is that caller. It is not a second product entry.
- **Minimum data in the context window**:
  - Classify: the question, plus search hits. An approved-article hit may include the passage the draft will cite. A loan-mock hit includes the record id and the type `loan_mock` only. The hidden expected result is never included.
  - Draft: the question, the stored reason set, and the approved passages. Loan-record field values are not included.
  - Router, customer view, and post-approval delivery: no model call. They read the stored set, the citation check, and the stored draft.
- **Identity**: The test distinguishes the customer role from the representative role in application code. The prompt does not grant either role. A real customer or employee directory is out of scope (PRD Integration Requirements).

Resolved runtime controls for the generated crew, from `.cursor/rules/adapter-crewai.mdc`, and not as product targets:

- Sequential process. Hierarchical process is not used.
- `memory=False`.
- `max_retry_limit` 2.
- `max_iter` stays at or below 12. The exact value is set when the crew is scaffolded, high enough for one fixture search and one structured output on each model task, and no higher.
- Temperature 0 for a repeatable fixture run. This is not an accuracy target.
- `max_rpm` is required at crew level by the adapter. No cost ceiling is sourced, so the number waits for an operator value. It is an Open Question, not a default invented here.
- No model name and no max-token figure are selected. Both wait for setup.

### Control Register

Training-time behavior is not a control this MVP can set. The application enforces the rows below. Side effects fail closed: a failed check does not deliver an answer and does not release an escalated draft.

The representative gate is the review route for the three escalation reasons. The approval item holds the question and the draft (FR-07). The reason set stays in the run record for the test. This table does not add the reason set to the representative’s item.

| ID | Risk category | Control | Control point (input / output / tool-call) | Failure direction (fail closed / fail open) | Owner | Evidence artifact | Review routing |
|----|---------------|---------|--------------------------------------------|---------------------------------------------|-------|-------------------|----------------|
| CR-01 | Direct injection in the question | The question is data. The stored reason set must match the closed schema. Any other shape is rejected. | input | fail closed | Workflow code | Stored reason set | Not the representative gate. The customer sees a non-answer, not a supported answer (US-003). |
| CR-02 | Indirect injection in a fixture article | A source id the resolver cannot match to an approved article is rejected. The model cannot add an id the fixture did not mark approved. | output | fail closed | Workflow code | Citation check | Not the representative gate. A citation failure is not shown as a supported answer (FR-03, US-003). |
| CR-03 | Token-budget exhaustion | A model call that stops on the runtime limit produces no customer answer and no approval item. No character or token cap is sourced. | input | fail closed | Workflow code | Task stop on the question trace | Non-answer. The numeric cap is an Open Question. |
| CR-04 | Tool abuse, including a live system of record | The only tool is fixture search. No call is bound to a mortgage servicing platform, CRM, Salesforce, Zendesk, or the open web. | tool-call | fail closed | Workflow code | Tool list in the crew config | None. The call is not available. |
| CR-05 | Loan-field exposure and a mock treated as a lookup | A loan-mock hit returns the record id and the type only. A question whose recorded set includes `customer-specific loan information` still opens the gate when that hit exists. | tool-call | fail closed | Workflow code | Tool-result schema and the stored reason set | The one representative gate when the set contains that reason (MN-05, US-004). |
| CR-06 | Direct delivery of an answer that is not exactly supported | The router delivers only when the stored set is exactly `supported` and every source reference resolves. Any escalation reason creates one approval item. | output | fail closed | Workflow code | Disposition on the question trace | The one representative gate. |
| CR-07 | Draft shown before approval | While the approval item is open, the customer view shows the waiting state and does not present the draft as the answer. | output | fail closed | Workflow code | Customer-view state | The representative already holds the item (FR-08). |
| CR-08 | A new draft after approval | Approval copies the stored draft and its source references. The model is not called. | output | fail closed | Workflow code | Same stored draft before and after approval | The representative’s approval is the release (FR-07). |
| CR-09 | A second approver or an approval on a supported answer | One approval item, one representative role, and no approval item when the set is exactly `supported`. | output | fail closed | Workflow code for the rule. The representative for the approval itself. | Approval record | The one gate. A second item fails the test (MN-03, US-004). |
| CR-10 | Secret exposure | The model credential is read from the environment. It is not written into prompts, fixtures, or logs the MVP retains. | input | fail closed | Workflow code | Env var name in `.env.example` only | None. |
| CR-11 | Decision log | Each question trace stores the question id, the reason set, the source ids, the citation check, the disposition, and, when used, the approval actor. | output | fail closed | Workflow code | Question trace | The test reads the trace. The representative reads the question and the draft. |

Fairness injection points are not a PRD requirement. No fairness row is added.

### Production Shape

- **Volume**: The expected volume is the FR-10 fixture set: one supported question, one question for each escalation condition alone, and one question whose hidden result lists two escalation reasons. This increment is not a production servicing operation (MRD §4). No concurrency, throughput, or uptime number is sourced (MC-01).
- **Retries**: A model task may retry up to `max_retry_limit` 2. A retry repeats that task. It does not change a reason set after it is stored, and it does not generate a new draft after approval.
- **Fallback**: When the retry limit is exhausted, the workflow does not deliver a supported answer and does not invent a fourth escalation reason. The customer sees a non-answer (US-003).
- **Circuit breaker**: The breaker opens for that question when the retry limit is exhausted. Later questions in the same fixture run may still call the model. No separate failure-count threshold is sourced.
- **Failure mode of this architecture**: A `supported` set that reaches the customer without passing the router. The customer view reads the router’s disposition only.
- **Traces and dashboards**: One trace per question, as in CR-11. No aggregate dashboard is sourced. MRD §4 leaves hosting and a monitoring product to a later deployment.

## Architecture Validation Checklist

- [ ] PRD requirements mapped to architectural components (`*create-sad`)
- [x] Work Ownership, Pattern Decision, Reference Architecture, Model and Context Strategy, and Control Register are complete
- [x] Runtime roles exist only as the two bounded model tasks inside the workflow
- [ ] Frontend and backend contracts (`*create-sad`)
- [x] Secrets via env vars only (CR-10)
- [x] MVP vs Future Work boundaries explicit: P2 in the PRD stays out of these decisions
- [x] Resolved `AAMAD_TARGET_RUNTIME` recorded in Audit

## Sources

- `project-context/1.define/prd.md` — FR-01 through FR-10, MD-02, MN-01, MN-03, MN-05, MC-01, MP-03, §§5–6, and Open Questions.
- `project-context/1.define/mrd.md` — Context (primary focus, approval policy, escalation conditions, mocked data), §2, §4, and Risk Assessment Matrix. Market figures are context and are not thresholds.
- `project-context/1.define/user-stories/US-001.md`, `US-002.md`, `US-003.md`, `US-004.md` — acceptance criteria already traced to the PRD. No story adds a requirement the PRD lacks.
- `aamad.config.yml` — `project.mode: greenfield`, `runtime.target: crewai`.
- `.cursor/templates/sad-template.md` — decision headings used here. Sections 1–10 are not filled.
- `.cursor/rules/adapter-crewai.mdc` — sequential process, YAML config, `allow_delegation`, `max_iter`, `max_retry_limit`, `max_rpm`, `memory=False`.
- System description: not present.
- `ux.md`: not present.

## Assumptions

- `AAMAD_TARGET_RUNTIME` is unset. Resolution uses `aamad.config.yml` `runtime.target`, which matches the adapter default `crewai`.
- “Existing system” in Work Ownership is the workflow code this SAD specifies.
- The classifier may call fixture search before the reason set is stored, so that `unsupported` and `cannot be determined to be safe` can be judged against the approved material. The stored set is what the test observes before the draft’s source references. The set is not edited after the draft.
- The representative’s only MVP action is to record approval. The PRD does not define reject, edit, or a second approver. Until approval, the customer remains on the waiting state. There is no timeout.
- A loan-mock search hit carries the record id and the type `loan_mock`, and does not carry loan field values, into the model context. The record is still in the searchable fixture data. This is the control for MRD Risk and MN-05. It is not a definition of which facts are customer-specific.
- An article is approved for the MVP when the fixture marks it approved. Conflict between two approved articles has no automatic rule.
- A notice of error, a qualified written request, or a request for information has no special route. It escalates only when the recorded set says so.
- Temperature 0 and `max_retry_limit` 2 are runtime controls for a repeatable fixture run. They are not quality or cost targets from the PRD.
- The model provider is not a system of record. The workflow may call it when the model credential is set. The PRD does not require the draft step to run without a model.
- The six-week cohort is a scope constraint from the MRD, not a schedule estimate.

## Open Questions

Carried from the PRD and the MRD. They are not answered here:

- Which facts count as customer-specific loan information, beyond the hidden expected result?
- What makes an article approved outside a fixture, and what happens when two approved articles conflict?
- Does a notice of error, a qualified written request, or a request for information always escalate?
- What evidence, beyond a source reference, shows that a draft is accurate? MP-03 stays open.
- What evidence shows that an answer is easy to understand? No readability test was supplied.
- Must mocked records be synthetic only?
- After this MVP, in a live operation, who may send the response once the representative has approved it?
- Are any latency, per-interaction cost, volume, response-time, throughput, concurrency, or uptime limits required? No number is set.
- What notice, other than the absence of the draft, should the customer see for a failure that is not one of the three escalation conditions?
- Does “mock every external system” include the model provider, or only systems of record?

Added by these decisions:

- What `max_rpm` value should the crew use? The adapter requires the setting. The PRD supplies no cost ceiling.
- What `max_iter` value, at or below 12, should each model task use once the crew is scaffolded?
- What `max_execution_time`, if any, should stop a hung model call? Until one is set, the retry limit is the stop, and duration is not a test failure.
- What token or character limit, if any, should CR-03 enforce?

## Audit

- Timestamp: 2026-11-14T13:26:13-05:00
- Persona id: `system-arch`
- Action: `decompose-work`
- Resolved runtime: `crewai`. `AAMAD_TARGET_RUNTIME` was unset. `aamad.config.yml` `runtime.target` is `crewai`. The adapter default is `crewai`. Warning: the environment variable was unset, so the config value was used; it matches the default.
- Prompt trace: omitted. This action records work ownership traced to the PRD, the MRD, and the user stories. It does not render a customer-facing model prompt.

- Timestamp: 2026-11-14T13:26:13-05:00
- Persona id: `system-arch`
- Action: `select-pattern`
- Resolved runtime: `crewai`, same resolution as above.
- Prompt trace: omitted. This action records the pattern decision. It does not render a customer-facing model prompt.

- Timestamp: 2026-11-14T13:26:13-05:00
- Persona id: `system-arch`
- Action: `define-controls`
- Resolved runtime: `crewai`, same resolution as above.
- Model controls recorded: temperature 0, `max_retry_limit` 2, `memory=False`, sequential process, `allow_delegation` false. `max_iter` is capped at 12 and not pinned. `max_rpm`, the model name, and max tokens are unset.
- Prompt trace: omitted. This action records the control register. It does not render a customer-facing model prompt.
- Scope honored: the three decisions only. No application code, no UX spec, no SFS, no diagram, and no change to the PRD or the MRD. Sections 1–10 of the SAD template are not filled.
