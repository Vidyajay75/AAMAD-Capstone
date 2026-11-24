# System Architecture Document — Mortgage Servicing Customer Support

## Context & Instructions

This is the lean MVP System Architecture Document (`*create-sad --mvp`). It records Work Ownership, Pattern Decision (with Reference Architecture), and Control Register (with Model and Context Strategy and Production Shape), then the MVP views in sections 1–10. It does not add product requirements. Scale, hosting product, concurrency, and uptime are deferred. The pattern decision and the control register are not deferred.

The product receives one natural-language question and a library of articles the fixture marks approved. It searches that library. It does not receive the hidden expected result, and it does not receive a pre-chosen set of matching articles. A supported answer is delivered to the customer with source references, and no approval item is created. Any of the three escalation conditions creates one approval item that holds the question and the draft. The customer does not see that draft as the answer until the representative records approval. After approval, the customer receives that same draft and its source references. The representative who records approval is the sender. A technical failure shows the FR-11 sentence, logs `technical_failure`, and creates no approval item. A provider timeout records `unsupported`, opens the one gate, and is not automatic FR-11 (CR-16). A 201st submitted question in a day is processed the same as questions 1–200, and the product logs that the day exceeded 200.

`project-context/1.define/system-description.md` is not present. `ux.md` is not present. Channel, layout, the waiting-notice wording, and a disability-access standard stay with `@ux.designer`. The technical-issue sentence is specified in FR-11.

## Input Requirements

**PRD Document**: `project-context/1.define/prd.md`  
**MRD**: `project-context/1.define/mrd.md`  
**System Description**: Not present at `project-context/1.define/system-description.md`  
**User Stories**: `project-context/1.define/user-stories/US-001.md`, `US-002.md`, `US-003.md`, `US-004.md`  
**MVP Scope**: PRD §4 Priority P0, FR-01 through FR-12. P1 is empty. P2 is future work.  
**Selected Runtime**: `crewai`

Resolution: `AAMAD_TARGET_RUNTIME` is unset. `aamad.config.yml` sets `runtime.target: crewai`. The adapter default is `crewai`. Resolved runtime is `crewai`. The PRD does not treat `runtime.target` as a product requirement. This SAD records the runtime for the generated application.

## System Architecture Specification

### Work Ownership

On this greenfield increment, “existing system” means the workflow code this document specifies. It does not mean a live mortgage servicer, CRM, Salesforce, or Zendesk. Those systems stay mocked and unconnected (MN-01, FR-09).

The four behavior properties used below are probabilistic output, knowledge cutoff, limited working memory, and steerability. A step whose output must be a closed set of reasons, a resolved citation, a claim that appears in a cited article, a send-or-hold decision, a daily count, or a cost record stays with code. A step that reads an open question and writes prose stays with the model, inside a schema the code checks.

| Step | Owner (model / existing system / human / collaborative) | Reversibility | Stakes | Accountable party | Rationale |
|------|----------------------------------------------------------|---------------|--------|-------------------|-----------|
| Customer submits one question (FR-01, US-001) | human | Reversible. A later question is a new trace. | Low for the act of submitting. | Customer | The customer is the primary user and starts the only happy path. The workflow has no representative-started entry. |
| Count the submitted question for the calendar day (FR-12, MC-01) | existing system | Reversible as a log event. The 201st question still runs. | Low for the customer, who must not see a capacity refusal. Medium for the test, which must read the log. | Workflow code | Two hundred questions in a day is a planning number, not a hard cap. A 201st question is processed the same as questions 1–200. Code increments the count and logs that the day exceeded 200. The log identifies the question and does not contain a secret. FR-11 is not used because the day exceeded 200. The model does not own volume. |
| Record the reason set (FR-02, MD-06, MD-07, MD-08, US-002, US-004) | collaborative | Reversible until delivery. A wrong `supported` set is weakly reversible after the customer has the answer. | High. The set is the only input to the send decision. | Workflow code for the schema and the closed definitions. The model proposes the set. | The judgment is probabilistic. The allowed values are a closed set: `supported`, or one or more of `unsupported`, `customer-specific loan information`, and `cannot be determined to be safe`. `supported` is exclusive. Customer-specific means the MD-06 fact list, or a question that is itself a notice of error, a qualified written request, or a request for information (MD-08). Two approved articles that disagree on a fact the answer needs add `cannot be determined to be safe` (MD-07). Code rejects any other shape. The customer is not required to see the set. The test reads it. |
| Search the approved library and the mock records (FR-03, FR-09, MN-05) | existing system | Reversible. A search does not send an answer. | High if a loan mock is treated as an approved source. | Workflow code | Search over fixtures is deterministic. The model does not choose the corpus from memory. An article is approved when the fixture marks it approved (MD-07). A loan mock may be found. It is not an approved article, and it is not a live loan lookup. |
| Draft the answer, name source references, and check claims (FR-04, MP-02, MP-03) | collaborative | Reversible until the customer receives it. | High. A sentence can state a fee, rate, account status, policy, or loan fact. | Workflow code for the citation check and the claim-in-article algorithm. The model writes the draft and emits `claims`. | Prose is probabilistic. Drafting may call a model (MN-01). Every source reference must resolve to an article the fixture marks approved. Claims are the Draft schema’s `claims` list, not scanned sentences. Code runs the CR-12 algorithm: normalized substring of each `text` in a cited approved article. A failing claim makes the recorded set include `unsupported` and not `supported`. That update is stored before disposition. It is not a fourth escalation reason and it is not FR-11. The drafter does not receive loan-record field values. The product records the model or provider cost of that question, which must be at most 20 cents (MC-01). |
| Choose direct delivery, the one gate, or the technical-issue outcome (FR-05, FR-06, FR-07, FR-11) | existing system | Direct delivery is weakly reversible once the customer has the answer. Opening the gate is reversible until approval. A technical-issue notice is a later retry by the customer. | High. Direct delivery of an unsupported fee, rate, account status, policy, or loan fact is the consumer-harm point in MRD Risk. | Workflow code | The rule is fixed. Direct delivery is allowed only when the recorded set is exactly `supported` and every source reference resolves. Any escalation reason opens one approval item. A `technical_failure` uses FR-11. A `provider_timeout` records `unsupported` and opens the one gate; FR-11 only if that item cannot be created (CR-16). A live success over 5 seconds keeps its disposition and sets `performance_miss`. The model does not own this choice. |
| Show the answer, the waiting state, or the technical-issue notice (FR-06, FR-08, FR-11, US-002, US-003) | existing system | Reversible while the gate is open, because the draft is withheld. A shown supported answer is weakly reversible. | High if the draft is shown before approval, or if a technical failure is shown as an answer. | Workflow code and the customer view | The customer view follows the disposition. Code always records `elapsed_ms`. Fixture/stand-in must finish in under 5 seconds. Live success over 5 seconds is a `performance_miss` (CR-16), not FR-11. Representative review is outside that interval and has no clock of its own (CR-17). While the approval item is open, the view shows that the question is waiting on representative review and does not present the draft as the answer. That waiting state continues until the representative records approval. Waiting-notice wording and layout stay with `@ux.designer`. The technical-issue sentence is specified in FR-11. |
| Representative records approval (FR-07, MD-09, US-004) | human | The approval stands. The MVP has no second approver and no undo. | High. Approval is what releases an escalated draft. The representative is the sender of that draft. | Representative role, as sender | The representative is the secondary user, the only approver, and the sender (MD-09). A separate sender role is out of scope, including in a later live operation. The MVP action is to open the item and record approval. That action is the only event that ends waiting (CR-17). The item holds the question and the draft. No edit, reject, auto-approve, or review-timeout action is in the PRD. |
| Deliver that same draft after approval (FR-07, MD-09) | existing system | The customer then has the draft. | High if a new draft is generated instead. | Workflow code for the copy. The representative remains the sender of that copy. | The stored draft, including its source references, is what the customer receives. The model is not called again. Code delivers because the representative recorded approval. It does not introduce a second sender. |

**Feasibility verdict**: `feasible with constraints`

