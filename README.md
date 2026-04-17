# Shipyrd Deploy Action

Notify [Shipyrd](https://shipyrd.io) of deploy lifecycle events from your GitHub Actions workflow.

## Usage

Add three steps around your deploy — one before, one on success, one on failure:

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Notify Shipyrd (pre-deploy)
    uses: shipyrd/deploy-action@v1
    with:
      api-token: ${{ secrets.SHIPYRD_API_KEY }}
      status: pre-deploy
      service: my-app

  # --- your deploy steps ---

  - name: Notify Shipyrd (post-deploy)
    if: success()
    uses: shipyrd/deploy-action@v1
    with:
      api-token: ${{ secrets.SHIPYRD_API_KEY }}
      service: my-app  # status defaults to post-deploy

  - name: Notify Shipyrd (failed)
    if: failure()
    uses: shipyrd/deploy-action@v1
    with:
      api-token: ${{ secrets.SHIPYRD_API_KEY }}
      status: failed
      service: my-app
```

## Setup

1. Go to your application's **Setup** page in Shipyrd and copy the deploy token.
2. In your GitHub repository, go to **Settings → Secrets and variables → Actions** and add a secret named `SHIPYRD_API_KEY` with that token as the value.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `api-token` | yes | — | Your Shipyrd deploy token |
| `status` | no | `post-deploy` | `pre-deploy`, `post-deploy`, or `failed` |
| `service` | yes | — | Application name as it appears in Shipyrd |
| `shipyrd-url` | no | `https://hooks.shipyrd.io` | Override for self-hosted Shipyrd |
| `destination` | no | `production` | Deploy destination (e.g. `staging`) |
| `performer` | no | `github.actor` | Who triggered the deploy |
| `version` | no | `github.sha` | Git SHA being deployed |
| `commit-message` | no | head commit message | Commit message for this deploy |

## Why `if: success()` and `if: failure()`?

Without these conditions, a failed deploy step would skip the `post-deploy` and `failed` notifications entirely, leaving the Shipyrd dashboard showing the deploy permanently stuck in `pre-deploy`. The `success()`/`failure()` split ensures Shipyrd always receives a terminal status.

## Testing locally

Use [act](https://github.com/nektos/act) to run the test workflow against a local Shipyrd instance:

```bash
act workflow_dispatch -W .github/workflows/test-local.yml \
  -j test-post-deploy \
  -s SHIPYRD_TOKEN=your-token \
  --input shipyrd-url=http://host.docker.internal:3000 \
  --env COMMIT_MESSAGE="your commit message"
```

`host.docker.internal` routes to your Mac's localhost from inside the Docker container that `act` uses. Replace the port with whatever your local Shipyrd instance is running on.
