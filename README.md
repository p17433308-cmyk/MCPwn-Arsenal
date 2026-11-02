```
  __  __  ____ ____                 
 |  \/  |/ ___|  _ \__      ___ __  
 | |\/| | |   | |_) \ \ /\ / / '_ \ 
 | |  | | |___|  __/ \ V  V /| | | |
 |_|  |_|\____|_|     \_/\_/ |_| |_|
```

# MCPwn — Offensive Security Framework for AI Agent Protocols

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Tests](https://img.shields.io/badge/tests-32%20passing-brightgreen.svg)]()
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey.svg)]()
[![Black Hat Arsenal](https://img.shields.io/badge/Black%20Hat-Asia%202027%20Arsenal-black.svg)]()

> **First comprehensive open-source offensive security framework, to our knowledge, for the Model Context Protocol (MCP) — the JSON-RPC standard powering AI agent-tool communication in Claude, GPT, Cursor, and 200,000+ exposed servers.**

---

## The Problem

MCP (Model Context Protocol) has become the de facto standard for connecting AI agents to external tools. Every major AI platform uses it — and it's broken by design:

| Attack Surface | Scale | Source |
|---|---|---|
| Exposed MCP servers | **200,000+** endpoints | Shodan/Censys, 2025 |
| Zero authentication | **36%** of exposed servers | Default SDK binds to 0.0.0.0 |
| Published CVEs | **30+** in 2025-2026 | OWASP MCP Top 10, NVD |
| Malicious packages | **postmark-mcp** confirmed | npm registry, 2025 |

The OWASP MCP Top 10 was published in 2025. Yet **no comprehensive open-source offensive tool existed** to systematically test for these vulnerabilities.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        MCPwn CLI                                │
│         scan · audit · attack · cve · forensics · demo          │
├───────────┬───────────┬───────────┬───────────┬─────────────────┤
│   RECON   │   AUDIT   │  ATTACKS  │ FORENSICS │    VULN DB      │
│           │           │           │           │                 │
│ Port Scan │ Path Trav │ Tool Pois │ Traffic   │ CVE-2025-49596  │
│ Endpoint  │ Cmd Inj   │ Data Exfil│ Capture   │ CVE-2026-27825  │
│ Discovery │ SSRF      │ URI Inj   │ Anomaly   │ Server          │
│ Auth Det  │ SQLi      │ Response  │ Detection │ Fingerprint     │
│ Server FP │ File Wrt  │ Injection │ SHA-256   │ Matching        │
│           │ No Auth   │ Priv Esc  │ Chains    │                 │
│           │ Tool Pois │           │           │                 │
│           │ Data Leak │           │           │                 │
│           │ Prompt Inj│           │           │                 │
├───────────┴───────────┴───────────┴───────────┴─────────────────┤
│                    MCP PROTOCOL CLIENT                          │
│    JSON-RPC 2.0 · HTTP+SSE Transport · stdio Transport          │
│    Initialization · Capability Negotiation · Tool Invocation     │
└─────────────────────────────────────────────────────────────────┘
```

### Module Breakdown

```
MCPwn/
├── mcpwn/
│   ├── protocols/          # MCP JSON-RPC 2.0 client
│   │   └── mcp_client.py   #   HTTP+SSE & stdio transports, handshake, tool invoke
│   ├── recon/              # Reconnaissance
│   │   └── scanner.py      #   Port scanning, endpoint discovery, auth detection
│   ├── audit/              # Automated vulnerability assessment
│   │   └── security_auditor.py  #  9 vuln classes, 70+ payloads
│   ├── attacks/            # Active exploitation
│   │   └── injector.py     #   Tool poisoning, exfil, injection, privesc
│   ├── forensics/          # Defensive capabilities
│   │   └── logger.py       #   Traffic capture, anomaly detection, SHA-256 chains
│   ├── vulndb/             # CVE intelligence
│   │   └── known_cves.py   #   CVE database, server fingerprinting
│   └── cli.py              # Full CLI interface
├── test_server/
│   └── vuln_server.py      # Intentionally vulnerable MCP server (all 9 classes)
├── scripts/
│   └── audit_real_servers.py  # Audit real npm/pip MCP packages
├── tests/
│   └── test_mcpwn.py       # 32 test cases
└── setup.py
```

---

## Vulnerability Classes

MCPwn's automated auditor tests for **9 vulnerability classes** with **70+ payloads**:

| # | Vulnerability | CWE | Payloads | Severity |
|---|---|---|---|---|
| 1 | Path Traversal in file-handling tools | CWE-22 | 12 variants (URL encoding, double encoding, platform-specific) | Critical |
| 2 | OS Command Injection via exec tools | CWE-78 | 15 payloads (`;`, `\|`, `$()`, `` ` ` ``, `&&`, `\|\|`) | Critical |
| 3 | SSRF via URL-fetching tools | CWE-918 | AWS/GCP/Azure metadata, internal services, `file://`, `gopher://` | Critical |
| 4 | SQL Injection in database tools | CWE-89 | UNION-based, time-based blind, error-based | Critical |
| 5 | Arbitrary File Write | CWE-73 | Webshell, cron job, SSH authorized_keys | Critical |
| 6 | Missing Authentication | CWE-306 | Unauthenticated tool access, default 0.0.0.0 binding | High |
| 7 | Tool Poisoning (prompt injection in descriptions) | CWE-94 | 8 patterns: hidden Unicode, role hijacking, cross-tool exploitation | High |
| 8 | Sensitive Data Exposure in tool metadata | CWE-200 | Credentials, API keys, internal paths in tool metadata | High |
| 9 | Prompt Injection via tool responses | — | AI agent behavior manipulation through crafted responses | High |

---

## Attack Modules

### Kill Chain

```
  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
  │   DISCOVERY  │───►│  ENUMERATION │───►│    AUDIT     │───►│ EXPLOITATION │
  │              │    │              │    │              │    │              │
  │ Port scan    │    │ List tools   │    │ 9 vuln       │    │ Exfiltrate   │
  │ Endpoint     │    │ List resources│   │ classes      │    │ Inject       │
  │ probe        │    │ List prompts │    │ 70+ payloads │    │ Escalate     │
  │ Auth detect  │    │ Fingerprint  │    │ CVE match    │    │ Poison       │
  └──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

| Module | Description | Impact |
|---|---|---|
| **Tool Poisoning Analysis** | Detect prompt injection payloads hidden in tool descriptions (hidden Unicode, role hijacking, cross-tool chains) | AI agent manipulation |
| **Data Exfiltration** | Extract sensitive files through legitimate file-reading tools: SSH keys, AWS credentials, Docker config, `.env`, `/etc/passwd` | Credential theft |
| **Resource URI Injection** | Bypass access controls via `file://`, internal IP, and cloud metadata resource URIs | Access control bypass |
| **Response Injection** | Craft tool responses that alter AI agent behavior — instruction override, context poisoning | AI agent hijacking |
| **Privilege Escalation** | Access admin-level and dangerous tools without authentication | Full server compromise |

---

## Demo Results

Against MCPwn's built-in vulnerable server:

```
MCPwn Security Audit Report
═══════════════════════════════════════════════════════

Target:     http://localhost:8080
Transport:  HTTP+SSE
Duration:   12.3 seconds

CHECKS:     176 total
FINDINGS:   26 vulnerabilities
  ├── CRITICAL:  24
  ├── HIGH:       2
  └── INFO:       0

ATTACKS:    33 successful
  ├── Data Exfiltration:    SSH keys, AWS credentials, Docker config
  ├── Command Injection:    Remote code execution via exec tools
  ├── SSRF:                 AWS metadata (169.254.169.254) accessed
  ├── Path Traversal:       /etc/passwd, /etc/shadow read
  └── SQL Injection:        Database dump via UNION-based injection

FORENSICS:  SHA-256 integrity chain generated
            14 anomalies flagged
```

---

## Usage

### Installation

```bash
git clone https://github.com/p17433308-cmyk/MCPwn-Arsenal.git
cd MCPwn-Arsenal
pip install -e .
```

### Quick Start — Full Demo

```bash
python -m mcpwn demo
```

This starts the vulnerable server, runs discovery, audit, and attacks — full pipeline.

### Individual Modules

**Reconnaissance:**
```bash
# Scan a single server
mcpwn scan --url http://localhost:3000

# Scan a host (all common MCP ports)
mcpwn scan --host 192.168.1.100

# Scan a subnet
mcpwn scan --network 10.0.0
```

**Security Audit:**
```bash
# Full 9-class audit with report
mcpwn audit http://localhost:3000 --output report.json
```

**Exploitation:**
```bash
# Run all attack modules
mcpwn attack http://localhost:3000 --module all

# Data exfiltration only
mcpwn attack http://localhost:3000 --module exfil

# Tool poisoning analysis
mcpwn attack http://localhost:3000 --module poison
```

**CVE Database:**
```bash
# List all known MCP CVEs
mcpwn cve --list

# Check server against CVE database
mcpwn cve --url http://localhost:3000
```

**Audit Real-World Servers:**
```bash
# Audit official MCP filesystem server
python scripts/audit_real_servers.py --target filesystem

# Audit all known targets
python scripts/audit_real_servers.py --all
```

---

## Vulnerable Test Server

`test_server/vuln_server.py` — an intentionally vulnerable MCP server with all 9 vulnerability classes for safe demonstration and training:

```bash
# Start the vulnerable server
python test_server/vuln_server.py --port 8080

# Then audit it
mcpwn audit http://localhost:8080
```

**Vulnerabilities included:**
- Path traversal — no path sanitization on file reads
- Command injection — `shell=True` on exec tools
- SSRF — no URL validation on fetch tools
- SQL injection — string concatenation in queries
- Prompt injection — malicious payloads in tool descriptions
- Credential exposure — secrets in tool responses
- Zero authentication — binds to `0.0.0.0` with no auth

---

## MCP Protocol Implementation

MCPwn implements the full MCP JSON-RPC 2.0 specification (protocol version `2025-03-26`):

| Transport | Method | Details |
|---|---|---|
| **HTTP+SSE** | POST to `/mcp` | Streamable HTTP, SSE for server events |
| **stdio** | stdin/stdout | Launch subprocess, JSON-RPC over pipes |

**Protocol operations:** Initialize → Capability negotiation → Tool/Resource/Prompt enumeration → Tool invocation → Response parsing

---

## Known CVE Database

| CVE | Target | CVSS | Impact |
|---|---|---|---|
| CVE-2025-49596 | MCP Inspector | 9.4 | Remote Code Execution |
| CVE-2026-27825 | mcp-atlassian | 8.8 | Arbitrary File Write → RCE |
| — | postmark-mcp | — | First confirmed malicious MCP package |
| — | 200,000+ servers | — | No-auth exposure (Shodan/Censys) |

Fingerprinting engine matches discovered servers against the CVE database automatically.

---

## Prior Art

| Tool | Scope | MCPwn Advantage |
|---|---|---|
| OWASP MCP Top 10 | Risk taxonomy | Automated testing for all categories |
| MCP Inspector | Debugging (had RCE itself) | Offensive security focus |
| Manual pentesting | Per-engagement | Systematic, reproducible, 176 checks |
| Burp Suite / ZAP | HTTP/REST APIs | MCP-protocol-native |

---

## Dependencies

**None required.** Pure Python standard library. The `requests` library is used when available but MCPwn falls back to raw socket HTTP when it is not installed.

## Tests

```bash
pytest tests/ -p no:cacheprovider    # 32 tests covering all modules
```

---

## Authors

- **Sinchana Shri Mewundi** — PES University
- **Adithya Ajay Kulkarni** — PES University
- **Prajwal Chowdary M** — PES University
- **Prasad H B** — PES University

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

## Disclaimer

MCPwn is designed for **authorized security testing, research, and educational purposes only**. All new vulnerabilities found in real MCP servers are reported to vendors before public disclosure. CVEs are assigned through standard processes. Users are responsible for ensuring compliance with all applicable laws. Do not use this tool against servers without explicit authorization.
