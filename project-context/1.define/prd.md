# Product Requirements Document — Mortgage Servicing Customer Support Crew

## Context & Instructions

This is the MVP Product Requirements Document for a greenfield capstone. It states what the MVP must do and how a test can tell that it did it. It does not select an application pattern, a runtime, or an agent roster. Those decisions belong to `@system.arch` in the SAD.

**MRD:** `project-context/1.define/mrd.md`  
**System description:** `project-context/1.define/system-description.md` is not present. Requirements are traced to the MRD or to the operator instruction for this `*create-prd`. Where the system description would have settled a test, the gap is an Open Question.  
**System concept:** A mortgage customer submits one servicing question in natural language. The solution classifies it, retrieves from approved mock knowledge, drafts an answer with source references, and either delivers that answer directly or places the question and the draft on one human approval gate.  
**Selected runtime:** Not selected in this PRD. Deferred to the SAD.

**MVP boundary from the operator instruction of 2026-11-13:** one user-facing happy path, one human approval gate, every external system mocked, acceptance criteria testable, no unsourced number used as a target.

The one happy path is direct delivery of a supported answer to the mortgage customer. The one human approval gate is the representative’s review of an escalation. That gate is the same review the MRD already requires for the three escalation conditions. It is not an approval of a supported answer, and it is not a second gate after the representative reviews.

## 1. Executive Summary

**Problem Statement**

Mortgage customers seek answers to servicing questions and report trouble reaching a servicer and receiving inaccurate or conflicting information. The CFPB’s 2025 Consumer Response Annual Report records approximately 30,400 mortgage complaints in calendar 2025. Trouble during the payment process was the top issue, and its monthly average was 8 percent higher than the prior two years. In 92 percent of complaints the Bureau sent to companies, consumers said they first tried to resolve the issue with the company. J.D. Power’s 23 July 2026 servicer study reports satisfaction of 607 on a 1,000-point scale, up 11 points this year, and reports that poor customer service is the leading reason customers consider switching, at 43 percent. These figures are context from the MRD. They are not targets for this MVP, and they are not evidence that this capstone will reduce complaints or raise satisfaction. [MRD Executive Summary; MRD §1; MRD Claims We Will Not Make]

The primary user is the mortgage customer. The secondary user is the mortgage customer support representative, who handles the one approval gate. [MRD Context]

**Solution Overview**

The MVP demonstrates one workflow on mocked data:

1. The customer submits a mortgage servicing question in natural language.
2. The solution classifies the question.
3. The solution retrieves relevant information from approved knowledge.
4. The solution drafts an answer with source references.
5. The solution determines whether to deliver the answer directly or escalate it for human review.

A supported answer is delivered directly to the customer. That is the only user-facing happy path. An escalation sends the question and the draft to the representative for review before a response is sent. That is the only human approval gate. The three escalation conditions are unchanged: the answer is unsupported, the question requires customer-specific loan information, or the answer cannot be determined to be safe. [MRD Context; operator `*create-prd`]

This document does not claim the workflow is superior to manual search, a customer-service platform, or an internal assistant. [MRD §5]

**Strategic Rationale**

The cohort is a six-week demonstration of that one workflow, not a live servicing operation and not a business case. No ROI, cost savings, complaint reduction, satisfaction gain, containment rate, or replacement of existing servicing platforms is a goal. Market timing is the context in the MRD. It does not set a launch window. [MRD Cohort; MRD Claims We Will Not Make]

## 2. Market Context & User Analysis

**Target Market / Users**

| User | Role in the MVP | Source |
| --- | --- | --- |
| Mortgage customer | Primary user. Submits the question and receives a supported answer directly. | MRD Context |
| Mortgage customer support representative | Secondary user. Receives the question and the draft on the one approval gate. | MRD Context |

Market segment size and growth are not calculated. The sourced context figures are the CFPB counts and the J.D. Power 2026 figures in the MRD. They are not a population the MVP must serve. Geographic expansion is N/A for this capstone. [MRD §1; MRD Claims We Will Not Make]

**User Needs Analysis**

