# workflows

Reusable GitHub workflows for kubed-io repos, built on [kubed-io/actions](https://github.com/kubed-io/actions).
The jobs live here; a repo keeps only the triggers, in caller files copied unchanged from
[app/](app/).

The workflows are in `.github/workflows/` because GitHub calls reusable workflows from there
only. Their one trigger is `workflow_call`, so none of them runs in this repo.

## Workflows

| Workflow | Does |
|---|---|
| [kubectl.yml](.github/workflows/kubectl.yml) | `kubectl build`, `plan` or `up` of the calling repo, its output in the job summary. `up` runs in the environment given, one at a time |
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
| `runner` | the caller's `vars.DEFAULT_RUNNER` | A `runs-on` label |
| `environment` | none | The GitHub environment `up` runs in |

## An app repo

Copy [app/](app/) into the repo's `.github/workflows/`:

| File | Runs |
|---|---|
| [deploy.yml](app/deploy.yml) | `up` on a merge to main |
| [pr.yml](app/pr.yml) | `build` and `plan` on a PR |
| [kubectl.yml](app/kubectl.yml) | any of them on demand, on `DEFAULT_RUNNER` or GitHub's |
| [renovate.yml](app/renovate.yml) | Renovate on a dashboard tick, a merge, weekly and on demand |

The repo needs these variables, and the environment they name:

| Variable | Is |
|---|---|
| `DEFAULT_RUNNER` | The `runs-on` label of the jobs |
| `DEPLOY_ENVIRONMENT` | The environment `up` runs in; give it a deployment branch rule of `main` alone |
| `GCP_PROJECT`, `GCP_WIF_PROVIDER` | What [kluster-konnect](https://github.com/kubed-io/actions/tree/main/kluster-konnect) and [app-token](https://github.com/kubed-io/actions/tree/main/app-token) sign in with |

The files are the same in every repo but for the Renovate cron: give each repo a day and hour
no other has, so the week spreads them out.
