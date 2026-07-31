# AEGIS User Guide

**Version:** 3.7.1
**Engine:** Rust native (14 languages, 115 detection patterns covering 126 CWE classes, 40 report formats)
**License tiers:** Community (free) / Pro / Premium / Enterprise

---

## Install

```bash
# Download the latest binary for your platform
# https://github.com/raknor-ai/aegis-releases/releases/latest
tar xzf aegis-cli-darwin-arm64.tar.gz   # macOS Apple Silicon
tar xzf aegis-cli-darwin-x64.tar.gz     # macOS Intel
tar xzf aegis-cli-linux-x64.tar.gz      # Linux x64
tar xzf aegis-cli-linux-arm64.tar.gz    # Linux ARM64 (Graviton)
chmod +x aegis
./aegis scan ./your-project
```

Available platforms: `darwin-arm64`, `darwin-x64`, `linux-x64`, `linux-arm64`, `win32-x64`

The binary is a single file (~33 MB), fully static on Linux (musl), with zero system dependencies.

## License activation

```bash
# CLI activation (writes to ~/.raknor/license.json)
aegis activate --key AEGIS-...

# Or environment variable (per-session / CI)
export AEGIS_LICENSE_KEY="AEGIS-..."

# Verify
aegis license
```

Without a key, AEGIS runs in **Community mode**: full Rust engine, 50-finding cap, SARIF + HTML + JSON output, compliance traffic lights.

---

## What each tier unlocks

Every tier includes everything below it. The same binary ships everywhere — the license key determines what's active.

### Community (free — no key required)

| What you get | Flag |
|---|---|
| Full Rust AST scan (14 languages, 115 detection patterns) | — |
| 50 findings shown (total count always reported) | — |
| Call graph, dead code, taint analysis | — |
| Compliance risk mapping (13 frameworks) — identifies which controls are at risk, not certification | — |
| SARIF 2.1.0 report | `--sarif` |
| Interactive HTML findings report | `--html` |
| JSON summary (machine-readable) | `--json-summary` |
| Delta scanning (changed files only) | `--changed-only`, `--since` |
| CI merge gating | `--fail-on` |
| Directory exclusion | `--exclude` |
| Engineer triage shorthand | `--engineer` |

Reports carry a "PREVIEW" watermark. Finding count is capped at 50 but the total is always shown so you know how much is behind the gate.

### Pro

Everything in Community, plus **unlimited findings** and:

| Bundle | What you get | Flags |
|---|---|---|
| **Remediate** | Unlimited findings, auto-fix patches (14 CWEs), trend analysis | `--auto-fix`, `--trend` |
| **Refactor** | Tech debt, bounded context, env divergence, IaC divergence, dep audit, dep accuracy, PII-in-logs, API surface | `--tech-debt`, `--bounded-context`, `--env-divergence`, `--iac-divergence`, `--dep-audit`, `--dep-accuracy`, `--log-risk`, `--api-surface` |
| **Shield** | STRIDE threat model, WAF rules (3 formats), DAST / ConfirmedFinding, IAM analysis, container audit | `--stride`, `--waf-rules`, `--dast`, `--iam`, `--container-audit` |

### Premium

Everything in Pro, plus:

| What you get | Flag |
|---|---|
| M&A Technical Due Diligence report | `--due-diligence` |
| FedRAMP Continuous Monitoring packages | `--conmon` |
| White-label branding | `--brand` |

### Enterprise

Everything in Premium, plus:

| Bundle | What you get | Flags |
|---|---|---|
| **Certify** | OSCAL 1.1.2 (SSP/AR/POA&M), DORA Pillar I-V, ISO 27001:2022, NIST CSF 2.0, OpenVEX, SBOM (CycloneDX + SPDX), 13-framework compliance risk map, evidence bundles. These map findings to framework controls — they identify which controls are at risk, not certify that they are satisfied. | `--oscal`, `--dora`, `--iso-27001`, `--nist-csf`, `--vex`, `--sbom`, `--compliance`, `--evidence-bundle` |

### Engineer mode

Interactive operator-assist tools that use the operator's own authenticated access. These are not batch-scan features — they run against your own infrastructure on demand.