**Load-bearing constraint**: The product must record the reason set and draft the answer from the question and the fixture library, without the hidden answer key. The test observes submit, then a classification outcome, then source references drawn from approved knowledge, then a draft, then exactly one of direct delivery or one approval item, or FR-11 when the workflow cannot complete (MD-02). Two of the reasons, `unsupported` and `cannot be determined to be safe`, are judgments about the answer against the approved material, so the classifier may use the search tool before the set is stored. The claim-in-article algorithm (CR-12, FR-04, MP-03) can add `unsupported` after the draft is written; that update is stored before the send-or-gate decision. A mock loan record may sit in the searchable data and still must not become a cited source, and it must not by itself clear the gate (MN-05, US-004). Code always records `elapsed_ms`. Fixture/stand-in must finish in under 5 seconds (EC-04). Live success over 5 seconds is a `performance_miss`, not FR-11 (CR-16). The recorded model or provider cost of that question is at most 20 cents (MC-01). A 201st question in a day still completes the workflow and is logged (FR-12). A fixture pass shows those rules. It does not make this capstone a live servicer.

### Pattern Decision

- **Primary shape**: workflow
- **Workflow sub-shape**: combination: chaining, then routing
- **Deciding factor**: Predictability rules out an agent. MD-02 already fixes the order: submit, classify, retrieve, draft, then exactly one of direct delivery or one approval item, or FR-11 when those steps cannot complete. FR-12 is that same path plus a log that the day exceeded 200. The path can be written in advance. Error cost then rules out an augmented call. A wrong direct delivery of a fee, rate, account status, policy, or loan fact is the high risk in the MRD, and the customer would have that answer with no representative in between. The disposition has to be a deterministic rule on the recorded reason set, the citation check, and the claim-in-article check (FR-04, MP-03). An augmented call would leave that disposition inside the same model response as the draft.
- **Rejected alternatives and reversal cost**:
  - Agent. Rejected on predictability. Reversal cost is high. The observable step order, the boundary that keeps loan-record fields out of the draft, and the code-owned router would be replaced by a model-chosen loop.
  - Augmented call. Rejected on error cost. Reversal cost is moderate. One prompt could later be split into the two tasks below, and the MVP would already have shipped with the model owning the send decision.
  - Evaluator-optimizer. Not used. Latency and cost rule it out as a third model call inside the under-5-second and 20-cent budgets (MC-01). MP-03 is the CR-12 algorithm (emitted `claims`, normalized substring). It is not a second-model grade, and it has no accuracy percentage.
  - Parallelization. Not used. The five steps, the claim check, and the router are ordered on one question.
  - There is no fourth pattern named multi-agent. Two model tasks in a sequence the crew config writes down remain a workflow.
- **Progression used**: prompt for the reason set and the draft, then retrieval over fixtures as a tool the workflow calls, then a code claim-and-citation check, then a code router. Evaluator-optimizer is not added. Fine-tuning is not recommended. No fine-tune set is in the inputs, and the cohort is a fixture demonstration.

Observability follows MD-02: the test can observe submit, then a classification outcome, then source references, then a draft, then one of the two dispositions or FR-11. Latency and cost do not change the primary shape. They cap the chain at two model tasks.

This is not an agent. The sequence is written in crew configuration. Classify runs, then draft runs, then code checks claims and citations and routes. Neither task chooses the next role. `allow_delegation` stays false. Process mode is sequential. There is no model-chosen tool loop, no turn budget for exploration, and no stop condition of the agent kind.

The two model tasks are bounded. Each task uses CrewAI `output_pydantic`. A parse miss is recorded as `unsupported` and opens the one gate. It is not FR-11 and not a repaired object that is treated as `supported`.

- Classify emits `ClassifyOutput` (`reason_set` only). It receives the question and may call fixture search. It does not receive the hidden expected result. Customer-specific, conflict, and notice-of-error / qualified-written-request / request-for-information meanings are MD-06, MD-07, and MD-08.
- Draft emits `DraftOutput` (`answer`, `source_refs`, `claims`). It receives the question, the stored `reason_set`, and the approved passages. It does not receive loan-record field values.

Daily count and the 201st-question log, search, the citation check, the CR-12 algorithm, the router, the customer view, and the copy of the stored draft after approval are code. The CR-12 algorithm may add `unsupported` and remove `supported` before the router runs. The router’s three automated exits are direct delivery, one approval item, and FR-11. After approval, code delivers that same stored draft; the representative who recorded approval is the sender (MD-09).

Figure: [`../diagrams/system-arch/workflow-boundary.svg`](../diagrams/system-arch/workflow-boundary.svg). The editable HTML is `project-context/diagrams/system-arch/workflow-boundary.html`.

### Reference Architecture

| Shape | Use when | Known failure to prevent |
|-------|----------|--------------------------|
| Routing | The question ends in direct delivery, the one gate, or FR-11 | No human escalation; an agent shipped before the routed workflow was measured; a technical failure shown as an answer |
| Retrieval | The draft cites a stable approved library | Retrieval used as a substitute for live state |

- **Selected reference architecture**: Routing
- **Known failure this design prevents**: A missing human gate, an agent loop shipped in place of the routed steps, a technical failure shown as a supported answer or turned into an approval item, and a waiting item closed by a clock instead of explicit approval. The router is code. Direct delivery happens only for an exactly `supported` set whose citations resolve and whose fee, rate, account status, policy, and loan-fact claims appear in those cited articles. Every escalation reason opens the one representative gate. A `technical_failure` uses FR-11, not the gate, and not a fourth escalation reason. A `provider_timeout` records `unsupported` and uses the gate (CR-16). A `performance_miss` keeps the functional disposition. Once `waiting`, only Record approval ends the wait (CR-17).
- **Second pattern**: Retrieval. Approved articles are the only citable corpus. Loan mocks and customer records are searchable fixtures, not live servicing state, and not approved articles. Finding a loan mock is a tool result. It is not a citation, and it does not answer a customer-specific question (MN-05, US-004). The CR-12 algorithm reads cited article text. It is not a live loan lookup.

### Model and Context Strategy

- **Default model tier**: balanced default. The pinned model id is `gpt-4o-mini`. A swap is a release under the rule below.
- **Context strategy**: progressive. Each step receives only the fields that step needs.
- **Swap-as-release rule**: A model change or a prompt change is a release. `@qa.eng` runs the evaluation criteria table in section 9 before the swap. The rollback criterion is written before the swap: the previous model and prompt pair remains runnable; the fixture set in FR-10, FR-11, and FR-12 still matches on that pair; the fixture/stand-in path still completes in under 5 seconds (EC-04); the recorded model or provider cost of that question stays at most 20 cents (MC-01). A live `performance_miss` is not by itself a rollback.
- **Entry point**: direct API
- **Entry-point tradeoff named at decision time**: The crew runtime calls the model provider. The application keeps the route in code, so a supported answer is the only text the customer receives on the happy path. A hosted chat would put the customer in a vendor thread where the model could answer before the router runs.
- **Entry-point responsibility map**: One route. The customer surface and the representative surface talk to the workflow. The workflow is the only caller of the model. `AAMAD_TARGET_RUNTIME=crewai` is that caller. It is not a second product entry.
- **Minimum data in the context window**:
  - Classify: the question, plus search hits. An approved-article hit may include the passage the draft will cite. A loan-mock hit includes the record id and the type `loan_mock` only. The hidden expected result is never included.
  - Draft: the question, the stored reason set, and the approved passages. Loan-record field values are not included.
  - Claim-in-article check: the Draft `claims` list and the cited approved article texts. No model call. No second judge.
  - Router, customer view, daily count, cost record, latency record (`elapsed_ms`, `performance_miss`, `failure_identity`), and post-approval delivery: no model call. They read the stored set, the citation check, the claim check, and the stored draft.
- **Identity**: The test distinguishes the customer role from the representative role in application code. The prompt does not grant either role. After approval, the representative who recorded it is the sender (MD-09). A real customer or employee directory is out of scope (PRD Integration Requirements).

This entry point is the product’s route to the model. It is not the IDE chosen at `aamad init`. It is the CrewAI runtime selected for the generated application.

Resolved runtime controls for the generated crew, from `.cursor/rules/adapter-crewai.mdc`, and the paid-run pins below:

- Model id: `gpt-4o-mini`. Credential env var name: `OPENAI_API_KEY`. Setup writes that name into `.env.example`. The value is never in this SAD.
- Sequential process. Hierarchical process is not used.
- `memory=False`.
- `max_retry_limit` 2.
- `max_iter` 6 per model task (still ≤ 12).
- Temperature 0 for a repeatable fixture run. This is not an accuracy target.
- `max_rpm` 60 at crew level.
- `max_execution_time` 15 seconds is the hang stop (provider timeout), not the 5-second tripwire. A successful live result after 5 seconds is a `performance_miss`, not a hang.
- Two model tasks only. No max-token figure is pinned; the 20-cent soft stop in CR-15 is the spend brake. The 5-second rule is CR-16.

### Control Register

Training-time behavior is not a control this MVP can set. The application enforces the rows below. Side effects fail closed: a failed check does not deliver an answer and does not release an escalated draft.

