# ff-app-docs

Documentation and **remote configuration** for the Feral File mobile app (`ff-app`). This repository replaces the mobile-specific config slice that previously lived under [`bitmark-inc/feral-file-docs`](https://github.com/bitmark-inc/feral-file-docs), so updates can be owned and reviewed in the **feral-file** organization.

## What ships here

| Path                  | Purpose                                                                                                                                                                            |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `configs/ff-app.json` | Remote config consumed by the app at runtime (feed operators, feature flags, and related settings). The client loads this file as **`ff-app.json`** from the deployed static host. |

On each deploy, CI also writes `configs/version.json` with the Git commit and deployment environment so you can tell which revision is live.

## Deployment

GitHub Actions deploys the **`configs/`** directory to **Cloudflare Pages** when you push to **`main`** or **`dev`**, or when you run the workflow manually.

- **`main`** → Production environment (default for `workflow_dispatch` when not specified inline).
- **`dev`** → Development environment.

### Required repository configuration

Configure these in the GitHub repo (Settings → Secrets and variables → Actions):

| Name                      | Type     | Description                                          |
| ------------------------- | -------- | ---------------------------------------------------- |
| `CLOUDFLARE_API_TOKEN`    | Secret   | Token with permission to deploy to Cloudflare Pages. |
| `CLOUDFLARE_ACCOUNT_ID`   | Variable | Cloudflare account ID.                               |
| `CLOUDFLARE_PROJECT_NAME` | Variable | Pages project name that serves the remote configs.   |

The workflow is defined in `.github/workflows/cloudflare-pages-deploy.yml`.

## Impact on production

Changes merged to the deployment branch you use for production (`main` by convention) affect **live app behavior** after clients refresh remote config. Coordinate with the mobile team before altering operator lists or behavioral flags.

## Maintenance

- Edit `configs/ff-app.json` in a branch, open a PR, merge when reviewed.
- After deploy, confirm the Pages URL serves `ff-app.json` and optional `version.json` as expected.

For broader Feral File documentation (agreements, web, TV, learning content), continue using the shared docs repository under Bitmark Inc. where those assets are still maintained.
