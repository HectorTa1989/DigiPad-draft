# DigaPad - SHB Digital Expert Agents

## Vietnam AI Innovation Challenge 2026

**Topic:** Digital Expert Agents — A Team of AI Specialists for Banking Operations

A multi-agent AI system in which each agent acts as a digital expert in a specific banking domain. The agents autonomously plan tasks, retrieve internal knowledge through domain-specific RAG, use tools and APIs, collaborate with one another, and execute controlled actions within banking operational systems.

> **Core value proposition:** A cross-department banking request is automatically decomposed and handled by specialized AI experts with separate permissions. Every recommendation is evidence-based, high-impact actions require human approval, and the entire workflow is auditable.

---

## 1. Problem Overview

Current banking AI applications frequently stop at question answering, document retrieval, or anomaly analysis. This project extends generative AI from **answering questions** to **performing controlled operational work**.

The system must demonstrate four essential capabilities:

1. **Plan:** understand a complex request and decompose it into structured tasks.
2. **Delegate:** assign each task to the correct banking specialist agent.
3. **Act:** call APIs, query databases, calculate results, and update workflow state.
4. **Govern:** enforce authorization, human approval, safety controls, and auditability.

This is not intended to be a general-purpose chatbot. It is an orchestration platform for controlled and explainable banking workflows.

---

## 2. Recommended MVP Use Case

### SME Credit Application Review

A bank employee submits a request such as:

> Evaluate a VND 5 billion working-capital loan application from ABC Company, verify credit eligibility, check internal policies and legal requirements, identify missing documents, and prepare a recommendation for approval.

This use case is suitable because it requires real collaboration across credit, legal/compliance, and operations teams.

### Expected outcome

The system should:

- Read the applicant and company profile.
- Retrieve financial statements and relevant documents.
- Calculate financial ratios and a credit-risk score.
- Check KYC, AML, sanctions, and internal credit policies.
- Detect missing, expired, or inconsistent documents.
- Produce a recommendation supported by evidence.
- Create a draft operational case.
- Pause before any consequential action and request human approval.
- Record all tools, evidence, decisions, and approvals in an audit trail.

---

## 3. Agent Team

### 3.1 Planner/Supervisor Agent

Responsibilities:

- Understand the overall objective.
- Split the request into tasks and dependencies.
- Select the appropriate specialist for each task.
- Run independent tasks in parallel when possible.
- Track task states and intermediate results.
- Resolve conflicts between specialist findings.
- Stop unsafe or non-compliant workflows.
- Request human review when confidence is low or risk is high.
- Aggregate specialist outputs into the final recommendation.

The Supervisor must be more than an intent router. It should manage a typed, stateful execution plan.

Example plan:

```json
{
  "request_id": "REQ-2026-001",
  "goal": "Evaluate an SME credit application",
  "tasks": [
    {
      "task_id": "T1",
      "agent": "credit_agent",
      "action": "calculate_financial_ratios",
      "dependencies": []
    },
    {
      "task_id": "T2",
      "agent": "compliance_agent",
      "action": "check_kyc_and_policy",
      "dependencies": []
    },
    {
      "task_id": "T3",
      "agent": "operations_agent",
      "action": "validate_required_documents",
      "dependencies": []
    },
    {
      "task_id": "T4",
      "agent": "supervisor",
      "action": "produce_recommendation",
      "dependencies": ["T1", "T2", "T3"]
    }
  ]
}
```

Required task states:

```text
pending
running
completed
failed
cancelled
waiting_for_input
waiting_for_approval
```

The orchestration layer must also support:

- Conditional branches.
- Bounded retries.
- Timeouts.
- Checkpoints and resume.
- Maximum-step limits.
- Idempotent tool execution.
- Human escalation.
- Workflow cancellation.

### 3.2 Credit Agent

Responsibilities:

