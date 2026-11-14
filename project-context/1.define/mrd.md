# Market Research Document — Mortgage Servicing Customer Support Crew

## Context & Instructions

This document is market and domain research for a greenfield Define-phase capstone. It does not select a runtime, an application pattern, an agent roster, or components. Those decisions belong to later phases.

Evidence rule: a figure appears only when a named source states it. The operator brief, the 2026-11-13 review, and the later user decision on 2026-11-13 are product scope, not market evidence. Sample use-case pages on Maven are selection briefs and are not cited. Vendor prices, vendor use-case pages, and figures from a document this research did not read are omitted. A missing figure is left out. It is not replaced with an estimate.

**Primary user:** the mortgage customer seeking answers to mortgage servicing questions. The customer submits a mortgage servicing question in natural language. A supported answer is delivered directly, with source references, without first reaching a representative. In this document, “accessible” means that direct path. It does not set a disability-access standard, and it does not choose a channel.

**Secondary user:** the mortgage customer support representative. The representative handles escalations under the three conditions below. Complex questions and exceptions in this document are those three conditions.

**Product vision.** Operator intent for this capstone is a better support experience for the mortgage customer: accessible, accurate, consistent, and timely answers. These words describe the intended experience. They are not measured outcomes, and they do not add a latency limit, a readability score, or a satisfaction gain.

**Primary focus:** One complete workflow, in this order:

- Customer submits a mortgage servicing question in natural language.
- Solution classifies the question.
- Solution retrieves relevant information from approved knowledge.
- Solution drafts an answer with source references.
- Solution determines whether to deliver the answer directly or escalate it for human review.

A supported answer is delivered directly to the customer. An escalation sends the question and the draft to the representative for review before a response is sent.

**Approval policy.** A supported answer is supported by the approved knowledge base, does not require customer-specific loan information, and can be determined to be safe. That answer is delivered directly to the customer. An escalation sends the question and the draft to the representative for review before a response is sent. The operator decision of 2026-11-13 closes the earlier question of whether every supported response also requires human approval.

**Escalation conditions.** The system stops and escalates the question and the draft to the representative for human review before a response is sent when any of these is true:

- The answer is unsupported.
- The question requires customer-specific loan information.
- The answer cannot be determined to be safe.

**What is mocked:** Mortgage customer records, loan servicing information, and knowledge articles. No live integration with mortgage servicing platforms, CRM systems, Salesforce, or Zendesk.

**Cohort:** A six-week project, as stated in the operator review of 2026-11-13. The length is a scope constraint for this capstone.

**Research date:** 2026-11-12. User-role revision: 2026-11-13. J.D. Power source and workflow-sequence revision: 2026-11-13.

**Product scope:** Operator brief of 2026-11-12, the operator review of 2026-11-13, and the later operator decision on 2026-11-13 that the mortgage customer is the primary user. Facts about the product below are that scope. Facts about the market are cited.

## Executive Summary

**Market opportunity.** The domain problem is mortgage servicing communication. The Consumer Financial Protection Bureau (CFPB) received approximately 30,400 mortgage complaints in calendar 2025. Trouble during the payment process was the top issue, and its monthly average was 8 percent higher than the monthly average of the prior two years. Consumers described unreachable servicer representatives, unreturned calls and emails, and inaccurate or conflicting information. In 92 percent of the complaints the Bureau sent to companies for review, consumers reported first trying to resolve the issue with the company. J.D. Power’s 2026 servicer study reports overall satisfaction of 607 on a 1,000-point scale, an increase of 11 points this year. The release says that increase comes as servicers improve digital experiences, communication around escrow and fees, and issue resolution. Poor customer service is the leading reason customers consider switching, at 43 percent. Quality customer service and strong self-service are the top loyalty drivers, each at 62 percent. These findings are the market context for the mortgage customer, who is the primary user of this capstone. They are not a business case, and the rise in the published score is not evidence that this capstone will improve satisfaction. No return, savings, complaint reduction, satisfaction gain, or containment rate is stated.

