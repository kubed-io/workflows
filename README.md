# workflows

Reusable GitHub workflows for kubed-io repos, built on [kubed-io/actions](https://github.com/kubed-io/actions).
The jobs live here; a repo keeps only the triggers, in caller files copied unchanged from
[app/](app/).

## Workflows

| Workflow | Does |
|---|---|
| [kubectl.yml](.github/workflows/kubectl.yml) | `kubectl build`, `plan` or `up` of the calling repo on `krm` (the kluster) or GitHub's runner, its output in the job summary. `up` runs in the caller's `kluster` environment |
| [renovate.yml](.github/workflows/renovate.yml) | Renovate's run for the calling repo: new versions on its Dependency Dashboard, a PR when a box is ticked |

```yaml
jobs:
  plan:
    uses: kubed-io/workflows/.github/workflows/kubectl.yml@main
    with:
      cmd: plan
    permissions:
      contents: read
      id-token: write
```

### kubectl.yml inputs

| Input | Default | Description |
|---|---|---|
| `cmd` | — | `build`, `plan` or `up` |
| `args` | `.` | The kustomization it runs on |
| `runner` | the caller's `vars.DEFAULT_RUNNER` | A `runs-on` label: `krm` or `ubuntu-26.04` |

## An app repo

Copy [app/](app/) into the repo's `.github/workflows/`:

| File | Runs |
|---|---|
| [deploy.yml](app/deploy.yml) | `up` on a merge to main |
| [pr.yml](app/pr.yml) | `build` and `plan` on a PR |
| [kubectl.yml](app/kubectl.yml) | any of them on demand, on the runner picked |
| [renovate.yml](app/renovate.yml) | Renovate on a dashboard tick, a merge, weekly and on demand |

The repo needs:

- the variables `GCP_PROJECT`, `GCP_WIF_PROVIDER` (the org has both) and `DEFAULT_RUNNER`
  (`krm` or `ubuntu-26.04`)
- an environment `kluster` whose deployment branches are `main` alone

The files are the same in every repo but for the Renovate cron: each repo takes a day and hour
no other has, so the week spreads them out.

| Repo | Cron | When (UTC) |
|---|---|---|
| grafana | `0 6 * * 1` | Monday 06:00 |
| n8n | `0 7 * * 2` | Tuesday 07:00 |