- Analyze financial statements.
- Calculate DSCR, leverage, liquidity, profitability, and debt ratios.
- Retrieve a simulated credit history.
- Evaluate repayment capacity.
- Evaluate collateral data.
- Run deterministic credit-scoring rules.
- Recommend limits, conditions, and risk grades.
- Clearly separate measured facts from AI-generated recommendations.

The LLM should explain and coordinate the analysis, while deterministic code should perform financial calculations.

### 3.3 Legal and Compliance Agent

Responsibilities:

- Check KYC and AML status.
- Check sanctions-screening results.
- Review legal documents.
- Retrieve applicable internal policies and regulations.
- Detect missing or expired documents.
- Produce findings with clause-level citations.
- Mark severe findings as requiring human review.

Example structured finding:

```json
{
  "finding": "The ultimate beneficial owner declaration is missing",
  "severity": "high",
  "source_document": "KYC Policy 2026",
  "clause": "5.3.2",
  "recommendation": "Request a signed UBO declaration",
  "requires_human_review": true
}
```

### 3.4 Operations Agent

Responsibilities:

- Check document completeness.
- Build an operational checklist.
- Create a draft case in the simulated workflow system.
- Assign or recommend a case owner.
- Request missing documents.
- Prepare a submission package.
- Submit only after receiving a valid human approval token.

### 3.5 Validator/Critic Agent

Responsibilities:

- Validate required output schemas.
- Check whether every important claim has supporting evidence.
- Detect contradictory findings between agents.
- Check calculations and decision-rule consistency.
- Verify that no agent exceeded its permissions.
- Reject incomplete output and send it back for bounded revision.

The Validator is not a replacement for human approval. It is a quality gate before the workflow reaches a human reviewer.

---

## 4. Proposed Architecture

```text
Bank Employee / Reviewer
          |
          v
Authentication + RBAC
          |
          v
Input Validation + PII Protection + Guardrails
          |
          v
Planner / Supervisor Agent
          |
          +----------------------+------------------------+
          |                      |                        |
          v                      v                        v
    Credit Agent        Compliance Agent         Operations Agent
          |                      |                        |
 Credit APIs/DBs        Policy and Legal RAG       Workflow APIs
 Scoring Engine         KYC/AML Tools              Document Service
 Financial Parser       Sanctions Check            Case Management
          |                      |                        |
          +----------------------+------------------------+
                                 |
                                 v
                         Validator Agent
                                 |
                                 v
                       Human Approval Gateway
                                 |
                                 v
                      Controlled Action Execution
                                 |
                                 v
                    Audit Log + Metrics + Dashboard
```

### Suggested technology stack

#### Frontend

- React or Next.js.
- React Flow for the workflow graph.
- Server-Sent Events or WebSocket for live state updates.
- Approval console and evidence viewer.

#### Backend

- Python 3.11+.
- FastAPI.
- Pydantic for strict request, tool, and agent-output schemas.
- LangGraph `StateGraph` for durable orchestration.

#### State and persistence

- PostgreSQL for workflow, cases, permissions, and audit metadata.
- LangGraph-compatible checkpoint persistence.
- Redis only if needed for transient locks, queues, or caching.

#### Knowledge and retrieval

- PostgreSQL with `pgvector`, or another supported vector database.
- Hybrid keyword and vector retrieval.
- Object storage for original source documents.

#### Observability

- OpenTelemetry.
- Langfuse or LangSmith for development traces.
- Structured JSON application logs.
- Prometheus/Grafana where infrastructure metrics are needed.

#### Integration

- REST/OpenAPI for banking and workflow services.
- MCP for controlled exposure of appropriate tool groups.
- Mock core-banking services for the hackathon demo.

#### Deployment

- Docker Compose for local demonstration.
- Cloud deployment to Azure, AWS, or GCP if required.
- Secrets supplied through environment variables or a secret manager.

---

## 5. Domain-Specific RAG

Do not place all banking documents into one unrestricted vector index. Use separate domain collections and enforce authorization filters during retrieval.

