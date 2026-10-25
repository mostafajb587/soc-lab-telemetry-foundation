# Telemetry Matrix

## 1. Purpose

This document maps telemetry sources to the security observations they can
support, the scenarios tested so far, and the limitations of the available
evidence.

The matrix distinguishes event availability from the strength of an
investigative conclusion. An event may establish that an action was observed
without proving malicious intent, compromise, or a causal relationship.

## 2. Status Vocabulary

| Status | Meaning |
|---|---|
| PASS | The specific stated test was completed and its result was verified. |
| PARTIAL | Some expected evidence was observed, but the complete scenario was not validated. |
| NEEDS RECHECK | Historical evidence exists, but the current state or reproducibility must be verified again. |
| DEFERRED | Deliberately postponed and not part of the currently validated capability. |
| PLANNED / NOT STARTED | Not yet implemented or validated. |

A status applies only to the specific test or capability described. It must
not be interpreted as proof that every related event type or detection path
works.

## 3. Telemetry Sources

| Source | Collection component | Relevant data | Documented status | Limitations |
|---|---|---|---|---|
| Windows client process telemetry | Sysmon on WIN-CLIENT, forwarded by Wazuh agent 001 | Process creation, process identifiers, command-line context when configured | NEEDS RECHECK | Historical collection was observed. Revalidate current service, configuration, and event delivery. |
| Windows client network telemetry | Sysmon Event ID 3, forwarded by Wazuh agent 001 | Process-associated network connections, local and remote endpoints | NEEDS RECHECK | A network event alone does not prove that a connection was malicious or successful at the application layer. |
| Windows client DNS telemetry | Sysmon Event ID 22, forwarded by Wazuh agent 001 | DNS query activity and process context | PARTIAL | A documented test observed a DNS query, but the test did not produce a complete process-to-DNS-to-network evidence chain. |
| Windows Security authentication events | Windows Security log on DC01, forwarded by Wazuh agent 002 | Logon events such as 4624 and failed logons such as 4625 | PASS for the documented subtests; NEEDS RECHECK for current state | An authentication event establishes an authentication-related observation, not compromise by itself. |
| Windows Security share-access auditing | Windows Security log on DC01, forwarded by Wazuh agent 002 | Event 5145 share-access checks and associated logon context | PASS for the documented test; NEEDS RECHECK for current state | An access check does not by itself prove that data was modified, exfiltrated, or accessed maliciously. |
| Network DNS, flow, and alert telemetry | Suricata EVE JSON on NET-SENSOR, forwarded by Wazuh agent 003 | DNS transactions, flow metadata, and rule-matched alerts | PARTIAL | Visibility depends on traffic traversing the sensor and on the configured logging outputs. |
| Centralized event archives | Wazuh archives | Events received by the central platform, subject to collection and retention configuration | NEEDS RECHECK | Archives are the primary historical event source for investigations; their contents must be queried and verified for each test. |
| Rule-matched alerts | Wazuh alerts | Events that matched configured alerting rules | NEEDS RECHECK | Alerts are a subset of collected telemetry. Absence of an alert does not prove absence of an event or attack. |
| Zeek network metadata | Not deployed in the current MVP | Potential future protocol metadata | DEFERRED | Do not treat Zeek-derived fields as available evidence. |
| Web server telemetry | WEB01 not deployed in the current MVP | Potential future web access and server-side activity | DEFERRED | The web-to-host-to-network scenario is not validated. |

## 4. Documented Validation Scenarios

### S1 — Process, DNS, and Network Activity

**Status: PARTIAL**

Historical test evidence documented:

- A Sysmon Event ID 22 DNS query for a test subdomain under `example.com`.
- A Sysmon Event ID 3 UDP connection from WIN-CLIENT to DC01 on port 53.
- A corresponding Suricata DNS request/response observation.
- A separate HTTPS test to `https://www.iana.org/` that returned HTTP 200.
- Sysmon DNS and network observations associated with the HTTPS test.
- Corresponding Suricata TLS/flow observations.

Limitations:

- The test-subdomain query did not return the expected A record.
- The DNS test and HTTPS test are distinct observations and must not be combined into one causal chain without matching evidence.
- The available results do not establish a complete process-to-file-hash-to-DNS-to-network chain.
- Current collection and repeatability require revalidation.

