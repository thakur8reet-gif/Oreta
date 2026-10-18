# Oreta — Product Requirements Document

- **Product:** Oreta
- **Working title:** Cloud Attack Path Intelligence Engine
- **Document status:** Draft v0.1
- **Date:** 2026-10-18
- **Product stage:** Concept / specification
- **Initial cloud provider:** AWS
- **Primary users:** Cloud security engineers, DevSecOps engineers, platform engineers, and security students/researchers

## 1. Executive summary

Oreta analyzes cloud infrastructure to identify how network exposure, workload identity, permissions, trust relationships, and sensitive resources may combine into multi-step attack paths.

Traditional configuration checks are useful for identifying known weaknesses, but isolated findings do not always explain their combined impact. Oreta will collect and normalize cloud state, construct a typed relationship/capability graph, evaluate explicit security policies, derive potential paths from defined entry points to sensitive targets, and rank results using explainable risk factors.

Oreta's core must be deterministic and evidence-driven. AI may later assist with summarization, but it will not be the source of truth for inventory, graph edges, policy outcomes, or risk calculations.

## 2. Problem statement

Cloud environments contain many interdependent resources and control planes. Network reachability, IAM permissions, role trust, secrets access, and data sensitivity are often evaluated by separate tools or presented as separate findings. Security teams must manually determine whether these facts combine into a plausible route to a high-value asset.

This creates three problems:

1. **Fragmented context:** Individual findings do not clearly show their relationship to other resources.
2. **Weak prioritization:** Severity alone may not reflect exposure, reachable impact, or the role a finding plays in an attack path.
3. **Opaque conclusions:** Security teams need evidence and assumptions behind a claimed path, not only a score or an AI-generated narrative.

Oreta aims to provide an explainable model of potential attack paths using collected cloud facts and explicit reasoning rules.

## 3. Product vision

Help a cloud owner answer:

> Given the cloud configuration and relationships we can observe, what high-impact assets could become reachable from plausible entry points, through which capability chains, and which changes would reduce that risk most?

## 4. Goals

- Discover a defined subset of AWS assets through read-only APIs.
- Normalize resource data into a provider-independent internal schema where practical.
- Model network, identity, trust, permission, and resource relationships as typed graph edges.
- Evaluate selected established security controls using explicit policies.
- Derive multi-step potential paths between configured entry points and sensitive targets.
- Explain every path with supporting evidence, assumptions, and limitations.
- Rank findings and paths using transparent, configurable factors.
- Show how proposed remediations change the graph and risk ranking.
- Support reproducible offline demos and automated tests before requiring cloud credentials.

## 5. Non-goals

The initial release will not:

- Prove that a path is exploitable in a live environment.
- Automatically exploit resources, execute payloads, or obtain credentials.
- Perform active internet-wide scanning.
- Automatically modify cloud resources or apply remediations.
- Guarantee complete AWS service coverage.
- Replace CSPM, CNAPP, penetration testing, or cloud security expertise.
- Use an LLM as the authoritative evaluator for security policy or reachability.
- Support AWS, Azure, and GCP simultaneously in the MVP.

## 6. Target users and primary jobs

### Persona A: Cloud security engineer
Needs to identify the most consequential relationships among exposures, permissions, and sensitive assets, then produce a defensible remediation plan.

### Persona B: DevSecOps / platform engineer
Needs actionable findings tied to specific resources and configuration evidence, with remediation guidance that can be validated after a change.

### Persona C: Student / security researcher
Needs to understand how cloud identity and network relationships combine, using a reproducible lab and a visual explanation of each derived path.

## 7. Main user journeys

### Journey 1: Analyze a fixture environment
1. User loads a bundled synthetic AWS environment.
2. Oreta parses and normalizes resources.
3. Oreta builds the relationship graph.
4. Policies run against applicable resource attributes.
5. Path analysis evaluates configured entry points and target assets.
6. User opens a path to inspect each edge, evidence source, assumptions, and remediation options.