The representative gate is the review route for the three escalation reasons. The approval item holds the question and the draft (FR-07). The reason set stays in the run record for the test. This table does not add the reason set to the representative’s item. A `technical_failure` uses FR-11, not the gate. A `provider_timeout` records `unsupported` and uses the gate (CR-16). A `performance_miss` is not a failure of this table. Once the gate is open, waiting continues until explicit approval (CR-17).

| ID | Risk category | Control | Control point (input / output / tool-call) | Failure direction (fail closed / fail open) | Owner | Evidence artifact | Review routing |
|----|---------------|---------|--------------------------------------------|---------------------------------------------|-------|-------------------|----------------|
| CR-01 | Direct injection in the question | The question is data. Classify must return `ClassifyOutput`. A parse miss, an extra reason, or `supported` together with an escalation reason is not stored as `supported`. Code records `unsupported` (and any valid escalation reasons that did parse) and opens the one gate. | output | fail closed | Workflow code | Stored `reason_set` | The one representative gate. Not FR-11. |
| CR-02 | Indirect injection in a fixture article | A source id the resolver cannot match to an article the fixture marks approved is rejected. The model cannot add an id the fixture did not mark approved. A draft whose citations do not all resolve is not a completed workflow. | output | fail closed | Workflow code | Citation check | Not the representative gate. FR-03 and FR-07 both require resolved citations. The customer sees the FR-11 sentence. No approval item is created. |
| CR-03 | Token-budget exhaustion | A model call that stops on the runtime limit is a `technical_failure` (CR-13), not a `performance_miss` and not a `provider_timeout`. No character or token cap is sourced. | input | fail closed | Workflow code | Task stop on the question trace with `failure_identity=technical_failure` | Not the representative gate. FR-11. The numeric token cap remains an Open Question. |
| CR-04 | Tool abuse, including a live system of record | The only tool is fixture search. No call is bound to a mortgage servicing platform, CRM, Salesforce, Zendesk, or the open web. | tool-call | fail closed | Workflow code | Tool list in the crew config | None. The call is not available. |
| CR-05 | Loan-field exposure and a mock treated as a lookup | A loan-mock hit returns the record id and the type only. A question whose answer needs a fact on the MD-06 list still opens the gate when that hit exists. A notice of error, a qualified written request, or a request for information includes `customer-specific loan information` (MD-08). | tool-call | fail closed | Workflow code | Tool-result schema and the stored reason set | The one representative gate when the set contains that reason (MN-05, US-004). |
| CR-06 | Direct delivery of an answer that is not exactly supported | The router delivers only when the stored set is exactly `supported`, every source reference resolves, and the claim-in-article check has passed. Any escalation reason creates one approval item. Two approved articles that disagree add `cannot be determined to be safe` (MD-07). | output | fail closed | Workflow code | Disposition on the question trace | The one representative gate. |
| CR-07 | Draft shown before approval | While the approval item is open, the customer view shows the waiting state and does not present the draft as the answer. Waiting does not expire. See CR-17. | output | fail closed | Workflow code | Customer-view state | The representative already holds the item (FR-08). |
| CR-08 | A new draft after approval, or a separate sender | Approval copies the stored draft and its source references. The model is not called. The representative who recorded approval is the sender (MD-09). | output | fail closed | Workflow code | Same stored draft before and after approval; approval actor on the trace | The representative’s approval is the release and the send (FR-07, MD-09). |
| CR-09 | A second approver or an approval on a supported answer | One approval item, one representative role, no second sender, and no approval item when the set is exactly `supported`. | output | fail closed | Workflow code for the rule. The representative for the approval itself. | Approval record | The one gate. A second item fails the test (MN-03, US-004). |
| CR-10 | Secret exposure | The model credential is read from `OPENAI_API_KEY`. It is not written into prompts, fixtures, or logs the MVP retains. Setup lists that name in `.env.example`. | input | fail closed | Workflow code | Env var name `OPENAI_API_KEY` in `.env.example` only | None. |
| CR-11 | Decision log | Each question trace stores the question id, the reason set, the source ids, the citation check, the claim-in-article result, the disposition, the recorded model or provider cost, `elapsed_ms`, `performance_miss` when set, the daily submitted-question count, and, when used, the approval actor as sender or a distinct `failure_identity` (`provider_timeout` or `technical_failure`). The 201st-question log is this same trace with the exceeded-200 mark. No secret value. | output | fail closed | Workflow code | Question trace | The test reads the trace. The representative reads the question and the draft. |
| CR-12 | Ungrounded fee, rate, account status, policy, or loan-fact claim | Code runs the claim-in-article algorithm below. A failing claim adds `unsupported` and removes `supported` before the router runs. | output | fail closed | Workflow code | Per-claim result on the question trace | The one representative gate (FR-04, MP-03). This is not FR-11 and not a fourth reason. |
| CR-13 | Incomplete workflow shown as an answer | When the chain cannot complete and the miss is a `technical_failure` — exhausted retries, a missing tool, a crash, or the CR-15 soft stop skipping a required model task — the customer sees “We are facing a technical issue at this point. Check back later.” The draft is not shown as the answer. No approval item is created. The error is logged with `failure_identity=technical_failure`. A Pydantic miss is CR-01. A provider hang is CR-16, not this row. | output | fail closed | Workflow code | FR-11 customer sentence and the error log | Not the representative gate. |
| CR-14 | A 201st question refused or treated as FR-11 | The daily count is a planning number. A 201st submitted question is processed the same as questions 1–200. The product logs that the day exceeded 200. The customer does not see a capacity refusal. | input | fail closed | Workflow code | Daily count and the exceeded-200 mark on the question trace | None. The question follows FR-06, FR-07, or FR-11 on its own merits. |
| CR-15 | Unrecorded or uncapped model cost | One submitted question is one call and one transaction. After each model call, code adds the provider cost to the question’s running total and stores it on the trace. Before starting the next model call, if that running total is already at or above 20 cents, the next call is not started. If a required Classify or Draft call is skipped, or if the total after a call is above 20 cents, the result is FR-11 with `failure_identity=technical_failure`, not a supported delivery and not a fourth path. | output | fail closed | Workflow code | Cost record on the question trace | Not the representative gate. FR-11. |
| CR-16 | Latency tripwire mixed with functional failure | Code always records `elapsed_ms` from submit until the customer-visible automated result. Fixture/stand-in runs must complete that result in under 5 seconds (EC-04). Live runs that finish with `supported` or `waiting` after 5 seconds set `performance_miss` and keep that disposition; they are not FR-11 and not a functional fail. A provider hang (`max_execution_time`) is `failure_identity=provider_timeout`: record `unsupported`, open the one gate, store whatever draft body exists; FR-11 only if that item cannot be created. A crash or missing tool is `technical_failure` (CR-13), logged separately from `provider_timeout` and from `performance_miss`. The 5-second tripwire and the 15-second hang stop do not apply after `waiting` is shown (CR-17). | output | fail closed | Workflow code | `elapsed_ms`, `performance_miss`, `failure_identity` on the question trace | `provider_timeout` uses the one gate. `technical_failure` is FR-11. `performance_miss` uses the functional disposition. |
| CR-17 | Waiting closed without explicit approval | Once disposition is `waiting`, it stays `waiting` until the representative role records approval on that item. Code does not auto-approve, auto-reject, auto-deliver, convert waiting to FR-11, or drop the item because of elapsed review time, `max_execution_time`, the 5-second tripwire, a session end, a calendar-day rollover, or the 201st-question log. A later customer read of the same `question_id` still returns `waiting` with the draft omitted. The PRD defines no reject action; an unapproved item remains open. | output | fail closed | Workflow code | Open approval item and customer-view state until the approval record exists | The one representative gate. Explicit approval is the only closer (FR-07, FR-08, MD-04). |

**Claim-in-article algorithm (CR-12 / MP-03)**

This is the code recipe. It is not a second model call and it does not scan free prose.

