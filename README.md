# .github

Organization-wide defaults for the Eunolabs GitHub organization.

GitHub falls back to the community health files in this repository for any
Eunolabs repository that does not define its own. These are **defaults, not
overrides** — a repository's own `SECURITY.md`, issue templates,
`CONTRIBUTING.md`, and so on always take precedence.

## What's here

| File | Purpose |
| --- | --- |
| [`SECURITY.md`](./SECURITY.md) | How to report a vulnerability privately. |
| [`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md) | Community standards (Contributor Covenant 2.1). |
| [`CONTRIBUTING.md`](./CONTRIBUTING.md) | Baseline contribution workflow. |
| [`SUPPORT.md`](./SUPPORT.md) | Where to get help. |
| [`.github/ISSUE_TEMPLATE/`](./.github/ISSUE_TEMPLATE) | Default bug-report and feature-request templates plus the chooser config. |
| [`.github/PULL_REQUEST_TEMPLATE.md`](./.github/PULL_REQUEST_TEMPLATE.md) | Default pull-request template. |

## How defaults resolve

For any repository in the organization, GitHub uses a health file from this
repository only when that repository has no file of the same type. To customize
the experience for a specific project, add the file directly to that project's
repository — it will shadow the default here.

See GitHub's documentation on
[creating a default community health file](https://docs.github.com/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file).