### Journey 2: Analyze an AWS account
1. User configures AWS credentials using the standard AWS credential chain or a read-only role.
2. Oreta verifies required permissions for supported collectors.
3. User starts a read-only scan.
4. Oreta stores scan metadata and collected configuration evidence while redacting sensitive values.
5. User reviews findings and paths, including coverage gaps caused by denied or unavailable APIs.

### Journey 3: Validate remediation
1. User selects a finding or path.
2. Oreta shows proposed changes and the graph relationships they are expected to affect.
3. User makes the change outside Oreta.
4. User runs a new scan.
5. Oreta compares results and reports whether the path disappeared, changed, or remains.
6. The system must not claim a fix worked until a subsequent scan supports that conclusion.

## 8. Functional requirements

Priority levels: **P0** = required for MVP; **P1** = important follow-up; **P2** = later enhancement.

### FR-1: Asset collection
- **P0:** Ingest a synthetic fixture format representing supported AWS resources.
- **P0:** Define collector interfaces and normalized schemas.
- **P1:** Read resource/configuration data from selected AWS APIs using read-only permissions.
- **P1:** Record scan time, collector status, source API, and collection errors.
- **P1:** Represent incomplete coverage explicitly; do not infer safety from missing data.

### FR-2: Normalization
- **P0:** Assign each resource a stable identifier, provider, resource type, name/label, region/account context where available, and source evidence.
- **P0:** Preserve relevant source attributes required for policy evaluation and graph construction.
- **P0:** Validate inputs with typed schemas and reject malformed relationships with useful diagnostics.
- **P1:** Version the schema to support future migrations.

### FR-3: Relationship and capability graph
- **P0:** Represent resources/identities as nodes and relationships as typed directed edges.
- **P0:** Support, at minimum, relationships such as `HAS_SECURITY_GROUP`, `ASSUMES_ROLE`, `TRUSTS_PRINCIPAL`, `CAN_READ`, `CAN_WRITE`, `CAN_REACH`, `RUNS_AS`, and `CONTAINS`.
- **P0:** Attach provenance and evidence references to derived edges.
- **P0:** Distinguish observed relationships from inferred capability edges.
- **P0:** Keep the graph builder separate from cloud collectors and policy evaluation.
- **P1:** Allow graph snapshots to be compared across scans.

### FR-4: Policy evaluation
- **P0:** Implement a small, documented set of deterministic controls for supported fixture resources.
- **P0:** Store policy identifiers, severity, applicable resource types, conditions, evidence requirements, and remediation text as versioned policy data.
- **P0:** Report `PASS`, `FAIL`, `NOT_APPLICABLE`, or `UNKNOWN`; missing evidence must not be treated as a pass.
- **P1:** Evaluate policies from established sources such as selected CIS AWS Foundations Benchmark and AWS security guidance, documenting the exact control mapping and version.
- **P1:** Consider OPA/Rego after validating the policy requirements; avoid building an unnecessarily complex custom policy language.

### FR-5: Attack-path analysis
- **P0:** Allow the user or fixture to define entry-point nodes and target/high-value nodes.
- **P0:** Find candidate paths over allowed edge types with bounded depth and cycle protection.
- **P0:** Return the ordered nodes/edges, evidence for each edge, assumptions, and any missing facts that prevent a stronger conclusion.
- **P0:** Distinguish network reachability from permission to access a resource; neither alone should imply the other.
- **P0:** Label results as potential paths, not confirmed exploits.
- **P1:** Support capability composition across role assumption, permission, network reachability, and secret/data relationships.
- **P1:** Add path pruning and ranking for large graphs.
- **P2:** Investigate more advanced graph algorithms or constrained attack-state search if simple path traversal proves insufficient.

### FR-6: Risk prioritization
- **P0:** Provide an explainable configurable ranking using factors such as target sensitivity, external exposure, privilege breadth, path length, control violations, and evidence confidence.
- **P0:** Show factor contributions rather than presenting an unexplained single number.
- **P0:** Keep severity labels separate from confidence and data coverage.
- **P0:** Avoid double-counting the same underlying condition across multiple correlated findings.
- **P1:** Support organization-defined asset criticality and business context.
- **P1:** Compare priority before and after a proposed remediation using the same scoring version.