**Technical feasibility.** A six-week cohort can demonstrate one complete workflow on mocked data. The customer submits a mortgage servicing question in natural language. The solution classifies the question, retrieves relevant information from approved knowledge, and drafts an answer with source references. It then determines whether to deliver the answer directly or escalate it for human review. A supported answer is delivered directly to the customer. An escalation sends the question and the draft to the representative for review before a response is sent. Mocked customer records, loan information, and knowledge articles are enough to show those steps, because the steps need an inquiry, an approved corpus, and a draft that can be checked against that corpus. A question that needs customer-specific loan information still escalates. Mocked records are not a live servicing lookup. A live servicing platform is outside the brief. NIST describes confabulation as a generative system confidently presenting false content. The CFPB’s June 2023 chatbot spotlight found inaccurate information and weaker help as problems become more complex. Those sources are why an unsupported, customer-specific, or unsafe answer stops. They are not a measured accuracy rate for this capstone.

**Recommended approach.** Keep Define on this cohort. The next product artifacts, when the operator asks for them, are the PRD and user stories. This revision does not write them. Leave runtime, pattern, roster, and components to later phases. Carry forward the mortgage customer as the primary user, the representative as the secondary user for the three escalation conditions, the five-step workflow, the mocked-data boundary, and a supported answer delivered directly to the customer. The vision words — accessible, accurate, consistent, and timely — stay operator intent, not acceptance scores.

## Detailed Findings by Dimension

### 1. Market Analysis and Opportunity Assessment

**Key insights**

1. The CFPB’s 2025 Consumer Response Annual Report is the named account of mortgage complaints used here. Consumers reported difficulty communicating with companies during servicing: they were unable to reach representatives, calls or emails went unreturned, or they received inaccurate or conflicting information. Trouble during the payment process was the top mortgage issue. [1]
2. J.D. Power’s 23 July 2026 press release is the named account of servicer satisfaction used here. Overall customer satisfaction with mortgage servicers increases 11 points to 607 on a 1,000-point scale. The release says that increase comes as servicers improve digital experiences, communication around escrow and fees, and issue resolution. Poor customer service, at 43 percent, is the leading reason customers consider switching servicers. Quality customer service, at 62 percent, ties strong self-service as a top driver of loyalty. Easy payment options are listed with them, at 61 percent. [2]
3. Neither source describes a representative’s desk procedure, and neither is a study of this capstone. On an escalation, the assumed current work is that the representative manually searches FAQs, knowledge articles, servicing procedures, and relevant customer information before preparing a response. No user research in this project observed that search. The customer’s published problem is the CFPB account above.

**Data points**

| Item | Figure | Period | Named source |
| --- | --- | --- | --- |
| Mortgage complaints | approximately 30,400 | Calendar 2025 | CFPB [1] |
| Top mortgage issue | Trouble during the payment process; monthly average 8 percent higher than the prior two years | Calendar 2025 | CFPB [1] |
| Consumers who first tried the company | 92 percent of complaints sent for company review | Calendar 2025 | CFPB [1] |
| Servicer satisfaction | 607 on a 1,000-point scale; increases 11 points this year | 2026 study | J.D. Power [2] |
| Leading reason customers consider switching | Poor customer service, 43 percent | 2026 study | J.D. Power [2] |
| Top drivers of loyalty | Quality customer service, 62 percent; strong self-service, 62 percent; easy payment options, 61 percent | 2026 study | J.D. Power [2] |

**Source citations.** [1], [2].

**Implications.** The PRD should treat the mortgage customer’s servicing question as the job. A supported answer is delivered directly to the customer. An escalation sends the question and the draft to the representative for review before a response is sent. Complaint counts and satisfaction points are context. They are not capstone acceptance tests, and they are not a target for complaint reduction or a satisfaction gain. The 11-point rise in the 2026 J.D. Power score is that study’s result. It is not evidence that this capstone will improve satisfaction.

### 2. Technical Feasibility and Requirements Analysis

**Key insights**

