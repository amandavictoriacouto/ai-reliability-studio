# Endpoint Verifier Security Model

> Status: **Proposed** (architecture review, M0) · Related: [ADR-0011](../adr/0011-sandboxed-verifiers.md), [threat model](threat-model.md) T03, T05, T09, T12
> Scope: the **LLM endpoint probe verifier**, which sends requests from our infrastructure to a customer-registered HTTP endpoint. It is the only MVP component that initiates outbound traffic to targets that customers choose, so it is the main SSRF and abuse surface.

## 1. Security objectives

| # | Objective |
|---|---|
| O1 | Only endpoints that a workspace has **proven it controls** and **explicitly authorized** can be probed. |
| O2 | The verifier can never reach our internal network, cloud metadata services, other tenants' resources or arbitrary third-party hosts (no SSRF). |
| O3 | A compromised or malicious endpoint cannot affect the verifier host, other jobs, the database or the evidence store. |
| O4 | Probing is bounded in time, volume, rate and cost, and can be stopped instantly. |
| O5 | The platform cannot be used as an attack tool against third parties. |
| O6 | Every run is attributable, reproducible and produces evidence. |

## 2. Authorization model

### 2.1 Who can do what

| Action | Allowed roles | Extra requirement |
|---|---|---|
| Register endpoint | `owner`, `admin`, `engineer`, `security` | — |
| Complete ownership verification | Same as above | Challenge passes (2.2) |
| Authorize probing (consent) | `owner`, `admin`, `security` | Recent authentication (≤ 10 min), accepts testing terms, records statement "I am authorized to test this system" |
| Start a run | `owner`, `admin`, `engineer`, `security` | Endpoint verified and authorized, not expired |
| Change probe limits | `owner`, `admin`, `security` | Limits can only be **lowered** below platform maximums |
| Revoke endpoint | `owner`, `admin`, `security` | Immediately cancels queued and running jobs |

### 2.2 Ownership verification

An endpoint is a normalized target: `https://<host>[:443]/<path-prefix>`. Ownership is proven for the exact host, using one of two methods:

1. **DNS TXT:** `_ars-verify.<host>` contains `ars-verify=<random 128-bit token>`.
2. **HTTPS well-known file:** `https://<host>/.well-known/ars-verify.txt` returns the token. The request is made with the same SSRF controls as probes, and redirects are not followed.

Rules:

- Tokens are single-use and expire after 72 h if not used.
- Verification **expires after 30 days** and is re-checked before each run if it is older than 24 h.
- Any change to scheme, host or port requires new verification. Path-prefix changes require re-authorization, not re-verification.
- Wildcard hosts and bare IP addresses are not accepted in MVP.

### 2.3 Endpoint credentials

- Customers may supply one auth header (e.g., `Authorization: Bearer …`) for a **dedicated test credential**. The UI recommends a least-privilege, revocable key.
- The credential is stored with envelope encryption (per-workspace data key). It is decrypted only inside the runner for a single job and is never written to logs, traces or evidence. Transcripts redact the header value.

## 3. SSRF controls (defense in depth)

| Layer | Control |
|---|---|
| **URL validation** | Scheme `https` only. Port 443 only in MVP. No userinfo (`user:pass@`). Hostname IDNA-normalized and lower-cased. IP literals rejected, including decimal, octal and hex forms (e.g., `2130706433`, `0x7f.1`). Path traversal normalized. Maximum URL length 2,048. |
| **DNS resolution** | The runner resolves the name itself. **All** A/AAAA answers must be public. If any answer is in a denied range, the request is rejected. |
| **Denied ranges** | IPv4: `0.0.0.0/8`, `10.0.0.0/8`, `100.64.0.0/10`, `127.0.0.0/8`, `169.254.0.0/16`, `172.16.0.0/12`, `192.0.0.0/24`, `192.0.2.0/24`, `192.88.99.0/24`, `192.168.0.0/16`, `198.18.0.0/15`, `198.51.100.0/24`, `203.0.113.0/24`, `224.0.0.0/4`, `240.0.0.0/4`, `255.255.255.255/32`. IPv6: `::/128`, `::1/128`, `::ffff:0:0/96` (checked against the embedded IPv4 as well), `64:ff9b::/96`, `100::/64`, `2001::/23`, `2001:db8::/32`, `fc00::/7`, `fe80::/10`, `ff00::/8`. Our own service and egress IPs are also denied. |
| **Anti-DNS-rebinding** | The connection is **pinned to the validated IP**. The runner connects to the IP and sends the original `Host` and SNI, so it never resolves the name a second time. |
| **Redirects** | Not followed in MVP. A 3xx response is recorded as a probe result. |
| **TLS** | Certificate validation is mandatory and cannot be disabled. Minimum TLS 1.2. |
| **Network egress** | Runners sit in an isolated network segment with no route to internal services, the database or the metadata endpoint (blocked at the network/firewall level; IMDSv2 with hop limit 1 where applicable). All traffic leaves through an **egress proxy** that enforces the same denied ranges, so application and network controls are independent. |
| **Response handling** | Body capped at 1 MB (stream cut-off). Content types allowed: `application/json`, `text/*`, `text/event-stream`. Responses are treated as **untrusted data**: never executed, rendered as HTML or fed to a tool-using model. |

