# runicRPC

**Production-Grade Solana RPC Load Balancer**

runicRPC is an enterprise-grade middleware layer for Solana RPC infrastructure. It provides intelligent request routing, automatic failover, latency-aware load balancing, and comprehensive observability for applications that depend on reliable blockchain connectivity.

---

## Overview

Modern Solana applications require consistent, low-latency access to RPC endpoints. Single-provider architectures introduce unacceptable risk: rate limits, regional outages, and transient failures can halt production systems without warning.

runicRPC eliminates this fragility by abstracting multiple RPC providers behind a unified, fault-tolerant interface. The system continuously monitors endpoint health, routes requests to optimal providers, and gracefully handles failures without application-level intervention.

### Design Principles

- **Zero runtime dependencies** beyond `@solana/web3.js`
- **Deterministic failover** with configurable circuit breaker thresholds
- **Provider-agnostic architecture** supporting Helius, Alchemy, QuickNode, and custom endpoints
- **Observable by default** with Prometheus-compatible metrics export

---

## Architecture

runicRPC operates as a transparent proxy layer between your application and RPC providers. All standard Solana JSON-RPC methods and WebSocket subscriptions are supported without modification.

```
┌─────────────────────────────────────────────────────────────────┐
│                        Application                               │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                         runicRPC                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   Router    │  │   Circuit   │  │    Cache    │              │
│  │  (Strategy) │  │   Breaker   │  │   (LRU)     │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   Health    │  │    Rate     │  │   Metrics   │              │
│  │   Monitor   │  │   Limiter   │  │   Export    │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
└─────────────────────────────────────────────────────────────────┘
                                │
                ┌───────────────┼───────────────┐
                ▼               ▼               ▼
        ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
        │   Helius    │ │   Alchemy   │ │  QuickNode  │
        └─────────────┘ └─────────────┘ └─────────────┘
```

### Core Components

| Component | Function |
|-----------|----------|
| **Router** | Selects optimal endpoint using configurable strategy (round-robin, latency-based, weighted, random) |
| **Circuit Breaker** | Isolates failing endpoints to prevent cascade failures; auto-recovers via half-open probes |
| **Health Monitor** | Continuous background health checks with configurable intervals |
| **Rate Limiter** | Token bucket algorithm prevents provider rate limit violations |
| **Cache** | LRU cache with TTL for idempotent read operations |
| **Metrics** | Prometheus-compatible counters, histograms, and gauges |

---

## Features

### Reliability

| Feature | Description |
|---------|-------------|
| Multi-provider failover | Automatic routing to healthy endpoints when primary fails |
| Circuit breaker pattern | Prevents repeated calls to degraded endpoints |
| Configurable retry logic | Exponential backoff with jitter for transient failures |
| Request deduplication | Coalesces identical in-flight requests |

### Performance

| Feature | Description |
|---------|-------------|
| Latency-based routing | EWMA algorithm routes to consistently fastest endpoint |
| Response caching | Configurable TTL cache for read-heavy workloads |
| Connection pooling | Persistent HTTP/2 connections reduce handshake overhead |
| WebSocket multiplexing | Single connection per provider with subscription management |

### Observability

| Feature | Description |
|---------|-------------|
| Prometheus metrics | Request latency histograms, error rates, cache hit ratios |
| Structured events | Typed event emitter for circuit state changes, failovers, reconnections |
| Debug logging | Optional verbose logging with automatic API key masking |
| Health endpoints | Programmatic access to per-endpoint health status |

### Supported Providers

| Provider | HTTP | WebSocket | Configuration |
|----------|------|-----------|---------------|
| Helius | Yes | Yes | API key |
| Alchemy | Yes | Yes | API key |
| QuickNode | Yes | Yes | RPC URL + WS URL |
| Public Solana | Yes | Yes | None (fallback) |
| Custom | Yes | Yes | URL configuration |

---

## Repository Contents

This repository provides documentation and reference implementations for runicRPC. The core routing engine is proprietary; this repository offers transparency through comprehensive documentation and working demonstrations.

| Repository | Purpose |
|-----------|---------|
| `docs` | Full documentation covering installation, configuration, and production deployment |
| `demo-app` | Working demonstration of runicRPC capabilities |

---

## Usage

### Basic Integration

```typescript
import { runicRPC } from '@runic-rpc/sdk';

const rpc = runicRPC.create({
  providers: {
    helius: { apiKey: process.env.HELIUS_API_KEY },
    alchemy: { apiKey: process.env.ALCHEMY_API_KEY }
  },
  strategy: 'latency-based',
  useFallback: true
});

// Use like standard Connection
const balance = await rpc.getBalance(publicKey);
const slot = await rpc.getSlot();

// Subscribe to account changes
const subscriptionId = await rpc.onAccountChange(
  publicKey,
  (account) => console.log('Account updated:', account)
);
```

### Configuration

```typescript
const rpc = runicRPC.create({
  // Provider configuration
  providers: {
    helius: { apiKey: 'your-key' },
    alchemy: { apiKey: 'your-key' },
    quicknode: { rpcUrl: 'https://...', wsUrl: 'wss://...' }
  },

  // Routing strategy
  strategy: 'latency-based', // 'round-robin' | 'latency-based' | 'weighted' | 'random'

  // Resilience settings
  retries: 3,
  retryDelay: 1000,
  circuitBreaker: {
    threshold: 5,
    timeout: 30000
  },

  // Performance settings
  cache: {
    enabled: true,
    maxSize: 1000,
    ttl: 1000
  },
  rateLimit: 100, // requests per second

  // Observability
  logLevel: 'info', // 'debug' | 'info' | 'warn' | 'error'

  // Fallback
  useFallback: true // Adds public Solana endpoint as last resort
});
```

### Monitoring

```typescript
// Event-based observability
rpc.on('request:success', (event) => {
  console.log(`${event.method} via ${event.endpoint}: ${event.latency}ms`);
});

rpc.on('circuit:open', (event) => {
  console.log(`Circuit opened for ${event.endpoint}: ${event.reason}`);
});

rpc.on('failover', (event) => {
  console.log(`Failover: ${event.from} -> ${event.to}`);
});

// Prometheus metrics export
const metrics = rpc.getPrometheusMetrics();
```

---

## Documentation

Comprehensive documentation is available in the [docs](./apps/docs) directory:

- Installation and setup
- Provider configuration
- Routing strategies
- Circuit breaker behavior
- WebSocket subscriptions
- Metrics and observability
- Production deployment guidelines
- Troubleshooting

---

## Access

runicRPC is available for integration. For SDK access and API credentials, refer to the documentation or open an issue on this repository.

---

## License

MIT License. See [LICENSE](./LICENSE) for details.

The core runicRPC engine is closed-source. This repository provides public documentation and demonstration code under the MIT license to ensure transparency and ease of integration.

---

## Security

For security vulnerabilities, please follow responsible disclosure. See [SECURITY.md](./SECURITY.md) for reporting guidelines.

---

## Status

runicRPC is production-ready and actively maintained. The system is designed as long-term Solana infrastructure, built for reliability at scale.

For inquiries, open an issue on this repository.
