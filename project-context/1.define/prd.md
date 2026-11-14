# Product Requirements Document — Mortgage Servicing Customer Support Crew

## Context & Instructions

This is the MVP Product Requirements Document for a greenfield capstone. It states what the MVP must do and how a test can tell that it did it. It does not select an application pattern, a runtime, or an agent roster. Those decisions belong to `@system.arch` in the SAD.

**MRD:** `project-context/1.define/mrd.md`  
**System description:** `project-context/1.define/system-description.md` is not present. Requirements are traced to the MRD, to the operator instruction for `*create-prd`, or to the operator decisions of 2026-11-14 that close the open questions below. Where the system description would have settled a test and the operator has not, the gap stays an Open Question.  
**System concept:** A mortgage customer submits one servicing question in natural language. The solution classifies it, retrieves from approved mock knowledge, drafts an answer with source references, and either delivers that answer directly or places the question and the draft on one human approval gate.  
**Selected runtime:** Not selected in this PRD. Deferred to the SAD.

**MVP boundary from the operator instruction of 2026-11-13:** one user-facing happy path, one human approval gate, every system of record mocked, acceptance criteria testable, no unsourced number used as a target. The operator decisions of 2026-11-14 set the customer-specific fact list, the fixture approval mark, the conflict rule, escalation of a notice of error, a qualified written request, or a request for information, the claim-in-article check, synthetic fixtures, the technical-issue notice, a model call for drafting, and the limits of 200 questions a day, under 5 seconds, and at most 20 cents per question.

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

Channel choice and a readability score are not set. The latency, cost, and daily volume limits are in section 5. “Accessible” in the MRD means the customer can receive a supported answer without first reaching a representative. It does not set a disability-access standard and does not choose a channel. [MRD Context]

**Competitive Landscape**

The MRD compares four existing approaches and separates what is established from what is only an assumption. This PRD does not add a feature comparison or a price.

| Approach | What the MVP may say | Source |
| --- | --- | --- |
| Manual FAQ and knowledge-base search | On an escalation, manual search is an assumption, not a measured baseline. | MRD §5 |
| Existing customer service platforms | Mortgage servicing platforms, CRM systems, Salesforce, and Zendesk are not connected. Their feature lists were not read. | MRD Context; MRD §5 |
| Internal AI assistants | The CFPB chatbot spotlight and NIST describe inaccurate or weakly grounded answers as a known failure of that class of system. This MVP does not claim to avoid that failure except by the source rule, the claim-in-article rule, and the one gate. | MRD §2; MRD §5; operator decision 2026-11-14 |
| Manual research and response preparation | No timed study is cited. The MVP does not claim to be faster or more accurate. | MRD §5 |

## 3. Technical Requirements & Architecture

This section states constraints. It does not choose a pattern, a runtime, components, or a roster. `@system.arch` records Work Ownership and Pattern Decision in `sad.md`.

**Discovery constraints**

`system-description.md` is absent. The rows below are taken from the MRD, from the operator instruction for this PRD, and from the operator decisions of 2026-11-14.

#### Must do

| ID | Item | Kind | Testable constraint |
| --- | --- | --- | --- |
| MD-01 | One user-facing happy path | requirement | The product is given a question and the approved library. The hidden expected result is supported. The customer receives the answer and its source references, and no approval item is created. |
| MD-02 | Five-step workflow | requirement | When the workflow completes, the observable order is submit, then a classification outcome, then source references drawn from approved knowledge, then a draft, then exactly one of the two outcomes. When it cannot complete, the result is FR-11. |
| MD-03 | Direct delivery of a supported answer | requirement | Supported means the approved knowledge supports the answer, the question does not require customer-specific loan information, and the answer can be determined to be safe. The customer receives it without a representative. |
| MD-04 | One human approval gate | requirement | The product is given a question and the approved library. The hidden expected result lists one or more of the three escalation conditions. Exactly one approval item is created with the question and the draft. The customer does not receive that response before approval is recorded. After approval, the customer receives that same draft and its source references. |
| MD-05 | Mock data for the workflow | requirement | Customer records, loan information, and knowledge articles used by a test are invented fixtures. They are not a servicing extract and do not use a real borrower’s name, loan number, or address. |
| MD-06 | Customer-specific facts | requirement | A question is customer-specific when the answer needs a fact about one borrower or one loan: loan number, borrower name, property address, balance, interest rate, payment amount, due date, last payment, escrow amount, payoff amount, or delinquency or account status. A general servicing question that an approved article answers can still be supported. |
| MD-07 | Approved mark and conflict | requirement | An article is approved when the fixture marks it approved. A draft may cite only those articles. When two approved articles disagree on a fact the answer needs, the recorded reason is cannot be determined to be safe. |
| MD-08 | Notice of error, qualified written request, or request for information | requirement | When the question itself is one of those three, the recorded reason is customer-specific loan information. A general question about what the term means can still be supported from an approved article. |

