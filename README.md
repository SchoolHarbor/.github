# SchoolHarbor `.github`

Org-level community health files and reusable workflows.

## Reusable workflows

### `security-marker-check.yml`

Fails CI if a known-malicious marker string (currently the [postcss/next.config worm campaign](https://github.com/orgs/community/discussions/188732)) is found in the consuming repo. Wire it as a `needs:` dependency for every other CI job:

```yaml
jobs:
  security-check:
    uses: SchoolHarbor/.github/.github/workflows/security-marker-check.yml@main
  build:
    needs: security-check
    # ...
```

Update the marker list in this repo once; every consuming repo picks up the change on its next run.