### S3 — Authentication and Network Share Access

**Status: PASS for the documented subscenario**

Historical test evidence documented:

- A network logon event, Security Event ID 4624, on DC01.
- A share-access check, Security Event ID 5145, associated with the same logon context.
- Matching source address, account context, and Logon ID in the documented records.
- Access-check context associated with the SYSVOL share.

Interpretation:

- The records support a network logon and a related SYSVOL access check in the same security context.
- They do not independently establish malicious lateral movement, compromise, or data exfiltration.
- Revalidate the current collection path before claiming that the test is reproducible.

#### S3 Test — Failed Network Authentication

**Status: PASS for the documented subtest; NEEDS RECHECK for current state**

A historical failed-authentication test from WIN-CLIENT toward DC01 was documented as producing Security Event ID 4625 evidence in Wazuh.

This result applies to the tested failure path only. It does not establish complete coverage for all authentication types or successful network-logon scenarios.

### S4 — Benign Simulated Activity

**Status: PARTIAL**

A complete end-to-end benign simulation covering the required process, file, registry, DNS, and network evidence has not been validated as one reproducible scenario.

Do not describe S4 as complete until its expected evidence, collection path, and test results are documented together.

## 5. Evidence Interpretation Rules

- Process names alone are insufficient to establish a relationship.
- Prefer stable identifiers such as ProcessGuid, SHA-256, user identity, source and destination addresses, logon/session identifiers, event IDs, timestamps, and original event references when available.
- Timestamp proximity or matching IP addresses alone must not establish causation.
- Security Event ID 4624 alone means authentication was observed; it does not establish compromise.
- A missing event may indicate missing coverage, collection failure, filtering, retention limits, or genuinely absent activity.
- Distinguish `NO SUPPORTING EVIDENCE` from a verified negative result.
- Preserve the original event reference and document any normalization or redaction.
- Do not infer that an event was collected merely because a service or configuration exists.

## 6. Collection and Coverage Caveats

- The sensor primarily observes traffic routed between the USERS and SERVERS network segments.
- Do not assume visibility into traffic that does not traverse the sensor.
- Wazuh archives and Wazuh alerts serve different purposes; alerts represent rule-matched events rather than the complete event population.
- Historical results are not a substitute for a fresh validation when current operational status matters.
- Current IP assignments, agent health, service state, configuration, and event delivery must be rechecked before a release or demonstration.

## 7. Out of Scope for the Current MVP

- Zeek telemetry.
- WEB01 server telemetry and the web-to-host-to-network scenario.
- Cloud telemetry collection.
- Velociraptor endpoint collection.
- Threat-intelligence API enrichment.
- Malware sandbox integration.
- Automated response and SOAR.
- Claims of complete telemetry coverage without measured validation.

## 8. Relationship to the Evidence Graph Agent

The companion `cross-telemetry-evidence-graph-agent` repository will consume
selected, provenance-preserving records and evaluate relationships against
explicit evidence requirements.

The agent must distinguish:

- `CONFIRMED`
- `PROBABLE`
- `UNCONFIRMED`
- `CONTRADICTED`

Blast-radius outcomes must distinguish:

- `CONFIRMED AFFECTED`
- `POTENTIALLY RELATED`
- `NO SUPPORTING EVIDENCE`

The agent must also account for telemetry coverage, missing evidence,
counter-evidence, and the difference between an unobserved event and a
verified negative finding.

## 9. Data Handling

- Never commit credentials, private keys, tokens, or other secrets.
- Do not commit raw event logs, packet captures, or unredacted snapshots.
- Use synthetic or sanitized fixtures for public examples.
- Keep detailed operational identifiers out of public documentation unless
  their publication has been explicitly reviewed and approved.
- Review `.gitignore` and every proposed file before committing.

## 10. Status Interpretation

- `PASS`: The specific documented test passed based on recorded evidence. This status does not imply that the current environment remains operational or that all related telemetry paths are covered.
- `PARTIAL`: Some expected evidence was observed, but the complete scenario or required evidence chain was not validated.
- `NEEDS RECHECK`: Current operational state, event delivery, configuration, or repeatability has not been freshly verified.
- `DEFERRED`: The capability is intentionally excluded from the current MVP.
- `PLANNED / NOT STARTED`: The capability is intended for future work but has not been implemented or validated.