The customer need in scope is one servicing question answered from approved knowledge, or stopped before send when one of the three conditions applies. The representative’s manual search of FAQs, knowledge articles, servicing procedures, and customer information is an assumption about escalation work. It was not observed in user research, and it is not an acceptance test. [MRD §1 insight 3; MRD §3]

Adoption barriers such as channel choice, a readability score, and a time limit are not set. “Accessible” in the MRD means the customer can receive a supported answer without first reaching a representative. It does not set a disability-access standard and does not choose a channel. [MRD Context]

**Competitive Landscape**

The MRD compares four existing approaches and separates what is established from what is only an assumption. This PRD does not add a feature comparison or a price.

| Approach | What the MVP may say | Source |
| --- | --- | --- |
| Manual FAQ and knowledge-base search | On an escalation, manual search is an assumption, not a measured baseline. | MRD §5 |
| Existing customer service platforms | Mortgage servicing platforms, CRM systems, Salesforce, and Zendesk are not connected. Their feature lists were not read. | MRD Context; MRD §5 |
| Internal AI assistants | The CFPB chatbot spotlight and NIST describe inaccurate or weakly grounded answers as a known failure of that class of system. This MVP does not claim to avoid that failure except by the source rule and the one gate. | MRD §2; MRD §5 |
| Manual research and response preparation | No timed study is cited. The MVP does not claim to be faster or more accurate. | MRD §5 |

## 3. Technical Requirements & Architecture

This section states constraints. It does not choose a pattern, a runtime, components, or a roster. `@system.arch` records Work Ownership and Pattern Decision in `sad.md`.

**Discovery constraints**

`system-description.md` is absent. The rows below are taken from the MRD and from the operator instruction for this PRD.

#### Must do

| ID | Item | Kind | Testable constraint |
| --- | --- | --- | --- |
| MD-01 | One user-facing happy path | requirement | The product is given a question and the approved library. The hidden expected result is supported. The customer receives the answer and its source references, and no approval item is created. |
| MD-02 | Five-step workflow | requirement | The observable order is submit, then a classification outcome, then source references drawn from approved knowledge, then a draft, then exactly one of the two outcomes. |
| MD-03 | Direct delivery of a supported answer | requirement | Supported means the approved knowledge supports the answer, the question does not require customer-specific loan information, and the answer can be determined to be safe. The customer receives it without a representative. |
| MD-04 | One human approval gate | requirement | The product is given a question and the approved library. The hidden expected result lists one or more of the three escalation conditions. Exactly one approval item is created with the question and the draft. The customer does not receive that response before approval is recorded. After approval, the customer receives that same draft and its source references. |
| MD-05 | Mock data for the workflow | requirement | Customer records, loan information, and knowledge articles used by a test are fixtures. |

#### Must not do

| ID | Item | Kind | Testable constraint |
| --- | --- | --- | --- |
| MN-01 | Live external systems | requirement | A test fails if the workflow calls a live mortgage servicing platform, CRM, Salesforce, Zendesk, or any other system of record. |
| MN-02 | A second user-facing happy path | requirement | The MVP test plan has one customer success path: direct delivery of a supported answer. |
| MN-03 | A second approval gate | requirement | A supported answer does not wait on a person. An escalated answer does not wait on a second person after the representative’s approval. |
| MN-04 | Extra escalation conditions | requirement | The only escalate reasons are unsupported, customer-specific loan information, and cannot be determined to be safe. |
| MN-05 | Answering customer-specific loan facts from mocks | requirement | A question whose hidden expected result is customer-specific still escalates when a mock loan record is in the data the product can search. |
| MN-06 | Unsourced business claims | requirement | Pass/fail does not depend on ROI, savings, complaint reduction, a satisfaction gain, a containment rate, or replacement of a servicing platform. |

#### Must cost

| ID | Item | Kind | Testable constraint |
| --- | --- | --- | --- |
| MC-01 | No sourced limit for latency, cost per interaction, or volume | assumption | No number is set. Any candidate number is an Open Question until a source or an operator decision supplies it. |

#### Must prove

