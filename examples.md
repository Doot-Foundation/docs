# Doot Oracle Client Examples

Practical ways to use the `@dootfoundation/client` package in real code.

## Examples

### 1. `doot-demo.ts` - Comprehensive Demo
A complete demonstration of all Doot Oracle client features:

- API Method: Fast price fetching from Doot servers
- L2 Method: Direct blockchain calls to Zeko L2
- L1 Method: Secure calls to Mina L1 blockchain
- Smart Fallback: Automatic API → L2 → L1 failover
- Multiple Tokens: Batch price fetching
- Key Validation: API key verification

Run the demo:
```bash
npm run build
node build/src/doot-demo.js
```

### 2. `interact.ts` - zkApp Integration
Shows how to integrate Doot price feeds into a Mina zkApp:

- Fetches price data using smart fallback
- Creates zkApp transactions with verified prices
- Demonstrates on-chain price updates
- Calculates exchange rates

Run the zkApp example:
```bash
npm run build
node build/src/interact.js
```

## Setup

1. Install dependencies:
   ```bash
   npm install
   ```

2. Add API key (create `.env` file):
   ```bash
   DOOT_API_KEY=your-api-key-here
   ```
   Get a free API key at: https://doot.foundation/dashboard

3. Build and run:
   ```bash
   npm run build
   node build/src/doot-demo.js
   ```

## Key Concepts Demonstrated

### Method Comparison
- API: Fastest (~100ms), requires valid key
- L2: Fast (~10-30s), decentralized via Zeko
- L1: Secure (~30-60s), maximum security via Mina

### Error Handling
All examples show proper error handling for network issues, invalid keys, and blockchain timeouts.

### Data Structure
All methods return identical format:
```javascript
{
  source: 'API',
  price_data: {
    token: 'bitcoin',
    price: '65432.12',
    oracle: 'B62q...',
    signature: 'ABC123...'
  }
}
```

## Notes

- L2/L1 methods compile contracts once per process; the SDK caches this
- Smart fallback automatically chooses the best available source
- API responses include signature and proof data fields
- Works in any Node.js environment; no special blockchain setup required

## Next Steps

- Read full documentation: https://docs.doot.foundation
- Install package: `npm install @dootfoundation/client`
- Get API key: https://doot.foundation/dashboard
