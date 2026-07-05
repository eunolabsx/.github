# Security Policy

Eunolabs builds security-enforcement software for the Model Context Protocol.
Getting vulnerabilities fixed quickly and disclosed responsibly is part of the
job, so we take every report seriously.

This is the **organization-wide default** policy. Some repositories ship their
own `SECURITY.md` with product-specific scope, supported-version tables, and
release-verification steps — that repository's policy takes precedence over this
one. When in doubt, read the policy in the repository you are reporting against.

## Reporting a vulnerability

**Please do not open a public GitHub issue, pull request, or discussion for a
security problem.** Public disclosure before a fix is available puts users at
risk.

Use one of these private channels instead:

1. **GitHub Private Vulnerability Reporting (preferred).** Go to the affected
   repository, open the **Security** tab, and choose **Report a vulnerability**.
   This opens an end-to-end private advisory thread between you and the
   maintainers, tracks the report through coordinated disclosure, and lets us
   request a CVE on your behalf.

2. **Email.** If you cannot use GitHub's reporting flow, email
   **security@eunolabs.ai**. Put the affected repository name in the subject
   line so we can route it quickly.

A good report includes:

- The repository, and the version, release tag, or commit SHA you tested.
- A minimal, self-contained reproduction — the smallest input and steps that
  demonstrate the issue.
- Your assessment of impact: what an attacker gains, and what preconditions the
  attack requires.
- Whether you intend to publish, and any timeline constraints on your side.

## What to expect

- **Acknowledgement within 2 business days** that we received your report.
- An initial assessment, and a request for anything else we need to reproduce
  it, shortly after.
- A target of a published advisory and patched release within **30 days** for
  high-severity issues and **90 days** for everything else. If a fix needs
  longer, we will tell you and explain why.

We will keep you updated as we investigate, and we credit reporters in the
published advisory unless you prefer to remain anonymous.

## Coordinated disclosure

We follow standard coordinated-disclosure practice:

1. You report privately through one of the channels above.
2. We confirm, investigate, and prepare a fix.
3. We agree a disclosure date with you — typically when the patched release
   ships.
4. We publish an advisory (with a CVE where applicable) and release the fix.

## Safe harbor

We will not pursue legal action against, or report to law enforcement,
researchers who:

- Make a good-faith effort to avoid privacy violations, data destruction, and
  service disruption.
- Report through the private channels above **before** any public disclosure,
  and give us reasonable time to remediate.
- Access only the minimum data necessary to demonstrate the issue, and do not
  exfiltrate data, pivot to other systems, or attack third parties.

If your good-faith research would otherwise violate the project's license or
applicable law, we authorize it for the limited purpose of finding and reporting
security issues under the conditions above.

## Out of scope

Reports about the following generally fall outside what we can act on, though
you are welcome to send them anyway if you are unsure:

- Vulnerabilities in third-party software we merely integrate with (upstream MCP
  servers, MCP hosts, identity providers) — those belong to the respective
  vendor.
- Findings that require an attacker to already control the host, the
  configuration, or the signing keys.
- Automated-scanner output with no demonstrated, exploitable impact.

Thank you for helping keep the Eunolabs ecosystem and its users safe.