1. **What counts as a claim.** The Draft task’s structured output includes `claims`: a list of objects `{ "kind": ..., "text": ... }`. Allowed `kind` values are exactly `fee`, `rate`, `account_status`, `policy`, and `loan_fact` (MP-03). Customer-facing prose is a separate `answer` field. Only items in `claims` are claims. Ordinary sentences in `answer` are not scanned.
2. **Normalize.** For a `text` string and for each cited approved article body: Unicode casefold, trim, collapse any whitespace to a single space. Do not strip `$` or digits. After normalize, `text` must be non-empty. Empty `text` fails that claim.
3. **“Appears in a cited article.”** The claim passes if the normalized `text` is a substring of at least one cited article’s normalized body, and that article’s id is in `source_refs` and the fixture marks it approved. The claim fails otherwise.
4. **Disposition.** If any claim fails, the recorded set includes `unsupported` and does not contain `supported`. The router then opens the one gate. If `claims` is empty, this check passes; MP-03 applies only to emitted claims of those kinds.
5. **Schema miss.** A `kind` outside the allowed list, a missing `claims` list, or any other `DraftOutput` / `ClassifyOutput` parse miss is `unsupported`, not FR-11. The recorded set includes `unsupported` and does not contain `supported`. The router opens the one gate. Code does not invent `kind` or `text` values.

MP-03 fixtures must make this observable: the supported case emits claim `text` that is already in a cited article; the unsupported case emits at least one `text` that is not.

**Latency tripwire (CR-16)**

The 5-second figure is measured. It is not a silent hope, and it is not an automatic FR-11.

1. **Measure.** Every question records `elapsed_ms` from submit until the customer-visible automated result (supported answer, waiting state, or FR-11). Representative review is outside this interval.
2. **Fixture / stand-in path.** That path must complete in under 5 seconds with the functional outcome the fixture requires. A stand-in model does not skip this budget. Exceeding 5 seconds fails EC-04. FR-11 is not a way to pass the clock.
3. **Live path.** Total processing time is always stored. If disposition is `supported` or `waiting` and `elapsed_ms` is over 5000, set `performance_miss`. Keep the disposition. Do not convert it to FR-11. A performance miss is not a functional failure.
4. **Provider timeout.** When `max_execution_time` (15 seconds) fires, log `failure_identity=provider_timeout`. Record `unsupported` and open the one gate, with the question and any draft body that exists. Do not automatically generate FR-11. Use FR-11 only if the approval item cannot be created.
5. **Technical failure.** Exhausted retries, missing tool, crash, or CR-15 skip: log `failure_identity=technical_failure` and use FR-11 (CR-13). That identity is never stored as `provider_timeout` or `performance_miss`.

**Explicit-approval wait (CR-17)**

The 5-second clock ends when the customer can see `waiting`. Review time is not that clock.

1. **Closer.** The only event that ends waiting is the representative role recording approval on that item (FR-07, MD-04).
2. **Persist.** Later reads of the same question still return `waiting` with the draft omitted (FR-08, US-003). Calendar-day rollover and the 201st-question log do not close open items (CR-14).
3. **No invented closer.** Code does not auto-approve, auto-reject, auto-deliver, or turn waiting into FR-11 because the representative has not acted. The PRD has no reject action.
4. **Not CR-16.** A provider hang happens before `waiting` is shown. After `waiting` is shown, `max_execution_time` and the 5-second tripwire do not fire again on that question.

Fairness injection points are not a PRD requirement. No fairness row is added. Corpus, prompt framing, examples, and routing stay as the fixture library, the two bounded tasks, and the code router above.

### Production Shape

- **Volume**: The planning number is 200 submitted questions in a day. A 201st question is processed and logged (FR-12, MC-01). The fixture set that must pass is FR-10 plus FR-11 plus FR-12: one supported question, one question for each escalation condition alone, one question whose hidden result lists two escalation reasons, the three written-request labels, one technical failure, and one 201st-question case. This increment is not a production servicing operation (MRD §4). No concurrency number and no uptime number are sourced.
- **Cost and latency at that volume**: Fixture/stand-in runs must finish in under 5 seconds (CR-16, EC-04). Live runs always record `elapsed_ms`. A live success over 5 seconds is a `performance_miss`, not FR-11. Cost: at most 20 cents recorded, with the CR-15 soft stop. Representative review is outside the interval and continues until explicit approval (CR-17).
- **Retries**: A model task may retry up to `max_retry_limit` 2. A retry repeats that task. It does not skip the claim-in-article check. It does not generate a new draft after approval. The only allowed edit of the reason set after the draft is the claim-in-article update before disposition (CR-12).
- **Fallback**: Exhausted retries, a missing tool, a crash, or a CR-15 skip use FR-11 with `technical_failure`. A hang at `max_execution_time` uses `provider_timeout` and the one gate (CR-16), not automatic FR-11. Neither path delivers a supported answer from a failed chain, invents a fourth escalation reason, or refuses a 201st question.
- **Circuit breaker**: The breaker opens for that question when the retry limit is exhausted. Later questions in the same day, including a 201st, may still call the model. No separate failure-count threshold is sourced. An open `waiting` item is not a broken circuit and is not closed by this breaker (CR-17).
- **Failure mode of this architecture**: A `supported` set that reaches the customer without passing the router, a technical failure shown as an answer, or a new draft sent after approval. Mitigation: the customer view reads the router’s disposition only (CR-06, CR-13); FR-11 is a logged sentence with no approval item; post-approval delivery copies the stored draft and attributes the send to the representative (CR-08).
- **Traces and dashboards**: One trace per question, as in CR-11. The exceeded-200 mark is on that trace. No aggregate dashboard is sourced. MRD §4 leaves hosting and a monitoring product to a later deployment.

### 1. MVP Architecture Philosophy & Principles

**MVP Design Principles**:

- Customer and representative feedback is the fixture set in FR-10, FR-11, and FR-12, not a live servicing panel.
- Simplest orchestration that delivers core value: the recorded workflow (chaining, then routing) with two bounded model tasks and a code router.
- Observable by default: one question trace (CR-11) and a health-check endpoint. No aggregate dashboard.
- Automated deploy scaffolding waits for Deliver. This increment must run its fixture tests without a live system of record.

**Core vs Future Features**:

- **MVP**: FR-01 through FR-12. Customer submits one natural-language question. Code counts the day. Classify and draft are the two model tasks. Search, citation check, claim-in-article check, and the router are code. Direct delivery of a supported answer is the only customer success path. The one representative gate holds the question and the draft; the representative who records approval is the sender. The customer sees a waiting state or the FR-11 sentence. Fixtures are synthetic. Drafting may call a model.
- **Future (P2)**: A live mortgage servicing platform, CRM, Salesforce, or Zendesk connection; a real identity system; Regulation X notice clocks; an uptime, concurrency, readability, or accuracy-percentage number; a second customer channel or happy path; measurement of complaint reduction, satisfaction, containment, savings, or ROI.
- **Explicit exclusions**: No second approval gate. No approval of a supported answer. No live loan lookup. No evaluator-optimizer. No agent loop. No default 3–4 agent crew.

**Technical Architecture Decisions**:

- Frontend framework and UI library are not selected. `aamad.config.yml` does not set `language` or `ui`. Channel, layout, waiting-notice wording, and a disability-access standard belong to `@ux.designer` in `ux.md`. This SAD does not default to a chat interface or a dashboard.
- Human–system interaction is two roles on one workflow: the customer starts the question and reads the supported answer, the waiting state, or the FR-11 sentence; the representative opens one approval item and records approval.
- Runtime communication is a synchronous HTTP API that invokes the sequential CrewAI crew and then the code checks and router. The customer-visible automated result is a complete response, not a stream. PRD does not require streaming.
- Selected runtime is `crewai`. Composition is sequential process, `allow_delegation=false`, YAML agent and task config, `Task.context` chaining from classify to draft.

### 2. Application Runtime Specification

The Pattern Decision is workflow (chaining, then routing). Runtime roles exist only as the two bounded model tasks. This is not an agent and not a 3–4 agent crew.

**Runtime roles**:

| Role | Goal | Tools | Memory | `output_pydantic` |
|------|------|-------|--------|-------------------|
| Classify | Propose the closed reason set from the question and fixture search | Fixture search only | None (`memory=False`) | `ClassifyOutput` |
| Draft | Write `answer`, `source_refs`, and `claims` from the stored set and approved passages | None. Search already ran. | None (`memory=False`) | `DraftOutput` |

**Pydantic output contracts**

These are the two named outputs. Build implements them as Pydantic models. Each CrewAI task sets `output_pydantic` to that class. A parse miss, a missing required field, an extra reason, an invalid `kind`, or `supported` together with an escalation reason is a schema miss: the recorded set includes `unsupported` and does not contain `supported`, and the one gate opens. It is not FR-11. Code does not treat a miss as `supported`.

`ClassifyOutput`

| Field | Type | Rule |
|-------|------|------|
| `reason_set` | list of strings | Required. Min length 1. Each item is exactly one of `supported`, `unsupported`, `customer-specific loan information`, `cannot be determined to be safe`. If `supported` is present, it is the only item. Duplicates are a miss. |