1. The operator boundary is the feasibility boundary. Mocked customer records, mocked loan information, and mocked knowledge articles are in scope. Live mortgage servicing platforms, CRM systems, Salesforce, and Zendesk are out of scope. [Operator brief]
2. **Why this thin slice is sufficient.** The cohort is six weeks, and the thing to demonstrate is one complete workflow rather than a servicing operation.    The workflow has five steps, in this order, on the same inquiry:
   - Customer submits a mortgage servicing question in natural language.
   - Solution classifies the question.
   - Solution retrieves relevant information from approved knowledge.
   - Solution drafts an answer with source references.
   - Solution determines whether to deliver the answer directly or escalate it for human review.
   A supported answer is delivered directly to the customer. An escalation sends the question and the draft to the representative for review before a response is sent. The escalation conditions remain the three already stated. A question that needs customer-specific loan information still escalates. Mocked loan records are not a live servicing lookup. Mocked data is sufficient because the steps can be shown when the records, the loan information, and the articles are present and the articles that may be used are identifiable as approved. A limited set of inquiries is sufficient because the demonstration is that one inquiry can travel through the five steps, including a supported answer delivered directly and an escalation under each of the three conditions. The same limited set cannot support a rate of complaint reduction, a change in satisfaction, a containment rate, or a savings figure. Those claims are listed under Claims We Will Not Make.
3. NIST describes confabulation as a generative system generating and confidently presenting erroneous or false content, including output that diverges from the prompt or contradicts earlier statements in the same context. NIST says this follows from predicting likely tokens. Retrieval from an approved knowledge base limits the draft when the draft is held to the retrieved passage and the passage is shown. The standard does not itself classify a mortgage question. NIST AI 600-1 was published in July 2024, which is older than the 18-month preference in the research template. It is included because it is the named definition of that failure. [3]
4. The CFPB’s 6 June 2023 chatbot spotlight, also older than 18 months and included because it is the Bureau’s named issue spotlight on this practice, found that chatbot effectiveness wanes as problems become more complex, that people receive inaccurate information, and that repeating system information back to a customer who is disputing that information does not handle the dispute. The Office of the Comptroller of the Currency (OCC) examiner framework attached to news release 2026-97 says hallucinations concerning fees, rates, account status, or policies could contribute to legal or customer-harm exposure. The PDF extract reviewed did not print a calendar date. This capstone is not a bank and is not a compliance determination under that framework. The framework also describes more than one human-oversight pattern. For this capstone, a supported answer is delivered directly to the customer. An escalation sends the question and the draft to the representative for review before a response is sent. [4][5]

**Data points.** No named source in this document reports an accuracy rate for mortgage answers grounded in an approved knowledge base. No latency, cost, or volume figure was in the operator brief.

**Source citations.** Operator brief; operator decisions of 2026-11-13; [3], [4], [5].

**Implications.** The outcome is an answer that cites the approved material it used. A supported answer is delivered directly to the customer. An escalation sends the question and the draft to the representative for review before a response is sent. Runtime, application pattern, components, and agent roster are not selected here.

### 3. User Experience and Workflow Analysis

**Key insights**

1. **Primary user.** The primary user is the mortgage customer seeking answers to mortgage servicing questions. The customer submits the question in natural language. A supported answer is delivered directly when that answer is supported by the approved knowledge base, does not require customer-specific loan information, and can be determined to be safe. The customer can receive that answer without first reaching a representative. No channel is chosen.
2. **Secondary user.** The secondary user is the mortgage customer support representative. On an escalation, the question and the draft are sent to the representative for review before a response is sent. That happens when the answer is unsupported, the question requires customer-specific loan information, or the answer cannot be determined to be safe. Complex questions and exceptions in this document are those three conditions.
3. **Customer journey.** The customer submits a mortgage servicing question in natural language. The solution classifies the question, retrieves relevant information from approved knowledge, and drafts an answer with source references. The solution then determines whether to deliver the answer directly or escalate it for human review. A supported answer is delivered directly to the customer. An escalation sends the question and the draft to the representative for review before a response is sent. Mocked loan records do not answer a question that needs customer-specific loan information. That question still escalates.
4. **Human approval.** Human review before a response is sent is required for an escalation under the three conditions. It is not required when a supported answer is delivered directly. The operator decision of 2026-11-13 closes that question. The CFPB spotlight is the published description of the customer-service failure this stop is meant to avoid: inaccurate information, weaker help as the problem becomes more complex, and a reply that repeats disputed system information. On an escalation, the assumed current work of the representative is a manual search of FAQs, knowledge articles, servicing procedures, and relevant customer information before a response is prepared. That search was not observed in user research. [1][4][5]