| ID | Item | Kind | Testable constraint |
| --- | --- | --- | --- |
| MP-01 | Happy path and each escalation condition | requirement | The fixture set includes one question whose hidden result is supported, one question for each escalation condition alone, and one question whose hidden result lists two escalation reasons. The expected result is hidden from the product. The recorded set matches that hidden result. Each escalation still creates one approval item. |
| MP-02 | Source references | requirement | On direct delivery, every source reference resolves to an article the fixture marks approved. |
| MP-03 | Accuracy beyond a source reference | assumption | Not proved by this PRD. What else shows a draft is accurate remains an Open Question. The MVP proof is MP-01 and MP-02. |

Trace: MD-02, MD-03, MD-05, MN-01, MN-04, MN-05, and MP-02 trace to MRD Context and MRD §2. MD-01, MD-04, MN-02, and MN-03 trace to MRD Context plus the operator `*create-prd` instruction. MN-06 traces to MRD Claims We Will Not Make. MC-01 and MP-03 trace to MRD Open Questions.

**Runtime constraints**

Runtime is not a product requirement. The SAD records it. This PRD does not set tool lists, iteration caps, hooks, or adapter controls.

Integration boundaries the product must honor, whatever runtime the SAD selects:

- Factual content comes from the approved mock knowledge set.
- Customer records and loan information come from mocks.
- No live system of record is called.
- The one approval decision is recorded and is attributable to the representative role in the test.
- Secrets are not written into the PRD, the MRD, fixtures that are committed as examples, or logs that the MVP retains for the test.

**Integration Requirements**

| Item | MVP | Later |
| --- | --- | --- |
| Mortgage servicing platform, CRM, Salesforce, Zendesk | Mocked. No live connection. | Out of scope until the operator adds it. |
| Knowledge articles, customer records, loan information | Fixtures local to the MVP. | A live store is out of scope. |
| Authentication of a real customer or employee directory | Not required. The test distinguishes the customer role from the representative role. | Open if a later release adds real identity. |
| Performance and scale targets | No number is sourced. | Open Questions. |

**Infrastructure Specifications**

No cloud provider, machine size, network design, uptime target, or monitoring product is sourced. The MRD states that this increment is not a production servicing operation. Hosting and observability for a later deployment belong to Deliver, after the SAD. The MVP must be able to run its fixture tests without a live external system. [MRD §4]

## 4. Functional Requirements

Priority P0 is the MVP. There is no P1. P2 is future work.

**Fixture rule**

A fixture has two parts. The product receives the customer’s question and the library of articles marked approved. It searches that library. It does not receive a pre-chosen set of matching articles, and it does not receive the expected result. The test holds the expected result. That result is either supported alone, or one or more of these escalation reasons: unsupported, customer-specific loan information, and cannot be determined to be safe. It is not both supported and an escalation reason. The test passes when the product’s recorded set matches that hidden result.

**Core Features (Priority P0)**

**FR-01. Customer submits one question.**  
Trace: MRD Context, primary focus step 1; operator `*create-prd` (one happy path).

Acceptance criteria:

- Given the customer role, when that role submits one mortgage servicing question as natural language, then the workflow starts for that question.
- A second customer entry path, such as a separate form for account numbers or a representative-started question, is not required for MVP pass.

**FR-02. Classify the question.**  
Trace: MRD Context, primary focus step 2.

Acceptance criteria:

- The product records a set of reasons. The set is exactly supported when none of the three escalation reasons apply.
- When any escalation reason applies, the set contains every one that applies, and it does not contain supported.
- The only escalation reasons are unsupported, customer-specific loan information, and cannot be determined to be safe.
- The customer is not required to see the reasons. The test reads the recorded set and compares it with the hidden answer key.

**FR-03. Retrieve from approved mock knowledge.**  
Trace: MRD Context, primary focus step 3; MRD §2.

Acceptance criteria:

- Given a fixture with a set of articles marked approved, when an answer is drafted, then every source reference on that draft resolves to an article in that approved set.
- A source reference that does not resolve to an approved article fails the test.
- Retrieval does not call a live servicing platform, CRM, Salesforce, or Zendesk.

**FR-04. Draft an answer with source references.**  
Trace: MRD Context, primary focus step 4; MRD Approval policy.

Acceptance criteria:

- On a supported result, every source reference resolves to an article in the library the product searched.
- On an unsupported result, the customer does not receive the draft as the answer.

