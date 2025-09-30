# Data Model

This section documents the JSON shapes used in caches, API responses,
and IPFS objects so that you can safely build integrations and tools.

## Per‑Token Cache Object (Redis)

Key: `TOKEN_TO_CACHE[token]`

```json
{
  "price": "1234567890",            
  "floatingPrice": 123.456789,       
  "decimals": 10,                    
  "aggregationTimestamp": 1720000000000,
  "signature": {
    "signature": "...",             
    "publicKey": "B62q...",        
    "data": "1234567890"          
  },
  "prices_returned": [ ... ],        
  "signatures": [ ... ],             
  "timestamps": [ ... ],             
  "urls": [ ... ]                    
}
```

Notes:
- `price` is an integer string scaled by `decimals` (fixed at 10)
- `floatingPrice` is the raw mean before scaling

## Aggregation Proof Cache (Redis)

Key: `TOKEN_TO_AGGREGATION_PROOF_CACHE[token]`

- Holds zk proof material for batch aggregation programs (20/100). Shape may
  evolve; treat as opaque and versioned.

## API Response

`GET /api/get/price?token=<name>`

```json
{
  "status": true,
  "message": "Price information found.",
  "data": {
    "price_data": { ... per‑token cache object ... },
    "proof_data": { ... aggregation proof object ... }
  },
  "asset": "mina"
}
```

## IPFS Object (Pinned by cron)

```json
{
  "assets": { "mina": { ... per‑token cache object ... }, ... },
  "merkle_map": {
    "pinnedAt": 1720000000000,
    "commitment": "<Field as string>",
    "keys": ["<Field>", ...],
    "values": ["<Field>", ...],
    "witnesses": [ { ... }, ... ]
  }
}
```

The contract stores `commitment` and the packed CID for this object and
verifies changes via OffchainState settlement proofs.

