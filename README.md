# SchoolHarbor `.github`

Org-level community health files and reusable workflows.

## Reusable workflows

### `supply-chain-audit.yml`

Fails CI if any JavaScript dependency in the consuming repo has an npm advisory at or above a configurable severity. Defaults to `critical` (so it catches the postcss-worm class of attack — those packages are tagged critical — without blocking on accumulated high-severity tech debt). Wire it as a `needs:` dependency for every other CI job:

```yaml
jobs:
  supply-chain:
    uses: SchoolHarbor/.github/.github/workflows/supply-chain-audit.yml@main
    # Optional: tighten the threshold for repos with clean audits
    # with:
    #   audit-level: high
  build:
    needs: supply-chain
    # ...
```

Skipped (with a `::notice::`) on repos without a `pnpm-lock.yaml`, so it's safe to wire into non-JS repos as a slot for future per-language checks.

This workflow is **reactive** — it relies on npm's advisory database, which is updated _after_ a malicious package is detected. For behavioral detection of install-script abuse and tampered-package signatures (the genuine zero-day defense), layer [Socket.dev](https://socket.dev) or [Snyk](https://snyk.io) on top.