**FR-05. Choose direct delivery or the one gate.**  
Trace: MRD Context, primary focus step 5; MRD Approval policy; operator `*create-prd`.

Acceptance criteria:

- Given a completed draft, when the workflow finishes, then the result is exactly one of: direct delivery to the customer, or one approval item for the representative.
- Direct delivery is allowed only when the recorded set is exactly supported.
- If the recorded set contains any escalation reason, the result is the one approval gate.

**FR-06. Happy path: deliver a supported answer directly.**  
Trace: MRD Context, primary user and approval policy; operator `*create-prd`.

Acceptance criteria:

- The product is given a question and the approved library. The hidden expected result is supported. The customer receives the answer and its source references, and no approval item is created.
- The representative does not act before the customer receives the answer.
- This is the only customer success path in the MVP test plan.

**FR-07. One human approval gate.**  
Trace: MRD Escalation conditions; operator `*create-prd`.

Acceptance criteria:

- The product is given a question and the approved library. The hidden expected result lists one or more of the three escalation conditions. Exactly one approval item is created with the question and the draft. The recorded set matches that hidden result. The customer does not receive that response before approval is recorded.
- The representative role can open the item and record approval.
- After approval is recorded, the customer receives that same draft, including its source references.
- The test fails if approval is recorded and the customer still has no answer.
- A second approval item, a second approver, or an approval step on a supported answer fails the test.
- A separate sender role is not part of the MVP. Who may send in a future live operation remains an Open Question carried from the MRD.

**FR-08. Customer sees a non-answer state while the gate is open.**  
Trace: MRD rule that an escalated response is not sent before review; operator requirement that acceptance criteria be testable. The wording of the notice is not specified here.

Acceptance criteria:

- Given an open approval item, when the customer views the result of that question, then the surface does not present the draft as the answer.
- The surface shows that the question is waiting on representative review.
- Layout, channel, and the exact words of that notice belong to `@ux.designer`.

**FR-09. Mock every external system.**  
Trace: MRD “What is mocked”; operator `*create-prd`.

Acceptance criteria:

- The knowledge articles, customer records, and loan information used by FR-01 through FR-08 are fixtures.
- The workflow does not require a network call to a mortgage servicing platform, a CRM, Salesforce, Zendesk, or another system of record in order to pass.
- A question whose hidden expected result is customer-specific still escalates when a mock loan record is in the data the product can search. The mock record is not a live loan lookup.

**FR-10. Fixture coverage for the proof.**  
Trace: operator `*create-prd` (testable acceptance); MRD §2 statement that a limited set can show direct delivery and a stop under each escalation condition. No market volume is sourced. The extra question below is the smallest set that makes two reasons at once observable.

Acceptance criteria:

- The fixture set includes one question whose hidden result is supported, one question for each escalation condition alone, and one question whose hidden result lists two escalation reasons.
- The expected result is hidden from the product. The recorded set matches that hidden result.
- Each of those fixtures passes the matching criteria in FR-06 or FR-07.
- One question per case is the smallest set that makes each outcome observable. It is not a sourced volume target.

**Enhanced Features (Priority P1)**

None. The operator limited this PRD to the MVP.

**Future Features (Priority P2)**

- A live mortgage servicing platform, CRM, Salesforce, or Zendesk connection.
- A real customer or employee identity system.
- A separate person who sends the response after the representative approves it.
- Regulation X notice clocks, and a rule for whether a notice of error, a qualified written request, or a request for information always escalates.
- Any latency, cost, volume, uptime, readability, or accuracy-rate target.
- A second customer channel or a second happy path.
- Measurement of complaint reduction, satisfaction change, containment, savings, or ROI.

## 5. Non-Functional Requirements

**Performance Requirements**

No response-time, throughput, concurrency, or uptime number appears in the MRD or in the operator instruction. None is set here. Candidate numbers belong in Open Questions. A test must not fail on a duration that has no sourced limit. [MRD Open Questions; MRD §4]

**Security & Compliance**