```text
knowledge/
├── credit/
│   ├── lending-policy/
│   ├── scoring-rules/
│   └── collateral-guidelines/
├── compliance/
│   ├── kyc/
│   ├── aml/
│   ├── internal-regulations/
│   └── legal-documents/
├── products/
│   ├── loan-products/
│   ├── interest-rates/
│   └── eligibility-rules/
└── operations/
    ├── standard-operating-procedures/
    ├── required-documents/
    └── workflow-guidelines/
```

### Required metadata

```json
{
  "document_id": "SHB-CREDIT-001",
  "title": "SME Credit Granting Policy",
  "department": "credit",
  "version": "3.2",
  "effective_date": "2026-01-01",
  "expiry_date": null,
  "status": "active",
  "confidentiality": "internal",
  "clause": "4.2.1"
}
```

### Retrieval requirements

- Hybrid lexical and semantic search.
- Filtering by domain, role, confidentiality, status, and applicable date.
- Effective-version selection.
- Clause-level source attribution.
- Retrieval-score and evidence-coverage tracking.
- No use of expired or unauthorized documents.
- Explicit “insufficient evidence” outcome when support is missing.
- Protection against prompt injection embedded in uploaded documents.
- Retrieved content must be treated as data, not as system instructions.

---

## 6. Tool and API Design

Practical tool use is a central deliverable. A tool action must query real demo data or change stored state; it must not merely return text claiming that an action occurred.

### Credit tools

```text
get_customer_profile()
get_credit_history()
extract_financial_statement()
calculate_financial_ratios()
run_credit_score()
evaluate_collateral()
simulate_repayment_schedule()
```

### Compliance tools

```text
search_policy()
check_kyc_status()
check_sanctions_screening()
check_aml_risk()
validate_legal_documents()
create_compliance_finding()
```

### Operations tools

```text
get_document_checklist()
validate_document_completeness()
create_case_draft()
assign_case_owner()
request_missing_document()
submit_for_approval()
```

### Tool risk levels

#### Read-only

May run automatically after authorization:

```text
get_customer_profile
search_policy
get_document_checklist
```

#### Calculation

May run automatically, but inputs, algorithm version, and outputs must be recorded:

```text
calculate_financial_ratios
run_credit_score
simulate_repayment_schedule
```

#### Draft action

May create reversible draft state:

```text
create_case_draft
create_compliance_finding
request_missing_document_draft
```

#### Approval-required action

Requires an authorized employee and a valid approval token:

```text
submit_credit_application
assign_case_owner
send_missing_document_request
```

#### Prohibited autonomous action

Must not be executed autonomously in the demo:

```text
approve_or_reject_loan
disburse_funds
close_customer_account
modify_core_ledger
```

Every mutating tool should support:

- Authorization checks.
- Input schema validation.
- Idempotency keys.
- Dry-run mode where appropriate.
- Timeout and bounded retry.
- Immutable execution record.
- Compensation or rollback for reversible actions.

---

## 7. Agent Authorization Model

Each agent must have a separate identity and least-privilege permissions.

### Credit Agent

- May read authorized customer financial data.
- May call scoring and calculation tools.
- May create a credit recommendation draft.
- Must not approve a loan.
- Must not modify legal documents.

### Compliance Agent

- May read authorized KYC and policy information.
- May create compliance findings.
- May block progression by raising a high-severity finding.
- Must not modify transactions or customer balances.

### Operations Agent

- May update a checklist and create draft cases.
- May submit only after human approval.
- Must not alter risk scores or compliance findings.

### Supervisor Agent

- May delegate tasks and aggregate outputs.
- May not bypass a specialist's permission boundary.
- May not grant itself new tools.
- May not approve a consequential business decision.

---

## 8. Human-in-the-Loop Controls

The system must not allow an LLM to autonomously approve credit, reject a customer, disburse funds, freeze an account, or perform another high-impact banking action.

