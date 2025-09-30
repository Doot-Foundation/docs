# Doot Oracle Client (npm)

Content adapted from `npm/main/README.md`.

## What is Doot?

Doot provides verified price data for 10 major cryptocurrencies using zero-knowledge proofs. Your app gets fast, reliable prices with automatic fallback across multiple sources.

## Quick Start

```bash
npm install @dootfoundation/client
```

```javascript
import { Client } from '@dootfoundation/client';

const client = new Client('your-api-key');
const price = await client.getData('bitcoin');

console.log(`Bitcoin: $${price.price_data.price}`);
```

## zkApp-CLI Usage

When using in zkApp-CLI projects, you may need to add `@ts-ignore` for TypeScript compilation:

```typescript
import dotenv from 'dotenv';
dotenv.config();

// @ts-ignore
import { Client } from '@dootfoundation/client';

const client = new Client(process.env.DOOT_API_KEY);
const price = await client.getData('bitcoin');
```

This is only a TypeScript compilation issue - the package works at runtime in zkApp-CLI environments.

## Supported Tokens

- bitcoin (BTC)
- ethereum (ETH)
- mina (MINA)
- solana (SOL)
- chainlink (LINK)
- ripple (XRP)
- dogecoin (DOGE)
- polygon (MATIC)
- avalanche (AVAX)
- cardano (ADA)

## How It Works

Doot uses a 3-layer fallback system:

1. API (fastest, ~100ms) - Direct from Doot servers
2. L2 (fast, ~10-30s) - Zeko Layer 2 blockchain
3. L1 (secure, ~30-60s) - Mina mainnet blockchain

If one source fails, it automatically tries the next one.

## API Methods

### `getData(token)`
Smart fallback through all sources (recommended)
```javascript
const price = await client.getData('ethereum');
```

### `getFromAPI(token)`
Direct from API (requires valid key)
```javascript
const price = await client.getFromAPI('bitcoin');
```

### `getFromL2(token)`
From Zeko L2 blockchain
```javascript
const price = await client.getFromL2('solana');
```

### `getFromL1(token)`
From Mina L1 blockchain
```javascript
const price = await client.getFromL1('mina');
```

### `isKeyValid()`
Check if your API key works
```javascript
const valid = await client.isKeyValid();
```

### `validtokens`
List of supported tokens
```javascript
import { validtokens } from '@dootfoundation/client';
console.log(validtokens);
```

## Response Format

All methods return the same format:

```javascript
{
  source: 'API',
  fromAPI: true,
  fromL2: false,
  fromL1: false,
  price_data: {
    token: 'bitcoin',
    price: '65432.12',
    decimals: '10',
    aggregationTimestamp: '1640995200000',
    signature: 'ABC123...',
    oracle: 'B62q...'
  },
  proof_data: '{...}'
}
```

## Requirements

- Node.js 18+
- Internet connection
- API key for fastest access (free at doot.foundation)

## Support

- Documentation: https://docs.doot.foundation
- Issues: GitHub Issues (Doot Foundation npm)
- Website: https://doot.foundation

