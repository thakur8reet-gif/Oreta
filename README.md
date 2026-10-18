# Oreta

**Cloud Attack Path Intelligence Engine**

Oreta is a cloud security analysis project that models cloud infrastructure as a capability graph and identifies potentially dangerous paths from exposed entry points to sensitive resources. Rather than stopping at isolated configuration findings, Oreta aims to correlate network reachability, identity permissions, trust relationships, and resource sensitivity to explain how weaknesses may combine into a larger risk.

> **Project status:** Early-stage / specification phase. The repository is being initialized; implementation, coverage, and detection quality are not yet established.

## Why Oreta?

A cloud account can pass many individual configuration checks and still contain risky combinations of permissions and relationships. A public workload may have a narrowly scoped role; that role may assume another role; the second role may read a secret; and that secret may provide access to a sensitive database. Looking at each setting in isolation can hide the significance of the chain.

Oreta is designed to help answer:

- What assets and identities exist in the cloud environment?
- Which resources are externally reachable, and through what network paths?
- What can each workload identity read, write, assume, or modify?
- Which combinations of relationships create plausible paths to sensitive assets?
- Which findings should be fixed first, and why?
- Which remediation would break the greatest number of high-impact paths?

## Design principles

1. **Evidence before inference.** Collect resource state from cloud APIs and preserve source evidence and scan timestamps.
2. **Deterministic core.** Asset collection, normalization, policy evaluation, graph traversal, and scoring must be reproducible for the same inputs and configuration.
3. **Graph-based reasoning.** Represent resources as nodes and relationships/capabilities as typed edges; derive paths from graph structure rather than hardcoding every possible multi-step scenario.
4. **Policies are evidence, not the whole product.** Established controls help identify known weaknesses, but path analysis considers how exposure, identity, permissions, connectivity, and asset impact combine.
5. **Explainable findings.** Each finding should show the evidence, assumptions, path, score factors, and remediation rationale.
6. **Safe by default.** Start with read-only collection and offline fixtures. No exploitation, credential harvesting, or automatic cloud changes.
7. **Honest uncertainty.** Distinguish confirmed configuration facts from inferred reachability and hypothetical attacker capabilities.

## Proposed workflow

```text
Cloud APIs / Local Fixtures
          |
          v
    Asset Collectors
          |
          v
  Normalization Layer
          |
          +-------------------+
          |                   |
          v                   v
   Policy Evaluation     Relationship Builder
          |                   |
          +---------+---------+
                    v
              Cloud Graph
                    |
                    v
         Capability / Path Analysis
                    |
                    v
          Risk Correlation & Ranking
                    |
                    v
       Findings, Evidence, Remediation
                    |
                    v
              Web Dashboard
```

## Planned initial scope

The first milestone should support an **offline AWS fixture dataset** and a narrow, well-tested subset of AWS resource types before connecting to a real account.

Candidate resource types:

- EC2 instances and network interfaces
- VPCs, subnets, route tables, and security groups
- IAM roles, policies, and role trust relationships
- S3 buckets and bucket/access policies
- RDS instances and network accessibility

The first release should prioritize correct relationship modeling and transparent path derivation over broad service coverage.

## Core capabilities (planned)

- Asset inventory with stable resource identifiers
- Normalized resource and relationship schema
- Policy-as-data evaluation for selected security controls
- Capability edges such as `CAN_REACH`, `CAN_ASSUME`, `CAN_READ`, and `CAN_WRITE`
- Graph traversal from defined entry points to high-value targets
- Explainable, configurable risk ranking
- Findings with evidence, confidence/limitations, and remediation guidance
- Scan history and before/after comparison
- Interactive graph and findings dashboard
- Unit tests and reproducible sample scenarios

## What Oreta is not

- Not a guarantee that a cloud environment is secure
- Not a replacement for cloud-native security products, penetration testing, or expert review
- Not an exploit framework
- Not an AI system that invents infrastructure or independently decides what is exploitable
- Not a complete cloud compliance suite in its initial version

A path is a **potential attack path under stated assumptions**, not proof that exploitation is possible. Real exploitability may depend on application vulnerabilities, runtime behavior, credential availability, service-specific semantics, and controls not visible to the collector.

## Proposed technology stack

The stack is provisional and may be adjusted after the initial architecture spike.

| Layer | Candidate |
|---|---|
| Language | Python |
| API | FastAPI |
| AWS collection | boto3, read-only IAM permissions |
| Validation / schemas | Pydantic |
| Graph analysis | NetworkX initially; evaluate a graph database only if justified |
| Policy evaluation | OPA/Rego or a small declarative policy format |
| Persistence | PostgreSQL for scans and findings |
| Frontend | Next.js + TypeScript |
| Graph visualization | React Flow |
| Local development | Docker Compose |
| Testing | pytest + synthetic cloud fixtures |

Avoid introducing Neo4j, a queue, or a complex distributed architecture until scale and query requirements demonstrate the need.

## Getting started

Implementation is not available yet. The initial development sequence is:

1. Define the normalized resource and edge schema.
2. Create synthetic AWS fixtures and expected graph outputs.
3. Implement collectors behind interfaces, beginning with offline fixtures.
4. Add a small set of explicit policies and test each policy.
5. Implement graph traversal with path explanations and constraints.
6. Add risk ranking, findings persistence, and a minimal dashboard.
7. Add an opt-in read-only AWS collector and verify it against fixtures.

## Safety and permissions

When AWS collection is introduced:

- Use a dedicated test account wherever possible.
- Use a dedicated IAM role with only the read permissions required by supported collectors.
- Never request access keys in the UI or commit credentials.
- Use temporary credentials or the standard AWS credential chain.
- Do not perform active exploitation or mutate cloud resources.
- Redact secrets and sensitive values from logs and stored evidence.
- Treat missing permissions and incomplete API responses as **unknown**, not as proof that a resource is safe.

## Contributing

Oreta is at the specification stage. Contributions should include tests and documented assumptions. A detection should identify its evidence source, logic, limitations, and remediation. A graph relationship should explain which cloud facts justify the edge.

## License

No license has been selected yet. Until a license is added, standard copyright applies and reuse is not automatically granted.