#### Must not do

| ID | Item | Kind | Testable constraint |
| --- | --- | --- | --- |
| MN-01 | Live systems of record | requirement | A test fails if the workflow calls a live mortgage servicing platform, CRM, Salesforce, Zendesk, or any other system of record. A model provider is not a system of record under this rule. Drafting may call a model. |
| MN-02 | A second user-facing happy path | requirement | The MVP test plan has one customer success path: direct delivery of a supported answer. |
| MN-03 | A second approval gate | requirement | A supported answer does not wait on a person. An escalated answer does not wait on a second person after the representative’s approval. |
| MN-04 | Extra escalation conditions | requirement | The only escalate reasons are unsupported, customer-specific loan information, and cannot be determined to be safe. |
| MN-05 | Answering customer-specific loan facts from mocks | requirement | A question whose hidden expected result is customer-specific still escalates when a mock loan record is in the data the product can search. |
| MN-06 | Unsourced business claims | requirement | Pass/fail does not depend on ROI, savings, complaint reduction, a satisfaction gain, a containment rate, or replacement of a servicing platform. |

#### Must cost

| ID | Item | Kind | Testable constraint |
| --- | --- | --- | --- |
| MC-01 | Latency, cost per question, and daily volume | requirement | One submitted question is one call and one transaction. The MVP supports 200 submitted questions in a day. Elapsed time from submit until the customer can see the supported answer, the waiting state, or the technical-issue notice is under 5 seconds. Representative review is outside that interval. The recorded model or provider cost of that question is at most 20 cents. |

#### Must prove

| ID | Item | Kind | Testable constraint |
| --- | --- | --- | --- |
| MP-01 | Happy path and each escalation condition | requirement | The fixture set includes one question whose hidden result is supported, one question for each escalation condition alone, and one question whose hidden result lists two escalation reasons. The expected result is hidden from the product. The recorded set matches that hidden result. Each escalation still creates one approval item. |
| MP-02 | Source references | requirement | On direct delivery, every source reference resolves to an article the fixture marks approved. |
| MP-03 | Claims match the cited article | requirement | Every factual claim in the draft about a fee, rate, account status, policy, or loan fact appears in a cited approved article. A claim of that kind that is absent from those articles makes the recorded set include unsupported, and the set does not contain supported. |

Trace: MD-02, MD-03, MD-05, MN-01, MN-04, MN-05, and MP-02 trace to MRD Context and MRD §2. MD-01, MD-04, MN-02, and MN-03 trace to MRD Context plus the operator `*create-prd` instruction. MN-06 traces to MRD Claims We Will Not Make. MD-06, MD-07, MD-08, MC-01, and MP-03 trace to the operator decisions of 2026-11-14. The MRD had left those items open.

**Runtime constraints**

Runtime is not a product requirement. The SAD records it. This PRD does not set tool lists, iteration caps, hooks, or adapter controls.

Integration boundaries the product must honor, whatever runtime the SAD selects:

- Factual content comes from the approved mock knowledge set.
- Customer records and loan information come from invented fixtures.
- No live system of record is called. Drafting may call a model.
- The one approval decision is recorded and is attributable to the representative role in the test.
- Secrets are not written into the PRD, the MRD, fixtures that are committed as examples, or logs that the MVP retains for the test.