`DraftOutput`

| Field | Type | Rule |
|-------|------|------|
| `answer` | string | Required. The customer-facing prose. Empty string is a schema miss (`unsupported`, one gate). |
| `source_refs` | list of strings | Required. May be empty. Each item is an article id. |
| `claims` | list of `{ kind, text }` | Required. May be empty. `kind` is exactly `fee`, `rate`, `account_status`, `policy`, or `loan_fact`. `text` is a string (empty `text` fails CR-12 after parse, it is not a schema miss). |

**Task / Turn Orchestration**:

1. Code accepts the question, checks the role, increments the daily count, and writes the exceeded-200 mark when the day is already at 200 (FR-12).
2. Classify runs with `output_pydantic=ClassifyOutput`. It may call fixture search. A parse miss stores `reason_set` as `unsupported` (plus any valid escalation reasons that did parse) and still runs Draft (CR-01). A valid object is stored as `reason_set`.
3. Draft runs with `output_pydantic=DraftOutput` and `Task.context` from classify plus the approved passages. A parse miss adds `unsupported`, removes `supported`, and opens the one gate. The approval item holds the question and the unparsed draft body so FR-07 can copy a stored body after approval. `source_refs` and `claims` are empty. The customer does not see that body until approval. CR-12 is not run. It does not receive loan-record field values.
4. When `DraftOutput` parsed, code runs the citation check and the CR-12 algorithm. A failing emitted claim adds `unsupported` and removes `supported`. Unresolved citations are FR-11 (CR-02).
5. Code routes: exactly `supported` with resolved citations and a passing claim check → deliver; any escalation reason → one approval item; `technical_failure` → FR-11; `provider_timeout` → `unsupported` and the one gate (FR-11 only if that item cannot be created). Record `elapsed_ms`. If disposition is `supported` or `waiting` and `elapsed_ms` is over 5000, set `performance_miss`.
6. After approval, code copies the stored `DraftOutput`. The model is not called. The representative who recorded approval is the sender. Until that approval is recorded, later reads keep `waiting` (CR-17).

Expected outputs:

- `ClassifyOutput.reason_set` as above.
- `DraftOutput` as above.
- Disposition: `supported` | `waiting` | `technical_issue`.
- Trace fields in CR-11, including recorded cost, `elapsed_ms`, `performance_miss`, and `failure_identity` when used.

Error handling: `max_retry_limit` 2 per model task. Exhausted retries and CR-15 skips use FR-11 with `technical_failure`. A hang at `max_execution_time` 15 seconds is `provider_timeout` (CR-16), not automatic FR-11.

**Runtime-Conditional Configuration — crewai**:

- Files: `config/agents.yaml`, `config/tasks.yaml`, `crew.py`.
- Process: sequential. Hierarchical is not used.
- Classify task, then draft task, with explicit `Task.context`. Classify sets `output_pydantic` to `ClassifyOutput`. Draft sets `output_pydantic` to `DraftOutput`.
- `allow_delegation=false`.
- `memory=False`.
- `max_retry_limit` 2. `max_iter` 6 per model task.
- Temperature 0. Model id `gpt-4o-mini`. `OPENAI_API_KEY` from the environment.
- `max_rpm` 60. `max_execution_time` 15 seconds (hang stop). The 5-second figure is CR-16, not this kill switch.
- The HTTP layer, not the crew, owns the daily count, citation check, claim check, router, customer view, approval copy, cost record, the CR-15 soft stop, the CR-16 latency record, and the CR-17 wait (open items persist until Record approval).

### 3. Frontend Architecture Specification

**Technology Stack** (from PRD or justified defaults):

- No framework, UI library, styling system, or device size is sourced. `@project.mgr` records the stack in `setup.md`. `@ux.designer` writes `ux.md` before `@frontend.eng` implements a surface.
- Type safety and state management wait for that stack choice. They are not selected here.

**Application Structure**:

- Two role surfaces, not a second product: customer and representative. Route and page names wait for `ux.md`.
- The frontend talks only to the Workflow API contract in section 4. It does not bind CrewAI, fixtures, or the model provider.
- Responsive behavior and accessibility baseline wait for `ux.md`. PRD “accessible” means the customer can receive a supported answer without first reaching a representative. It does not set a disability-access standard.

**Interface Requirements**:

- Primary interaction surface, from the PRD, not a presumed chat: the customer can submit natural language and read the supported answer and its source references; on an escalation the customer sees that the question is waiting on representative review and does not see the draft as the answer; on a technical failure the customer sees “We are facing a technical issue at this point. Check back later.” The representative can open the one approval item, read the question and the draft, and record approval.
- Loading lasts until the customer-visible automated result. Fixture/stand-in must finish in under 5 seconds. Live may be slower; that is a `performance_miss`, not FR-11. Recoverable technical failure is FR-11. Provider timeout uses the waiting gate (CR-16). The waiting state is not a failure and is not loading. Once `waiting` is shown, the surface keeps that state until the representative records approval (CR-17). It does not time out into FR-11 or auto-deliver.
- Future Work (P2) is not shown as a working control. If a later surface mentions a live servicing connection, it is marked future work.

### 4. Backend Architecture Specification

**API Architecture**:

Non-streaming JSON. One question is one request and one transaction (MC-01).

| Operation | Actor | Request | Response |
|-----------|-------|---------|----------|
| Submit question | customer | `question_id` optional; `question` string | `question_id`; `disposition` (`supported` \| `waiting` \| `technical_issue`); on `supported`, `answer` and `source_refs`; on `waiting`, no draft; on `technical_issue`, the FR-11 sentence; test-readable `reason_set`, `recorded_cost`, `elapsed_ms`, `performance_miss` when set, `failure_identity` when used, citation and claim-check results |
| Read question result | customer | `question_id` | Same shape. While `waiting`, draft is omitted and the disposition stays `waiting` on every later read until approval is recorded (CR-17). After approval, the stored draft and `source_refs` |
| Open approval item | representative | `question_id` | Question text, draft, `source_refs`. Not the hidden expected result |
| Record approval | representative | `question_id`; actor is the representative role | Approval recorded. This is the only closer of `waiting` (CR-17). Subsequent customer read returns that same stored draft. Actor stored as sender |

Validation: role is checked in application code. `reason_set` must match the closed schema before the router runs (CR-01). Unknown fields are ignored. Rate limiting does not refuse a 201st question (CR-14).

Error envelope: a `technical_failure` returns `disposition=technical_issue` with the FR-11 sentence and `failure_identity=technical_failure`. A `provider_timeout` returns `disposition=waiting` with `failure_identity=provider_timeout` (FR-11 only if the approval item cannot be created). Neither is a fourth escalation reason. A `performance_miss` is not this envelope.

**Data Architecture**:

Minimal store, required by FR-07, FR-08, FR-12, and CR-11:

- Invented fixture library (approved articles, customer records, loan mocks).
- Question traces (CR-11).
- Open approval items and the stored draft. An open item survives calendar-day rollover until Record approval (CR-17).
- Calendar-day submitted-question count.

No live servicing database. A durable production store is Future Work. Setup may use an in-process or file store so tests can read traces and the exceeded-200 log.

**Runtime Integration Layer**:

- The HTTP handler increments the daily count, invokes the sequential crew (classify, then draft), runs citation and claim checks, records cost, and writes disposition.
- Crew config is YAML. Secrets are environment variables named in `.env.example`.
- Prompt Trace and task lifecycle logs follow the CrewAI adapter. Secrets are redacted. Logs the MVP retains for the test live under `project-context/2.build/logs` once Build starts.

**Authentication & Secrets**:

- The test distinguishes the customer role from the representative role in application code. No real customer or employee directory.
- Model credential: `OPENAI_API_KEY` only. Setup lists that name in `.env.example`. No secret values in this SAD, fixtures, or traces (CR-10).

### 5. DevOps & Deployment Architecture

**CI/CD** (minimal MVP): lint, test, and build, aligned to the stack in `setup.md`.  
**Hosting**: smallest MVP-appropriate target that can run the fixture tests and the Workflow API. No cloud provider is sourced. Health-check endpoint: a liveness route that does not call the model. Port and image wait for Deliver.  
**IaC / multi-region / advanced monitoring**: Future Work.  
**Observability**: per-request traces (CR-11) are in scope. Aggregate dashboards are not.

### 6. Data Flow & Integration Architecture

Request path: customer or representative surface → Workflow API → daily count → sequential crew (classify may search fixtures; draft uses approved passages) → citation check and claim-in-article check → router → customer-visible result or one approval item. After approval, the API copies the stored draft to the customer read. The model provider is called only from the crew. Live servicing systems are not called.

