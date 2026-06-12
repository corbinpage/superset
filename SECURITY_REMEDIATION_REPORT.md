<!--
 Licensed to the Apache Software Foundation (ASF) under one
 or more contributor license agreements.  See the NOTICE file
 distributed with this work for additional information
 regarding copyright ownership.  The ASF licenses this file
 to you under the Apache License, Version 2.0 (the
 "License"); you may not use this file except in compliance
 with the License.  You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing,
 software distributed under the License is distributed on an
 "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 KIND, either express or implied.  See the License for the
 specific language governing permissions and limitations
 under the License.
-->

# Security Remediation Report

Snapshot of the security issues filed against this repository by the **Devin Security Scanner** and the remediation pull requests opened by the **Devin Remediation Pipeline**.

- Dashboard: https://devin-cve-issue-action.fly.dev/
- Pipeline service: https://github.com/corbinpage/devin-cve-issue-action
- Trigger: a `devin-fix` label on a `[Security]` issue → GitHub Action → remediation webhook → triage → Devin session → PR.

## Active findings

| Issue | Advisory | Package | Severity | Disposition | Remediation PR |
| --- | --- | --- | --- | --- | --- |
| [#74](https://github.com/corbinpage/superset/issues/74) | CVE-2026-48523 (+4 related) | `pyjwt` 2.12.0 | High | Fix ready | [#75](https://github.com/corbinpage/superset/pull/75) |
| [#77](https://github.com/corbinpage/superset/issues/77) | GHSA-55h3-fm53-wq99 | `eslint-plugin-i18n-strings` | Critical (malware) | Fix ready | [#79](https://github.com/corbinpage/superset/pull/79) |
| [#78](https://github.com/corbinpage/superset/issues/78) | CVE-2026-44405 | `paramiko` 3.5.1 | High | Escalated to human (no fix available) | — |

> A duplicate of the eslint rename fix was also opened as [#76](https://github.com/corbinpage/superset/pull/76); it is superseded by #79 and should be closed.

---

## #74 — CVE-2026-48523: pyjwt high (Algorithm Allow-List Bypass)

**Package:** `pyjwt@2.12.0` → fixed in `2.13.0` · **Severity:** High · **Ecosystem:** python
**Related CVEs:** CVE-2026-48522, CVE-2026-48524, CVE-2026-48525, CVE-2026-48526

PyJWT 2.12.0 has 5 known vulnerabilities, the most severe being **CVE-2026-48523** — a verifier-side algorithm allow-list bypass that could allow an attacker to use unintended algorithms when verifying JWTs. This is particularly critical for Superset since PyJWT is used for authentication (`flask-jwt-extended`). Additional CVEs fixed in 2.13.0: SSRF via `PyJWKClient` (CVE-2026-48522), redundant HTTP fetches (CVE-2026-48524), detached-JWS verification (CVE-2026-48525), and a JWT decoding issue (CVE-2026-48526).

**Remediation — PR [#75](https://github.com/corbinpage/superset/pull/75)** (triaged: auto-fix, low complexity / non-breaking):
- `requirements/base.in`: added `pyjwt>=2.13.0,<3.0.0` security pin
- `requirements/base.txt`: `pyjwt==2.12.0` → `pyjwt==2.13.0`
- `requirements/development.txt`: `pyjwt==2.12.0` → `pyjwt==2.13.0`
- Patch-level bump, no API changes; CI green (37 checks).

---

## #77 — GHSA-55h3-fm53-wq99: eslint-plugin-i18n-strings critical (Malware)

**Package:** `eslint-plugin-i18n-strings` (all versions) · **Severity:** Critical · **CWE:** CWE-506 (Embedded Malicious Code)

The npm package name `eslint-plugin-i18n-strings` is flagged as malware (no fix available). Superset's `superset-frontend` uses a **local** ESLint plugin that shares this name (referenced via the `file:` protocol, never fetched from npm), but the name collision triggers scanners and invites dependency-confusion risk.

**Remediation — PR [#79](https://github.com/corbinpage/superset/pull/79)** (triaged: auto-fix — local rename, functionality preserved):
- Renamed `eslint-rules/eslint-plugin-i18n-strings/` → `eslint-rules/eslint-plugin-superset-i18n/`
- `superset-frontend/package.json` dependency → `"eslint-plugin-superset-i18n": "file:eslint-rules/eslint-plugin-superset-i18n"`
- `.eslintrc.js` / `.eslintrc.minimal.js`: plugin `'superset-i18n'`, rules `'superset-i18n/no-template-vars'`, `'superset-i18n/sentence-case-buttons'`
- Lint rules `no-template-vars` and `sentence-case-buttons` unchanged; CI green (43 checks).

---

## #78 — CVE-2026-44405: paramiko high (SHA-1 Algorithm Allowed)

**Package:** `paramiko@3.5.1` · **Severity:** High · **Ecosystem:** python · **Fixed in:** no patched release (affects through 4.0.0)

In Paramiko through 4.0.0, `rsakey.py` allows the cryptographically broken SHA-1 algorithm for SSH key verification.

**Disposition — Escalated to human.** No patched version is available, and this repo already pins `paramiko<4.0` to keep SSH tunneling working (master commit `74845eaf0b`), so an automated bump is not possible. The pipeline correctly routed this to a human to monitor upstream rather than opening a speculative PR.

---

## Pipeline history

The scanner and remediation pipeline ran several times against this fork, filing `[Security]` issues (label `devin-scanner`, plus `devin-fix` for the ones in remediation scope) and opening remediation PRs. Earlier remediation PRs for these same advisories (e.g. #31/#32, #38/#39, #49–#61, #69–#72) were superseded as the pipeline re-ran and were closed in favor of the current set (#75, #76, #79). Triage routing observed:

- **Auto-fix** — simple, non-breaking dependency bumps / local renames (pyjwt, joi, the eslint rename).
- **Escalated to human** — critical-severity items and findings with no available fix (paramiko SHA-1; the malicious-pkg / critical test issues).

See the live dashboard for throughput, auto-resolution rate, engineer-hours saved, and per-week breakdowns.