**Integration Requirements**

| Item | MVP | Later |
| --- | --- | --- |
| Mortgage servicing platform, CRM, Salesforce, Zendesk | Mocked. No live connection. | Out of scope until the operator adds it. |
| Knowledge articles, customer records, loan information | Invented fixtures local to the MVP. | A live store is out of scope. |
| Model provider used to draft an answer | A call is allowed. It is not a system of record. | A requirement to draft without a model is out of scope. |
| Authentication of a real customer or employee directory | Not required. The test distinguishes the customer role from the representative role. | Open if a later release adds real identity. |
| Performance and scale targets | 200 submitted questions in a day. Under 5 seconds until the customer-visible automated result. At most 20 cents of model or provider cost per question. | Behavior above 200 questions in a day is unresolved. No concurrency number and no uptime number. |

**Infrastructure Specifications**

No cloud provider, machine size, network design, uptime target, or monitoring product is sourced. The MRD states that this increment is not a production servicing operation. Hosting and observability for a later deployment belong to Deliver, after the SAD. The MVP must be able to run its fixture tests without a live system of record. The operator set no uptime number on 2026-11-14. [MRD §4]

## 4. Functional Requirements

Priority P0 is the MVP. There is no P1. P2 is future work.

**Fixture rule**

A fixture has two parts. The product receives the customer’s question and the library of articles marked approved. It searches that library. It does not receive a pre-chosen set of matching articles, and it does not receive the expected result. The test holds the expected result. That result is either supported alone, or one or more of these escalation reasons: unsupported, customer-specific loan information, and cannot be determined to be safe. It is not both supported and an escalation reason. The test passes when the product’s recorded set matches that hidden result.

Operator decisions of 2026-11-14 that the fixture tests use:

- A question is customer-specific when the answer needs one of these facts about one borrower or one loan: loan number, borrower name, property address, balance, interest rate, payment amount, due date, last payment, escrow amount, payoff amount, or delinquency or account status. A general servicing question that an approved article answers can still be supported. A mock loan record that contains the fact still escalates.
- An article is approved when the fixture marks it approved. When two approved articles disagree on a fact the answer needs, the hidden result is cannot be determined to be safe.
- When the question itself is a notice of error, a qualified written request, or a request for information, the hidden result includes customer-specific loan information. A general question about what the term means can still be supported.
- Every factual claim in the draft about a fee, rate, account status, policy, or loan fact appears in a cited approved article. A claim of that kind that is absent from those articles is an unsupported result.
- Customer records, loan facts, and articles are invented for this capstone.

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
- A question is customer-specific when the answer needs a loan number, borrower name, property address, balance, interest rate, payment amount, due date, last payment, escrow amount, payoff amount, or delinquency or account status for one borrower or one loan.
- When the question itself is a notice of error, a qualified written request, or a request for information, the recorded set includes customer-specific loan information. A general question about what that term means follows the same supported-or-escalate rules as any other question.
- When two approved articles disagree on a fact the answer needs, the recorded set includes cannot be determined to be safe.
- The customer is not required to see the reasons. The test reads the recorded set and compares it with the hidden answer key.

**FR-03. Retrieve from approved mock knowledge.**  
Trace: MRD Context, primary focus step 3; MRD §2.

Acceptance criteria:

- Given a fixture with a set of articles marked approved, when an answer is drafted, then every source reference on that draft resolves to an article in that approved set.
- A source reference that does not resolve to an approved article fails the test.
- For this MVP, the approved mark on the fixture is the approval rule.
- Retrieval does not call a live servicing platform, CRM, Salesforce, or Zendesk. Drafting may call a model.

**FR-04. Draft an answer with source references.**  
Trace: MRD Context, primary focus step 4; MRD Approval policy.

Acceptance criteria:

- On a supported result, every source reference resolves to an article in the library the product searched.
- Every factual claim in the draft about a fee, rate, account status, policy, or loan fact appears in a cited approved article. A claim of that kind that is absent from those articles makes the recorded set include unsupported, and the set does not contain supported.
- On an unsupported result, the customer does not receive the draft as the answer.
- No accuracy percentage is set.

