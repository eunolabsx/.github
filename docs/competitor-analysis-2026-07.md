# Competitive Landscape: eunox & mcp-capability-manifest

**Date:** July 2026
**Scope:** MCP security gateways/proxies, agent action-layer runtimes, guardrails & scanners, authorization platforms, and identity vendors moving into agent authorization — what they offer today, where they're heading, and how they compete with Eunolabs' protocol-boundary, deny-by-default capability enforcement.

**Method note.** Findings were gathered from primary sources (vendor announcements, official docs, the MCP blog). Most passed adversarial 3-vote verification (22 of 25 top claims confirmed, 1 refuted, 2 unverified — marked ⚠ where relied upon). Claims from the broader extraction pass that were not individually verified are marked †. The Arcade.dev section was added in a follow-up research pass from primary vendor sources and press coverage (cited inline, single-pass — not adversarially verified).

---

## 1. Executive summary

The market Eunolabs plays in has gone from empty to crowded in roughly eighteen months. A widely referenced community catalog now lists **~47 distinct MCP gateway products (~20 open-source, ~27 commercial)**†. Four distinct groups are converging on "control what an AI agent can do":

1. **API-gateway incumbents** (Kong, Solo.io, Cloudflare, Pomerium) bundling per-tool MCP ACLs into existing gateways.
2. **Agent action-layer runtimes** (Arcade.dev) that execute tools inside their own platform and enforce per-user OAuth delegation at the point of execution.
3. **Authorization platforms** (Oso, Permit.io) repositioning their policy engines as "AI agent access control."
4. **Identity vendors** (Okta, Descope; Cisco via the Astrix acquisition) anchoring agent control at the identity/credential layer — and, in Okta's case, successfully pushing their approach *into the MCP spec itself*.

**No verified competitor currently combines eunox's specific bundle**: deny-by-default enforcement, argument-level and call-ordering constraints, listing filtering, *signed tamper-evident* audit logs, a vendor-neutral file-based manifest spec, and a single open-source static binary. But every individual piece of that bundle now has at least one well-funded competitor, and the biggest strategic threats are (a) standards absorption — Okta's Cross App Access is now an official MCP authorization extension and Arcade co-authored an accepted authorization SEP with Anthropic — and (b) platform gravity, where enforcement gets bundled into runtimes and suites the customer already owns.

Section 9 sets out the counter-strategy, including the current architectural decision to **delegate token minting to existing authZ/identity platforms** and position eunox as the vendor-neutral enforcement point beneath them.

---

## 2. MCP gateways & proxies (direct competitors)

### Kong AI Gateway — the most direct feature overlap

