# SOC Lab & Telemetry Foundation

**Status:** Documentation baseline in progress
**Last documented baseline:** 2026-10-24

## 1. Project Purpose

This repository documents a security operations lab used to generate, collect,
validate, and preserve security telemetry for investigation and detection-engineering
experiments.

It records the lab architecture, telemetry sources, validation scenarios,
known limitations, and supporting proof-of-concept (PoC) evidence.

The lab is the telemetry foundation for the separate
**Cross-Telemetry Evidence Graph & Blast Radius Agent** project.

## 2. Lab Overview

The lab includes the following major components:

| Component | Role |
|---|---|
| Windows client | Generates endpoint telemetry, including Sysmon events |
| Windows Server / Domain Controller | Provides Active Directory and DNS services and Windows Security events |
| Network sensor | Observes traffic between lab subnets and exports Suricata telemetry |
| Wazuh platform | Centralizes collected events and provides searchable archives and alerts |
| WireGuard tunnel | Carries the configured sensor-to-Wazuh communication path |

The local lab uses separate USERS and SERVERS network segments. Detailed
operational addresses and resource identifiers are intentionally omitted
from this overview.

## 3. Architecture Diagrams

### SOC Lab Network Topology

![SOC Lab Network Topology](docs/diagrams/SOC%20Lab%20Network%20Topology.jpg)

This diagram illustrates the lab network architecture and the connectivity
between its main components and network segments.

### Telemetry Collection and Data Flow

![Telemetry Collection and Data Flow](docs/diagrams/TELEMETRY%20COLLECTION%20AND%20DATA%20FLOW.jpg)

This diagram illustrates the documented telemetry collection and forwarding
paths from the lab data sources to the central Wazuh platform.

## 4. Telemetry Sources

The documented telemetry sources include:

- Windows Security events
- Sysmon process, DNS, and network events
- Suricata EVE JSON events
- Wazuh archives and alert records

Wazuh archives are the primary event source for investigation snapshots.
Wazuh alerts represent events that matched configured detection rules and
must not be treated as a complete substitute for archived telemetry.

## 5. Validation Status at the Documented Baseline

The following statuses reflect the project handoff documented on 2026-10-24.
They are historical status records, not a claim that every component was
revalidated on the date this README was created.

| Area | Status | Evidence and limitations |
|---|---|---|
| Project definition | PASS | Project scope and design principles documented |
| Network foundation | BUILT — NEEDS RECHECK | Lab networks were configured; current state requires revalidation when relevant |
| Network sensor | BUILT — PARTIALLY VALIDATED | Suricata and Wazuh telemetry paths were tested in selected scenarios |
| Wazuh and WireGuard | PASS FOR TESTED PATH | Tested connectivity and telemetry paths were documented |
| Windows agents and Sysmon | BUILT — NEEDS RECHECK | Agents and telemetry collection were configured; current state may require revalidation |
| DNS correlation | PARTIAL | Cross-source DNS events were observed; the documented test did not produce an A-record answer |
| HTTPS correlation | PARTIAL | DNS, endpoint network activity, and a Suricata TLS flow were observed; a complete process-to-file-hash chain was not established |
| Authentication and share access | PASS FOR TESTED SUBSCENARIO | Events 4624 and 5145 shared relevant logon context; this does not establish malicious lateral movement |

A successful test validates only the behavior and scope actually tested.
It must not be generalized to all telemetry sources or scenarios.

## 6. Proof-of-Concept Evidence

Screenshots and other PoC artifacts will be organized by lab phase and scenario.

Each evidence item should document:

- Evidence identifier
- Capture date, if known
- Related phase and test scenario
- Source application or data source
- Claim supported by the evidence
- Validation status and known limitations
- Redaction status, and a file hash when useful

Screenshots provide supporting visual evidence; they do not replace underlying
logs, query results, or repeatable validation steps.

Only sanitized screenshots, synthetic fixtures, and approved documentation
should be committed to a public repository.

## 7. Evidence Handling and Security

- Do not commit credentials, API keys, WireGuard private keys, or secret files.
- Do not commit raw telemetry, packet captures, memory dumps, or sensitive exports.
- Keep raw evidence separate from sanitized fixtures used for development.
- Preserve source references and document any transformations applied to evidence.
- Treat `.gitignore` as a safeguard, not as a substitute for reviewing files before committing.
- Do not publish detailed operational network information without a deliberate review.

## 8. Relationship to the Agent Repository

The companion repository,
`cross-telemetry-evidence-graph-agent`, implements the investigation and evaluation
software that consumes selected, documented telemetry.

This repository focuses on the lab, telemetry generation and collection,
validation procedures, and provenance of the supporting evidence. The agent
repository focuses on data contracts, normalization, evidence correlation,
blast-radius analysis, and evaluation against an independent ground truth.

Only sanitized, documented datasets should cross into the agent repository.

## 9. Scope and Deferred Work

The current lab documentation does not imply that every planned telemetry
source or scenario is implemented.

Additional sources and scenarios, including a dedicated web server and Zeek,
remain deferred according to the project handoff baseline.

## 10. Status Vocabulary

- `PASS`: Passed for the explicitly tested scope.
- `BUILT — NOT FULLY VALIDATED`: Implemented, but not fully verified.
- `PARTIAL`: Some intended behavior was demonstrated; limitations remain.
- `NEEDS RECHECK`: Historical state exists, but current state has not been confirmed.
- `DEFERRED`: Deliberately excluded from the current scope.
- `PLANNED / NOT STARTED`: Work has not yet been implemented.

## 11. Documentation Principles

This repository distinguishes observed facts from assumptions and planned work.
No phase should be marked complete without evidence appropriate to its acceptance
criteria. Known gaps and limitations must remain visible in the documentation.
