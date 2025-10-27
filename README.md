# Pool Discovery gRPC Service

This service provides gRPC endpoints for decentralized exchange (DEX) pool discovery and cache management, complementing the REST API layer.

---

## 📘 Table of Contents
- [Overview](#overview)
- [Configuration](#configuration)
- [Service Definition](#service-definition)
- [Messages](#messages)
- [Example Usage](#example-usage)
- [Running the Service](#running-the-service)
- [Deployment](#deployment)
- [Authentication](#authentication)
- [Error Handling](#error-handling)
- [Sequence Diagram](#sequence-diagram)

---

## 🧠 Overview

The **Pool Discovery gRPC Service** allows internal services and clients to perform pool discovery and retrieve supported decentralized exchanges (DEXes) efficiently over gRPC.

It is implemented in Go using **gRPC** and **Protocol Buffers (protobuf)**.  
All communication uses Basic Authentication for request authorization.

---

## ⚙️ Configuration

The gRPC server uses environment variables defined in the `.env` file located in the root project directory.

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

## 📜 Service Definition

The service definition resides in:
```
api/proto/v1/discovery.proto
```

### Discovery.proto

```proto
syntax = "proto3";

package api.proto.v1;
option go_package = "gitlab.com/priceprovider/pool-discovery/api/proto/v1;pooldiscoveryv1";

import "google/protobuf/struct.proto";

service PoolDiscoveryService {
  rpc DiscoverPool(DiscoverPoolRequest) returns (DiscoverPoolResponse);
  rpc SupportedDexs(SupportedDexsRequest) returns (SupportedDexsResponse);
  rpc ClearCache(ClearCacheRequest) returns (ClearCacheResponse);
}
```

---

## 🧾 Messages

### DiscoverRequest
```proto
message DiscoverPoolRequest {
  string chain = 1;
  string pool_address = 2;
  string dex = 3; // optional
  bool convert_to_usd = 4; // optional
}
```

### DiscoverResponse
```proto
message DiscoverPoolResponse {
  google.protobuf.Struct result = 1;
  
  PoolMeta meta = 2;
}

message PoolMeta {
  string discovered_at = 1;
  bool cached = 2;
}
```

### SupportedDexResponse
```proto
message SupportedDexsResponse {
  repeated SupportedDexEntry data = 1;
}

message SupportedDexEntry {
  string chain = 1;
  repeated string supported_dexs = 2;
}

```

###  ClearCacheRequest
```proto
message ClearCacheRequest {
  string chain = 1;
  string pool_address = 2;
}
```

###  ClearCacheResponse
```proto
message ClearCacheResponse {
}
```

---

## 💡 Example Usage

### Discover Pool Example (Go Client)
```go
conn, _ := grpc.Dial("localhost:50051", grpc.WithInsecure())
client := discoveryv1.NewPoolDiscoveryServiceClient(conn)

req := &discoveryv1.DiscoverRequest{
  PoolAddress: "0x6c545e78638c8c1db7a48b282bb8ca79da107993fcb185f75cedc1f5adb2f535",
  Chain:       "sui",
  Dex:         "cetus",
  ConvertToUsd: true,
}

res, err := client.DiscoverPool(context.Background(), req)
if err != nil {
  log.Fatalf("Error: %v", err)
}

fmt.Printf("Response: %+v\n", res)
```

---

## 🚀 Running the Service

Run gRPC server locally:
```bash
make run
```

or build manually:
```bash
go build -o ./bin/pool-discovery-server cmd/server/main.go
./bin/pool-discovery-server
```

Default gRPC port: **50051**

---

## 🐳 Deployment

Deployment is managed via the **Makefile** at the project root.

### Common Commands

| Command | Description |
|----------|-------------|
| `make run` | Run the API & gRPC server locally |
| `make build` | Build the gRPC server binary |
| `make proto` | Generate protobuf and gRPC stubs |
| `make docker-build` | Build Docker image |
| `make docker-push` | Push Docker image to registry |
| `make up` | Start production service using Docker Compose |
| `make down` | Stop production service |
| `make dev-up` | Start local development stack |
| `make dev-down` | Stop local development stack |

Dockerfile: `deployments/docker/Dockerfile`  
Compose (Production): `deployments/docker/docker-compose.yaml`  
Compose (Development): `deployments/docker/docker-compose.dev.yaml`

---

## 🔒 Authentication

The gRPC service uses Basic Authentication metadata in each request.

### Metadata Header Example
```go
md := metadata.Pairs("authorization", "Basic base64(username:password)")
ctx := metadata.NewOutgoingContext(context.Background(), md)
```

---

## ⚠️ Error Handling

gRPC errors follow standard gRPC status codes:

| Code | Description |
|------|--------------|
| `OK` | Success |
| `InvalidArgument` | Bad or invalid request parameters |
| `Unauthenticated` | Invalid or missing credentials |
| `NotFound` | Resource not found |
| `Internal` | Internal server error |

---

## 📊 Sequence Diagram

<img width="3796" height="3650" alt="gRPC_Sequence" src="https://drive.usercontent.google.com/download?id=1XQ9jh5MSXtEEzMu10dtLORLSW1dv03tS&export=view" />