| What you get | Flag |
|---|---|
| New Relic APM correlation — maps findings to hot execution paths | `--newrelic` |
| CloudWatch Logs Insights correlation | `--cloudwatch` |
| ELB/ALB log analysis | `aegis log elb-*` |
| RDS STIG/CIS database compliance check | `--rds-compliance` |

---

## Quick reference

```bash
# Scan everything, generate all reports
aegis scan ./target --all

# Engineer triage mode (findings + auto-fix + STRIDE + tech debt — no compliance)
aegis scan ./target --engineer

# Specific reports
aegis scan ./target --sarif --html --scoring

# CI gate — fail on critical findings
aegis scan ./target --sarif --fail-on critical

# Delta scan — only files changed since main
aegis scan ./target --since origin/main --sarif --fail-on critical

# Pre-commit hook — only staged + unstaged changes
aegis scan ./target --changed-only --fail-on high

# White-label
aegis scan ./target --all --brand .aegis-brand.json

# Trend analysis (compare against a baseline)
aegis scan ./target --all --trend previous-scan/aegis-summary.json

# GRC summary for questionnaires
aegis scan ./target --grc-summary --project-name "My Product"

# Explain a code flow (scoped to a function/route)
aegis explain-flow ./target --entry "POST /api/purchase"

# Query logs (parallel-safe, no MCP required)
aegis log search --logs ./logs --query "error"
aegis log errors --logs ./logs --top 15
aegis log trace --logs ./logs --request-id abc-123
aegis log elb-summary --elb-logs ./alb-logs

# Start MCP server (for Claude Code / IDE integration)
aegis mcp-serve --scan-dir ./aegis-reports
```

---

## Flags — complete reference

### Output control

| Flag | Default | Description | Tier |
|---|---|---|---|
| `--output <DIR>` | `aegis-reports` | Directory for report files | All |
| `--all` | — | Generate every report format | All |
| `--engineer` | — | Engineer-focused reports only (no compliance/M&A/evidence) | All |
| `--json-summary` | — | Machine-readable metrics (findings, severity, score, architecture) | All |
| `--project-name <NAME>` | Directory name | Project display name for GRC and compliance reports | All |

### Security reports

