  # Pool Discovery Logic API

This service provides HTTP REST APIs for discovering decentralized exchange (DEX) pools and managing cached pool data.  
It is built using the [Fiber](https://github.com/gofiber/fiber) web framework in Go.

---

## 📘 Table of Contents
- [Overview](#overview)
- [Configuration](#configuration)
- [Endpoints](#endpoints)
  - [Ping](#ping)
  - [Health](#health)
  - [Discover Pool](#discover-pool)
  - [Supported DEXes](#supported-dexes)
  - [Delete Cached Pool](#delete-cached-pool)
- [Response Format](#response-format)
- [Installation](#installation)
- [Running the Service](#running-the-service)
- [Deployment](#deployment)
- [Authentication](#authentication)
- [Error Handling](#error-handling)
- [Sequence Diagram](#sequence-diagram)

---

## 🧠 Overview

The **Pool Discovery Logic API** enables:
- Pool discovery on supported DEX platforms  
- Cache management of discovered pools  
- Retrieval of supported blockchain networks and DEXes  
- Health and readiness checks  

All API responses are wrapped in a unified `ResponseWrapper` structure for consistent client integration.

---

## ⚙️ Configuration

The project uses environment variables defined in a `.env` file located in the root directory.  
You can copy the `.env.example` file to `.env` and configure it as needed.

### Example `.env.example`

```bash
# ---------------------------------
# SERVICE & ENVIRONMENT
# ---------------------------------
ENV=
MODE=
GO_MOD_TAG=

SERVICE_NAME=
TIMEZONE=Asia/Jakarta 

# ---------------------------------
# API & GRPC SERVER & AUTH
# ---------------------------------
API_PORT=8001
GRPC_PORT=50051

API_BASE_URL=
AUTH_USERNAME=
AUTH_PASSWORD=

# ---------------------------------
# LOGGING
# ---------------------------------
LOG_LEVEL=debug
LOG_PATH=./data/
LOG_NAME=

# ---------------------------------
# DATABASE (MySQL/MariaDB)
# ---------------------------------
DB_HOST=
DB_PORT=3306
DB_USER=
DB_PASS=
DB_NAME=
DB_PREFIX=

# ---------------------------------
# CACHE (Redis)
# ---------------------------------
REDIS_HOST=
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=

# ---------------------------------
# SECRET MANAGEMENT (HashiCorp Vault)
# ---------------------------------
VAULT_ADDRESS=
VAULT_TOKEN=
VAULT_MOUNT=
VAULT_SECRET=
VAULT_GLOBAL=

VAULT_CR_ADDRESS=
VAULT_CR_TOKEN=
VAULT_CR_MOUNT=
VAULT_CR_SECRET=
VAULT_GLOBAL_CR=

# ---------------------------------
# GITLAB MANAGEMENT
# ---------------------------------
GITLAB_USERNAME=
GITLAB_PASSWORD=
```

---

## 🌐 Endpoints

### 🩵 Ping
| Method | Endpoint | Description | Auth |
|--------|-----------|-------------|------|
| GET | `/ping` | Simple liveness check | ❌ |

#### Sample Request
```bash
curl http://localhost:8080/ping
```

#### Sample Response
```json
{
    "status_number": "200",
    "message": "pong!",
    "ts": 1761529418,
    "data": null
}
```

---

### ❤️ Health
| Method | Endpoint | Description | Auth |
|--------|-----------|-------------|------|
| GET | `/health` | Returns API and dependency health status | ❌ |

#### Sample Request
```bash
curl http://localhost:8080/health
```

#### Sample Response
```json
{
    "message": "Health check completed",
    "ts": 1761529368,
    "data": {
        "api": {
            "status": "healthy",
            "timestamp": 1761529368
        },
        "redis": {
            "status": "healthy",
            "timestamp": 1761529368
        },
        "oracle_db": {
            "status": "healthy",
            "timestamp": 1761529368
        }
    }
}
```

---

### 🔍 Discover Pool
| Method | Endpoint | Description | Auth |
|--------|-----------|-------------|------|
| POST | `/v1/discover` | Initiates discovery for a given pool | ✅ |

#### Sample Request
```bash
curl -X POST http://localhost:8080/v1/discover   -u username:password   -H "Content-Type: application/json"   -d '{
    "pool_address": "0x6c545e78638c8c1db7a48b282bb8ca79da107993fcb185f75cedc1f5adb2f535",
    "chain": "sui",
    "dex": "cetus",
    "convert_to_usd": true
  }'
```

#### Sample Response
```json
{
    "status_number": "200",
    "ts": 1761529222423,
    "data": {
        "cetus_chain": "1",
        "cetus_chain_convertx": "Crypto:ALL:SUI/USDT",
        "cetus_chain_dec0": 9,
        "cetus_chain_dec1": 9,
        "cetus_chain_pair0": "0x549e8b69270defbfafd4f94e17ec44cdbdd99820b33bda2278dea3b9a32d3f55::cert::CERT",
        "cetus_chain_pair1": "0x0000000000000000000000000000000000000000000000000000000000000002::sui::SUI",
        "cetus_chain_pool": "0x6c545e78638c8c1db7a48b282bb8ca79da107993fcb185f75cedc1f5adb2f535",
        "cetus_chain_reverse": "1"
    },
    "meta": {
        "discovered_at": "2025-10-27T08:40:22+07:00",
        "cached": false
    }
}
```

---

### 🧭 Supported DEXes
| Method | Endpoint | Description | Auth |
|--------|-----------|-------------|------|
| GET | `/v1/supported` | Lists all supported blockchain networks and their DEXes | ✅ |

#### Sample Request
```bash
curl -u username:password http://localhost:8080/v1/supported
```

#### Sample Response
```json
{
    "status_number": "200",
    "ts": 1761529242723,
    "data": [
        {
            "chain": "abstract",
            "supported_dexs": [
                "abstractswapv2",
                "abstractswapv3"
            ]
        },
        {
            "chain": "arbitrum",
            "supported_dexs": [
                "camelotv3",
                "curve",
                "pancakeswapv3",
                "uniswapv3"
            ]
        }
    ]
}
```

---

### 🗑️ Delete Cached Pool
| Method | Endpoint | Description | Auth |
|--------|-----------|-------------|------|
| DELETE | `/v1/cache/:chain/:pool_address` | Deletes cached pool discovery result for a specific chain | ✅ |

#### Sample Request
```bash
curl -X DELETE   -u username:password   http://localhost:8080/v1/cache/sui/0x6c545e78638c8c1db7a48b282bb8ca79da107993fcb185f75cedc1f5adb2f535
```

#### Sample Response
```json
{
    "status_number": "200",
    "ts": 1761529268649,
    "data": null
}
```

---

## 🧾 Response Format

All API responses share this format:

```json
{
  "status_number": "200",
  "message": "Success message",
  "ts": 1761529222423,
  "data": {},
  "meta": {}
}
```

| Field | Type | Description |
|--------|------|-------------|
| `status_number` | string | HTTP status as string |
| `message` | string | Response message (optional) |
| `ts` | int64 | Unix timestamp in milliseconds |
| `data` | any | Main payload |
| `meta` | any | Optional metadata |

---

## 💾 Installation

```bash
git clone https://gitlab.com/priceprovider/pool-discovery.git
cd pool-discovery
go mod tidy
cp .env.example .env
```

---

## 🚀 Running the Service

```bash
go run cmd/api/main.go
```

or build and run:

```bash
go build -o pool-discovery-api cmd/api/main.go
./pool-discovery-api
```

Default API URL:
```
http://localhost:8080
```

---

## 🐳 Deployment

The project includes a `Makefile` for simplified development, build, and deployment tasks.

### Common Commands

| Command | Description |
|----------|-------------|
| `make run` | Run the API & gRPC server locally |
| `make build` | Build the service binary into `./bin` |
| `make clean` | Clean build artifacts |
| `make get-module` | Fetch and update Go dependencies |
| `make proto` | Generate protobuf code |
| `make docker-build` | Build Docker image |
| `make docker-push` | Push Docker image to registry |
| `make up` | Start production service via Docker Compose |
| `make down` | Stop production services |
| `make dev-up` | Start local development stack |
| `make dev-down` | Stop local development stack |

Dockerfile: `deployments/docker/Dockerfile`  
Docker Compose (production): `deployments/docker/docker-compose.yaml`  
Docker Compose (development): `deployments/docker/docker-compose.dev.yaml`

---

## 🔒 Authentication

All `/v1` endpoints require **Basic Authentication**.

Header format:
```
Authorization: Basic base64(username:password)
```

---

## ⚠️ Error Handling

Errors are wrapped in the standard `ResponseWrapper`:

```json
{
  "status_number": "400",
  "message": "Invalid pool address",
  "ts": 1761529300000
}
```
List of error code:

| Status Code | Status Number | Description |
|---|---|---|
| 400 | `invalid_request` | Invalid request parameters |
| 400 | `unsupported_chain` | Unsupported blockchain |
| 400 | `unsupported_provider` | Unsupported DEX provider |
| 404 | `pool_not_found` | Pool not found |
| 404 | `pair_not_found` | Trading pair not found |
| 429 | `rate_limited` | Rate limit exceeded |
| 500 | `internal_error` | Internal server error |
| 500 | `enrichment_failed` | Failed to enrich with on-chain data |
| 503 | `rpc_client_unavailable` | RPC service unavailable |


---

## 📊 Sequence Diagram

<img width="3796" height="3650" alt="API_Sequence" src="https://drive.usercontent.google.com/download?id=10OxGcjTbmxjqPy9nLhOBDn6ECVM5WwE2&export=view" />