**FR-05. Choose direct delivery or the one gate.**  
Trace: MRD Context, primary focus step 5; MRD Approval policy; operator `*create-prd`.

Acceptance criteria:

- Given a completed draft, when the workflow finishes, then the result is exactly one of: direct delivery to the customer, or one approval item for the representative.
- Direct delivery is allowed only when the recorded set is exactly supported.
- If the recorded set contains any escalation reason, the result is the one approval gate.
- When the workflow cannot complete those steps, the result is the technical-issue outcome in FR-11. That outcome is not a second customer success path.

**FR-06. Happy path: deliver a supported answer directly.**  
Trace: MRD Context, primary user and approval policy; operator `*create-prd`.

Acceptance criteria:

- The product is given a question and the approved library. The hidden expected result is supported. The customer receives the answer and its source references, and no approval item is created.
- The representative does not act before the customer receives the answer.
- This is the only customer success path in the MVP test plan.
- Elapsed time from submit until the customer can see the answer is under 5 seconds. The recorded model or provider cost of that question is at most 20 cents. The product records that cost.

**FR-07. One human approval gate.**  
Trace: MRD Escalation conditions; operator `*create-prd`.

Acceptance criteria:

- The product is given a question and the approved library. The hidden expected result lists one or more of the three escalation conditions. Exactly one approval item is created with the question and the draft. The recorded set matches that hidden result. The customer does not receive that response before approval is recorded.
- The representative role can open the item and record approval.
- After approval is recorded, the customer receives that same draft, including its source references.
- The test fails if approval is recorded and the customer still has no answer.
- Elapsed time from submit until the customer can see the waiting state is under 5 seconds. Representative review time is outside that interval. The recorded model or provider cost of that question is at most 20 cents. The product records that cost.
- A second approval item, a second approver, or an approval step on a supported answer fails the test.
- A separate sender role is not part of the MVP. Who may send in a future live operation remains an Open Question carried from the MRD.

**FR-08. Customer sees a non-answer state while the gate is open.**  
Trace: MRD rule that an escalated response is not sent before review; operator requirement that acceptance criteria be testable. The wording of the waiting notice is not specified here. The technical-issue sentence is specified in FR-11.

Acceptance criteria:

- Given an open approval item, when the customer views the result of that question, then the surface does not present the draft as the answer.
- The surface shows that the question is waiting on representative review.
- Layout, channel, and the exact words of that notice belong to `@ux.designer`.

**FR-09. Mock every external system.**  
Trace: MRD “What is mocked”; operator `*create-prd`.

Acceptance criteria:

- The knowledge articles, customer records, and loan information used by FR-01 through FR-11 are invented fixtures. They are not a servicing extract and do not use a real borrower’s name, loan number, or address.
- The workflow does not require a network call to a mortgage servicing platform, a CRM, Salesforce, Zendesk, or another system of record in order to pass. Drafting may call a model.
- A question whose answer needs a fact on the MD-06 list still escalates when a mock loan record is in the data the product can search. The mock record is not a live loan lookup.

**FR-10. Fixture coverage for the proof.**  
Trace: operator `*create-prd` (testable acceptance); MRD §2 statement that a limited set can show direct delivery and a stop under each escalation condition; operator decisions of 2026-11-14. The extra questions below are the smallest set that makes each new rule observable. They are not the daily capacity of 200 questions.

Acceptance criteria:

- The fixture set includes one question whose hidden result is supported, one question for each escalation condition alone, and one question whose hidden result lists two escalation reasons.
- The supported question is a general servicing question. It is not itself a notice of error, a qualified written request, or a request for information. Every factual claim in its delivered answer about a fee, rate, account status, policy, or loan fact appears in a cited approved article.
- The customer-specific question asks for one fact on the MD-06 list.
- The cannot-be-determined-to-be-safe question has two approved articles that disagree on a fact the answer needs.
- The fixture set also includes one question that is itself a notice of error, one that is itself a qualified written request, and one that is itself a request for information. Each hidden result includes customer-specific loan information, and each creates one approval item.
- The fixture set includes one case in which the workflow cannot complete, used by FR-11.
- The expected result is hidden from the product. The recorded set matches that hidden result.
- Each completed-workflow fixture passes the matching criteria in FR-06 or FR-07.
- One question per case is the smallest set that makes each outcome observable. The daily capacity of 200 submitted questions is MC-01. It is not this fixture count.

