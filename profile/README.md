# Eunolabs

**Capability enforcement for AI agents.**

Your identity provider already says *who* an agent is. What's missing is a
control point for *what* it is allowed to do. Eunolabs builds least-privilege
security tooling for the [Model Context Protocol (MCP)](https://modelcontextprotocol.io):
enforce which tools an agent may call, with which arguments, in which order —
at the protocol boundary, before the call runs.

## Projects

**[eunox](https://github.com/eunolabs/eunox)** — A capability firewall for MCP
servers. A deny-by-default policy proxy that sits between an MCP host and any MCP
server (local subprocess or remote HTTPS), checks every call against a YAML
capability manifest, filters tool / resource / prompt listings down to what's
permitted, and writes a signed, tamper-evident audit log. Ships as one static Go
binary.

**[mcp-capability-manifest](https://github.com/eunolabs/mcp-capability-manifest)**
— The vendor-neutral specification and JSON Schemas behind eunox: a file-based
format for declaring what an agent may do against an MCP server, plus a companion
JWT claim for narrowing that scope per invocation. Any enforcement proxy can
implement it.

## Why

MCP standardizes how an agent discovers and invokes server-side tools, but is
deliberately silent on authorization — who may invoke them, and under what
conditions. Eunolabs fills that gap with a declarative capability manifest,
enforced deny-by-default at the boundary, and an audit trail you can verify.

## Learn more

- **Website** — <https://eunolabs.ai>
- **Security** — found a vulnerability? Please report it privately; see our
  [security policy](https://github.com/eunolabs/.github/blob/main/SECURITY.md).
- **Contributing** — everything here is open source under the Apache-2.0 license.
  Start with [`CONTRIBUTING.md`](https://github.com/eunolabs/.github/blob/main/CONTRIBUTING.md)
  and each repository's own guide.
