# MCPwn — Black Hat Asia 2027 Arsenal Submission

## Tool Name
MCPwn: Offensive Security Framework for AI Agent Protocols

## Category
AI/ML Security / Exploitation

## Description (short)
MCPwn is the first comprehensive open-source offensive security framework, to our knowledge, for the Model Context Protocol (MCP) — the JSON-RPC standard powering AI agent-tool communication in Claude, GPT, Cursor, and 200,000+ exposed server endpoints. MCPwn automates discovery, enumeration, vulnerability assessment, and exploitation of MCP servers across 9 vulnerability classes.

## Detailed Description

### Problem
MCP (Model Context Protocol) has become the de facto standard for connecting AI agents to external tools. Adopted by Anthropic (Claude), OpenAI (GPT), Cursor, and hundreds of AI platforms, it enables agents to read files, execute commands, query databases, and call APIs on remote servers.

The security problem is severe:
- **200,000+ MCP server endpoints** exposed on the internet (Shodan/Censys, 2025)
- **36% have zero authentication** — default SDK binds to 0.0.0.0 with no auth
- **30+ CVEs** in 2025-2026 (per OWASP MCP Top 10, NVD records)
- **First malicious MCP package** (postmark-mcp) found in the wild
- Tool arguments passed **unsanitized** to shell commands, file operations, SQL queries
- Tool descriptions can contain **prompt injection** payloads that manipulate AI agent behavior

The OWASP MCP Top 10 was published in 2025, yet no comprehensive tool exists to systematically test for these vulnerabilities.

### Solution — MCPwn
MCPwn is a modular Python framework (6,114 LOC, 32 tests) that provides:

**1. MCP Protocol Client**
Full implementation of MCP JSON-RPC 2.0 over both transports:
- HTTP+SSE (Streamable HTTP) — the web-facing transport
- stdio — the local process transport
Handles initialization handshake, capability negotiation, tool/resource/prompt enumeration, and tool invocation.

**2. Network Scanner**
- Probes common MCP ports (3000, 3001, 5000, 8000, 8080, 9000)
- Tests standard endpoints (/mcp, /sse, /api/mcp)
- Multi-threaded subnet scanning
- Auth detection and server fingerprinting

**3. Automated Security Auditor (9 vulnerability classes)**
| Class | CWE | Payloads |
|---|---|---|
| Path Traversal | CWE-22 | 12 variants (URL encoding, double encoding, platform-specific) |
| OS Command Injection | CWE-78 | 15 payloads (;, |, $(), ``, &&, ||) |
| SSRF | CWE-918 | AWS/GCP/Azure metadata, internal services, file://, gopher:// |
| SQL Injection | CWE-89 | UNION-based, time-based blind, error-based |
| Arbitrary File Write | CWE-73 | Webshell, cron, SSH key injection |
| Missing Authentication | CWE-306 | Unauthenticated tool access |
| Tool Poisoning | CWE-94 | 8 prompt injection patterns in tool descriptions |
| Sensitive Data Exposure | CWE-200 | Credentials in tool metadata |
| Prompt Injection via Response | — | AI agent behavior manipulation |

**4. Attack Framework**
- **Data Exfiltration**: Extract SSH keys, AWS credentials, Docker config, .env files through standard MCP file-reading tools
- **Resource URI Injection**: Bypass access controls via file://, internal IPs, cloud metadata URIs
- **Response Injection**: Craft tool responses that manipulate the AI agent's behavior
- **Privilege Escalation**: Access admin-level tools without authentication
- **Tool Poisoning Analysis**: Detect and demonstrate prompt injection in tool descriptions

**5. CVE Database**
Known MCP CVEs with server name/version fingerprinting:
- CVE-2025-49596: MCP Inspector RCE (CVSS 9.4)
- CVE-2026-27825: mcp-atlassian arbitrary file write → RCE
- Automated matching against discovered servers

**6. Vulnerable Test Server**
Intentionally vulnerable MCP server with all 9 vulnerability classes for safe demonstration and training.

**7. Forensic Logger**
MCP traffic capture with SHA-256 integrity chains, anomaly detection for suspicious tool descriptions, hidden Unicode, and sensitive data leaks.

### Real Results
- Demo against vulnerable server: **176 checks, 26 findings (24 Critical, 2 High), 33 successful attacks**
- Real data exfiltration: SSH keys, AWS credentials, Docker config extracted through standard MCP tool calls
- Audit scripts identifying unvalidated file paths, unrestricted command execution, and missing authentication in real npm/pip MCP packages

### What Makes This Different
| Existing Work | Limitation | MCPwn |
|---|---|---|
| OWASP MCP Top 10 | Risk taxonomy, no tooling | Automated testing for all 10 categories |
| MCP Inspector | Official debugging tool (itself had RCE — CVE-2025-49596) | Offensive security focus |
| Manual pentesting | Per-engagement, not systematic | Automated, reproducible, 176 checks |
| No dedicated tools | — | First comprehensive MCP security framework |

## Demo Plan (50 minutes)

1. **The MCP Attack Surface** (5 min) — Protocol overview, why 200K+ servers are exposed, real-world incidents
2. **Discovery & Enumeration** (10 min) — Scan network, find MCP servers, enumerate tools/resources, fingerprint server versions
3. **Automated Security Audit** (10 min) — Run full 9-class audit against vulnerable server, walk through findings
4. **Exploitation Chain** (15 min) — Tool poisoning → data exfiltration (SSH keys, AWS creds) → response injection → privilege escalation. Full kill chain.
5. **Real-World Findings** (5 min) — Vulnerabilities found in popular npm/pip MCP packages
6. **Defense & Detection** (5 min) — How to secure MCP deployments, forensic logging capabilities
7. **Q&A** (5 min)

**Environment:** MCPwn's built-in vulnerable server + real MCP package examples. All attacks against controlled targets.

## Requirements
- Python 3.8+
- No external dependencies
- Network access for scanner (optional — works against local servers)

## Speaker
**Dr. Sapna V M** — Associate Professor, PES University, Bangalore. Research focus: AI security, protocol exploitation, digital forensics. Published work on offensive security tooling for emerging protocols including IoT, automotive, and industrial systems.

## Repository
https://github.com/youraccount/MCPwn

## License
MIT

## Release
Open-source, released before conference. Fully functional with pip install.

## Responsible Disclosure
All vulnerabilities found in real MCP servers are reported to maintainers before conference presentation. CVEs assigned through standard process. MCPwn targets only servers explicitly provided by the user for testing.