**FR-11. Technical failure.**  
Trace: operator decision of 2026-11-14.

Acceptance criteria:

- Given a failure that is not one of the three escalation conditions, when the customer views the result, then the customer sees this sentence: “We are facing a technical issue at this point. Check back later.”
- The error is logged in a record the test can read. The record identifies the question and the failure. It does not contain a secret value.
- The draft is not shown as the answer. No approval item is created.
- Elapsed time from submit until the customer can see that sentence is under 5 seconds. The recorded model or provider cost of that question, when a model was called, is at most 20 cents.

**Enhanced Features (Priority P1)**

None. The operator limited this PRD to the MVP.

**Future Features (Priority P2)**

- A live mortgage servicing platform, CRM, Salesforce, or Zendesk connection.
- A real customer or employee identity system.
- A separate person who sends the response after the representative approves it.
- Regulation X notice clocks.
- An uptime number, a concurrency number, a readability score, or an accuracy percentage.
- A second customer channel or a second happy path.
- Measurement of complaint reduction, satisfaction change, containment, savings, or ROI.

## 5. Non-Functional Requirements

**Performance Requirements**

The operator set these limits on 2026-11-14. A call and a transaction are one submitted question.

- The MVP supports 200 submitted questions in a day.
- Latency, and the response time for this MVP, is under 5 seconds from submit until the customer can see the supported answer, the waiting state, or the technical-issue notice. Representative review is outside that interval.
- The model or provider cost recorded for that question is at most 20 cents. The product records that cost.

No concurrency number and no uptime number are set. No throughput number is set beyond the 200 questions in a day. What the product does with a 201st question in a day is unresolved. [Operator decision 2026-11-14; MRD §4]

**Security & Compliance**

- The MVP does not connect to a live system of record. [FR-09]
- Fixtures are synthetic. They are invented for this capstone, they are not a servicing extract, and they do not use a real borrower’s name, loan number, or address. [Operator decision 2026-11-14]
- Secret values are not stored in this PRD or in committed fixtures.
- This capstone is not a compliance determination under the OCC framework or Regulation X. Inaccurate statements about fees, account status, or policy are why unsupported and unsafe answers use the gate. [MRD §2; MRD Risk]
- `@security.eng` assesses a built increment later. This PRD does not write that assessment.

**Scalability & Reliability**

Scaling triggers, recovery-time objectives, and fault-tolerance procedures are not sourced. They are deferred with P2. The MVP reliability bar is FR-10 plus FR-11: the supported result, each escalation condition alone, one question with two escalation reasons, the three written-request labels, and one technical failure behave as specified on the fixtures. Daily capacity is the 200 questions in MC-01.

## 6. User Experience Design

**Interface Requirements**

The MVP has two roles and one customer success path.

- The customer can submit natural language and, on the happy path, read the answer and its source references.
- On an escalation, the customer sees the waiting state in FR-08 and does not see the draft as the answer.
- On a technical failure, the customer sees the sentence in FR-11, the error is logged, and no approval item is created.
- The representative can open the one approval item, read the question and the draft, and record approval. After that approval, the customer receives that same draft and its source references.

Channel, screen layout, visual design, and a disability-access standard are not decided. `@ux.designer` owns them if a surface is in scope. This PRD does not name a UI library or a device size. [MRD §3 Implications]

**Human–system interaction**

- The customer starts the only happy path and does not pass through the representative when the answer is supported.
- The representative enters only through the one gate.
- Explainability on the happy path is the source references on the answer. No further explanation format is specified.
- Errors that are the three escalation conditions use FR-07 and FR-08. A failure that is not one of those conditions uses FR-11. The customer sees “We are facing a technical issue at this point. Check back later.” The draft is not shown as the answer.

