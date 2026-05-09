# CEMS — Conference and Event Management Software

Conference and Event Management Software for a federal training facility, built on Salesforce Government Cloud Plus (FedRAMP High) with Service Cloud, Sales Cloud, and Agentforce.

## What this replaces

The existing system managing the facility's hotel accommodations, conference facilities, classroom scheduling, training program enrollment, badging/access control, grading, and certificates.

## Platform

- **Salesforce Government Cloud Plus** (FedRAMP High)
- Service Cloud + Sales Cloud + Experience Cloud
- Salesforce Scheduler (rooms and instructors)
- Agentforce (3 domain agents)
- Shield Platform Encryption + Event Monitoring + Field Audit Trail

## Scope

| Phase | Scope | Status |
|---|---|---|
| 1 | People model, Training Programs, Enrollment (nomination + self-service portal) | Planning |
| 2 | Scheduling — rooms, instructors, group event bookings | Not started |
| 3 | Lodging — hotel room inventory and reservations | Not started |
| 4 | Badging — tiered access, clearance seam, ACS integration | Not started |

## Docs

- [Architecture and Design Spec](docs/superpowers/specs/2026-05-08-cems-design.md)
- [Phase 1 Implementation Plan](docs/superpowers/plans/2026-05-08-phase1-enrollment.md)

## Contributing — especially if you know the existing system

If you have domain expertise on this facility, your input is critical before any code ships. The design was built from high-level requirements and will have gaps and wrong assumptions.

**Start here:** [Issue #1 — Design Review (domain expert input needed)](https://github.com/runloco-dev/cems/issues/1)

Open questions filed as issues:
- [#2 — Nomination workflow and agency quotas](https://github.com/runloco-dev/cems/issues/2)
- [#3 — Training program catalog migration](https://github.com/runloco-dev/cems/issues/3)
- [#4 — Certificate and transcript format requirements](https://github.com/runloco-dev/cems/issues/4)
- [#5 — Experience Cloud portal SSO and IdP](https://github.com/runloco-dev/cems/issues/5)
- [#6 — ATO process and system owner](https://github.com/runloco-dev/cems/issues/6)

Comment on any issue where the design doesn't match reality. Open new issues for anything missing.

## Development setup

Requires Salesforce CLI (`sf`) and access to the GovCloud+ sandbox.

```bash
# Authorize your org
sf org login web --alias cems-dev --set-default

# Deploy all metadata
sf project deploy start --source-dir force-app --target-org cems-dev

# Run tests
sf apex run test --class-names EnrollmentService_Test,WaitlistService_Test,CertificateService_Test \
  --target-org cems-dev --result-format human --wait 10
```

## Data classification

CEMS handles CUI (Controlled Unclassified Information) and below. Salesforce GovCloud+ is not authorized for classified data. Clearance tier is stored as a reference field only — the authoritative adjudication record lives in the agency clearance system (DISS/NBIS or equivalent).