## 4. Isolation

- **Separate runner pool.** Probe jobs run in a dedicated `verifier-runner` process type, separate from the general worker.
- **No database or storage credentials.** The runner receives a job payload (target, decrypted credential, probe suite) and a **job-scoped token** that only allows it to submit results for that job. The API stores results and evidence.
- **One job per process.** The process runs as non-root with a read-only filesystem, a writable tmpfs of at most 64 MB, dropped Linux capabilities, the default seccomp profile, and no shell utilities in the image.
- **No secrets beyond the job.** The runner has no LLM provider keys, GitHub credentials or signing keys.
- **LLM-as-judge disabled by default.** Detection uses deterministic checks: canary tokens planted in system-prompt probes, PII and secret regex detectors, length and format checks. If a judge model is enabled later, it runs in the API with hardened prompts and receives transcripts as quoted data.

## 5. Execution limits

| Limit | MVP default (platform maximum) |
|---|---|
| Connect / read timeout | 5 s / 30 s |
| Requests per run | 50 (fixed by the probe suite) |
| Wall-clock per run | 10 min (hard kill) |
| Request rate per endpoint | 1 request/s (customers may only lower it) |
| Concurrent runs per endpoint | 1 |
| Concurrent runs per workspace | 2 |
| Runs per workspace per day | 20 |
| Response size | 1 MB |
| Circuit breaker | Abort after 5 consecutive 5xx/timeouts; honor `Retry-After`; abort on 429 after one backoff |
| Kill switch | Per workspace, per endpoint and global (admin), effective on the next request |

## 6. Abuse prevention

- **Fixed probe content.** Probe suites are versioned files in `catalog/probes/`, reviewed by the code owner. Customers cannot supply arbitrary payloads in MVP, so the platform is not a generic HTTP client.
- **Identifiable traffic.** Every request sends the header `User-Agent: AIReliabilityStudio-Verifier/<version> (+<docs-url>)` and an `X-ARS-Run-Id` header.
- **Accountability.** Every registration, verification, authorization, run start/stop and limit change is an audit event.
- **Anomaly controls.** Alerts fire when the same host is registered by several workspaces or when run failure rates spike. A global host deny-list is maintained.

## 7. Evidence produced

Each run produces one evidence object (JSONL transcript plus summary). It records the probe suite version, runner version, resolved IP, TLS certificate fingerprint, request/response pairs with credentials redacted, detector results, timestamps and the final status. The evidence is hashed and stored per the [evidence model](../architecture/evidence-model.md).

## 8. Verification of these controls

| Control | How it is validated |
|---|---|
| URL/IP validation, denied ranges, IP encodings, IPv4-mapped IPv6 | Automated unit tests (table-driven, ≥ 60 cases) |
| DNS rebinding, redirect refusal, oversize response, timeouts | Automated integration tests with a controllable fake DNS server and HTTP target |
| Network-level egress block | Automated smoke test from a runner in each environment (tries the metadata IP and an internal service and expects failure) + human review of network config per environment |
| Ownership verification flows | Automated integration tests; manual test once per release |
| Runner privileges (non-root, read-only FS, no DB creds) | Automated container policy check in CI + human review of deployment manifest |

## 9. Residual risks

- A verified host may later change hands, or point to a third party, within the 30-day window. This is mitigated by the 24 h pre-run re-check.
- Probes that succeed against production endpoints may cost the customer money or trigger alerts. Mitigation: we recommend staging endpoints and enforce rate limits.
- Prompt-injection detection is probabilistic. A "passed" probe run proves only that **these probes** did not trigger **these detectors** at that time; it does not prove the endpoint is immune.