- The MVP does not connect to a live system of record. [FR-09]
- Whether fixtures must be synthetic only is unresolved. Until that question is closed, the MVP test does not require production customer data, and this PRD does not authorize loading a real servicing extract. [MRD §4; MRD Open Questions]
- Secret values are not stored in this PRD or in committed fixtures.
- This capstone is not a compliance determination under the OCC framework or Regulation X. Inaccurate statements about fees, account status, or policy are why unsupported and unsafe answers use the gate. [MRD §2; MRD Risk]
- `@security.eng` assesses a built increment later. This PRD does not write that assessment.

**Scalability & Reliability**

Scaling triggers, recovery-time objectives, and fault-tolerance procedures are not sourced. They are deferred with P2. The MVP reliability bar is FR-10: the supported result, each escalation condition alone, and one question with two escalation reasons behave as specified on the fixtures.

## 6. User Experience Design

**Interface Requirements**

The MVP has two roles and one customer success path.

- The customer can submit natural language and, on the happy path, read the answer and its source references.
- On an escalation, the customer sees the waiting state in FR-08 and does not see the draft as the answer.
- The representative can open the one approval item, read the question and the draft, and record approval. After that approval, the customer receives that same draft and its source references.

Channel, screen layout, visual design, and a disability-access standard are not decided. `@ux.designer` owns them if a surface is in scope. This PRD does not name a UI library or a device size. [MRD §3 Implications]

**Human–system interaction**

- The customer starts the only happy path and does not pass through the representative when the answer is supported.
- The representative enters only through the one gate.
- Explainability on the happy path is the source references on the answer. No further explanation format is specified.
- Errors that are the three escalation conditions use FR-07 and FR-08. Other failure behavior, such as a fixture that cannot be loaded, must not be shown to the customer as a supported answer. The exact error notice is an Open Question for UX, with the testable rule that it is not presented as a sourced answer.

## 7. Success Metrics & KPIs

**Business / Operational Metrics**

No business target is set. The CFPB counts and the J.D. Power score stay context. They are not baseline or goal values for the MVP. ROI, cost savings, complaint reduction, a satisfaction gain, and a containment rate are claims this product will not make. [MRD Claims We Will Not Make]

**Technical Metrics**

The MVP succeeds when FR-10 passes. No accuracy percentage, latency budget, or cost per answer is set. Task effectiveness beyond “the fixture outcome matches, and sources resolve” is MP-03 and remains open.

**User Experience Metrics**

No satisfaction score, task-time limit, or readability score is set. “Easy to understand” remains operator intent without a test. [MRD Open Questions]

## 8. Implementation Strategy

**Development Phases**

- Phase 1, Define: this PRD is the requirements artifact. The SAD is next and is owned by `@system.arch`. User stories are not part of this action.
- Phase 2, Build: setup, then frontend and backend, then integration, then QA, against FR-01 through FR-10.
- Phase 3, Deliver: deploy configuration and runbook after the build artifacts exist. This PRD does not authorize a live deploy.

**Resource Requirements**

The six-week length is the operator’s description of the cohort in the MRD. It is a scope constraint, not a staffed plan and not a sourced estimate. No team size or budget is stated. [MRD Assumptions]

**Risk Mitigation**

| Risk | MVP response | Source |
| --- | --- | --- |
| A customer receives an answer whose source note does not point at an article in the approved library. | FR-03 and FR-04 fail that answer. An unsupported result goes to the one gate under FR-05. | FR-03; FR-04; FR-05 |
| A cited article is real, but a sentence states a fee, rate, account status, policy, or loan fact the article does not contain. | This MVP does not test that. Whether the sentence is accurate beyond the source note remains MP-03. | MRD Open Questions; MP-03 |
| Real customer data is loaded as a mock. | Not authorized by this PRD. Synthetic-only remains open. | MRD Risk |
| An unsourced rate or savings figure becomes a success metric. | Section 7 forbids it. | MRD Claims We Will Not Make |
| “Approved” is undefined, so a source can point at an article the operator has not approved. | Tests use the fixture’s approved mark. The general approval rule remains open. | MRD Open Questions; MP-02 |

## 9. Launch & Go-to-Market Strategy

N/A. This is a six-week capstone on mocked data. It does not pick a commercial segment, a price, or a launch market. [MRD §5; MRD Market positioning]

## Quality Assurance Checklist

