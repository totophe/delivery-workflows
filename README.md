# delivery-workflows

Reusable GitHub Actions workflows for projects that are built on GitHub and
run on a self-hosted platform behind a deploy gateway.

The pipelines hold no deployment key. A job asks GitHub for an identity token
and presents it to the gateway; the gateway checks the token and decides what
that run may do.

## `platform-deploy.yml`

Makes one call to the gateway, waits until the application reports the
expected commit, and posts the result as a status on that commit.

| Input | |
|---|---|
| `action` | `deploy`, `open-preview`, `remove-preview` or `rollback` |
| `app` | the application's identifier on the platform |
| `stage` | `staging`, `preview` or `production` |
| `commit` | the full commit expected to run |
| `pr` | the pull request number, for previews |
| `url` | the application's public address |
| `health_path` | path that answers with the running commit (default `/health`) |

The calling repository sets two variables, `DEPLOY_GATEWAY_URL` and
`DEPLOY_GATEWAY_AUDIENCE`, and gives the job these permissions:

```yaml
jobs:
  staging:
    permissions: { id-token: write, statuses: write, contents: read }
    uses: totophe/delivery-workflows/.github/workflows/platform-deploy.yml@v1
    with:
      action: deploy
      app: <application id>
      stage: staging
      commit: ${{ github.event.workflow_run.head_sha }}
      url: https://staging.example.org
```

The application's health endpoint must include the full commit it runs,
anywhere in its body.