## 7. Success Metrics & KPIs

**Business / Operational Metrics**

No business target is set. The CFPB counts and the J.D. Power score stay context. They are not baseline or goal values for the MVP. ROI, cost savings, complaint reduction, a satisfaction gain, and a containment rate are claims this product will not make. [MRD Claims We Will Not Make]

**Technical Metrics**

The MVP succeeds when FR-10 and FR-11 pass and each question meets MC-01. No accuracy percentage is set. A draft’s fee, rate, account status, policy, and loan-fact claims must appear in a cited approved article. That check is MP-03.

**User Experience Metrics**

No satisfaction score and no readability score are set. “Easy to understand” remains operator intent without a test. The operator confirmed that on 2026-11-14. The under-5-second limit in section 5 is a latency limit, not a readability test.

## 8. Implementation Strategy

**Development Phases**

- Phase 1, Define: this PRD is the requirements artifact. User stories are in `project-context/1.define/user-stories/`. The SAD is next and is owned by `@system.arch`.
- Phase 2, Build: setup, then frontend and backend, then integration, then QA, against FR-01 through FR-11.
- Phase 3, Deliver: deploy configuration and runbook after the build artifacts exist. This PRD does not authorize a live deploy.

**Resource Requirements**

The six-week length is the operator’s description of the cohort in the MRD. It is a scope constraint, not a staffed plan and not a sourced estimate. No team size or budget is stated. [MRD Assumptions]

**Risk Mitigation**

| Risk | MVP response | Source |
| --- | --- | --- |
| A customer receives an answer whose source note does not point at an article in the approved library. | FR-03 and FR-04 fail that answer. An unsupported result goes to the one gate under FR-05. | FR-03; FR-04; FR-05 |
| A cited article is real, but a sentence states a fee, rate, account status, policy, or loan fact the article does not contain. | MP-03 and FR-04 fail that draft. The recorded set includes unsupported. | Operator decision 2026-11-14; FR-04; MP-03 |
| Real customer data is loaded as a mock. | Fixtures are invented. A real servicing extract is not authorized. | Operator decision 2026-11-14 |
| An unsourced rate or savings figure becomes a success metric. | Section 7 forbids it. The 200 questions, 5 seconds, and 20 cents are operator limits, not those claims. | MRD Claims We Will Not Make; operator decision 2026-11-14 |
| “Approved” is undefined, so a source can point at an article the operator has not approved. | Tests use the fixture’s approved mark. Two approved articles that disagree escalate as cannot be determined to be safe. | Operator decision 2026-11-14; MD-07 |

## 9. Launch & Go-to-Market Strategy

N/A. This is a six-week capstone on mocked data. It does not pick a commercial segment, a price, or a launch market. [MRD §5; MRD Market positioning]

## Quality Assurance Checklist

- [x] Requirements traceable to the MRD, to the operator `*create-prd` instruction, or to the operator decisions of 2026-11-14. The system description is absent and is not used as a source.
- [x] Technical constraints are stated without a runtime, a pattern, or a roster. The SAD owns those choices.
- [x] Success metrics match the fixture tests and the operator limits of 200 questions a day, under 5 seconds, and at most 20 cents per question. Unsourced business outcomes are not targets.
- [x] MVP scope is P0. P1 is empty. P2 lists deferred work.
- [x] Market sections use the MRD. The MRD was not skipped.

## Sources

