# API Reference

This document provides an overview of the APIs available for interacting with the Cross-Chain DeFi Oracle with Zero-Knowledge Privacy Layer system.

## API Categories

The system exposes several categories of APIs:

1. [Oracle API](oracle-api.md) - For requesting and providing external data
2. [Cross-Chain Bridge API](bridge-api.md) - For cross-chain asset transfers and communication
3. [Privacy Layer API](privacy-api.md) - For confidential transactions and selective disclosure
4. [Layer 2 API](layer2-api.md) - For scalable transaction processing
5. [Integration API](integration-api.md) - For developers building on top of the platform

## Authentication

API access requires authentication using one of the following methods:

### Web3 Authentication

```javascript
// Example of Web3 authentication
async function authenticateWithWeb3() {
  // Request account access
  const accounts = await ethereum.request({ method: 'eth_requestAccounts' });
  const account = accounts[0];
  
  // Generate message to sign
  const message = `Sign this message to authenticate with the API: ${Date.now()}`;
  
  // Sign the message
  const signature = await ethereum.request({
    method: 'personal_sign',
    params: [message, account]
  });
  
  // Return authentication details
  return {
    account,
    message,
    signature
  };
}
```

### API Key Authentication

For server-side applications, API key authentication is available:

```javascript
// Example of API key authentication
const apiKey = 'your-api-key';
const apiSecret = 'your-api-secret';

// Generate HMAC signature
const timestamp = Date.now().toString();
const message = timestamp + method + endpoint;
const signature = crypto
  .createHmac('sha256', apiSecret)
  .update(message)
  .digest('hex');

// Include in request headers
const headers = {
  'X-API-Key': apiKey,
  'X-API-Timestamp': timestamp,
  'X-API-Signature': signature
};
```

## Error Handling

All APIs use standard HTTP status codes and return a consistent error format:

```json
{
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "The provided parameter is invalid",
    "details": {
      "parameter": "amount",
      "reason": "Must be a positive number"
    }
  }
}
```

Common error codes include:

| Code | Description |
|------|-------------|
| UNAUTHORIZED | Authentication failed |
| INVALID_PARAMETER | One or more parameters are invalid |
| INSUFFICIENT_FUNDS | Not enough balance for the operation |
| RATE_LIMITED | Too many requests |
| SERVER_ERROR | Internal server error |
| NOT_FOUND | Resource not found |

## Rate Limiting

API calls are subject to rate limiting based on the following tiers:

| Tier | Requests per minute | Burst |
|------|---------------------|-------|
| Free | 60 | 100 |
| Developer | 300 | 500 |
| Enterprise | 3000 | 5000 |

Rate limit headers are included in all responses:

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1625097036
```

## Pagination

List endpoints support pagination using the following parameters:

- `limit`: Number of items per page (default: 20, max: 100)
- `cursor`: Pagination cursor from previous response

Example:

```
GET /api/v1/transactions?limit=50&cursor=eyJpZCI6MTAwfQ==
```

Paginated responses include a `pagination` object:

```json
{
  "data": [...],
  "pagination": {
    "next_cursor": "eyJpZCI6MTUwfQ==",
    "has_more": true
  }
}
```

## Versioning

The API uses semantic versioning in the URL path:

```
https://api.example.com/v1/resource
```

Breaking changes will only be introduced in new major versions. Minor versions may add new endpoints or parameters but will maintain backward compatibility.

## Webhooks

The API supports webhooks for event notifications:

1. Register a webhook URL:

```
POST /api/v1/webhooks
{
  "url": "https://your-service.com/webhook",
  "events": ["transaction.confirmed", "oracle.update"]
}
```

2. Webhook payload format:

```json
{
  "id": "evt_1234567890",
  "created": 1625097036,
  "type": "transaction.confirmed",
  "data": {
    "transaction_id": "0x1234...",
    "status": "confirmed",
    "block_number": 12345678
  }
}
```

3. Webhook authentication:

All webhook requests include a signature header for verification:

```
X-Webhook-Signature: t=1625097036,v1=5257a869e7ecebeda32affa62cdca3fa51cad7e77a0e56ff536d0ce8e108d8bd
```

## SDKs

Official SDKs are available for the following platforms:

- [JavaScript/TypeScript](https://github.com/mubashirrao1122/cross-chain-oracle-zkp-js)
- [Python](https://github.com/mubashirrao1122/cross-chain-oracle-zkp-py)
- [Java](https://github.com/mubashirrao1122/cross-chain-oracle-zkp-java)
- [Go](https://github.com/mubashirrao1122/cross-chain-oracle-zkp-go)
- [Rust](https://github.com/mubashirrao1122/cross-chain-oracle-zkp-rs)

Example usage (JavaScript):

```javascript
import { OracleClient } from '@cross-chain-oracle-zkp/sdk';

// Initialize client
const client = new OracleClient({
  apiKey: 'your-api-key',
  network: 'mainnet'
});

// Request price data
const ethPrice = await client.oracle.getPrice({
  asset: 'ETH',
  currency: 'USD'
});

console.log(`Current ETH price: $${ethPrice.value}`);
```

## API Status

Current API status and incident history are available at:
[status.cross-chain-oracle-zkp.io](https://status.cross-chain-oracle-zkp.io)