The dashboard should support:

- Approve.
- Reject.
- Request revision.
- Request more evidence.
- Reassign.
- Cancel workflow.
- Inspect source documents and calculations.

An approval token should be bound to:

```text
user_id
user_role
action
request_id
payload_hash
data_version
issued_at
expires_at
```

Binding the approval to a payload hash prevents an agent from changing the action after approval.

---

## 9. Dashboard and Explainability

The dashboard should display a structured business trace rather than a model's private chain of thought.

Example trace:

```text
✓ Supervisor created four tasks
✓ Credit Agent retrieved authorized financial data
✓ Credit Agent calculated DSCR and leverage ratios
✓ Compliance Agent retrieved three applicable policy clauses
! Compliance Agent detected a missing UBO declaration
✓ Operations Agent created a document checklist and draft case
⏸ Workflow is waiting for a compliance officer
```

### Dashboard views

- Request summary.
- Live workflow graph.
- Agent and task status.
- Tool calls and execution outcome.
- RAG evidence and source version.
- Financial calculations.
- Findings and risk severity.
- Decision summary.
- Human approval queue.
- Retry and error history.
- Audit timeline.
- Latency, token usage, and estimated cost.
- Single-agent versus multi-agent benchmark.

### Explainability rule

Display:

- Decision summary.
- Evidence.
- Rules and calculations.
- Tool and action trace.

Do not display hidden model reasoning or token-by-token chain of thought.

---

## 10. Security and Governance

Minimum controls:

- User and agent RBAC.
- Tool-level authorization.
- Separate agent identities.
- Encryption in transit and at rest.
- Secret management.
- PII masking and redaction.
- Prompt-injection scanning.
- Tool and outbound-domain allowlists.
- Strict input/output schema validation.
- Append-only audit logs.
- Action idempotency.
- Bounded retries and execution limits.
- Workflow kill switch.
- Data-retention policy.
- Separation of prompts, knowledge documents, and transactional data.

### Audit event example

```json
{
  "event_id": "EVT-2026-00042",
  "request_id": "REQ-2026-001",
  "task_id": "T2",
  "actor_type": "agent",
  "actor_id": "compliance_agent",
  "event_type": "tool_completed",
  "tool": "check_kyc_status",
  "input_hash": "sha256:...",
  "output_hash": "sha256:...",
  "status": "success",
  "timestamp": "2026-07-17T08:00:00Z"
}
```

Do not store raw sensitive data in general-purpose logs.

---

## 11. Single-Agent vs Multi-Agent Benchmark

A performance comparison is a required deliverable and should be designed at the beginning, not added just before the demonstration.

### Single-agent baseline

- One LLM receives the complete request.
- One shared knowledge base.
- One shared tool collection.
- The same model produces the final outcome.

### Multi-agent implementation

- Supervisor creates a plan.
- Specialist agents operate within separate domains and permissions.
- Validator checks completeness and consistency.
- Supervisor aggregates findings.
- Human approval is required before consequential actions.

### Evaluation dataset

Create 30–50 representative test cases covering:

- Complete and valid applications.
- Missing required documents.
- Expired policies or documents.
- Conflicting customer information.
- Borderline credit scores.
- AML or sanctions alerts.
- Tool/API failures.
- Unauthorized action attempts.
- Prompt injection inside an uploaded document.
- Requests requiring human escalation.

### Metrics

- End-to-end task success rate.
- Policy-citation accuracy.
- Evidence coverage.
- Tool-call success rate.
- Missing-document detection rate.
- Calculation accuracy.
- Unauthorized-action rate.
- Hallucination rate.
- Decision consistency.
- Human-escalation precision.
- Recovery rate after tool failure.
- Completion latency.
- Token usage.
- Estimated cost per request.

The target is not to claim that multi-agent is universally better. The report should identify where specialization, permissions, and parallel execution improve quality, and where they add latency or cost.