Figure for the in-process chain: [`../diagrams/system-arch/workflow-boundary.svg`](../diagrams/system-arch/workflow-boundary.svg). The editable HTML is `project-context/diagrams/system-arch/workflow-boundary.html`.

Error propagation: escalation reasons open the waiting state (FR-08). That state persists until Record approval (CR-17). A `technical_failure` returns FR-11 and writes the error log (CR-13). A `provider_timeout` records `unsupported` and opens the waiting gate (CR-16). The frontend does not invent a different sentence.

### 7. Performance & Scalability Specifications

- Response time: always record `elapsed_ms`. Fixture/stand-in must complete in under 5 seconds. Live success over 5 seconds is `performance_miss` (CR-16). Representative review is outside that interval and has no SLA clock; waiting continues until explicit approval (CR-17).
- Cost: recorded model or provider cost at most 20 cents per question (MC-01, CR-15). Two model tasks only. Soft stop: do not start the next model call if the running total is already at or above 20 cents; skipped required call or total above 20 cents is FR-11.
- Volume: 200 submitted questions in a day as a planning number. A 201st is processed and logged (FR-12).
- Concurrency, throughput beyond that planning number, and uptime are not sourced. Scaling path is deferred with P2. This `--mvp` SAD does not invent them.

Token and cost controls at the runtime layer: `gpt-4o-mini`, temperature 0, `max_retry_limit` 2, `max_iter` 6, `max_rpm` 60, `max_execution_time` 15 seconds (hang stop), two sequential tasks, CR-15 soft stop at 20 cents, CR-16 latency tripwire.

### 8. Security & Compliance Architecture

- AuthN/AuthZ for MVP: role distinction in application code (customer vs representative). No SSO.
- Encryption and input validation: question is data (CR-01); HTTPS and at-rest encryption wait for Deliver if hosting requires them. They are not sourced for this capstone.
- Control Register (CR-01 through CR-17) is the application-layer safety design. `@security.eng` verifies it later. `security.require_security_assessment` is the 1.0 default and is not set in this project’s `aamad.config.yml`; section 9 still requires that assessment before Deliver unless the operator sets the flag false.
- This capstone is not a compliance determination under the OCC framework or Regulation X (PRD §5). Inaccurate fee, rate, account status, policy, or loan-fact claims are why CR-12 and the gate exist.

### 9. Testing & Quality Assurance Specifications

- Unit: `ClassifyOutput` and `DraftOutput` parse (pass and miss), citation resolver, CR-12 algorithm, router, daily count, cost record, `elapsed_ms` / `performance_miss` / `failure_identity`, waiting persistence until Record approval.
- Integration: sequential crew with fixture search; HTTP operations in section 4.
- Smoke/acceptance: FR-10, FR-11, and FR-12 fixtures. Hidden expected result is not given to the product.
- Runtime-specific: YAML-referenced tools bind; classify and draft use `output_pydantic`; a schema miss records `unsupported` and opens the one gate; traces include `elapsed_ms`, `performance_miss`, and distinct `failure_identity`; secrets redacted.
- `*verify-change` before Deliver.
- Security assessment before Deliver when `security.require_security_assessment` is true (1.0 default).
- `quality.require_evals` is the 1.0 default. The table below is the pass/fail contract. `@qa.eng` `*run-evals` builds the golden dataset, graders, and `evals.md`.

**Evaluation Criteria**:

Derived from PRD KPIs and MC-01. No accuracy percentage and no readability score are set. Thresholds not in the PRD are not invented.

| ID | Dimension | Metric | Threshold | Grading Method | Source |
|----|-----------|--------|-----------|-----------------|--------|
| EC-01 | Accuracy | Recorded reason set vs hidden expected result | Exact set match. `supported` is exclusive of escalation reasons. | Code-based | MP-01; FR-02; FR-10 |
| EC-02 | Accuracy | Source references on a completed draft | Every source id resolves to an article the fixture marks approved. Unresolved citations are FR-11, not a supported or gated pass. | Code-based | MP-02; FR-03; CR-02 |
| EC-03 | Accuracy | Fee, rate, account status, policy, and loan-fact claims | Every emitted `claims[].text`, after CR-12 normalize, is a substring of a cited approved article. A failing claim makes the recorded set include `unsupported` and not `supported`. No second model. | Code-based | MP-03; FR-04; CR-12 |
| EC-04 | Latency | Fixture/stand-in wall clock | Under 5 seconds to the functional outcome the fixture requires (supported, waiting, or the FR-11 fixture). A late fixture run fails. Live over-5-second success is EC-10, not this fail. | Code-based | MC-01; CR-16 |
| EC-05 | Safety | Direct delivery vs the one gate | Direct delivery only when the recorded set is exactly `supported` and EC-02 and EC-03 pass. Any escalation reason creates exactly one approval item. The customer does not receive that draft before approval. | Code-based | FR-05; FR-06; FR-07; CR-06; CR-07 |
| EC-06 | Safety | Incomplete workflow (`technical_failure`) | Customer sees “We are facing a technical issue at this point. Check back later.” Draft is not the answer. No approval item. Error log identifies the question and `failure_identity=technical_failure` and contains no secret. Provider timeout is EC-10, not this row. | Code-based | FR-11; CR-13 |
| EC-07 | Security | Systems of record, fixtures, secrets | No call to a live servicing platform, CRM, Salesforce, Zendesk, or other system of record. Fixtures are invented and are not a servicing extract. Model credential is an env var. Logs and fixtures contain no secret value. | Code-based | MN-01; FR-09; CR-04; CR-10 |
| EC-08 | Cost | Recorded model or provider cost per question | At most 20 cents. The product records that cost. A required model call skipped by the soft stop, or a total above 20 cents, is FR-11. | Code-based | MC-01; CR-15 |
| EC-09 | Safety | 201st submitted question in a day | Processed the same as questions 1–200. Log that the day exceeded 200. No capacity refusal. Not FR-11 because the day exceeded 200. | Code-based | FR-12; CR-14 |
| EC-10 | Latency | Live `elapsed_ms` and event class | Every live question stores `elapsed_ms`. Success over 5 seconds sets `performance_miss` and keeps the disposition. `provider_timeout` and `technical_failure` are distinct identities. A `performance_miss` is not FR-11. | Code-based | CR-16 |
| EC-11 | Safety | Open waiting item without approval | Later customer reads still return `waiting` with the draft omitted. No auto-approve, auto-reject, auto-deliver, or conversion to FR-11. Calendar-day rollover does not close the item. Waiting ends only when Record approval runs. | Code-based | FR-07; FR-08; CR-17 |

### 10. MVP Launch & Feedback Strategy

- This is a six-week capstone on mocked data. There is no commercial beta or launch market (PRD §9).
- Success is FR-10, FR-11, and FR-12 passing, fixtures meeting EC-04 and EC-08, and live questions recording `elapsed_ms` under CR-16. CFPB counts and J.D. Power scores stay context. They are not before/after metrics.
- Outcome document fields for Deliver: before metric = fixture pass rate against EC-01 through EC-11 on the FR-10/11/12 set; auditable control = CR-06, CR-12, CR-13, CR-16, and CR-17; measurement owner = `@qa.eng` for evals, `@security.eng` for the Control Register assessment; after metric = the same fixture set after the first Deliver package, recorded when the operator authorizes that run.

## Implementation Guidance for AI Development Agents

1. Foundation setup per `setup.md` epic (`@project.mgr`). Use model `gpt-4o-mini`, `OPENAI_API_KEY` in `.env.example`, `max_iter` 6, `max_rpm` 60, `max_execution_time` 15 seconds (hang stop), the CR-15 soft stop, the CR-16 latency tripwire, and persist open approval items until Record approval (CR-17).
2. UX (`ux.md`) — in scope. Two roles, one customer success path, waiting state, FR-11 sentence (`@ux.designer`).
3. Frontend MVP UI without backend wiring (`@frontend.eng`).
4. Backend runtime scaffolding that implements the Pattern Decision (`@backend.eng`): sequential CrewAI crew, fixture search, code checks, router, traces.
5. Integration epic wires FE ↔ BE (`@integration.eng`).
6. QA validates unit, integration, smoke, evals against EC-01 through EC-11, and `*verify-change` (`@qa.eng`). Security assessment (`@security.eng`).
7. Deliver packages deploy/CI/runbook and the outcome document (`@devops.eng`). No live deploy without operator authorization.

## Architecture Validation Checklist