**Data points.** The J.D. Power figures and the CFPB complaint figures in dimension 1. No task-success rate, handle time, or customer-effort score was in the operator brief, and none is added.

**Source citations.** Operator brief; operator decisions of 2026-11-13; [1], [2], [4], [5].

**Implications.** Visual design, channel, and screen layout are not decided here. `@ux.designer` owns interaction design if a surface is in scope after the PRD. The workflow constraint the PRD must keep is the five-step sequence, direct delivery of a supported answer to the customer, and, on an escalation, review by the representative of the question and the draft before a response is sent.

### 4. Production and Operations Requirements

**Key insights**

1. This increment is a six-week demonstration on mocked data. It is not a production servicing operation. The notes below are constraints a later live deployment would have to face. They are not a deployment design.
2. The control the brief sets, before any hosting choice, is retrieval from an approved knowledge base that the draft can cite. A supported answer is delivered directly to the customer. An escalation sends the question and the draft to the representative for review before a response is sent. The OCC framework says testing can examine hallucination, verified sources, and categories that require human verification. An evaluation set for this capstone still has to be agreed with the operator. [5]
3. Whether the mocked records must be entirely synthetic is an open question. The brief did not provide a dataset.
4. No uptime target, recovery-time objective, or cost ceiling was in the brief. No budget figure is stated.

**Data points.** None added. Operational cost and capacity figures were not in the brief and are not invented here.

**Source citations.** Operator brief; operator decisions of 2026-11-13; [5].

**Implications.** The PRD should keep evaluation on mocked records and an approved article set, and should keep a live servicing platform out of scope until the operator adds it. `@security.eng` assesses a built increment later. This document does not write that assessment.

### 5. Innovation and Differentiation Analysis

**Key insights**

1. The job, as the operator specified it, is the five-step workflow on mocked data. A supported answer is delivered directly to the customer. An escalation sends the question and the draft to the representative for review before a response is sent. That statement is the selection scope. It is not a market measurement, and it is not a claim that the workflow outperforms the approaches below.
2. **Existing alternatives.** Established points are separated from assumptions. Limitations that were not read in a named source are not stated.

| Approach | Established in this document | Assumption, not a finding | What this document does not claim |
| --- | --- | --- | --- |
| Manual FAQ and knowledge-base searches | A person can read an FAQ or a knowledge article. No named source here measures how mortgage servicing representatives search. | On an escalation, the representative manually searches FAQs, knowledge articles, servicing procedures, and relevant customer information before preparing a response. | That the search is slow, error-prone, or improved by this capstone. |
| Existing customer service platforms | The operator brief names mortgage servicing platforms, CRM systems, Salesforce, and Zendesk as live systems this capstone will not connect to. Their current feature lists were not read. | None about their limits. A limit would be an assumption, so none is recorded. | That this capstone replaces those platforms or is more capable than they are. |
| Internal AI assistants | The CFPB spotlight found that consumer-finance chatbots give less useful help as problems grow more complex, that people receive inaccurate information, and that repeating disputed system information does not resolve the dispute. NIST describes confabulation for generative systems. These findings describe that class of system. They are not a census of servicers’ internal assistants. [3][4] | That a given servicer’s assistant has those failures, or that this capstone would avoid them. | Any accuracy, containment, or superiority comparison. |
| Manual research and response preparation | A representative can prepare a customer response without this system. No timed study of that preparation is cited. | On an escalation, the representative prepares the response by hand from the materials in the row above. | That the customer-facing workflow is faster or more accurate than that manual preparation. |

3. No patent search was performed. This document does not claim the job is free of existing patents.
4. Monetization is outside the brief. The brief defines a capstone with mocked data. No price is stated.

**Data points.** None. Containment, price per resolution, cost per contact, and savings are not stated, because no named source read for this revision supplies them.

**Source citations.** Operator brief; operator decisions of 2026-11-13; [3], [4].