- `project-context/1.define/mrd.md`, including its Context, Executive Summary, §§1–5, Claims We Will Not Make, Assumptions, and Open Questions.
- Operator instruction for `*create-prd`, 2026-11-13: MVP only; one user-facing happy path; one human approval gate; mock every external system; testable acceptance criteria; trace to the MRD or the system description; leave pattern, runtime, and agent roster to the SAD; unsourced numbers go to Open Questions.
- Operator decisions, 2026-11-14: accept the customer-specific fact list; the fixture approved mark; conflict escalates as cannot be determined to be safe; a notice of error, a qualified written request, or a request for information escalates as customer-specific; a fee, rate, account status, policy, or loan-fact claim must appear in a cited approved article; no readability test; fixtures are synthetic; who may send in a live operation stays deferred; 200 calls a day, latency under 5 seconds, and at most 20 cents per transaction; no separate concurrency or uptime number; log the error and show “We are facing a technical issue at this point. Check back later.”; drafting may call a model.
- `aamad.config.yml`: `project.mode` is `greenfield`. The file also contains `runtime.target`. That value is not a requirement of this PRD.
- System description: not present at `project-context/1.define/system-description.md`.

## Assumptions

- The MRD is the product scope for users, the five steps, the three escalation conditions, direct delivery of a supported answer, mocked data, and the claims this product will not make.
- “One human approval gate” means the representative’s review of an escalation. It does not add a review of supported answers. The earlier MRD decision that a supported answer is delivered directly remains in force.
- For the MVP, the representative’s approval delivers that same draft and its source references to the customer. This does not decide who may send in a future live servicer.
- The customer-visible waiting state in FR-08 is required so the escalation path can be tested. The words and layout of that waiting state are not specified. The technical-issue sentence is specified in FR-11.
- The expected result is the tester’s answer key. It may list more than one escalation reason. It is not given to the product. The product receives the question and the approved-article library and searches that library. The MVP meanings of customer-specific, approved, and a matching claim are the operator decisions of 2026-11-14.
- “At least one” fixture per outcome is the smallest set that makes FR-10 and FR-11 observable. It is not the daily capacity of 200 questions.
- A call and a transaction are one submitted question. The 20 cents is the model or provider charge recorded for that question. Representative labor is outside it. The under-5-second interval ends when the customer can see the supported answer, the waiting state, or the technical-issue notice. Representative review time is outside it.
- The MRD file still lists the closed questions as open. It is the research snapshot. This PRD records the later operator decisions.
- “Easy to understand” stays intent. No readability score was added.
- The six-week cohort is a scope constraint from the MRD, not a delivery estimate.
- `runtime.target` in `aamad.config.yml` is not adopted here, because the operator assigned runtime to the SAD.
- Launch and go-to-market are N/A, as stated in section 9.

## Open Questions

Closed by the operator on 2026-11-14: the customer-specific fact list, the fixture approval mark, a conflict between approved articles, escalation of a notice of error, a qualified written request, or a request for information, the claim-in-article check, no readability test, synthetic fixtures, the technical-issue notice, the model-provider rule, and the limits of 200 questions a day, under 5 seconds, and at most 20 cents per question. No concurrency number and no uptime number were added.

Still unresolved:

- After this MVP, in a live operation, who may send the response once the representative has approved it?
- What should the product do when more than 200 questions are submitted in a day?

## Audit

- Timestamp: 2026-11-14T13:56:00-05:00
- Persona id: `product-mgr`
- Action: `create-prd` (revision: operator decisions of 2026-11-14 close the open questions listed above and set MC-01, MP-03, MD-06, MD-07, MD-08, and FR-11)
- Prior revision: 2026-11-14T11:15:00-05:00, the product records every escalation reason that applies. 2026-11-14T11:10:00-05:00, the risk table claims only the source-note check. 2026-11-14T10:59:00-05:00, after approval the customer receives that same draft. 2026-11-14T10:46:00-05:00, expected fixture result is hidden from the product. Original write: 2026-11-13T22:44:29-05:00, action `create-prd`.
- Resolved runtime: not recorded. Pattern, runtime, and agent roster are left to the SAD. `runtime.target` in `aamad.config.yml` is not a requirement of this document. `AAMAD_TARGET_RUNTIME` was not applied.
- Prompt trace: omitted. This artifact is Define-phase requirements traced to the MRD and the operator decisions. It is not a production model response and it does not select an application pattern.
- Scope honored: MVP only; one customer happy path; one human approval gate; systems of record mocked; drafting may call a model; operator-sourced limits only; no SAD; no application code.