### FR-7: Findings and remediation
- **P0:** Each finding includes a stable ID, title, affected resource, severity, evidence, policy/path references, explanation, limitations, and suggested remediation.
- **P0:** Clearly distinguish directly observed facts from derived conclusions.
- **P1:** Link each remediation to the graph edges or findings it is expected to affect.
- **P1:** Track remediation status across scans without automatically applying changes.

### FR-8: User interface
- **P0:** Display scan summary, coverage status, findings, and ranked paths.
- **P0:** Allow a user to inspect an individual path and its evidence.
- **P1:** Interactive graph visualization with node/edge details and filters.
- **P1:** Scan-to-scan comparison.
- **P2:** Export reports in JSON and Markdown/PDF.

### FR-9: Security and auditability
- **P0:** No cloud write APIs in the MVP.
- **P0:** Never ask users to paste long-lived cloud secrets into application forms.
- **P0:** Redact credentials, secret values, and sensitive resource data from logs.
- **P0:** Document minimum permissions and collection behavior.
- **P1:** Add authentication, tenant isolation, and access controls before multi-user hosted deployment.
- **P1:** Encrypt stored scan data and support deletion/retention controls.

## 9. MVP scope

The MVP should prove the reasoning model with a small set of synthetic AWS resources before adding broad cloud collection.

### Included
- Offline fixture loader.
- Normalized schemas for EC2, security groups, VPC/subnet/route relationships, IAM roles/policies/trust, S3, and RDS.
- A small set of deterministic policy controls.
- Typed graph construction with provenance.
- Bounded path search from defined entry points to defined sensitive targets.
- Explainable ranking.
- Findings/path API and minimal UI.
- Automated tests and at least three synthetic scenarios: safe baseline, isolated policy violations, and a multi-hop path.

### Excluded from MVP
- Full AWS service coverage.
- Multi-cloud support.
- Active exploitation or attack simulation against real services.
- Automatic remediation.
- LLM-based security decisions.
- Production-grade multi-tenant SaaS billing.

## 10. Technical approach

Proposed components:

1. **Collectors:** obtain source facts from fixtures or read-only AWS APIs.
2. **Normalizer:** transform source data into typed, versioned resource records.
3. **Graph builder:** create observed relationships and evidence-backed capability edges.
4. **Policy evaluator:** evaluate explicit controls against applicable facts.
5. **Path engine:** search the graph under defined traversal constraints and assumptions.
6. **Risk engine:** calculate transparent ranking from path and asset attributes.
7. **Persistence/API:** store scans, resources, edges, policies, findings, and paths.
8. **UI:** present coverage, findings, graph, evidence, and remediation comparisons.

Candidate stack: Python, FastAPI, Pydantic, boto3, NetworkX, pytest, PostgreSQL, Next.js, TypeScript, and React Flow. Technology choices are provisional; the MVP should favor a modular monolith over premature microservices.

## 11. Determinism and correctness requirements

For identical normalized inputs, policy versions, graph rules, and scoring configuration:

- Resource normalization must produce equivalent output.
- Policy results must be repeatable.
- Graph edges must be traceable to evidence or an explicitly documented inference rule.
- Path results must be repeatable, subject to documented tie-breaking and depth limits.
- Risk ranking must be reproducible and expose its factor contributions.
- Unknown or missing information must remain unknown.
- AI-generated text, if added later, must not modify underlying facts, scores, or policy outcomes.

## 12. Evaluation and success metrics

MVP quality will be measured using a labeled synthetic test suite.

- **Asset parsing:** all valid fixture resources are loaded; malformed fixtures fail with actionable errors.
- **Graph correctness:** expected nodes and edges match golden test cases.
- **Policy correctness:** each control has positive, negative, not-applicable, and unknown test cases where relevant.
- **Path correctness:** expected paths are found; prohibited/infeasible transitions are not silently traversed.
- **Evidence coverage:** every reported finding and derived path edge has a source or inference explanation.
- **Reproducibility:** repeated runs over identical inputs produce identical policy outcomes and risk scores.
- **Remediation validation:** synthetic before/after fixtures demonstrate whether targeted paths are removed or remain.
- **Performance baseline:** record runtime and memory on documented fixture sizes; establish numeric targets after the first benchmark rather than inventing them in advance.