**Implications.** The PRD positions a single customer-initiated workflow on mocked servicing questions. The comparison above is context for that demonstration. It is not a competitive ranking.

## Critical Decision Points

**Go / no-go factors**

- Go, for Define and a later mocked build, if the product remains this cohort: the five-step sequence on mocked records and an approved knowledge base, a supported answer delivered directly to the customer, and an escalation that sends the question and the draft to the representative for review before a response is sent.
- No-go, for this increment, if the goal becomes a live answer from a servicing platform, CRM, Salesforce, or Zendesk, or a replacement for those platforms.
- No-go, as a planning input, if a business case requires a dollar return, a complaint reduction, a satisfaction gain, or a containment rate. This document does not provide those figures.

**Technical architecture choices**

Not made. Runtime, application pattern, reference architecture, components, and agent roster belong to later phases. Constraints for that later work, without a design, are: the mortgage customer is the primary user; the representative is the secondary user for escalation; a supported answer from the approved corpus is delivered directly to the customer; an escalation sends the question and the draft to the representative for review before a response is sent; data in this cohort is mocked. Customer-specific loan questions escalate. Mocked records are not a live loan lookup.

**Market positioning**

The primary user is the mortgage customer. The secondary user is the mortgage customer support representative, who handles the three escalation conditions. The value inside the capstone is one completed pass through the five steps: a supported answer is delivered directly to the customer, or the question and the draft go to the representative for review before a response is sent. The intended experience is accessible, accurate, consistent, and timely, as operator intent. The named market context is the CFPB 2025 complaint account and the J.D. Power 2026 servicer release. This project does not pick a commercial segment or a price, and it does not claim to be superior to the alternatives in dimension 5.

**Resource requirements**

The operator described a six-week cohort. No team size or budget was stated. The next product work, when requested, is a PRD and user stories from this scope. Architecture, UX if a new surface is in scope, Build, and Deliver follow the existing phase order. This revision stops at the MRD.

## Risk Assessment Matrix

**High risk**

- A draft that states a fee, rate, account status, policy, or customer-specific loan fact the approved material does not contain, and that reaches the customer because it was treated as supported. The CFPB spotlight and the OCC framework describe inaccurate statements of that kind as consumer harm. The brief requires a stop when the answer is unsupported, requires customer-specific loan information, or cannot be determined to be safe. The in-scope response is to escalate the question and the draft to the representative for human review before a response is sent. [4][5][Operator brief]
- Using real customer data inside a mock. The brief did not provide a dataset. Whether the records must be synthetic is an open question.

**Medium risk**

- Treating ROI, cost savings, complaint reduction, a satisfaction gain, or a containment rate as a success metric. No such figure is sourced in this document. A supported answer delivered directly to the customer is a workflow rule, not a containment rate.
- An approved knowledge base that is undefined, so a source reference can point at material the operator has not called approved. The approval rule for an article is an open question.
- Stating a limitation of an existing customer service platform or an internal assistant that this research did not verify, and then treating the capstone as the remedy. Dimension 5 does not do that.

**Low risk**

- The age of NIST AI 600-1 (July 2024) and the CFPB chatbot spotlight (June 2023). They are kept as the named documents for confabulation and for inaccurate chatbot answers. They are not acceptance tests. [3][4]

## Actionable Recommendations

**Immediate next steps**

This revision completes the MRD only. It does not write the PRD or the user stories. Before those artifacts, resolve the open questions that change acceptance tests: what counts as customer-specific, what “approved” means for an article, what evidence shows a draft is accurate, and what evidence shows an answer is easy to understand.

**Short-term priorities**

When the operator asks for stories, write them for the five-step sequence and the two outcomes: a supported answer delivered directly to the customer, and an escalation that sends the question and the draft to the representative for review before a response is sent. Keep stories free of runtime, component, and roster language.

**Long-term strategy**

A live servicer product, with a system of record, is a different scope from this six-week cohort. It is not scheduled here. The named sources say inaccurate customer-service statements can harm consumers. They do not set this cohort’s build plan, and they do not make this capstone a replacement for an existing servicing platform.

## Claims We Will Not Make