| Flag | Output file | What it does | Tier |
|---|---|---|---|
| `--sarif` | `aegis-report.sarif.json` | SARIF 2.1.0 — upload to GitHub Security, Defender, or any SARIF consumer. All findings with CWE, file:line, severity, trust level, verification class (M/J), calibrated confidence. | Community |
| `--html` | `aegis-report.html` | Interactive HTML findings report with severity toggles, file search, sortable columns, collapse/expand, CWE distribution, and trust classification. Criticals sort first. | Community |
| `--stride` | `aegis-stride.html` | STRIDE threat model — Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege. | Pro |
| `--dep-audit` | `aegis-dep-audit.html` | Dependency vulnerability audit. Checks `requirements.txt`, `package.json`, `go.mod` against known CVEs. | Pro |
| `--log-risk` | `aegis-log-risk.html` | PII-in-logs detection. Five categories: PiiVariable, PiiPattern, DebugPii, PrintInProd, ExceptionLeak. | Pro |
| `--iam` | `aegis-iam-analysis.html/json` | IAM policy analysis (FedRAMP KSI #1). Scans Terraform/CloudFormation for wildcard actions, open ingress, missing auth, public S3, hardcoded account IDs, missing TLS, CORS misconfig. | Pro |
| `--dast` | — | ConfirmedFinding: derives exploit probes from static taint paths, executes against a running target, observes the vulnerability. Demonstrated for CWE-89 (SQL injection) and CWE-94 (code injection). | Pro |
| `--container-audit` | `aegis-container-audit.html` | Discovers Dockerfiles, classifies containers (Service/Base/Infra/Emulator/DevTest/Ops), maps coupling. | Pro |

### Remediation reports

| Flag | Output file | What it does | Tier |
|---|---|---|---|
| `--auto-fix` | `aegis-autofix/` | Git-apply compatible auto-fix patches for 14 CWE categories. | Pro |
| `--waf-rules` | `aegis-waf-*.conf/json` | WAF rules from findings — ModSecurity, AWS WAF, and Cloudflare formats. | Pro |

### Compliance and evidence

| Flag | Output file | What it does | Tier |
|---|---|---|---|
| `--oscal` | `aegis-oscal.json` | OSCAL 1.1.2 — Assessment Results, POA&M, Component Definition. Machine-readable for FedRAMP 20x. | Enterprise |
| `--dora` | `aegis-dora-evidence.json` | DORA (EU 2022/2554) conformity assessment. Maps to Pillar I-V. | Enterprise |
| `--iso-27001` | `aegis-iso27001-evidence.json` | ISO/IEC 27001:2022 evidence package. Annex A controls with SOC 2 bridge. | Enterprise |
| `--nist-csf` | `aegis-nist-csf.json` | NIST Cybersecurity Framework 2.0 evidence. All 6 functions. | Enterprise |
| `--vex` | `aegis-vex.json` | OpenVEX vulnerability exchange. Machine-readable for supply chain transparency. | Enterprise |
| `--sbom` | `aegis-sbom.cdx.json`, `aegis-sbom.spdx.json` | Software Bill of Materials in CycloneDX 1.5 and SPDX formats. | Enterprise |
| `--scoring` | `aegis-scoring.html` | Weighted security indicators across 6 domains. Informational — not a certification or grade. | Enterprise |
| `--compliance` | `aegis-compliance.html` | 13-framework risk map. Every finding mapped to FedRAMP, SOC 2, ISO 27001, PCI-DSS, HIPAA, DORA, NIST CSF, CMMC, OWASP, EU AI Act, SEC/FINRA, DoD SRG. Identifies which controls are at risk. | Enterprise |
| `--evidence-bundle` | Multiple files | All compliance formats + scoring + index. Input for compliance workflows — not standalone certification evidence. | Enterprise |
| `--conmon` | `aegis-conmon.json` | FedRAMP Continuous Monitoring package. POA&M, assessment, deviations, SLA tracking. | Premium |
| `--due-diligence` | `aegis-due-diligence.html` | M&A Technical Due Diligence. Traffic-light summary, risk rating, remediation estimate. | Premium |
| `--grc-summary` | `aegis-grc-summary.html` | GRC Executive Summary for third-party questionnaires. | Pro |
| `--grc-baseline <FILE>` | — | Baseline GRC summary for remediation progress comparison. | Pro |

### Architecture and code quality

| Flag | Output file | What it does | Tier |
|---|---|---|---|
| `--tech-debt` | `aegis-techdebt.html` | Tech debt: config typos (Levenshtein), dead code, heavyweight deps, TODO/FIXME, wildcard imports, long functions. | Pro |
| `--bounded-context` | `aegis-bounded-context.html` | Architecture analysis. Call graph entry point tracing, container split suggestions (70% Jaccard threshold). | Pro |
| `--api-surface` | `aegis-api-surface.txt` | API endpoint inventory. REST/GraphQL/gRPC across 13 framework patterns. | Pro |
| `--dep-accuracy` | `aegis-dep-accuracy.html` | Declared vs imported dependencies. Finds bloat (unused) and phantom (undeclared). | Pro |
| `--env-divergence` | `aegis-env-divergence.html/json` | Environment config drift. Prod vs dev divergences in env vars, feature flags, connection strings. | Pro |
| `--iac-divergence` | `aegis-iac-divergence.json` | IaC resource divergence. Terraform tfvars, container defs, circuit breakers across environments. | Pro |
| `--complexity` | `aegis-complexity.html` | Cyclomatic and cognitive complexity per function. Risk level classification (high/moderate/low). | Pro |
| `--gap-analysis` | `aegis-gap-analysis.html` | Per-framework compliance gap identification with remediation guidance. | Pro |

### Scan modifiers

| Flag | What it does | Tier |
|---|---|---|
| `--changed-only` | Only scan files changed in the git working tree (staged + unstaged). For pre-commit hooks. | All |
| `--since <REF>` | Only scan files changed since a git ref (e.g., `origin/main`, `HEAD~3`). For CI delta scans. | All |
| `--fail-on <SEV>` | Exit non-zero if findings at this severity or above. Values: `critical`, `high`, `medium`, `low`. | All |
| `--env <NAME>` | Tag scan with environment (e.g., `production`, `staging`). Auto-detects CI provider if omitted. | All |
| `--trend <FILE>` | Compare against a baseline (SARIF or `aegis-summary.json`). New, resolved, and changed findings. | Pro |
| `--brand <FILE>` | White-label branding (JSON). HTML reports use your brand. SARIF/OSCAL/JSON stay standards-formatted. | Premium |
| `--false-positive-threshold <FLOAT>` | Suppress findings with FP score >= threshold (0.0-1.0). | All |
| `--engineer` | Engineer triage mode — findings, STRIDE, log risk, tech debt, auto-fix, dep audit, dep accuracy, SARIF, JSON summary. Excludes compliance/M&A/evidence. | All |
| `--compliance-profile` | Compliance profile — OSCAL, DORA, ISO 27001, NIST CSF, scoring, GRC summary, gap analysis, JSON summary. For auditors and GRC analysts. | Enterprise |
| `--exclude <DIRS>` | Exclude directories. Repeatable (`--exclude vendor --exclude staging`) or comma-separated. Built-in excludes: `node_modules`, `.git`, `vendor`, `target`, `__pycache__`, `dist`, `build`, `.next`, `coverage`, `.aegis`, `deployments`, `.aws-sam`, `.terraform`, `.serverless`, `.venv`, `venv`, `.env`, `env`. | All |

---

## Other commands

### `aegis explain-flow`

Generate a scoped flow explainer (Markdown + HTML with Mermaid diagrams). Traces execution from an entry point through call graph, state machines, taint flows, and security findings.

```bash
aegis explain-flow ./target --entry "create_order" --project-name "AcmeShop"
aegis explain-flow ./target --entry "/api/v1/checkout"
aegis explain-flow ./target --entry "routes/applicant.py" --max-depth 15
```

### `aegis log`

Interactive log analysis (JSON output). Each subcommand parses logs independently — no persistent state.

```bash
aegis log search --logs ./logs --query "timeout" --level error
aegis log errors --logs ./logs --top 20
aegis log trace --logs ./logs --request-id abc-123
aegis log correlate --logs ./logs --scan-dir ./aegis-reports
aegis log elb-summary --elb-logs ./alb-logs --endpoints
aegis log elb-calltree --elb-logs ./alb-logs --trace-id "1-abc-def"
aegis log elb-duplicates --elb-logs ./alb-logs
aegis log elb-search --elb-logs ./alb-logs --status-min 500
```

For persistent sessions with IDE integration, use `aegis mcp-serve` instead.

### `aegis mcp-serve`

Run as an MCP server (stdio transport) for Claude Code integration. Exposes scan data as tools: `aegis_scan`, `aegis_findings`, `aegis_summary`, `aegis_explain`, `aegis_iam`, `aegis_taint`.

```bash
aegis mcp-serve --scan-dir ./aegis-reports
aegis mcp-serve --logs ./logs --elb-logs ./alb-logs
```

### `aegis scan-host`

Scan the current host for STIG/CIS compliance (reads `/etc`, `/proc`, `/sys`).

```bash
aegis scan-host --stig --cis --all
```

### `aegis config`

```bash
aegis config setup                   # Interactive setup
aegis config show                    # Show current config
aegis config set newrelic.api-key XYZ
aegis config add-target ./my-repo --label "Backend"
aegis config discover                # Discover infrastructure (containers, DBs, cloud, CI/CD)
aegis config targets                 # List configured targets
```

---

## Terminal output

Every scan prints a summary to the terminal:

```
Files scanned:    265
Lines of code:    126,334
Findings:         468 (50 shown, community tier)
Call graph:       4,094 nodes, 9,967 edges
Dead code:        3,162 functions (1,205 confirmed)
Tech debt:        1,619 findings

Severity:  █ 1 critical  ███ 4 high  █████████ 20 medium  ████████████ 25 low
Trust:     ████ 3 untrusted  ████████ 9 medium-trust  ██████ 4 trusted

▼ Action Required — 5 findings need attention
────────────────────────────────────────────────────────────────────────
  CRITICAL CWE-798 src/config/database.js:14
           Hardcoded database password
      HIGH CWE-89  src/routes/users.js:47
           SQL injection via string concatenation
      HIGH CWE-79  src/views/profile.js:23
           XSS via innerHTML assignment
  ...

Compliance Risk Map
────────────────────────────────────────────────────
FedRAMP High       [████████████████████]  85%  BLOCKED
FedRAMP Moderate   [████████████████████] 100%  AT RISK
SOC2 Type II       [████████████████████] 100%  AT RISK
PCI-DSS v4         [████████████████████]  85%  BLOCKED
HIPAA              [████████████████████]  95%  BLOCKED
DORA               [████████████████████] 100%  AT RISK
ISO 27001          [████████████████████] 100%  AT RISK
CMMC Level 2       [████████████████████]  85%  BLOCKED
NIST CSF           [████████████████████] 100%  AT RISK

Reports (8):
  aegis-reports/aegis-report.sarif.json
  aegis-reports/aegis-report.html
  ...

Scan: 3.3s | License: Enterprise | Engine: Rust native
```

### "Action Required" section

After every scan, critical and high findings are listed with file:line locations, sorted by severity. This is the "what to fix first" view — no scrolling through HTML reports to find the worst problems.

### Compliance risk indicators

These show which compliance frameworks have findings that map to their controls. They identify risk areas — they do not certify compliance.

| Status | Meaning |
|---|---|
| **READY** | No critical or high findings map to this framework's controls |
| **AT RISK** | Findings exist that map to this framework's controls — review recommended |
| **BLOCKED** | Critical or high findings map to controls that would likely block certification |

### Trust levels

| Level | Meaning |
|---|---|
| **Untrusted** | Reachable from external/user input (highest risk) |
| **Medium-trust** | Reachable from internal service calls |
| **Trusted** | Only reachable from trusted/authenticated paths |

---

## HTML report features

The `--html` vulnerability report is interactive:

- **Severity toggles** — Click Critical / High / Medium / Low buttons to show/hide by severity
- **Reachability filter** — Click Reachable / Unreachable counts to filter findings by reachability status
- **File search** — Type a path to filter findings to a specific file or directory
- **Sortable columns** — Click Severity, CWE, File, or Line headers to re-sort
- **Collapse/expand** — Toggle finding details open or closed in bulk
- **Criticals-first** — Default sort puts critical findings at the top
- **Trust badges** — Each finding shows its trust classification (Untrusted through Trusted)
- **Cross-linked reports** — Breadcrumb navigation and clickable counts linking to related reports

All filtering is client-side (vanilla JS, no dependencies). The HTML file is fully self-contained — share it as a single file.

---

## Languages supported

| Language | Extensions | Parser | AST features |
|---|---|---|---|
| C | `.c`, `.h` | tree-sitter | Functions, calls, includes |
| C++ | `.cpp`, `.cc`, `.cxx`, `.hpp` | tree-sitter | Functions, calls, includes |
| Python | `.py` | tree-sitter | Functions, calls, imports (from/import) |
| JavaScript | `.js`, `.mjs`, `.cjs`, `.jsx` | tree-sitter | Functions, calls, imports/require |
| TypeScript | `.ts`, `.tsx`, `.mts`, `.cts` | tree-sitter | Functions, calls, imports |
| Java | `.java` | tree-sitter | Methods, calls, imports |
| Go | `.go` | tree-sitter | Functions, calls, imports |
| C# | `.cs` | tree-sitter | Methods, calls, using directives |
| Rust | `.rs` | tree-sitter | Functions, impl items, use declarations |
| HCL/Terraform | `.tf`, `.hcl`, `.tfvars` | tree-sitter | Blocks, resources, modules |
| Bash | `.sh`, `.bash`, `.zsh` | tree-sitter | Functions |
| PHP | `.php` | tree-sitter | Functions, methods, namespace use |
| Kotlin | `.kt`, `.kts` | tree-sitter (via Java) | Functions, imports |
| Swift | `.swift` | tree-sitter | Functions, import declarations |

All languages get: function extraction, call graph construction, CWE pattern matching, and dead code detection. Inter-procedural dataflow and taint analysis depth varies by language — TypeScript has measured parity; C/C++ are scoped to OS-command-injection; others are AST-pattern with heuristic analysis.

---

## CWE coverage

115 detection patterns covering 126 CWE classes. Key categories:

| CWE | Name | Severity |
|---|---|---|
| CWE-78 | OS Command Injection | Critical |
| CWE-79 | Cross-Site Scripting (XSS) | High |
| CWE-89 | SQL Injection | Critical |
| CWE-94 | Code Injection | Critical |
| CWE-120 | Buffer Overflow | High |
| CWE-295 | Improper Certificate Validation | Critical |
| CWE-319 | Cleartext Transmission | High |
| CWE-327 | Weak Cryptography | High |
| CWE-396 | Catch Generic Exception | Medium |
| CWE-401 | Memory Leak | Medium |
| CWE-502 | Unsafe Deserialization | Critical |
| CWE-798 | Hardcoded Credentials | Critical |
| CWE-918 | Server-Side Request Forgery | High |

Full list: 115 patterns including buffer overflows, race conditions, path traversal, open redirects, prototype pollution, format strings, null dereference, use-after-free, double-free, IAM misconfigurations, and more.

### Language-aware suppressions

The engine suppresses known false positives by language:
- **Rust/Go:** CWE-401 (`::new()` is a constructor, not a malloc) and CWE-1321 (`.extend()` is type-safe)
- **Rust:** CWE-798 suppressed on const verification key declarations and `#[cfg(test)]` blocks
- **Python:** Defensive `free(p); p = NULL;` idiom suppresses CWE-415/416
- **All:** Context-aware severity adjustment based on trust level, reachability, and compensating controls

---

## Configuration files

| File | Purpose |
|---|---|
| `~/.raknor/license.json` | Persistent license key |
| `~/.raknor/config.json` | New Relic API key, DAST endpoints, scan targets |
| `.aegis-brand.json` | White-label branding (company, logo, colors) |
| `.aegis-suppress.json` | Finding suppressions (CWE + file path + reason) |

### White-label branding (`.aegis-brand.json`)

```json
{
  "company_name": "Your Company",
  "product_name": "Your Security Scan",
  "logo_url": "https://your-domain.com/logo.png",
  "primary_color": "#1a5276",
  "accent_color": "#2ecc71",
  "footer_text": "Powered by Your Company",
  "contact_url": "https://your-domain.com",
  "support_email": "security@your-domain.com"
}
```

---

## Exit codes

| Code | Meaning |
|---|---|
| 0 | Scan completed, no findings at or above `--fail-on` threshold |
| 1 | Findings found at or above `--fail-on` threshold |
| 2 | Scan error (invalid target, missing files, configuration error) |

---

## Architecture

```
aegis scan ./target
        │
        ▼
┌─────────────────────────────────────┐
│  aegis-cli (Rust binary, ~33 MB)    │
│                                     │
│  Parser (tree-sitter, 14 languages) │
│  → AST extraction                   │
│  → Call graph (petgraph)            │
│  → Taint analysis (sources → sinks)│
│  → 115 CWE pattern matching         │
│  → ConfirmedFinding (CWE-89, CWE-94)│
│                                     │
│  40 report generators               │
│  Ed25519 + FIPS P-256 license       │
│  Everything local — nothing uploaded│
└─────────────────────────────────────┘
```

- **No cloud dependency.** All analysis runs locally.
- **No source upload.** Code never leaves the machine.
- **No LLM.** Deterministic — same input, same output.
- **No runtime agent.** Point-in-time scan, no persistent process.

---

## What AEGIS is

AEGIS confirms static vulnerability findings with dynamically-derived probes — not
inference from patterns, but observation of the vulnerability being exercised against
a running target. Today this is demonstrated for SQL injection (CWE-89) and code injection
(CWE-94); command injection and path traversal are in progress. It finds those findings
via AST-based CWE detection across 14 languages, with inter-procedural taint analysis
at varying depth (TypeScript at measured parity; C/C++ scoped; others heuristic).

Each finding is classified as **Class-M** (mechanically verified) or **Class-J**
(judgment/pattern-only). A finding earns Class-M through one of three criteria:
taint-flow confirmation (source-to-sink dataflow traced), exploit proof (sandbox-validated
PoC), or structural match (reachable CWE in a deterministic class). Findings that don't
meet any criterion remain Class-J. This classification appears in SARIF output and
drives downstream grading — Class-J findings never reach the confidence tier that
Class-M evidence occupies.

Findings map to 13 compliance frameworks (NIST, ISO, PCI-DSS, DORA, and nine others); these
mappings identify which controls are at risk, not certify that they are satisfied. Output
is SARIF 2.1.0, OSCAL, and HTML. The scanner is a single Rust binary, runs locally and
offline, and maintains a cryptographic provenance chain across scan results.

---

*AEGIS v3.7.1 — Pareidolia LLC (d/b/a Raknor AI / Equilateral AI)*
*https://aegis.raknor.ai*
