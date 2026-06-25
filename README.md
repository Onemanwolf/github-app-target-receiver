# github-app-target-receiver (TARGET)

The **target** side of a cross-repo GitHub Actions trigger built on a **GitHub App**.
This repo receives a `repository_dispatch` event from the **source** repo
([`github-app-source-trigger`](https://github.com/Onemanwolf/github-app-source-trigger)),
does its work, and reports back.

📖 **Full guide:** [`IMPLEMENTATION.md` in the source repo](https://github.com/Onemanwolf/github-app-source-trigger/blob/main/IMPLEMENTATION.md).

## Workflows in this repo

| Workflow | Trigger | What it does |
|----------|---------|--------------|
| **Target - Cross-Repo Receiver** | `repository_dispatch: cross-repo-trigger` | Reads the payload, "deploys", resolves a container name, and uploads a `result.json` artifact (`cross-repo-result`) |
| **Target - Evaluator** | `workflow_run` of the receiver | Evaluates the receiver's result and sends an async callback (`cross-repo-result`) back to the source |

The receiver stamps its **run name** with a `correlation_id` from the payload so the
source's synchronous workflow can find exactly this run while polling.

## Secrets required (this repo)

Needed because the evaluator authenticates as the App to send the callback **back** to the source:

| Secret | Value |
|--------|-------|
| `APP_CLIENT_ID` | The GitHub App's Client ID (`Iv23...`) |
| `APP_PRIVATE_KEY` | The App's private key (`.pem` contents) |

```bash
gh secret set APP_CLIENT_ID   --repo Onemanwolf/github-app-target-receiver --body "Iv23li..."
gh secret set APP_PRIVATE_KEY --repo Onemanwolf/github-app-target-receiver < path/to/key.pem
```

## Data in / out

- **In:** `client_payload` fields (e.g. `container_name`, `image_tag`) read as `${{ github.event.client_payload.<key> }}`.
- **Out:** values produced during the run are written to `result.json` and uploaded as the `cross-repo-result` artifact; the source downloads and parses it.
