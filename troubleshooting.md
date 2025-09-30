# Troubleshooting

Practical fixes for common errors seen while running or integrating Doot.

## API: 401 Unauthorized

- Ensure header is `Authorization: Bearer <DOOT_API_KEY>`
- Validate key with the SDK: `client.isKeyValid()`

## API: 429 Too Many Requests

- Upstash Ratelimit is active; slow down client or contact support for higher limits

## L2/L1: OffchainState read fails with Field.assertEquals

- This typically means the state hasn’t settled yet after an `update()`
- Wait 20–60 seconds and retry

## IPFS: New CID not accessible

- Verify `NEXT_PUBLIC_PINATA_GATEWAY` and that the gateway returns JSON for the CID
- Check that the upload body is valid JSON with the expected structure

## SDK: All sources failed

- The error includes API, L2, and L1 failure messages
- Check network connectivity and whether Zeko/Mina devnets are up