- [x] PRD requirements mapped to architectural components (sections 2–6; FR-01 through FR-12)
- [x] Work Ownership, Pattern Decision, Reference Architecture, Model and Context Strategy, and Control Register are complete
- [x] Runtime roles exist only as the two bounded model tasks inside the workflow
- [x] Frontend and backend contracts agree on schemas / streaming: non-streaming JSON operations in section 4; frontend consumes only that API; draft omitted while `waiting`
- [x] Secrets via env vars only (CR-10)
- [x] MVP vs Future Work boundaries explicit: P2 in the PRD stays out of these decisions; scale deferred in section 7
- [x] Resolved `AAMAD_TARGET_RUNTIME` recorded in Audit

## Sources

- `project-context/1.define/prd.md` — FR-01 through FR-12, MD-01 through MD-09, MN-01 through MN-06, MC-01, MP-01 through MP-03, §§5–6. Operator decisions of 2026-11-14 and 2026-11-17 close the questions this file previously carried from the PRD. Open Questions in that PRD: none remaining.
- `project-context/1.define/mrd.md` — Context (primary focus, approval policy, escalation conditions, mocked data), §2, §4, and Risk Assessment Matrix. Market figures are context and are not thresholds.
- `project-context/1.define/user-stories/US-001.md`, `US-002.md`, `US-003.md`, `US-004.md` — acceptance criteria already traced to the PRD. No story adds a requirement the PRD lacks. US-002 criterion 12 carries FR-12. US-004 criterion 13 carries MD-09.
- `aamad.config.yml` — `project.mode: greenfield`, `runtime.target: crewai`.
- `.cursor/templates/sad-template.md` — decision headings and sections 1–10. This write is `*create-sad --mvp`.
- `.cursor/rules/adapter-crewai.mdc` — sequential process, YAML config, `allow_delegation`, `max_iter`, `max_retry_limit`, `max_rpm`, `memory=False`.
- `.cursor/templates/aamad-config-template.yml` — 1.0 defaults `security.require_security_assessment: true` and `quality.require_evals: true`. This project’s `aamad.config.yml` does not set those keys.
- System description: not present.
- `ux.md`: not present.

## Assumptions

- `AAMAD_TARGET_RUNTIME` is unset. Resolution uses `aamad.config.yml` `runtime.target`, which matches the adapter default `crewai`.
- “Existing system” in Work Ownership is the workflow code this SAD specifies.
- The classifier may call fixture search before the reason set is stored, so that `unsupported` and `cannot be determined to be safe` can be judged against the approved material. The test observes a classification outcome before the draft’s source references (MD-02). The CR-12 algorithm may then add `unsupported` and remove `supported` before disposition. That is the only allowed edit of the set after the draft.
- CR-12 checks only emitted `claims`. It does not scan `answer` prose. MP-03 fixtures must emit the claims the test needs to observe. Empty `claims` passes this check. Empty `text` after normalize fails that claim. `$` and digits are kept. There is no second-model judge and no fuzzy or embedding match.
- Customer-specific, approved, conflict, notice-of-error / qualified-written-request / request-for-information, claim-in-article, synthetic fixtures, the technical-issue sentence, the model-provider rule, the 200 / 5-second / 20-cent limits, the 201st-question log, and the representative-as-sender rule are the operator decisions recorded in the PRD. Work Ownership uses those closed meanings. It does not invent a further definition for a live servicer.
- The representative’s only MVP action is to record approval. The PRD does not define reject, edit, or a second approver. The representative who records approval is the sender (MD-09). Until that approval is recorded, the customer remains on the waiting state (CR-17). Provider hang and the 5-second tripwire are CR-16, not a review SLA.
- A loan-mock search hit carries the record id and the type `loan_mock`, and does not carry loan field values, into the model context. The record is still in the searchable fixture data. This is the control for MRD Risk and MN-05.
- An article is approved for the MVP when the fixture marks it approved. Two approved articles that disagree on a fact the answer needs add `cannot be determined to be safe`.
- Temperature 0 and `max_retry_limit` 2 are runtime controls for a repeatable fixture run. They are not quality targets from the PRD. `max_rpm` 60 is the crew brake. `max_execution_time` 15 seconds is the hang stop. The 5-second figure is the CR-16 tripwire: hard on fixtures, measured on live. The 20-cent figure is the per-question cost cap and the CR-15 soft stop.
- The model provider is not a system of record. Drafting may call a model. The PRD does not require the draft step to run without a model.
- The six-week cohort is a scope constraint from the MRD, not a schedule estimate.
- Pattern Decision and Reference Architecture in this file are the 2026-11-17 pass. They keep the primary shape `workflow` and the sub-shape chaining then routing. FR-11 is a router exit, not a fourth escalation reason. The CR-12 algorithm is a code step in the chain, not an evaluator-optimizer.
- An unresolved source reference cannot complete either customer path, so CR-02 uses FR-11. A failing CR-12 claim completes the path as `unsupported` and uses the one gate. A `ClassifyOutput` or `DraftOutput` parse miss also records `unsupported` and uses the one gate. It is not FR-11. On a Draft parse miss, the unparsed body is stored for the representative and is not shown to the customer until approval, so FR-07 still has a stored draft to copy. A provider hang follows the same gate path (CR-16) and is not FR-11 unless the item cannot be created.
- Paid-run pins: model `gpt-4o-mini`; env var name `OPENAI_API_KEY`; `max_iter` 6; `max_rpm` 60; `max_execution_time` 15 seconds (hang stop). A model swap is a release. Setup writes the env var name into `.env.example` and does not write the secret.
- Operator latency policy (2026-11-24): a live success over 5 seconds is a `performance_miss`, not FR-11. Provider timeout uses the one gate unless the item cannot be created. Technical failure stays FR-11. Fixture/stand-in still must finish in under 5 seconds. The 15-second hang stop is what lets a slow live success exist so it can be logged as a miss rather than killed at 4 seconds.
- Operator wait policy (2026-11-24): the system continues waiting until the representative explicitly records approval. No review clock, session end, day rollover, or CR-16 tripwire closes an open item.
- This `--mvp` SAD does not select a frontend framework, UI library, cloud provider, or port. Those wait for `ux.md`, `setup.md`, and Deliver.
- CrewAI implies a Python runtime for the generated backend. That is an adapter consequence, not a `language.primary` value from this project’s config.
- Section 9 evaluation criteria are the PRD pass/fail contract. Filling them here is the `*create-sad --mvp` write of that table. Golden dataset and graders remain `@qa.eng` `*run-evals`.
- The Workflow API operations in section 4 are the smallest contract that makes FR-01 through FR-12 observable. Field names may be refined in `backend.md` without changing the dispositions.

## Open Questions

Closed by the PRD of 2026-11-17 and no longer open for architecture:

- Which facts count as customer-specific loan information? MD-06.
- What makes an article approved in this MVP, and what happens when two approved articles conflict? MD-07.
- Does a notice of error, a qualified written request, or a request for information always escalate? MD-08: the recorded set includes `customer-specific loan information`.
- What evidence, beyond a source reference, shows that a draft’s fee, rate, account status, policy, or loan-fact claims are grounded? MP-03 and FR-04.
- Must mocked records be synthetic only? FR-09 and the operator decision of 2026-11-14.
- After this MVP, in a live operation, who may send the response once the representative has approved it? MD-09: the representative who recorded approval.
- Are latency, per-question cost, and daily volume limits required? MC-01 and FR-12. No concurrency number and no uptime number remain unset.
- What notice should the customer see for a failure that is not one of the three escalation conditions? FR-11.
- Does “mock every external system” include the model provider? MN-01: drafting may call a model.

Closed by this paid-run revision:

- Model id: `gpt-4o-mini`.
- Credential env var name: `OPENAI_API_KEY`.
- Soft stop: CR-15, 20 cents, FR-11 if a required call is skipped or the total is above 20 cents.
- `max_iter` 6, `max_rpm` 60, `max_execution_time` 15 seconds (hang stop).
- Latency tripwire (CR-16): fixture/stand-in under 5 seconds; live measures `elapsed_ms`; late success is `performance_miss`; `provider_timeout` and `technical_failure` are distinct; timeout is not automatic FR-11.
- Explicit-approval wait (CR-17): `waiting` continues until the representative records approval. No auto-approve, auto-reject, auto-deliver, or review-timeout FR-11.

Still unresolved, and not answered here:

- What evidence shows that an answer is easy to understand? No readability test was supplied. That stays with `@ux.designer` as intent, not as a control threshold.
- What token or character limit, if any, should CR-03 enforce in addition to the 15-second hang stop and the 20-cent soft stop?
- What frontend framework, UI library, and device size should `setup.md` and `ux.md` record? Not sourced in the PRD or `aamad.config.yml`.
- What hosting target and port should Deliver and setup use?