## 13. Key risks and mitigations

| Risk | Mitigation |
|---|---|
| False positive attack paths | Mark assumptions, use typed edges and traversal constraints, test against labeled fixtures |
| Missing cloud permissions mistaken for safety | Report coverage gaps and `UNKNOWN` outcomes |
| Overly broad capability inference | Keep observed and inferred edges separate; require evidence and documented semantics |
| Risk score creates false precision | Show factors, confidence, and limitations; make scoring configurable |
| AWS semantics are more complex than generic permissions | Start narrow; test service-specific semantics; document unsupported cases |
| Scope becomes too large | Begin with fixtures and a small resource subset; expand only after correctness |
| Sensitive cloud inventory is exposed | Read-only access, least privilege, redaction, encryption, retention controls |
| AI produces plausible but unsupported explanations | Keep AI optional and grounded in structured evidence; deterministic outputs remain authoritative |

## 14. Milestones

### M0 — Specification and data model
- Finalize resource, edge, evidence, policy, and path schemas.
- Define fixture format and labeled scenarios.

### M1 — Deterministic core
- Fixture loader, normalization, graph builder, policy evaluator, unit tests.

### M2 — Path intelligence
- Entry/target configuration, bounded graph search, evidence-backed path explanations, scoring.

### M3 — API and interface
- FastAPI endpoints, persistence, findings list, graph/path inspection.

### M4 — AWS read-only collector
- Minimum permissions, coverage reporting, account/region scoping, comparison against fixtures.

### M5 — Validation and polish
- Regression suite, sample environments, documentation, threat model, demo, and limitations.

Milestone dates should be set after the initial implementation effort is estimated.

## 15. Open questions

- Which AWS regions and account-scoping model should the first collector support?
- Should graph persistence remain in NetworkX plus PostgreSQL, or is a dedicated graph database justified by measured query needs?
- Which exact version of CIS AWS Foundations Benchmark should be mapped first?
- How should asset criticality be supplied: user labels, tags, or fixture metadata?
- Which relationship types are sufficiently evidenced for the first path engine?
- What graph sizes should define the initial performance benchmark?
- What license should the project use?

## 16. Acceptance criteria for the first demonstrable release

The first demo is acceptable when:

1. A user can load a documented fixture without cloud credentials.
2. Oreta produces a normalized inventory and graph.
3. At least a few deterministic controls produce correct results on positive and negative test cases.
4. A multi-hop potential path from an entry point to a sensitive target is found from graph relationships, not from a hardcoded whole-path signature.
5. The path view explains each edge, source evidence, assumptions, and limitations.
6. The risk ranking exposes its factors and is reproducible.
7. A before/after fixture shows the impact of a remediation.
8. Tests run locally and the README accurately describes what is and is not implemented.

---

## Appendix A — Example conceptual path

Illustrative only; not evidence of a real AWS account:

```text
Internet-reachable application
        |
        | RUNS_AS (observed workload identity)
        v
IAM Role A
        |
        | CAN_ASSUME (derived from trust + caller permission)
        v
IAM Role B
        |
        | CAN_READ (derived from effective permission)
        v
S3 Bucket (tagged as sensitive)
```

Oreta should report this only when each edge is supported by collected facts and the derivation rules for that edge. It must explicitly state any assumptions about workload compromise, credential availability, network reachability, or runtime behavior that have not been verified.

## Appendix B — Finding versus path

- **Finding:** a specific condition, such as a security group allowing SSH from an unrestricted IPv4 source.
- **Path:** an ordered chain of relationships/capabilities from an entry point to a target.
- **Risk ranking:** a prioritization aid that combines evidence-backed conditions and impact context; it is not proof of exploitability.
