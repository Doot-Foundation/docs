# Doot Oracle Client (npm)

Production SDK for consuming Doot price feeds. It prioritizes speed while
preserving verifiability and provides a single response format regardless of
source.

## Quick Start

```bash
npm install @dootfoundation/client
```

```javascript
import { Client } from '@dootfoundation/client';

const client = new Client(process.env.DOOT_API_KEY || '');
const price = await client.getData('bitcoin');
console.log(price.source, price.price_data.price);
```

## Supported Tokens

- mina, bitcoin, ethereum, solana, ripple, cardano, avalanche, polygon, chainlink, dogecoin

Internally, the SDK maps these names to on‑chain vector indexes. See Architecture → Token Ordering.

## Fallback Logic and Caching

- `getData(token)` tries sources in order: API → L2 → L1
- Each step logs internally; the thrown error contains the chain of failures
- On‑chain reads share a one‑time compilation cache and a 3‑minute result cache per token/source

Notes:
- Default endpoints are devnet (Zeko and Mina). Override only if you know what you’re doing.
- Both L1 and L2 point to the same hardcoded contract address in this build. If you deploy your own, set `client.DootL1Address`/`client.DootL2Address` after constructing the client.

## Methods

- `getData(token)`
  - Smart fallback. Recommended for most apps.
- `getFromAPI(token)`
  - Fastest. Requires `DOOT_API_KEY`. Hits `/api/get/price` on `client.BaseURL`.
- `getFromL2(token)`
  - Reads `getPrices()` from `Doot` on Zeko, extracts the token index.
- `getFromL1(token)`
  - Same as L2 but against Mina.
- `isKeyValid()`
  - Calls `/api/get/user/getKeyStatus` to validate your key.

## Response

All methods return the same shape:

```javascript
{
  source: 'API' | 'L2' | 'L1',
  fromAPI: boolean,
  fromL2: boolean,
  fromL1: boolean,
  price_data: {
    token: string,
    price: string,                // integer string with 10 decimals
    decimals: '10',               // fixed decimal precision
    aggregationTimestamp: string, // ms since epoch
    signature: string,            // for API source
    oracle: string                // base58 public key of oracle/contract
  },
  proof_data: string              // JSON string for API calls
}
```

## Advanced Configuration

- `client.BaseURL`
  - Defaults to `https://doot.foundation`. Point it at your own API mirror if needed.
- `client.MinaL1Endpoint` / `client.MinaL1ArchiveEndpoint`
  - Defaults to Minascan devnet endpoints.
- `client.ZekoL2Endpoint`
  - Defaults to `https://devnet.zeko.io/graphql`.
- `client.DootL1Address` / `client.DootL2Address`
  - Defaults to the public oracle address. Override after instantiation to read your deployments.

## Errors and Retries

- `getData()` throws a combined error when all sources fail: `All sources failed - API: <…>, L2: <…>, L1: <…>`
- L1/L2 errors commonly indicate network unavailability or OffchainState still settling. Retry after 20–60 seconds.

## zkApp‑CLI Usage

Some `o1js` builds require a `// @ts-ignore` in zkApp‑CLI projects. This is a
TypeScript setup quirk only; runtime behavior is correct.

```typescript
import dotenv from 'dotenv';
dotenv.config();
// @ts-ignore
import { Client } from '@dootfoundation/client';

const client = new Client(process.env.DOOT_API_KEY!);
const price = await client.getData('mina');
```
