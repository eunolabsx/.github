# Competitive Landscape: eunox & mcp-capability-manifest

**Date:** July 2026
**Scope:** MCP security gateways/proxies, agent guardrails & scanners, authorization platforms, and identity vendors moving into agent authorization — what they offer today, where they're heading, and how they compete with Eunolabs' protocol-boundary, deny-by-default capability enforcement.

**Method note.** Findings below were gathered from primary sources (vendor announcements, official docs, the MCP blog) and passed through adversarial 3-vote verification; 22 of 25 top claims were confirmed, 1 refuted, 2 left unverified (marked ⚠ where relied upon). Claims drawn only from the broader extraction pass (not individually verified) are marked †.

---

## 1. Executive summary

The market Eunolabs plays in has gone from empty to crowded in roughly eighteen months. A widely referenced community catalog now lists **~47 distinct MCP gateway products (~20 open-source, ~27 commercial)**†. Three distinct groups are converging on "control what an AI agent can do":

1. **API-gateway incumbents** (Kong, Solo.io, Cloudflare, Pomerium) bundling per-tool MCP ACLs into existing gateways.
2. **Authorization platforms** (Oso, Permit.io) repositioning their policy engines as "AI agent access control."
3. **Identity vendors** (Okta, Descope; Cisco via the Astrix acquisition) anchoring agent control at the identity/credential layer — and, in Okta's case, successfully pushing their approach *into the MCP spec itself*.

**No verified competitor currently combines eunox's specific bundle**: deny-by-default enforcement, argument-level and call-ordering constraints, listing filtering, *signed tamper-evident* audit logs, a vendor-neutral file-based manifest spec, and a single open-source static binary. But every individual piece of that bundle now has at least one well-funded competitor, and the biggest strategic threat is standards absorption: Okta's Cross App Access is now an official MCP authorization extension, and the 2026-07-28 spec release hardens OAuth-layer authorization in core.

---

## 2. MCP gateways & proxies (direct competitors)

