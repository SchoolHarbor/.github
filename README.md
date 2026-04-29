# SchoolHarbor `.github`

Org-level community health files and reusable workflows.

## Reusable workflows

### `supply-chain-audit.yml` — proof of concept (disabled)

> **Status:** PoC. The org ruleset that referenced this workflow is currently `enforcement: disabled` pending a decision on a broader off-the-shelf tool (Socket.dev / Snyk / OSV-Scanner / GitHub Dependency Review). The file and ruleset config are preserved so the wiring can be revived by flipping the ruleset back to `active`.

Fails CI if any JavaScript dependency in the consuming repo has an npm advisory at or above a configurable severity. Defaults to `critical` (so it catches the postcss-worm class of attack — those packages are tagged critical — without blocking on accumulated high-severity tech debt).

Skipped (with a `::notice::`) on repos without a `pnpm-lock.yaml`, so it's safe to enforce against non-JS repos.

#### Primary: org-level enforcement

Enforced across every repo in the org via a Repository Ruleset's `workflows` rule, triggered on `pull_request` and `merge_group`. Failed audits block PR merge. Per-repo wiring is not required; new repos pick up the check automatically.

The org rule always runs at the default `audit-level: critical` because ruleset-injected workflows can't pass inputs.

#### Opt-in: same-run `needs:` gate

A repo that wants the audit gate to also block subsequent jobs inside its own CI run (defense-in-depth above the org rule) can call this workflow via `uses:` and override the threshold:

```yaml
jobs:
  supply-chain:
    uses: SchoolHarbor/.github/.github/workflows/supply-chain-audit.yml@main
    with:
      audit-level: high # tighter than the org default
  build:
    needs: supply-chain
    # ...
```

The org rule already blocks merge of poisoned PRs, so for most repos this opt-in mode isn't worth the per-repo wiring drift.

#### Caveat

This workflow is **reactive** — it relies on npm's advisory database, which is updated _after_ a malicious package is detected. For behavioral detection of install-script abuse and tampered-package signatures (the genuine zero-day defense), layer [Socket.dev](https://socket.dev) or [Snyk](https://snyk.io) on top.
