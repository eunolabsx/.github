# Contributing to Eunolabs projects

Thanks for considering a contribution. This is the **organization-wide default**
guide; it covers the baseline that applies across Eunolabs repositories. Several
repositories also ship their own `CONTRIBUTING.md` with project-specific build,
test, and review requirements — **when a repository has its own guide, it takes
precedence over this one.** Always read it first.

## Before you start

- **Security vulnerabilities do not go through public issues or pull requests.**
  Follow [`SECURITY.md`](./SECURITY.md) and report privately.
- **Search existing issues and pull requests** before opening a new one, to
  avoid duplicates.
- **Discuss large or user-visible changes first.** For anything substantive — a
  new feature, a change to a public contract, or a breaking change — open an
  issue (or a draft PR with a sketch) describing the use case before writing a
  lot of code. Small fixes, typo corrections, and dependency bumps can go
  straight to a PR.
- By participating you agree to abide by our
  [Code of Conduct](./CODE_OF_CONDUCT.md).

## Pull request workflow

1. **Fork** the repository and create a topic branch off the default branch. A
   short, descriptive slug is fine (`fix/typo-in-readme`,
   `feat/new-condition-type`).
2. **Keep each PR focused on a single change.** A refactor mixed with a behavior
   change is much harder to review and to roll back — split them.
3. **Add tests** for new behavior and bug fixes, and run the repository's test
   and lint targets locally before pushing.
4. **Update the docs** in the same PR when you change behavior, configuration,
   or a public interface.
5. **Open the PR** against the default branch and fill in the pull-request
   template. Explain *why*, not just *what*.

## Commit conventions

- Use [Conventional Commits](https://www.conventionalcommits.org/) prefixes
  (`feat:`, `fix:`, `docs:`, `test:`, `ci:`, `chore:`, and `sec:` for security
  hardening). Add `!` for a breaking change and describe the migration in the
  body.
- **Sign off every commit** to certify the
  [Developer Certificate of Origin](https://developercertificate.org/):

  ```bash
  git commit -s -m "feat: ..."
  ```

  We use a DCO sign-off, not a CLA.

## Licensing of contributions

Eunolabs projects are open source (see each repository's `LICENSE`). By
submitting a contribution you agree it is provided under that repository's
license, including any patent grant it contains. Do not contribute material you
cannot license this way.

## Getting help

Questions about using a project, rather than contributing to it, belong in
[`SUPPORT.md`](./SUPPORT.md).