### Kong AI Gateway — the most direct feature overlap
- **Offering today:** Kong AI Gateway 3.13 (announced January 2026) introduced **MCP Tool ACLs** via its AI MCP Proxy plugin: allow/deny rules at a default level (all tools) plus per-tool overrides, keyed to authenticated Kong Consumers and Consumer Groups. ([Kong blog](https://konghq.com/blog/product-releases/mcp-tool-acls-ai-gateway), verified 3-0.) It also filters `tools/list` responses per identity so agents never see restricted tools (⚠ verification incomplete due to workflow rate limit; 1-0 in favor).
- **Direction:** Actively tracking the MCP spec — in 3.14 it changed ACL denial responses from JSON-RPC errors to HTTP 403 to match the MCP 2025-11-25 authorization spec†. Kong extended into MCP with plugins and a registry service across AI Gateway and Konnect in early 2026†.
- **vs. eunox:** Closest head-to-head competitor at the protocol boundary. Two verified/reported differentiators for Eunolabs: Kong's MCP ACLs are **allow-by-default when no rules are configured**† (opposite of eunox's deny-by-default), and the feature sits in Kong's **commercial enterprise tier** (⚠ unverified) versus eunox's open-source single binary. Kong ACLs are per-tool; no evidence of argument-level constraints, call-ordering rules, or signed audit logs.

### Pomerium
- **Offering today:** Open-source, identity-aware "agentic gateway" purpose-built for securing MCP tool calls — sits in the network path between agents and MCP servers with tool-level authorization and comprehensive audit logging†. Its own market framing cites research finding **1,862 internet-exposed MCP servers with no authentication**†.
- **Direction:** Positioning hard around the "agentic gateway" category; its published evaluation criteria (tool-level authZ granularity, audit logging of calls/parameters/decisions, upstream OAuth 2.1, session-aware policy, prompt-injection resilience) read like a spec of the category Eunolabs competes in†.
- **vs. eunox:** Same enforcement point, open-source like eunox. Differentiation rests on eunox's declarative manifest (argument/ordering constraints), signed logs, and vendor-neutral spec vs. Pomerium's identity-policy model.

### Solo.io agentgateway, Cloudflare AI Gateway, Aembit, MintMCP, PolicyLayer
- **agentgateway** (Solo.io-originated, open source): drop-in security, observability, and governance for agent-to-agent (A2A) *and* agent-to-tool (MCP) traffic†.
- **MintMCP:** commercial enterprise gateway with OAuth/SSO, monitoring, and real-time guardrails†.
- **PolicyLayer:** markets **deterministic allow-list policy enforcement on every tool call outside the LLM loop** — essentially eunox's value proposition in different words†. Worth tracking closely.
- Pomerium's own competitive set for 2026 names Pomerium, Kong, Cloudflare AI Gateway, agentgateway, and workload-identity vendor Aembit†.
- Notably, the e2b-dev awesome-mcp-gateways catalog contains **no entry described as a deny-by-default capability firewall, capability manifest, or signed-audit-log solution**† — the manifest-plus-signed-audit combination is not yet a named category.

---

## 3. Guardrails & scanners (adjacent, mostly non-overlapping)

### Snyk agent-scan (Invariant Labs lineage)
- **Offering today:** A **static scanner, not a runtime enforcement proxy** — inventories installed agent components (harnesses, MCP servers, skills) and scans for 15+ risk categories: prompt injection, tool poisoning, tool shadowing, malware payloads, credential handling, hardcoded secrets. ([github.com/snyk/agent-scan](https://github.com/snyk/agent-scan), verified 3-0.) Client is Apache-2.0 but **scans require a Snyk account/API token** — open-core with a commercial backend (verified 3-0). Active and adopted: v0.5.12 released June 23, 2026, ~2.7k stars†; supports scanning configs of Claude Desktop/Code, Cursor, Windsurf, Gemini CLI, and others†.
- **Direction:** Snyk explicitly describes runtime enforcement as **still an area of active research**† — its strategy is visibility (AI-BOM inventory + MCP scanning), and it frames the core MCP risk as "toxic flows" (unsafe combinations of tools, permissions, and data paths)† — the same problem class eunox's call-ordering constraints address at runtime.
- **vs. eunox:** Complementary more than competitive today (scan-time vs. enforcement-time). The threat is that Snyk graduates from research into runtime enforcement with an installed enterprise base.

### Zenity
- **Offering today:** Launched runtime protection for OpenAI AgentKit agents (November 2025) — deterministic rule-based policies enforced "at the endpoint," detecting/blocking data leakage, secret exposure, and unsafe responses†.
- **Direction:** Per-platform runtime coverage of major agent-building ecosystems (AgentKit, Microsoft Copilot Studio, ChatGPT Enterprise)†.
- **vs. eunox:** Different enforcement point (agent platform/endpoint, content-focused) — not tool-call capability allowlisting, argument constraints, or sequence enforcement. Adjacent, not head-on.

### Prompt Security, Knostic
Both are active in agentic-AI security marketing, but no claims from their materials survived source-quality screening in this research pass. Treat as watch-list.

---

## 4. Authorization platforms (repositioning into agents)

### Oso — "Oso for Agents"
- **Offering today:** Oso has repositioned its flagship as an **AI agent security and control platform**: discover every agent, monitor approved AI traffic, detect anomalies/policy violations, enforce rules, and produce complete audit logs ([osohq.com](https://www.osohq.com/), verified 3-0). It supports **MCP-aware deny-style rules** ("block unknown MCP servers," "deny all delete operations") — enforcing at the same layer eunox targets (verified 2-1). Headline marketing is **"automated least privilege"** — directly contesting Eunolabs' language (verified 3-0).
- **Model:** Hosted cloud platform (free dev tier; Startup from $149/mo); the platform is not open source, though Oso maintains an OSS authorization library†. Policies in its Polar language (RBAC/ABAC/ReBAC) with ingested Facts†.
- **vs. eunox:** Direct overlap on enforcement-plus-audit positioning, but delivered as a cloud policy-decision platform vs. eunox's self-hosted protocol-boundary binary with a file-based manifest. Eunolabs' angle: no cloud dependency, deterministic wire-level enforcement, signed logs.

### Permit.io — AI Access Control
- **Offering today:** Extends its FGA platform to AI identity via a "Four-Perimeter Framework" (prompt filtering, RAG data protection, securing external access, response enforcement), with an **MCP integration that applies identity-based checks before agents execute actions through MCP servers**†. Delivered as SDK/framework integrations: PydanticAI, LangChain, LangFlow, MCP†. Also assigns machine identities to agents with per-action permissions†.
- **vs. eunox:** Enforcement embedded in application/orchestration code, not an out-of-process proxy. Competes for the same budget but a different architectural conviction; eunox wins where teams can't or won't modify agent code.

### Cerbos
Marketing agentic authorization; no claims survived source screening this pass. Watch-list.

---

## 5. Identity vendors moving into agent authorization

### Okta — the biggest strategic threat
- **Offering today:** "Okta for AI Agents" — a commercial platform to **discover/register known and unknown agents, standardize agent access, and instantly revoke access** — generally available **April 30, 2026** ([Showcase 2026 release](https://www.okta.com/newsroom/press-releases/showcase-2026/), verified 3-0). Okta Privileged Access enforces policies for agents using static credentials (verified 3-0). Its "blueprint for the secure agentic enterprise" claims identity as the control plane for *where are my agents, what can they connect to, and what can they do* — overlapping Eunolabs' core question, anchored at the identity layer (verified 3-0).
- **Standards play (critical):** Okta's **Cross App Access (XAA)** has been **incorporated into MCP as the "Enterprise-Managed Authorization" extension, with support landing in official MCP SDKs (TypeScript and Java first)** (verified 3-0). Okta claims an XAA ecosystem of **25+ software makers including Anthropic (Claude), Atlassian, Slack, Cloudflare, and Datadog** as of June 2026†.
- **vs. eunox:** Okta does not (per these releases) do argument-level or call-sequence enforcement at the MCP wire protocol — its granularity is agent identity, access grants, and revocation. But XAA-in-the-spec means the *identity-vendor-led OAuth extension* is becoming the default answer to "MCP authorization," which crowds the mindshare Eunolabs' vendor-neutral manifest spec needs.

### Descope — Agentic Identity Hub
- **Offering today:** Agentic Identity Hub 2.0 (January 26, 2026): a dedicated auth and access-control layer for agents and MCP servers with **ephemeral credentials, tool-level access control, and policy controls** based on roles, JWT claims, tenants, and agent types (verified 3-0). Includes agent logging/auditing with SIEM streaming — overlapping eunox's audit story, though **without any signed/tamper-evident claim** (verified 3-0). Also ships "Comprehensive MCP Auth" for server developers: OAuth 2.1 consent flows, hardened DCR, CIMD, per-agent/per-tool scopes, tenant isolation (verified 3-0).
- **Direction:** Commercial SaaS with a free tier; CEO explicitly markets "least-privilege access for AI agents" — Eunolabs' own language. A 2.5 release with enhanced policy controls followed in June 2026†, i.e., rapid movement deeper into policy enforcement.
- **vs. eunox:** Attribute-driven platform policies vs. eunox's declarative file-based manifests; server-side OAuth scoping vs. eunox's host-side protocol-boundary proxy. Descope's per-tool scopes + JWT-claims policies are conceptually close to the mcp-capability-manifest JWT companion claim.

### Cisco / Astrix, Oasis
- **Cisco announced its intent to acquire Astrix Security** (early May 2026; ~$400M per Calcalist), folding non-human-identity security into Cisco Identity Intelligence, Secure Access, and Duo — explicitly framed around extending zero trust to the "agentic workforce"†. Astrix is posture/detection (visibility, lifecycle, remediation of over-privileged NHI access), not inline enforcement†.
- **Oasis Security** reportedly raised $120M for "agentic access management" (source did not survive quality screening — unconfirmed).
- **Takeaway:** consolidation is real; standalone agent-identity vendors are being absorbed into platform suites, which will bundle "good-enough" agent controls into products enterprises already own.

---

## 6. The MCP spec itself: absorption risk, measured

This is the existential question for a third-party enforcement layer, and the evidence cuts both ways:

**Reassuring (verified):**
- The official 2026 MCP roadmap (March 2026) lists four priority areas — transport evolution/scalability, agent communication (Tasks), governance maturation, enterprise readiness. **Deeper security and authorization work is explicitly deferred** to an "On the Horizon" category where maintainers will only *support a community-formed working group*, not drive it ([MCP blog](https://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/), verified 3-0). Tool-level permissioning / capability scoping is not mentioned at all†.
- The 2026-07-28 release candidate's six authorization SEPs all harden the **OAuth/token layer** (issuer validation per RFC 9207, OIDC application_type, credential binding, refresh tokens, scope accumulation) — not capability manifests or deny-by-default tool policy (verified 3-0). The fine-grained enforcement niche remains unowned by core.

**Threatening (verified):**
- **Enterprise readiness is a top-4 priority**, and maintainers name **audit trails, SSO-integrated auth, gateway behavior, and configuration portability** as the problems to solve — squarely Eunolabs' problem space (verified 3-0). Note: a tempting inference that this work will land only as extensions (not core) was **refuted** in verification — the RC already shipped authorization hardening in core, so don't count on core staying out.
- **Okta's XAA is now the MCP "Enterprise-Managed Authorization" extension** with official SDK support (verified 3-0).
- The RC makes MCP **stateless** (initialize handshake and `Mcp-Session-Id` removed; version/capabilities travel in `_meta` on every request) — eunox must adapt, and interception by generic infrastructure gets easier (verified 3-0). New **`Mcp-Method` / `Mcp-Name` headers let gateways route and filter tool calls without body inspection**, lowering the barrier for Kong/Cloudflare/Solo.io-class vendors to do per-tool policy without MCP-specific engineering (verified 3-0).
- Timeline: RC locked May 21, 2026; final spec **July 28, 2026**; Tier-1 SDK support expected within ten weeks†. SEP-1932 (DPoP) and SEP-1933 (Workload Identity Federation) are in review†.

---

## 7. Summary table

| Competitor | Category | Enforcement point | Deny-by-default | Arg-level / ordering | Listing filtering | Signed audit logs | OSS | Direction |
|---|---|---|---|---|---|---|---|---|
| **eunox (us)** | Capability firewall | MCP protocol boundary (proxy) | ✅ | ✅ | ✅ | ✅ signed, tamper-evident | ✅ Apache-2.0 | Vendor-neutral manifest spec |
| Kong AI Gateway | API gateway | MCP proxy plugin | ❌ allow-by-default† | ❌ per-tool only | ✅ ⚠ | ❌ | ❌ enterprise tier ⚠ | Fast-following MCP spec |
| Pomerium | Agentic gateway | Network path proxy | ? | ❌ tool-level† | ? | ❌ (logs, unsigned) | ✅ | Defining "agentic gateway" category |
| agentgateway (Solo.io) | Agentic gateway | Proxy (MCP + A2A) | ? | ? | ? | ❌ | ✅ | A2A + MCP governance |
| PolicyLayer | Tool-call firewall | Outside LLM loop | ✅ allow-list† | ? | ? | ? | ? | Closest stated value prop† |
| Snyk agent-scan | Scanner | Static / scan-time | n/a | n/a | n/a | n/a | client OSS, backend commercial | Runtime enforcement "in research" |
| Zenity | Runtime guardrails | Agent platform endpoint | rule-based | ❌ content-focused | n/a | ❌ | ❌ | Per-platform coverage (AgentKit, Copilot) |
| Oso for Agents | AuthZ platform | Cloud decision engine, MCP-aware | deny rules | ❌ | ? | ❌ (logs, unsigned) | ❌ platform (lib OSS) | Pivoted flagship to agents |
| Permit.io | AuthZ platform | In-app SDKs (LangChain, MCP) | ? | action-level | n/a | ❌ | partially | Four-perimeter AI pipeline |
| Okta | Identity | Identity/credential layer + XAA | n/a | ❌ | n/a | ❌ | ❌ | **XAA now an official MCP extension** |
| Descope | Identity | Auth layer, per-tool OAuth scopes | ? | ❌ scope-level | n/a | ❌ unsigned | ❌ SaaS | 2.5 in June '26, deeper policy |
| Cisco/Astrix | NHI security | Posture/detection | n/a | n/a | n/a | n/a | ❌ | Absorbed into Cisco suite (~$400M†) |

---

## 8. Assessment: how they compete with eunox, and what to do about it

**Where we're still differentiated (verified across the landscape):**
1. **Deny-by-default at the protocol boundary.** Kong — the nearest competitor — is allow-by-default when unconfigured†. Nobody in the verified set ships fail-closed capability enforcement as the default posture.
2. **Argument-level and call-ordering constraints.** Every competitor's granularity stops at tool-level allow/deny or OAuth scopes. Snyk's "toxic flows" framing† validates ordering as a real problem — but Snyk only *detects* it at scan time; eunox *prevents* it at run time.
3. **Signed, tamper-evident audit logs.** Descope, Oso, and Pomerium all advertise audit logging; none claims cryptographic tamper evidence.
4. **Vendor-neutral, file-based spec.** Everyone else's policy lives inside their platform (Kong Consumers, Polar, Descope policies, Okta grants).

**The three real threats, ranked:**
1. **Standards capture by Okta.** XAA is in the spec, in the SDKs, and has 25+ ecosystem vendors including Anthropic†. If "MCP authorization" becomes synonymous with identity-layer OAuth grants, a capability-manifest spec needs to position as *complementary* (fine-grained, post-authentication enforcement) rather than competing. Concretely: document how mcp-capability-manifest composes with XAA/Enterprise-Managed Authorization tokens — the manifest's JWT scope-narrowing claim is a natural fit to ride that rail rather than fight it.
2. **Commoditization by gateway incumbents.** The `Mcp-Method`/`Mcp-Name` headers make basic per-tool filtering nearly free for any L7 gateway (verified 3-0). Tool-level ACLs will be table stakes within a year; differentiation must live in what headers *can't* express — argument constraints, ordering, manifest portability, signed logs.
3. **Bundling/consolidation.** Cisco/Astrix and Okta-for-AI-Agents show enterprises will get "agent security" checkboxes from suites they already buy. The counter is the same one that worked for OPA and cert-manager: be the neutral, embeddable open standard the suites themselves adopt.

**Near-term to-do signals from the research:**
- **Track the 2026-07-28 spec now** (final lands July 28; SDKs within ~10 weeks†): the stateless redesign (`_meta` on every request, no session header) directly affects eunox's proxy implementation (verified 3-0) — being compatible on day one is a marketing moment, being late is a credibility problem.
- **Engage the "On the Horizon" security/authZ working group.** Maintainers explicitly said they'll support a community-formed WG (verified 3-0). That's an open lane to make mcp-capability-manifest *the* community proposal for fine-grained capability scoping before an incumbent fills it.
- **Watch PolicyLayer** — the only vendor found stating a deterministic-allowlist-outside-the-LLM-loop proposition†, i.e., our pitch in someone else's mouth.

---

## Sources

Primary (verified against): [github.com/snyk/agent-scan](https://github.com/snyk/agent-scan) · [osohq.com](https://www.osohq.com/) · [Okta platform innovation release](https://www.okta.com/newsroom/press-releases/okta-platform-innovation/) · [Okta Showcase 2026 release](https://www.okta.com/newsroom/press-releases/showcase-2026/) · [Descope Agentic Identity Hub 2.0](https://www.descope.com/press-release/agentic-identity-hub-2.0) · [MCP 2026 roadmap](https://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/) · [MCP 2026-07-28 release candidate](https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/) · [Kong MCP Tool ACLs](https://konghq.com/blog/product-releases/mcp-tool-acls-ai-gateway)

Secondary/context (†): [Pomerium: top agentic gateways 2026](https://www.pomerium.com/blog/top-5-agentic-gateways-for-securing-mcp-tool-calls-in-2026) · [e2b-dev/awesome-mcp-gateways](https://github.com/e2b-dev/awesome-mcp-gateways) · [Snyk Labs: guardrails for agentic AI](https://labs.snyk.io/resources/guardrails-agentic-ai-mcp-aibom/) · [Zenity: runtime protection for AgentKit](https://zenity.io/blog/product/closing-the-guardrail-gap-runtime-protection-for-openai-agentkit) · [Permit.io AI Access Control](https://www.permit.io/blog/announcing-permit-ai-access-control-ai-identity-fga) · [SecurityWeek: Cisco to acquire Astrix](https://www.securityweek.com/cisco-moves-to-acquire-astrix-security-to-tackle-non-human-identity-risks/)