- [x] Requirements traceable to the MRD or to the operator `*create-prd` instruction. The system description is absent and is not used as a source.
- [x] Technical constraints are stated without a runtime, a pattern, or a roster. The SAD owns those choices.
- [x] Success metrics match the fixture tests. Unsourced rates and business outcomes are not targets.
- [x] MVP scope is P0. P1 is empty. P2 lists deferred work.
- [x] Market sections use the MRD. The MRD was not skipped.

## Sources

- `project-context/1.define/mrd.md`, including its Context, Executive Summary, §§1–5, Claims We Will Not Make, Assumptions, and Open Questions.
- Operator instruction for `*create-prd`, 2026-11-13: MVP only; one user-facing happy path; one human approval gate; mock every external system; testable acceptance criteria; trace to the MRD or the system description; leave pattern, runtime, and agent roster to the SAD; unsourced numbers go to Open Questions.
- `aamad.config.yml`: `project.mode` is `greenfield`. The file also contains `runtime.target`. That value is not a requirement of this PRD.
- System description: not present at `project-context/1.define/system-description.md`.

## Assumptions

- The MRD is the product scope for users, the five steps, the three escalation conditions, direct delivery of a supported answer, mocked data, and the claims this product will not make.
- “One human approval gate” means the representative’s review of an escalation. It does not add a review of supported answers. The earlier MRD decision that a supported answer is delivered directly remains in force.
- For the MVP, the representative’s approval delivers that same draft and its source references to the customer. This does not decide who may send in a future live servicer.
- The customer-visible waiting state in FR-08 is required so the escalation path can be tested. The words and layout of that state are not specified.
- The expected result is the tester’s answer key. It may list more than one escalation reason. It is not given to the product. The product receives the question and the approved-article library and searches that library. A passing test does not settle the general meanings of customer-specific, approved, accurate, or safe.
- “At least one” fixture per outcome is the smallest set that makes FR-10 observable. It is not a market volume.
- The six-week cohort is a scope constraint from the MRD, not a delivery estimate.
- `runtime.target` in `aamad.config.yml` is not adopted here, because the operator assigned runtime to the SAD.
- Launch and go-to-market are N/A, as stated in section 9.

## Open Questions

Carried from the MRD, still unresolved:

- Which facts count as customer-specific loan information, beyond the hidden expected result used by the test in FR-09?
- What makes an article approved outside a fixture, and what happens when two approved articles conflict?
- Does a notice of error, a qualified written request, or a request for information always escalate?
- What evidence, beyond a source reference, shows that a draft is accurate?
- What evidence shows that an answer is easy to understand? No readability test was supplied.
- Must mocked records be synthetic only?
- After this MVP, in a live operation, who may send the response once the representative has approved it?
- Are any latency, per-interaction cost, or volume limits required? No number is set.

Added because this PRD has no source for them:

- What response-time, throughput, concurrency, or uptime number, if any, should the MVP meet?
- What notice, other than “not the draft,” should the customer see for a failure that is not one of the three escalation conditions?
- Does “mock every external system” include a model provider, or only systems of record? This PRD mocks systems of record. It does not require the drafting step to run without a model.

## Audit

- Timestamp: 2026-11-14T11:15:00-05:00
- Persona id: `product-mgr`
- Action: `create-prd` (revision: the product records every escalation reason that applies; supported is used only when none apply)
- Prior revision: 2026-11-14T11:10:00-05:00, the risk table claims only the source-note check. 2026-11-14T10:59:00-05:00, after approval the customer receives that same draft. 2026-11-14T10:46:00-05:00, expected fixture result is hidden from the product. Original write: 2026-11-13T22:44:29-05:00, action `create-prd`.
- Resolved runtime: not recorded. Pattern, runtime, and agent roster are left to the SAD. `runtime.target` in `aamad.config.yml` is not a requirement of this document. `AAMAD_TARGET_RUNTIME` was not applied.
- Prompt trace: omitted. This artifact is Define-phase requirements traced to the MRD and the operator instruction. It is not a production model response and it does not select an application pattern.
- Scope honored: MVP only; one customer happy path; one human approval gate; external systems of record mocked; no unsourced numeric target; no user stories; no SAD; no application code.