---

## 12. Suggested Project Structure

```text
shb-digital-expert-agents/
├── README.md
├── .env.example
├── docker-compose.yml
├── docs/
│   ├── architecture.md
│   ├── security.md
│   ├── evaluation.md
│   └── demo-script.md
├── backend/
│   ├── pyproject.toml
│   ├── app/
│   │   ├── main.py
│   │   ├── api/
│   │   ├── agents/
│   │   │   ├── supervisor.py
│   │   │   ├── credit.py
│   │   │   ├── compliance.py
│   │   │   ├── operations.py
│   │   │   └── validator.py
│   │   ├── orchestration/
│   │   │   ├── graph.py
│   │   │   ├── state.py
│   │   │   ├── routing.py
│   │   │   └── checkpoints.py
│   │   ├── tools/
│   │   │   ├── credit_tools.py
│   │   │   ├── compliance_tools.py
│   │   │   └── operations_tools.py
│   │   ├── rag/
│   │   │   ├── ingestion.py
│   │   │   ├── retrieval.py
│   │   │   └── authorization.py
│   │   ├── security/
│   │   │   ├── auth.py
│   │   │   ├── permissions.py
│   │   │   ├── guardrails.py
│   │   │   └── pii.py
│   │   ├── approvals/
│   │   ├── audit/
│   │   ├── models/
│   │   └── services/
│   └── tests/
│       ├── unit/
│       ├── integration/
│       ├── security/
│       └── evaluation/
├── frontend/
│   ├── package.json
│   └── src/
│       ├── components/
│       ├── features/
│       │   ├── workflow/
│       │   ├── evidence/
│       │   ├── approvals/
│       │   └── benchmark/
│       └── services/
├── mock-services/
│   ├── customer-api/
│   ├── credit-api/
│   ├── compliance-api/
│   └── workflow-api/
├── knowledge/
│   ├── credit/
│   ├── compliance/
│   ├── products/
│   └── operations/
└── evaluation/
    ├── cases/
    ├── expected-results/
    └── reports/
```

---

## 13. Recommended Delivery Phases

### Phase 1 — Foundation

- Define the SME credit workflow.
- Define typed graph state and task schemas.
- Build mock customer, credit, compliance, and workflow APIs.
- Implement authentication, user roles, and agent identities.

### Phase 2 — Specialists

- Implement Credit, Compliance, and Operations agents.
- Implement deterministic calculation and validation tools.
- Add separate RAG domains and document metadata.

### Phase 3 — Orchestration

- Build Supervisor planning and delegation.
- Add parallel execution, dependencies, checkpointing, and retry.
- Add Validator quality gates.

### Phase 4 — Controlled execution

- Add draft mutations and idempotency.
- Add human approval tokens.
- Add audit logging and permission enforcement.

### Phase 5 — Dashboard

- Add live workflow graph and task timeline.
- Add evidence, calculations, and tool-call views.
- Add approval and audit screens.

### Phase 6 — Evaluation

- Build the single-agent baseline.
- Run both systems against the same test set.
- Produce metric comparisons and failure analysis.

---

## 14. Demo Scenario

A strong five-to-seven-minute demonstration can follow this sequence:

1. A bank employee opens an SME loan request.
2. The dashboard displays the Supervisor's structured plan.
3. Credit, Compliance, and Operations agents begin independent tasks.
4. Live traces show actual API and retrieval calls.
5. The Credit Agent calculates financial ratios.
6. The Compliance Agent cites applicable policy clauses and detects a missing UBO declaration.
7. The Operations Agent creates a draft case and checklist in the mock workflow database.
8. The Validator detects that the missing UBO document prevents submission.
9. The workflow pauses for a compliance officer.
10. The officer inspects evidence and requests the missing document.
11. The action executes through an approved tool and appears in the audit timeline.
12. The dashboard compares the result with the single-agent baseline.

---