## Audit

- Timestamp: 2026-11-24T08:10:00-05:00
- Persona id: `system-arch`
- Action: `create-sad --mvp` (revision: CR-17 explicit-approval wait; waiting is not closed by a clock)
- Resolved runtime: `crewai`. `AAMAD_TARGET_RUNTIME` was unset. `aamad.config.yml` `runtime.target` is `crewai`. The adapter default is `crewai`. Warning: the environment variable was unset, so the config value was used; it matches the default.
- Prompt trace: omitted. Operator policy: the system continues waiting until the representative explicitly approves.
- Scope honored: SAD only. Reject, edit, and a review SLA remain out of the PRD.

- Timestamp: 2026-11-24T07:55:00-05:00
- Persona id: `system-arch`
- Action: `create-sad --mvp` (revision: CR-16 latency tripwire; `performance_miss` vs `provider_timeout` vs `technical_failure`; hang stop 15s)
- Resolved runtime: `crewai`. `AAMAD_TARGET_RUNTIME` was unset. `aamad.config.yml` `runtime.target` is `crewai`. The adapter default is `crewai`. Warning: the environment variable was unset, so the config value was used; it matches the default.
- Prompt trace: omitted. Operator policy: a performance miss is not FR-11. Timeout is not automatic FR-11.
- Scope honored: SAD only. Fixture EC-04 remains a hard under-5-second pass. Live late success is EC-10.

- Timestamp: 2026-11-23T17:20:00-05:00
- Persona id: `system-arch`
- Action: `create-sad --mvp` (revision: pin `gpt-4o-mini`, `OPENAI_API_KEY`, CR-15 soft stop, `max_iter` 6, `max_rpm` 60, `max_execution_time` 4s)
- Resolved runtime: `crewai`. `AAMAD_TARGET_RUNTIME` was unset. `aamad.config.yml` `runtime.target` is `crewai`. The adapter default is `crewai`. Warning: the environment variable was unset, so the config value was used; it matches the default.
- Prompt trace: omitted. This revision pins the paid run. It does not write `.env.example` or a secret value.
- Scope honored: SAD only. Setup still creates `.env.example` with the name `OPENAI_API_KEY`.

- Timestamp: 2026-11-23T17:10:00-05:00
- Persona id: `system-arch`
- Action: `create-sad --mvp` (revision: Pydantic schema miss is `unsupported` and the one gate, not FR-11)
- Resolved runtime: `crewai`. `AAMAD_TARGET_RUNTIME` was unset. `aamad.config.yml` `runtime.target` is `crewai`. The adapter default is `crewai`. Warning: the environment variable was unset, so the config value was used; it matches the default.
- Prompt trace: omitted. Operator decision: wrong shape is a human-review miss, not a technical-issue path.
- Scope honored: SAD only. FR-11 remains for retries exhausted, hung call, missing tool, or crash.

- Timestamp: 2026-11-23T16:55:00-05:00
- Persona id: `system-arch`
- Action: `create-sad --mvp` (revision: name `ClassifyOutput` and `DraftOutput`; `output_pydantic`; schema miss is FR-11)
- Resolved runtime: `crewai`. `AAMAD_TARGET_RUNTIME` was unset. `aamad.config.yml` `runtime.target` is `crewai`. The adapter default is `crewai`. Warning: the environment variable was unset, so the config value was used; it matches the default.
- Prompt trace: omitted. This revision names structured task outputs. It does not pick a model id or env var.
- Scope honored: SAD only. No application code. No `.env.example` write (that file does not exist until setup). Model id and spend knobs not pinned.

- Timestamp: 2026-11-23T15:55:00-05:00
- Persona id: `system-arch`
- Action: `create-sad --mvp` (revision: pin CR-12 / MP-03 as the claim-in-article algorithm)
- Resolved runtime: `crewai`. `AAMAD_TARGET_RUNTIME` was unset. `aamad.config.yml` `runtime.target` is `crewai`. The adapter default is `crewai`. Warning: the environment variable was unset, so the config value was used; it matches the default.
- Prompt trace: omitted. This revision records a code recipe for MP-03. It does not render a customer-facing model prompt.
- Scope honored: SAD only. Claims are Draft `{ kind, text }` fields. “Appears in” is normalized substring of a cited approved article. No second model. No application code, no PRD rewrite, no new diagram.

- Timestamp: 2026-11-17T09:15:00-05:00
- Persona id: `system-arch`
- Action: `create-sad --mvp`
- Resolved runtime: `crewai`. `AAMAD_TARGET_RUNTIME` was unset. `aamad.config.yml` `runtime.target` is `crewai`. The adapter default is `crewai`. Warning: the environment variable was unset, so the config value was used; it matches the default.
- Prompt trace: omitted. This action writes the lean MVP SAD from the PRD, the MRD, the user stories, and the five architecture decisions. It does not render a customer-facing model prompt.
- Scope honored: sections 1–10 filled as lean MVP views. Scale, hosting product, concurrency, and uptime deferred. Pattern Decision and Control Register not deferred. Evaluation criteria table filled from PRD thresholds only. No application code, no UX spec, no SFS, no new diagram, and no change to the PRD or the MRD.
- Diagram reused: `project-context/diagrams/system-arch/workflow-boundary.html`, `project-context/diagrams/system-arch/workflow-boundary.svg`. Type: architecture. diagram-design commit `0b9b10cab3dbf695642fa1a6c4b4f60df9083689`.
- Handoff: `@ux.designer` `*design-ux`. The PRD includes a user-facing surface. Then `@project.mgr` `*setup-project`.

- Timestamp: 2026-11-17T09:10:00-05:00
- Persona id: `system-arch`
- Action: `define-controls`
- Resolved runtime: `crewai`. `AAMAD_TARGET_RUNTIME` was unset. `aamad.config.yml` `runtime.target` is `crewai`. The adapter default is `crewai`. Warning: the environment variable was unset, so the config value was used; it matches the default.
- Model controls recorded: temperature 0, `max_retry_limit` 2, `memory=False`, sequential process, `allow_delegation` false. `max_iter` is capped at 12 and not pinned. `max_rpm`, the model name, max tokens, and `max_execution_time` are unset. Rollback now includes the FR-10 through FR-12 fixtures, under 5 seconds, and at most 20 cents per question.
- Prompt trace: omitted. This action records the control register. It does not render a customer-facing model prompt.
- Scope honored: Model and Context Strategy, Control Register, and Production Shape only. Work Ownership and Pattern Decision were not rewritten. No application code, no UX spec, no SFS, no new diagram, and no change to the PRD or the MRD. Sections 1–10 of the SAD template are not filled.

- Timestamp: 2026-11-17T09:05:00-05:00
- Persona id: `system-arch`
- Action: `select-pattern`
- Resolved runtime: `crewai`. `AAMAD_TARGET_RUNTIME` was unset. `aamad.config.yml` `runtime.target` is `crewai`. The adapter default is `crewai`. Warning: the environment variable was unset, so the config value was used; it matches the default.
- Prompt trace: omitted. This action records the pattern decision. It does not render a customer-facing model prompt.
- Scope honored: Pattern Decision and Reference Architecture only. Work Ownership was not rewritten. Control Register and Production Shape were not rewritten. No application code, no UX spec, no SFS, and no change to the PRD or the MRD. Sections 1–10 of the SAD template are not filled.
- Diagram: `project-context/diagrams/system-arch/workflow-boundary.html`, `project-context/diagrams/system-arch/workflow-boundary.svg`. Type: architecture. diagram-design commit `0b9b10cab3dbf695642fa1a6c4b4f60df9083689`.

- Timestamp: 2026-11-17T08:40:00-05:00
- Persona id: `system-arch`
- Action: `decompose-work`
- Resolved runtime: `crewai`. `AAMAD_TARGET_RUNTIME` was unset. `aamad.config.yml` `runtime.target` is `crewai`. The adapter default is `crewai`. Warning: the environment variable was unset, so the config value was used; it matches the default.
- Prompt trace: omitted. This action records work ownership traced to the PRD of 2026-11-17, the MRD, and the user stories. It does not render a customer-facing model prompt.
- Scope honored: Work Ownership and the feasibility verdict only. Pattern Decision, Control Register, and Production Shape were not rewritten. No application code, no UX spec, no SFS, no diagram, and no change to the PRD or the MRD. Sections 1–10 of the SAD template are not filled.

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
