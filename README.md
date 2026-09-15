# ff-app-docs

Documentation and **remote configuration** for the Feral File mobile app (`ff-app`). This repository replaces the mobile-specific config slice that previously lived under [`bitmark-inc/feral-file-docs`](https://github.com/bitmark-inc/feral-file-docs), so updates can be owned and reviewed in the **feral-file** organization.

## What ships here

| Path | Purpose |
|------|--------|
| `configs/ff-app.json` | Remote config consumed by the app at runtime (feed operators, feature flags, and related settings). The client loads this file as **`ff-app.json`** from the deployed static host. |

Before each deploy, CI parses every JSON config and verifies every signed DP-1
playlist with the pinned `@feralfile/cli` release. It then writes
`configs/version.json` with the Git commit and deployment environment so you can
tell which revision is live.

## Deployment

GitHub Actions deploys the **`configs/`** directory to **Cloudflare Pages** when you push to **`main`** or **`dev`**, or when you run the workflow manually.

- **`main`** → Production environment (default for `workflow_dispatch` when not specified inline).
- **`dev`** → Development environment.

### Required repository configuration

Configure these in the GitHub repo (Settings → Secrets and variables → Actions):

| Name | Type | Description |
|------|------|--------------|
| `CLOUDFLARE_API_TOKEN` | Secret | Token with permission to deploy to Cloudflare Pages. |
| `CLOUDFLARE_ACCOUNT_ID` | Variable | Cloudflare account ID. |
| `CLOUDFLARE_PROJECT_NAME` | Variable | Pages project name that serves the remote configs. |

The workflow is defined in `.github/workflows/cloudflare-pages-deploy.yml`.

## Impact on production

Changes merged to the deployment branch you use for production (`main` by convention) affect **live app behavior** after clients refresh remote config. Coordinate with the mobile team before altering operator lists or behavioral flags.

## Maintenance

- Edit `configs/ff-app.json` in a branch, open a PR, merge when reviewed.
- After deploy, confirm the Pages URL serves `ff-app.json` and optional `version.json` as expected.

For broader Feral File documentation (agreements, web, TV, learning content), continue using the shared docs repository under Bitmark Inc. where those assets are still maintained.

## Temporary Daily recovery — September 10, 2026

`configs/ff-app.json` temporarily points Daily at `daily-recovery-2026-09-10.json`. This separately signed recovery copy omits only **Each and Every Command AP** (`ff4ea942-d73d-4355-b370-a6cb9966d183`, scheduled September 10). Its 30.75 MB text preview exhausts Android WebView memory in app 1.8.0. All 76 other entries, artwork metadata, and scheduled dates are preserved: September 10 falls back to **Predictive Art Bot AP**, and September 11 selects **Minos AP** normally.

The original signed playlist is unchanged. The recovery copy has a new identity and is signed by the available operator key; it does not claim the original curator's signature. It is a static snapshot, so subsequent edits to the original feed will not reach Daily while this URL is configured.

After September 10 has ended for app users, or a safe renderer ships, restore `daily.playlist_url` to `https://feed.feralfile.com/api/v1/playlists/feral-file-daily-f77fe04c` and deploy. Keep the recovery JSON available for clients that still have the temporary URL cached. Restoring the config is a separate deployment; it is not scheduled automatically. Clients refresh remote config on cold launch or a Daily refresh.

### Extension — September 15, 2026

The recovery copy is also the only Daily clients read, and it ran out on **October 16**. On 2026-09-15 it was extended in place with 30 entries for **October 17 through November 15** so the URL already cached by clients keeps working. Entries rotate across the feed playlists of the NODE and Refik Anadol Studio channels — `OCCUPY` (`4036c79e-23db-4d86-8f27-21713e32aea5`), `Unsupervised — MoMA — Dreams` (`ebe2f3f2-68e4-4b81-8b0e-f570939c576d`), `Machine Hallucinations — Coral` (`2494954f-40e4-412c-bd45-10e3b51e414f`), `Unsupervised — MoMA — Burned — MoMA Dreams` (`2ed0b439-479d-4e07-8f61-9dcd11b016d9`) — one item per day in playlist order, moving to the next playlist each day and skipping a playlist once it is exhausted. Item bodies are copied verbatim from `https://feed.feralfile.com/api/v1/playlists/<id>`; only `displayAt` is added. The existing 76 entries and their dates are unchanged.

The extended document was signed by the Feral File curator wallet (`did:pkh:eip155:1:0x7D5dA3DAAdf000b9e239e45a106a9F241bDaA8b0`) and published to the feed as `https://feed.feralfile.com/api/v1/playlists/feral-file-daily-temporary-recovery-60647c56` (new playlist id `60647c56-db21-4d14-9496-088e2ae3b48a`, feed co-signature included). `configs/daily-recovery-2026-09-10.json` is the exact bytes served by that feed URL, kept under the filename clients already have cached. The original feed playlist `feral-file-daily-f77fe04c` was not extended; restoring `daily.playlist_url` to it would drop back to an October 16 end date. The next extension should follow the same step: append entries after the last `displayAt`, sign, publish, copy the served bytes here, verify.