- **Offering today:** Kong AI Gateway 3.13 (announced January 2026) introduced **MCP Tool ACLs** via its AI MCP Proxy plugin: allow/deny rules at a default level (all tools) plus per-tool overrides, keyed to authenticated Kong Consumers and Consumer Groups. ([Kong blog](https://konghq.com/blog/product-releases/mcp-tool-acls-ai-gateway), verified 3-0.) It also filters `tools/list` responses per identity so agents never see restricted tools (⚠ verification incomplete; 1-0 in favor).
- **How it works:** policy lives in Kong's control plane as configuration attached to Kong-specific *Consumer* objects. Conceptually:

  ```
  # Kong AI MCP Proxy ACLs (conceptual)
  consumer_group: data-analysts
    default_acl: allow            # ← allow-by-default when unconfigured†
    tools:
      delete_records: deny
  ```

  versus the eunox model, where the policy is a portable file with argument and ordering constraints:

  ```yaml
  # eunox capability manifest (illustrative)
  default: deny                   # ← fail closed, always
  tools:
    query_db:
      allow: true
      args:
        statement: { pattern: "^SELECT " }
    send_email:
      allow: true
      not_after: [read_secrets]   # ordering constraint: no exfil flows
  ```

  The difference in *where the policy lives* (platform object vs. reviewable file) and *what it can express* (tool-level identity ACLs vs. argument/ordering constraints) is the whole competitive story in miniature.
- **Direction:** Actively tracking the MCP spec — in 3.14 it changed ACL denial responses from JSON-RPC errors to HTTP 403 to match the MCP 2025-11-25 authorization spec†. Kong extended into MCP with plugins and a registry service across AI Gateway and Konnect in early 2026†.
- **vs. eunox:** Closest head-to-head competitor at the protocol boundary. Kong's MCP ACLs are **allow-by-default when no rules are configured**† (opposite of eunox), sit in the **commercial enterprise tier** (⚠ unverified), stop at tool granularity, and cannot reach local stdio MCP servers. See §9.

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

## 3. Agent action-layer runtimes: Arcade.dev

Arcade is not a gateway and not an identity provider — it is a third architecture: a **runtime that executes agent tools inside its own platform** and enforces authorization at the point of execution. It is the best-funded company in eunox's immediate neighborhood and the most standards-embedded, which makes it the most dangerous *narrative* competitor even though its architecture differs from ours.

### What they offer today

- **Product:** ["The MCP runtime for production AI agents"](https://www.arcade.dev/) — user authentication, authorization, policy enforcement, and tool execution in one managed layer, plus a large pre-built tool catalog (marketed as 7,500+ integrations) and enterprise governance (SOC 2, GDPR/CCPA, audit trails).
- **Security model:** agents act **as the authenticated end user through delegated OAuth** — never bot tokens or service accounts. Arcade's Engine brokers the OAuth flow with the service provider, manages refresh, scopes each call, and injects tokens at runtime so **the client and the LLM never see credentials** ([Arcade docs](https://docs.arcade.dev/en/guides/create-tools/tool-basics/create-tool-auth)).
- **How it works in practice** — a tool built on their open-source [`arcade-mcp`](https://github.com/arcadeai/arcade-mcp) Python framework declares its auth requirement in code:

  ```python
  from arcade_mcp_server import MCPApp, Context
  from arcade_mcp_server.auth import GitHub

  app = MCPApp(name="gh", version="1.0.0")

  @app.tool(requires_auth=GitHub(scopes=["repo"]))
  async def list_my_repos(context: Context) -> list[str]:
      """List the authenticated user's GitHub repositories."""
      token = context.get_auth_token_or_empty()
  ```

  If the user hasn't granted the required scopes, the Engine automatically interrupts the tool call, runs the OAuth consent flow with the provider, then resumes execution. 22 OAuth providers are built in, plus a generic `OAuth2` class.
- **Licensing/deployment split (the open-core seam):** the tool-building framework is open source and runs standalone over stdio/HTTP with self-managed tokens — but **the authorization value (user OAuth flows, token refresh, per-call scoping) requires the Arcade Engine**, delivered as Arcade Cloud (`arcade deploy`) or a [self-hosted Engine for enterprise customers](https://docs.arcade.dev/en/guides/deployment-hosting/configure-engine). The security layer is the commercial product.

### Direction

- **Funding:** [$60M Series A (June 15, 2026)](https://www.businesswire.com/news/home/20260615229631/en/Arcade-Raises-$60M-to-Become-the-Secure-Action-Layer-Behind-Every-Production-AI-Agent) led by SYN Ventures with strategic investment from Morgan Stanley and Wipro — $72M total. Stated ambition: "the secure action layer behind every production AI agent," proving **which agent took which action on behalf of which user**.
- **Standards:** working with Anthropic, Arcade [authored the URL Elicitation SEP accepted into the MCP spec](https://www.businesswire.com/news/home/20251125080912/en/Arcade.dev-Authors-Core-MCP-Capability-Unlocking-Secure-AI-Agents-at-Scale) (November 2025) — a standardized secure flow for tool authorization consent — and claims seats on the MCP security and governance steering committees. Note where their standards footprint sits: the **OAuth/consent layer**, consistent with the broader pattern (§7) that the spec is absorbing authorization at the token layer while the capability-enforcement layer remains unclaimed.
- **Distribution:** tools surfaced through ecosystem channels (e.g., LangSmith Fleet), pushing the catalog as the default way agents get capabilities.

### Overlap map vs. eunox

| Dimension | Arcade | eunox |
|---|---|---|
| Enforcement point | Inside their runtime, at tool execution | Protocol boundary, in front of *any* MCP server |
| Covers third-party/existing MCP servers | ❌ only tools in their catalog or rebuilt on their framework | ✅ any server, unmodified (local subprocess or remote HTTPS) |
| Local stdio servers on dev machines | ❌ (Engine-mediated model) | ✅ |
| Identity model | Per-user delegated OAuth (strong) | Consumes external tokens; JWT capability claim (integration in progress) |
| Granularity | OAuth scopes (provider-defined vocabulary) | Tool + argument + call-ordering constraints |
| Default posture | Consent-driven per user; no static ceiling artifact | Deny-by-default manifest ceiling, git-reviewable |
| Audit | "Complete audit trail," SOC 2 | Signed, tamper-evident logs |
| Openness | Open-source framework, commercial Engine (auth requires it) | Fully open-source binary + vendor-neutral spec |
| Standards | Authored accepted MCP SEP (consent flow); steering committee seats | mcp-capability-manifest (not yet in MCP process) |

**The buyer-conversation overlap is high** — least privilege, audit, "prove what the agent did" — even though the architectural overlap is moderate. Their delegated-OAuth model also does not answer the confused-deputy problem: a prompt-injected agent misusing access the user *legitimately delegated* is fully authorized in Arcade's model. Scope-granularity is capped by what each provider's OAuth scopes can express — nothing like argument constraints or ordering rules.

### Dual nature: rail and threat

Because eunox's strategy is to **rely on existing authZ platforms to mint tokens** (§9), Arcade is two things at once:

- **As a rail:** Arcade is a token broker. An Arcade-brokered credential could carry the mcp-capability-manifest claim like any other issuer's, and eunox can sit in front of Arcade-hosted MCP endpoints as the customer-side ceiling. An Arcade minting recipe belongs next to the Okta/Auth0/Descope ones.
- **As a threat:** unlike pure identity vendors, Arcade *enforces and executes* — every tool that moves into their runtime is a tool where the enforcement question is answered before eunox arrives. And they are already inside the MCP steering rooms where capability scoping will eventually be discussed.

---

## 4. Guardrails & scanners (adjacent, mostly non-overlapping)

### Snyk agent-scan (Invariant Labs lineage)

- **Offering today:** A **static scanner, not a runtime enforcement proxy** — inventories installed agent components (harnesses, MCP servers, skills) and scans for 15+ risk categories: prompt injection, tool poisoning, tool shadowing, malware payloads, credential handling, hardcoded secrets. ([github.com/snyk/agent-scan](https://github.com/snyk/agent-scan), verified 3-0.) Client is Apache-2.0 but **scans require a Snyk account/API token** — open-core with a commercial backend (verified 3-0). Active and adopted: v0.5.12 released June 23, 2026, ~2.7k stars†; scans configs of Claude Desktop/Code, Cursor, Windsurf, Gemini CLI, and others†.
- **Direction:** Snyk explicitly describes runtime enforcement as **still an area of active research**† — its strategy is visibility (AI-BOM inventory + MCP scanning), and it frames the core MCP risk as "toxic flows" (unsafe combinations of tools, permissions, and data paths)† — the same problem class eunox's call-ordering constraints address at runtime.
- **vs. eunox:** Complementary more than competitive today (scan-time vs. enforcement-time). The threat is that Snyk graduates from research into runtime enforcement with an installed enterprise base.

### Zenity

- **Offering today:** Runtime protection for OpenAI AgentKit agents (November 2025) — deterministic rule-based policies enforced "at the endpoint," detecting/blocking data leakage, secret exposure, and unsafe responses†.
- **Direction:** Per-platform runtime coverage of major agent-building ecosystems (AgentKit, Microsoft Copilot Studio, ChatGPT Enterprise)†.
- **vs. eunox:** Different enforcement point (agent platform/endpoint, content-focused) — not tool-call capability allowlisting, argument constraints, or sequence enforcement. Adjacent, not head-on.

### Prompt Security, Knostic

Both are active in agentic-AI security marketing, but no claims from their materials survived source-quality screening in this research pass. Watch-list.

---

## 5. Authorization platforms (repositioning into agents)

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

## 6. Identity vendors moving into agent authorization

### Okta — the biggest standards threat

- **Offering today:** "Okta for AI Agents" — a commercial platform to **discover/register known and unknown agents, standardize agent access, and instantly revoke access** — generally available **April 30, 2026** ([Showcase 2026 release](https://www.okta.com/newsroom/press-releases/showcase-2026/), verified 3-0). Okta Privileged Access enforces policies for agents using static credentials (verified 3-0). Its "blueprint for the secure agentic enterprise" claims identity as the control plane for *where are my agents, what can they connect to, and what can they do* — overlapping Eunolabs' core question, anchored at the identity layer (verified 3-0).
- **Standards play (critical):** Okta's **Cross App Access (XAA)** has been **incorporated into MCP as the "Enterprise-Managed Authorization" extension, with support landing in official MCP SDKs (TypeScript and Java first)** (verified 3-0). Okta claims an XAA ecosystem of **25+ software makers including Anthropic (Claude), Atlassian, Slack, Cloudflare, and Datadog** as of June 2026†.
- **vs. eunox:** Okta does not (per these releases) do argument-level or call-sequence enforcement at the MCP wire protocol — its granularity is agent identity, access grants, and revocation. Under the delegated-minting strategy (§9), Okta is a *supplier* of tokens eunox consumes; the risk is mindshare, not architecture.

### Descope — Agentic Identity Hub

- **Offering today:** Agentic Identity Hub 2.0 (January 26, 2026): a dedicated auth and access-control layer for agents and MCP servers with **ephemeral credentials, tool-level access control, and policy controls** based on roles, JWT claims, tenants, and agent types (verified 3-0). Includes agent logging/auditing with SIEM streaming — overlapping eunox's audit story, though **without any signed/tamper-evident claim** (verified 3-0). Also ships "Comprehensive MCP Auth" for server developers: OAuth 2.1 consent flows, hardened DCR, CIMD, per-agent/per-tool scopes, tenant isolation (verified 3-0).
- **Direction:** Commercial SaaS with a free tier; CEO explicitly markets "least-privilege access for AI agents" — Eunolabs' own language. A 2.5 release with enhanced policy controls followed in June 2026†, i.e., rapid movement deeper into policy enforcement.
- **vs. eunox:** Attribute-driven platform policies vs. eunox's declarative file-based manifests; server-side OAuth scoping vs. eunox's host-side protocol-boundary proxy. Descope's per-tool scopes + JWT-claims policies are conceptually close to the mcp-capability-manifest JWT companion claim — making Descope a natural early minting partner (§9).

### Cisco / Astrix, Oasis

- **Cisco announced its intent to acquire Astrix Security** (early May 2026; ~$400M per Calcalist), folding non-human-identity security into Cisco Identity Intelligence, Secure Access, and Duo — explicitly framed around extending zero trust to the "agentic workforce"†. Astrix is posture/detection (visibility, lifecycle, remediation of over-privileged NHI access), not inline enforcement†.
- **Oasis Security** reportedly raised $120M for "agentic access management" (source did not survive quality screening — unconfirmed).
- **Takeaway:** consolidation is real; standalone agent-identity vendors are being absorbed into platform suites, which will bundle "good-enough" agent controls into products enterprises already own.

---

## 7. The MCP spec itself: absorption risk, measured

This is the existential question for a third-party enforcement layer, and the evidence cuts both ways:

**Reassuring (verified):**

- The official 2026 MCP roadmap (March 2026) lists four priority areas — transport evolution/scalability, agent communication (Tasks), governance maturation, enterprise readiness. **Deeper security and authorization work is explicitly deferred** to an "On the Horizon" category where maintainers will only *support a community-formed working group*, not drive it ([MCP blog](https://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/), verified 3-0). Tool-level permissioning / capability scoping is not mentioned at all†.
- The 2026-07-28 release candidate's six authorization SEPs all harden the **OAuth/token layer** (issuer validation per RFC 9207, OIDC application_type, credential binding, refresh tokens, scope accumulation) — not capability manifests or deny-by-default tool policy (verified 3-0). Arcade's accepted SEP likewise standardizes the *consent flow*, not capability constraints. The fine-grained enforcement niche remains unowned by core.

**Threatening (verified):**

- **Enterprise readiness is a top-4 priority**, and maintainers name **audit trails, SSO-integrated auth, gateway behavior, and configuration portability** as the problems to solve — squarely Eunolabs' problem space (verified 3-0). Note: a tempting inference that this work will land only as extensions (not core) was **refuted** in verification — the RC already shipped authorization hardening in core, so don't count on core staying out.
- **Okta's XAA is now the MCP "Enterprise-Managed Authorization" extension** with official SDK support (verified 3-0), and **Arcade holds MCP security/governance steering seats** — the authorization conversation inside the spec process is currently led by vendors whose answer is tokens and consent, not capabilities.
- The RC makes MCP **stateless** (initialize handshake and `Mcp-Session-Id` removed; version/capabilities travel in `_meta` on every request) — eunox must adapt, and interception by generic infrastructure gets easier (verified 3-0). New **`Mcp-Method` / `Mcp-Name` headers let gateways route and filter tool calls without body inspection**, lowering the barrier for Kong/Cloudflare/Solo.io-class vendors to do per-tool policy without MCP-specific engineering (verified 3-0).
- Timeline: RC locked May 21, 2026; final spec **July 28, 2026**; Tier-1 SDK support expected within ten weeks†. SEP-1932 (DPoP) and SEP-1933 (Workload Identity Federation) are in review†.

---

## 8. Summary table

| Competitor | Category | Enforcement point | Deny-by-default | Arg-level / ordering | Listing filtering | Signed audit logs | OSS | Direction |
|---|---|---|---|---|---|---|---|---|
| **eunox (us)** | Capability firewall | MCP protocol boundary (proxy) | ✅ | ✅ | ✅ | ✅ signed, tamper-evident | ✅ Apache-2.0 | Vendor-neutral manifest spec; token-claim integration in progress |
| Kong AI Gateway | API gateway | MCP proxy plugin | ❌ allow-by-default† | ❌ per-tool only | ✅ ⚠ | ❌ | ❌ enterprise tier ⚠ | Fast-following MCP spec |
| Arcade.dev | Action-layer runtime | Inside own runtime, at execution | consent-driven, no static ceiling | ❌ OAuth scopes only | n/a (own catalog) | ❌ (audit trail, unsigned) | framework OSS, Engine commercial | $60M Series A; authored accepted MCP SEP; steering seats |
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

## 9. How eunox counters

### 9.0 Architectural premise (and an honest constraint)

The strategy below assumes the **two-layer model**: the manifest is the *ceiling* (static, file-based, git-reviewable, deny-by-default), and per-invocation tokens can only *narrow* it, never broaden it. **Constraint to be honest about:** the token/capability-claim side of this is not yet fully supported or integrated in eunox. The decision is to **not build an issuer** — token minting is delegated to existing authZ/identity platforms (Okta, Auth0, Descope, Keycloak, and potentially Arcade) — and eunox verifies and intersects. This is the classic PEP/PDP split (OPA didn't mint; Envoy didn't decide), it keeps eunox a stateless static binary with no key ceremony, and it converts the identity vendors from threats into suppliers. Until the integration ships, **messaging leads only with what is true today** (see 9.4).

### 9.1 Reframe the fight: identity says *who*, capabilities say *what*

Identity-keyed tool ACLs (Kong) and delegated per-user OAuth (Arcade) both answer "who is calling." The dominant agent threat is different: a **prompt-injected or confused agent misusing access it legitimately holds**. An identity-keyed ACL happily lets the right consumer call `send_email`; Arcade's consent flow happily executes what the user once delegated. Neither can express:

- **Argument constraints** — `query_db` allowed, but only `SELECT`, only these schemas.
- **Ordering constraints** — block "read secrets → call external webhook" toxic flows. Snyk's own research names toxic flows as *the* MCP risk class† but only detects them at scan time; Kong can't see them; Arcade's scope model can't express them. eunox prevents them at runtime.

Per-tool allow/deny is about to be commoditized anyway — the new `Mcp-Method`/`Mcp-Name` headers make it nearly free for any L7 gateway (verified 3-0). Do not compete on it. Compete on what headers, ACLs, and OAuth scopes cannot express.

### 9.2 Countering Kong (feature incumbency)

Press four verified/reported weaknesses:

1. **Allow-by-default.** Kong's ACL evaluation permits everything when unconfigured†; eunox fails closed. "Forgot to configure = wide open" is the centerpiece of every comparison asset.
2. **Platform-locked policy.** Kong ACLs are control-plane objects bound to Kong Consumers; eunox policy is a portable YAML file — PR-reviewable, diffable, portable to any conforming enforcer, with the spec as the exit guarantee.
3. **No reach into local stdio.** A network gateway cannot sit in front of subprocess MCP servers on developer machines (Claude Desktop/Code, Cursor) — a segment eunox covers natively. Own it.
4. **Unsigned logs, enterprise tier.** Signed tamper-evident audit vs. ordinary logging (⚠ tier claim pending verification); open source vs. paid enterprise plugin.

Kong's spec-conformance habit (it reshaped ACL denials to match MCP authorization semantics within one release†) is exploitable: if the capability manifest becomes a community spec, Kong's own instincts pressure it to implement *our* format — see 9.5.

### 9.3 Countering Arcade (platform gravity)

Arcade's model is "bring your tools into our secure house." eunox's is "we guard the door of whatever house you already have." Concretely:

1. **Coverage asymmetry.** Arcade secures tools in its catalog or rebuilt on its framework, mediated by its Engine. eunox governs **existing, unmodified, third-party MCP servers** — local or remote — with zero migration. Most enterprises' MCP estate is exactly that: servers they didn't write and won't rebuild.
2. **The ceiling they don't have.** Arcade has consent trails; it has no static, reviewable, deny-by-default policy artifact. "What is this agent *ever* allowed to do, show me the file, show me who approved the diff" is a question only the manifest answers — and it's the question compliance asks.
3. **Confused-deputy gap.** Delegated OAuth is authorization *of the user's access*, not control *of the agent's behavior within it*. eunox's argument/ordering constraints are the control that survives prompt injection; Arcade's granularity is capped at provider-defined OAuth scopes.
4. **Co-opt, don't collide.** Treat Arcade as an issuer: publish a minting recipe so Arcade-brokered credentials can carry the capability claim, and document eunox-in-front-of-Arcade-endpoints as a supported topology. Every tool that runs on Arcade *with an eunox ceiling in front of it* is a win, not a loss.
5. **Respect the standards clock.** Their accepted SEP proves small companies can land authorization SEPs. That validates — and adds urgency to — 9.5.

### 9.4 Messaging discipline until the claim integration ships

Lead only with what is shipped and verified today: **deny-by-default posture; portable git-reviewable manifests; argument and ordering enforcement; listing filtering; local + remote server coverage; signed tamper-evident audit logs; single OSS binary.** Frame token-based narrowing as **"works with the authZ platform you already have"** — a roadmap architecture, not a shipped claim. When the integration lands, the identity story arrives as an *and*: Kong makes you move identity into its control plane, Arcade makes you move tools into its runtime — eunox meets your issuer and your servers where they already are.

### 9.5 Sequencing

1. **Spec first, code second.** Nail intersection semantics in mcp-capability-manifest now: how a claim narrows a manifest; absent/malformed claims fail closed; issuer/audience validation requirements. Cheap to write, and it's what issuers need to see before minting the claim.
2. **Ship the verification path** (JWKS fetch, `iss`/`aud` validation, claim→manifest intersection). Small, high-leverage, verification-only — no minting complexity.
3. **Publish minting recipes** for Okta (including XAA/Enterprise-Managed Authorization tokens carrying the claim — ride their rail), Auth0, Descope, Keycloak, and Arcade. Custom claims cost issuers nothing; every recipe entrenches the format as the neutral one and pre-empts each platform inventing its own vocabulary.
4. **Be day-one compatible with the 2026-07-28 spec** (final lands July 28; Tier-1 SDKs within ~10 weeks†). The stateless redesign (`_meta` on every request, no session header) directly affects the proxy (verified 3-0). On-time is a marketing moment; late is a credibility problem.
5. **Form or join the MCP security/authZ working group.** Maintainers explicitly said they'll support a community-formed WG (verified 3-0), the capability-scoping lane is empty (verified 3-0), and both Okta and Arcade are already in adjacent rooms. Make mcp-capability-manifest *the* community proposal for fine-grained capability scoping before an incumbent does — then compete as spec steward and reference implementation, not as one gateway among ~47.
6. **Watch PolicyLayer** — the only vendor found stating a deterministic-allowlist-outside-the-LLM-loop proposition†, i.e., our pitch in someone else's mouth.

---

## Sources

Primary (verified against): [github.com/snyk/agent-scan](https://github.com/snyk/agent-scan) · [osohq.com](https://www.osohq.com/) · [Okta platform innovation release](https://www.okta.com/newsroom/press-releases/okta-platform-innovation/) · [Okta Showcase 2026 release](https://www.okta.com/newsroom/press-releases/showcase-2026/) · [Descope Agentic Identity Hub 2.0](https://www.descope.com/press-release/agentic-identity-hub-2.0) · [MCP 2026 roadmap](https://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/) · [MCP 2026-07-28 release candidate](https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/) · [Kong MCP Tool ACLs](https://konghq.com/blog/product-releases/mcp-tool-acls-ai-gateway)

Arcade.dev (single-pass, primary/vendor + press): [arcade.dev](https://www.arcade.dev/) · [Arcade docs — tool auth](https://docs.arcade.dev/en/guides/create-tools/tool-basics/create-tool-auth) · [Arcade docs — self-hosted Engine config](https://docs.arcade.dev/en/guides/deployment-hosting/configure-engine) · [arcade-mcp on GitHub](https://github.com/arcadeai/arcade-mcp) · [BusinessWire — $60M Series A](https://www.businesswire.com/news/home/20260615229631/en/Arcade-Raises-$60M-to-Become-the-Secure-Action-Layer-Behind-Every-Production-AI-Agent) · [PYMNTS — funding](https://www.pymnts.com/news/investment-tracker/2026/arcade-raises-60-million-to-control-ai-agents/) · [BusinessWire — MCP SEP authorship](https://www.businesswire.com/news/home/20251125080912/en/Arcade.dev-Authors-Core-MCP-Capability-Unlocking-Secure-AI-Agents-at-Scale) · [SiliconANGLE — Arcade + Anthropic](https://siliconangle.com/2025/11/25/arcade-dev-anthropic-advance-mcp-new-secure-authorization-flow/) · [WorkOS — What is Arcade.dev](https://workos.com/blog/what-is-arcade-dev-an-llm-tool-calling-platform)

Secondary/context (†): [Pomerium: top agentic gateways 2026](https://www.pomerium.com/blog/top-5-agentic-gateways-for-securing-mcp-tool-calls-in-2026) · [e2b-dev/awesome-mcp-gateways](https://github.com/e2b-dev/awesome-mcp-gateways) · [Snyk Labs: guardrails for agentic AI](https://labs.snyk.io/resources/guardrails-agentic-ai-mcp-aibom/) · [Zenity: runtime protection for AgentKit](https://zenity.io/blog/product/closing-the-guardrail-gap-runtime-protection-for-openai-agentkit) · [Permit.io AI Access Control](https://www.permit.io/blog/announcing-permit-ai-access-control-ai-identity-fga) · [SecurityWeek: Cisco to acquire Astrix](https://www.securityweek.com/cisco-moves-to-acquire-astrix-security-to-tackle-non-human-identity-risks/)