This cohort does not supply the evidence these claims would need. They are not implied by the CFPB complaint counts, the J.D. Power scores, the existence of the five-step workflow, the vision words, or the use of mocked data.

- **ROI.** No investment figure and no return figure are in the brief or in the sources retained here.
- **Cost savings.** No cost per contact, wage-based savings, or avoided-work figure is used. The representative’s manual search on an escalation is an assumption, not a timed baseline.
- **Complaint reduction.** The CFPB count describes calendar 2025 complaints. It does not predict a change from this workflow.
- **Customer satisfaction improvements.** The J.D. Power score describes the 2026 servicer study. It is not a before-and-after measure for this capstone. The vision of a better customer experience is operator intent, not this claim. The 11-point rise in that study is not a result for this capstone.
- **Containment rates.** No share of inquiries resolved without a person is sourced. Direct delivery of a supported answer is a workflow rule, not a measured containment rate.
- **Replacement of existing servicing platforms.** The brief forbids live integration with mortgage servicing platforms, CRM systems, Salesforce, and Zendesk. Dimension 5 does not rank this capstone against them.

## Sources

1. Consumer Financial Protection Bureau, *Consumer Response Annual Report*, calendar 2025 complaints, section 4.6 Mortgage, PDF dated March 2026 in the file name. https://files.consumerfinance.gov/f/documents/cfpb_2025-cr-annual-report_2026-03.pdf
2. J.D. Power, “2026 U.S. Mortgage Servicer Satisfaction Study,” press release, 23 July 2026. https://www.jdpower.com/business/press-releases/2026-u-s-mortgage-servicer-satisfaction-study/
3. National Institute of Standards and Technology, *Artificial Intelligence Risk Management Framework: Generative Artificial Intelligence Profile*, NIST AI 600-1, July 2024. https://doi.org/10.6028/NIST.AI.600-1
4. Consumer Financial Protection Bureau, “Chatbots in consumer finance,” 6 June 2023. https://www.consumerfinance.gov/data-research/research-reports/chatbots-in-consumer-finance/chatbots-in-consumer-finance/ and https://files.consumerfinance.gov/f/documents/cfpb_chatbot-issue-spotlight_2023-06.pdf
5. Office of the Comptroller of the Currency, examiner framework attached to news release 2026-97, *Supervisory Framework for Bank Use of Super Intelligence*. https://www.occ.gov/news-issuances/news-releases/2026/nr-occ-2026-97a.pdf The extract reviewed did not print a calendar date.

Operator sources, not market evidence:

- Operator brief, 2026-11-12. The brief’s product facts are the job, the three escalation conditions, the mocked-data boundary, and the exclusion of live mortgage servicing platforms, CRM systems, Salesforce, and Zendesk. The brief named the mortgage customer as the user and the support agent as the person who handles escalations.
- Operator review, 2026-11-13, earlier the same day. That review named the representative as the primary operational user and the customer as the secondary user. It also set the six-week cohort, the five-step workflow, and the claims this document will not make, and it treated the representative’s manual search as an assumption. The user assignment in that review is superseded by the decision below. The cohort, the workflow, the claims list, and the assumption rule are not.
- Operator decision, 2026-11-13, later the same day. The mortgage customer is the primary user and submits the question in natural language. A supported answer is delivered directly to the customer. The representative is the secondary user and handles the three escalation conditions. “Accessible” means the customer can receive a supported answer without first reaching a representative. Complex questions and exceptions are those three conditions. Vision words are intent, not measured outcomes.
- Operator correction, 2026-11-13. The 2025 J.D. Power servicer study is replaced by the 23 July 2026 press release. The five-step sequence is stated in one order. Users, the three escalation conditions, the six-week cohort, and the mocked-data boundary are unchanged.

Not used as evidence: sample use-case pages on Maven. They are selection briefs.

## Assumptions

