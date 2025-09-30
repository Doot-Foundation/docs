# Configuration

This page lists only public configuration that integrators need. Internal
credentials are managed in the deployment environment and are not documented here.

## Public Variables

- `DOOT_API_KEY` — client key used with the public API and SDK
- `NEXT_PUBLIC_PINATA_GATEWAY` — gateway host used to fetch pinned IPFS JSON
- `NEXT_PUBLIC_ZEKO_DOOT_PUBLIC_KEY` — Zeko L2 contract public key
- `NEXT_PUBLIC_MINA_DOOT_PUBLIC_KEY` — Mina L1 contract public key (if used)

## SDK Overrides (optional, in code)

- `client.BaseURL = 'https://your-api.example'`
- `client.ZekoL2Endpoint = 'https://devnet.zeko.io/graphql'`
- `client.MinaL1Endpoint = 'https://api.minascan.io/node/devnet/v1/graphql'`
- `client.MinaL1ArchiveEndpoint = 'https://api.minascan.io/archive/devnet/v1/graphql'`
- `client.DootL1Address = PublicKey.fromBase58('...')`
- `client.DootL2Address = PublicKey.fromBase58('...')`

## Internal Credentials (not public)

The backend and cron workers rely on internal credentials such as:
- Redis service credentials
- Oracle signer used for aggregated signatures and transactions
- IPFS pinning token for JSON uploads
- Optional data‑provider API keys (exchanges/indexers)

These are not exposed or documented in public docs. Operators should manage and rotate them in their secret manager/CI.

## Cron Schedules

- `cron-update-all-prices`: every 10 minutes on the minute (`*/10 * * * *`)
- `cron-update-historical`: every 10 minutes starting at minute 5 (`5-59/10 * * * *`)
- `cron-zkapp-update-doot-zeko`: every 10 minutes at minutes 2,12,22,32,42,52 (`2-59/10 * * * *`)
- `cron-zkapp-update-doot-mina`: every 20 minutes at minutes 2,22,42 (`2-59/20 * * * *`)

The offsets ensure Zeko/Mina on-chain updates run after price aggregation and historical IPFS pinning complete, reducing contention and read-after-write issues.
