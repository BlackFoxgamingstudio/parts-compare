# Architecture: Sovereign Parts Compare

## Overview

**Package ID:** `PKG-005`  
**Domain:** Web Scraping & E-Commerce NLP  
**Microservice Port:** `8783`  
**n8n Webhook Path:** `parts-compare-trigger`  
**GitHub:** [BlackFoxgamingstudio/parts-compare](https://github.com/BlackFoxgamingstudio/parts-compare)

Cross-supplier parts comparison engine with async web scraping, NLP attribute normalization, price tracking, availability alerts, and procurement recommendations.

---

## System Architecture

```
                     ┌──────────────────────────────────┐
                     │       Sovereign Parts Compare       │
                     │       Port: 8783            │
                     ├──────────────┬───────────────────┤
   n8n Webhook ────▶ │  REST API    │   Core Engine     │
   HTTP POST         │  /api/v1/*   │   Dispatcher      │
                     └──────┬───────┴────────┬──────────┘
                            │                │
              ┌─────────────▼────────────────▼─────────┐
              │          Component Layer                 │
              │  AsyncScraper    | AttributeNormal | PriceTracker  │
              └────────────────────────┬────────────────┘
                                       │
              ┌────────────────────────▼────────────────┐
              │      n8n Central Event Bus (:5678)       │
              └─────────────────────────────────────────┘
```

## Core Components

### `AsyncScraper`
Handles all asyncscraper operations. Exposes async methods callable from the core dispatcher.

### `AttributeNormalizer`
Handles all attributenormalizer operations. Exposes async methods callable from the core dispatcher.

### `PriceTracker`
Handles all pricetracker operations. Exposes async methods callable from the core dispatcher.

### `AvailabilityMonitor`
Handles all availabilitymonitor operations. Exposes async methods callable from the core dispatcher.

### `ProcurementAdvisor`
Handles all procurementadvisor operations. Exposes async methods callable from the core dispatcher.

---

## API Contract

All interactions follow the SBB standard envelope:

```http
POST /api/v1/execute
Content-Type: application/json
X-SBB-API-Key: <api-key>

{
  "action": "<operation>",
  "payload": {},
  "trace_id": "optional-uuid"
}
```

**Success Response (HTTP 200):**
```json
{
  "status": "success",
  "data": {},
  "trace_id": "...",
  "timestamp": "2025-01-01T00:00:00Z"
}
```

**Health Check:**
```http
GET /health
→ {"status": "healthy", "service": "sovereign-parts-compare", "port": 8783}
```

## Integration Matrix

| System | Protocol | Direction | Purpose |
|--------|----------|-----------|---------|
| n8n Event Bus (:5678) | HTTP POST | Outbound | Event forwarding |
| n8n Webhook | HTTP POST | Inbound | Trigger execution |
| SBB Codebase Vault (:8766) | HTTP | Outbound | Code analysis |
| SBB Patterns Bible (:8794) | HTTP | Outbound | Standards validation |
| External APIs | HTTPS | Outbound | Domain-specific data |

## Deployment Architecture

```yaml
# docker-compose excerpt
sovereign-parts-compare:
  image: sovereign-parts-compare:latest
  ports: ["8783:8783"]
  healthcheck:
    test: curl -f http://localhost:8783/health
    interval: 30s
```

## Security Model

| Control | Implementation |
|---------|---------------|
| Authentication | `X-SBB-API-Key` header (env: `SBB_API_KEY`) |
| Rate Limiting | 100 req/min per client IP |
| Input Validation | Pydantic models (strict mode) |
| Container Security | Non-root user (`appuser:1001`) |
| Secrets | Environment variables only (never hardcoded) |
| TLS | Terminate at reverse proxy (nginx/caddy) |

## Tags
`scraping`, `e-commerce`, `nlp`, `parts`