## 15. Success Criteria

The MVP is successful when it can demonstrate all of the following:

- At least three specialist agents collaborate on one complex request.
- A Supervisor decomposes and delegates the work.
- Agents invoke functional APIs or data tools.
- RAG results include document and clause evidence.
- Agent permissions are enforced.
- A draft case is actually persisted.
- High-impact actions stop for human approval.
- The dashboard shows tasks, tools, decisions, evidence, and status.
- Every action has an auditable event.
- A repeatable benchmark compares single-agent and multi-agent behavior.

---

## 16. Repositories for Reference

Use these projects as architectural references rather than copying one repository wholesale. Review each license before reusing code.

### Agentic Banking Ecosystem

- Repository: https://github.com/denniszielke/agentic-banking-ecosystem
- Useful patterns: enterprise banking agents, MCP, A2A communication, Microsoft Foundry, Entra ID, Azure AI Search, and OpenTelemetry.
- Best use: identity, tool authorization, enterprise integration, and observability.

### RAG Retail Banking Agent with LangGraph

- Repository: https://github.com/saswanth/RAG-RETAIL-BANKING-AGENT-WITH-LANGGRAPH
- Useful patterns: loan-origination workflow, Supervisor/Worker architecture, LangGraph state, tools, API, tests, and UI.
- Best use: fast MVP foundation for a credit workflow.

### Banking Compliance Agent

- Repository: https://github.com/romit-98/banking-compliance-agent
- Useful patterns: policy retrieval, compliance reasoning, report generation, PII redaction, prompt-injection defense, and hash-chained audit logs.
- Best use: Compliance Agent, evidence-based output, and security controls.

### AI Multi-Agent Loan Underwriting

- Repository: https://github.com/shruti-gavhane/AI-Multi-Agent-Loan-Underwriting
- Useful patterns: customer verification, financial analysis, fraud detection, risk assessment, loan decision, and letter generation.
- Best use: understanding how an underwriting workflow can be split across specialists.

### Banking Multi-Agent System

- Repository: https://github.com/ayeshashafiq2036-source/Banking-Multi-Agent-System
- Useful patterns: a small LangGraph example with Router, RAG, and Action agents.
- Best use: learning the basic routing and tool-action pattern.

### LangGraph Multi-Agent Supervisor

- Repository: https://github.com/langchain-ai/langgraph-supervisor-py
- Useful patterns: hierarchical agent coordination and tool-based handoffs.
- Important note: for greater context and policy control, consider implementing the supervisor pattern directly with tools and a typed LangGraph workflow.

### Microsoft Agent Framework

- AutoGen repository: https://github.com/microsoft/autogen
- Microsoft Agent Framework announcement: https://devblogs.microsoft.com/foundry/introducing-microsoft-agent-framework-the-open-source-engine-for-agentic-ai-apps/
- Best use: evaluating an enterprise-oriented alternative with MCP, A2A, OpenAPI, identity, and observability integration.

---

## 17. Final Recommendation

Build the project as a clean-room, focused SME credit workflow rather than a generic banking assistant or a cosmetic fork of one repository.

Combine the strongest patterns from the reference projects:

1. Use an enterprise agent identity, integration, and observability model.
2. Use a stateful Supervisor/Worker workflow for loan origination.
3. Use domain-specific, evidence-based compliance RAG.
4. Enforce least privilege and human approval before consequential actions.
5. Demonstrate actual state changes and a measurable comparison against a single-agent baseline.

The strongest competition narrative is:

> **From banking chatbot to governed digital workforce: specialized AI agents plan, collaborate, use real tools, cite bank policies, request approval at the correct control points, and leave a complete audit trail.**

---

## Disclaimer

This repository describes a hackathon prototype. It must use synthetic or properly anonymized data and must not be used to make real credit, legal, compliance, or financial decisions without full institutional validation, security review, regulatory review, model-risk governance, and authorized human oversight.