- The operator brief of 2026-11-12 and the operator decisions of 2026-11-13 are the product scope. Market figures in this file are not acceptance thresholds.
- The mortgage customer is the primary user, and the mortgage customer support representative is the secondary user, because the later operator decision on 2026-11-13 says so. That decision supersedes the user assignment in the earlier review on the same day. The 2026-11-12 brief had already described the customer as the user and the support agent as the escalation handler.
- A supported answer is delivered directly to the customer. An escalation sends the question and the draft to the representative for review before a response is sent. That applies when the answer is unsupported, the question requires customer-specific loan information, or the answer cannot be determined to be safe. This closes the earlier open question on approval of every supported response.
- The 2025 J.D. Power figures previously cited here — satisfaction of 596, a 51 percent switching reason, and a 32 percent communication rating — are not in the 23 July 2026 press release. They are not carried forward. Other 2026 study figures, including brand rankings, financial-health shares, and escrow results, are not added because they do not describe the inquiry workflow.
- On an escalation, the current work is an assumption: the representative manually searches FAQs, knowledge articles, servicing procedures, and relevant customer information before preparing a response. No named source and no user research in this project establish that procedure. This file does not invent a separate workaround for the customer.
- The six-week length is the operator’s description of this cohort. It is not a sourced schedule or a delivery estimate.
- `system-description.md` is not present under `project-context/1.define/` at this revision. The three escalation conditions are taken from the 2026-11-12 brief. The delivery rule for a supported answer is taken from the later operator decision on 2026-11-13, not from that missing file.
- The CFPB annual report also publishes how many mortgage complaints were sent to companies and how companies closed them. Those figures are not restated here, because they describe complaint administration rather than the customer’s inquiry or the escalation path.
- Household mortgage balances, origination volumes, and occupational wage and employment counts were in an earlier revision. They are removed because they do not describe the inquiry workflow. No replacement market-size or wage figure is added.
- Regulation X written-response clocks for a live servicer were in an earlier revision. They are removed because this mocked cohort does not implement live servicer correspondence. Whether a notice of error, a qualified written request, or a request for information always escalates remains an Open Question, without those clocks attached.
- NIST AI 600-1 (July 2024) and the CFPB chatbot spotlight (June 2023) are older than 18 months. They are kept because each is the named document used for that point. The 2026 OCC framework is the newer named supervisory text. The OCC framework applies to banks the OCC supervises. This capstone is not asserted to be in that supervisory population.
- The source count is below the template’s preference of 15 to 20 because statistics and regulatory procedures that do not bear on this workflow were removed, and no further statistics were invented to refill the list.
- “Accessible,” “accurate,” “consistent,” “timely,” and “easy to understand” are operator intent. No latency limit and no readability test were supplied.
- Runtime is intentionally unrecorded. It belongs to a later phase.

## Open Questions

- Which facts count as customer-specific loan information?
- What makes an article approved, and what happens when two approved articles conflict?
- Does a notice of error, a qualified written request, or a request for information always escalate?
- What evidence, beyond a source reference, shows that a draft is accurate?
- What evidence shows that an answer is easy to understand? No readability test was supplied.
- Must mocked records be synthetic only?
- After human review of an escalated question, who may send the response?
- Are any latency, per-interaction cost, or volume limits required? “Timely” in the vision does not set one.

## Audit

- Timestamp: 2026-11-13T19:15:00-05:00
- Persona id: `product-mgr`
- Action: `create-mrd` (revision of the J.D. Power source and the five-step sequence)
- Resolved runtime: not recorded. Runtime, application pattern, components, and agent roster are deferred to later phases.
- Prompt trace: omitted. This artifact is Define-phase research from the named sources and the operator brief and decisions. It is not a production model response, and it does not select a runtime or an application pattern.
- Earlier revision, 2026-11-13T11:15:00-05:00: named the representative as the primary operational user. That user assignment is superseded. The six-week cohort, the claims list, the mocked-data boundary, and the three escalation conditions remain.
- Earlier revision, 2026-11-13T18:40:00-05:00: named the mortgage customer as the primary user and the representative as the secondary user. Those roles remain.
- Revision: replaced the 24 July 2025 J.D. Power servicer release with the 23 July 2026 release, using only figures printed in that release; standardized the five steps and the two outcomes. The CFPB 2025 complaint figures are unchanged. No new integration, runtime, or satisfaction claim was added.
- Scope honored: no PRD, no user stories, no agent roster, no architecture, no component diagram, no visual design. No other file was modified.
