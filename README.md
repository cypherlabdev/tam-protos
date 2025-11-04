# TAM Protos

Shared Protocol Buffer definitions for the TAM (Total Addressable Market) sports betting platform microservices.

## Overview

This repository serves as the **single source of truth** for all gRPC service definitions across the TAM platform. It contains proto definitions for 7 microservices handling 1M+ bets per day.

## Services

### Core Services

- **wallet-service**: Balance management, reservations, and transactions
  - 2PC-style balance reservations (Reserve → Commit/Cancel)
  - Idempotent operations with saga support
  - Complete audit trail

- **user-service**: Authentication and session management
  - JWT-based authentication
  - Redis session storage
  - Session validation middleware

- **order-validator**: Saga orchestration using Temporal workflows
  - Stateless orchestrator
  - Distributed transaction coordination
  - Automatic compensation on failure

- **order-book**: Bet placement and settlement
  - Order matching engine
  - Settlement processing (win/loss/push)
  - Integration with wallet-service

### Supporting Services

- **notification-service**: User notifications (event-driven)
  - WebSocket real-time updates
  - Email, SMS, and push notifications
  - NATS event consumption

- **risk-analyzer**: Fraud detection and risk scoring (Python)
  - ML-based risk assessment
  - Rate limiting and pattern detection
  - Real-time risk checks

- **reporting-service**: Analytics and reporting (Python)
  - Daily/weekly/monthly reports
  - Real-time dashboards
  - Event-driven analytics

## Usage

### Go

```go
import (
    walletv1 "github.com/tam/tam-protos/gen/go/wallet/v1"
    userv1 "github.com/tam/tam-protos/gen/go/user/v1"
    orderv1 "github.com/tam/tam-protos/gen/go/order/v1"
)

// Create wallet service client
conn, _ := grpc.Dial("wallet-service:9090", grpc.WithTransportCredentials(insecure.NewCredentials()))
walletClient := walletv1.NewWalletServiceClient(conn)

// Reserve balance
resp, err := walletClient.ReserveBalance(ctx, &walletv1.ReserveBalanceRequest{
    UserId:         "user-123",
    Amount:         "100.00",
    SagaId:         "saga-456",
    IdempotencyKey: "idempotency-789",
})
```

### Python

```python
from tam_protos.gen.python.wallet.v1 import wallet_service_pb2
from tam_protos.gen.python.wallet.v1 import wallet_service_pb2_grpc

# Create channel and stub
channel = grpc.insecure_channel('wallet-service:9090')
wallet_client = wallet_service_pb2_grpc.WalletServiceStub(channel)

# Reserve balance
response = wallet_client.ReserveBalance(
    wallet_service_pb2.ReserveBalanceRequest(
        user_id="user-123",
        amount="100.00",
        saga_id="saga-456",
        idempotency_key="idempotency-789"
    )
)
```

## Development

### Prerequisites

- [Buf](https://buf.build/docs/installation) (for proto management)
- Go 1.21+ (for Go code generation)
- Python 3.11+ (optional, for Python code generation)

### Generate Code

```bash
# Generate Go code
buf generate

# The generated code will be in:
# - gen/go/common/v1/
# - gen/go/wallet/v1/
# - gen/go/user/v1/
# - gen/go/order/v1/
# - gen/go/notification/v1/
# - gen/go/risk/v1/
# - gen/go/reporting/v1/
```

### Validate Changes

```bash
# Lint proto files
buf lint

# Check for breaking changes against main branch
buf breaking --against '.git#branch=main'
```

### Running Tests

```bash
# Ensure generated code compiles
cd gen/go
go mod tidy
go build ./...
```

## Versioning

See [docs/VERSIONING.md](docs/VERSIONING.md) for proto versioning strategy.

### Current Version: v1

All services are currently on v1. Breaking changes will require a v2 migration.

## Critical Requirements

### Idempotency

**ALL write operations MUST include an `idempotency_key` field.** This ensures safe retries and prevents duplicate transactions in a distributed system.

Example:
```protobuf
message ReserveBalanceRequest {
    string user_id = 1;
    string amount = 2;
    string saga_id = 3;
    string idempotency_key = 4;  // CRITICAL
}
```

### Saga Support

Operations participating in distributed transactions (sagas) MUST include a `saga_id` field for tracking and correlation.

### Decimal Precision

**ALL money amounts use `string` type** (NOT float/double) to avoid floating-point precision issues.

Example:
```protobuf
message GetBalanceResponse {
    string balance = 1;  // "500.00" NOT 500.0
    string currency = 2; // "EUR"
}
```

## Architecture Patterns

### 2PC-Style Reservations (wallet-service)

```
1. ReserveBalance  → Creates reservation (status=PENDING)
2. CommitReservation → Deducts balance (status=COMMITTED)
   OR
   CancelReservation → Refunds balance (status=CANCELLED)
```

### Saga Pattern (order-validator)

```
Temporal Workflow:
1. ReserveBalance (COMPENSATABLE)
2. PlaceBet (COMPENSATABLE)
3. CommitReservation (FINAL - NO COMPENSATION)
4. ConfirmBet (IDEMPOTENT - best effort)

On failure at any step: Automatic compensation
```

## Contributing

### Adding a New RPC Method

1. Add the method to the appropriate `.proto` file
2. Run `buf lint` to ensure it passes linting
3. Run `buf breaking` to check for breaking changes
4. Generate code with `buf generate`
5. Update service implementation
6. Create PR with changes

### Creating a New Service

1. Create directory: `proto/{service}/v1/`
2. Add service proto file: `{service}_service.proto`
3. Update `buf.gen.yaml` if needed
4. Run `buf generate`
5. Tag release: `git tag v0.{x}.0`

## CI/CD

### Pull Request Checks

- Buf lint
- Breaking change detection (against main)
- Code generation verification

### Release Process

1. Create tag: `git tag v0.1.0`
2. Push tag: `git push origin v0.1.0`
3. GitHub Actions automatically:
   - Generates Go code
   - Creates GitHub release
   - Publishes artifacts

## License

Proprietary - TAM Platform

## Support

For questions or issues, please contact the platform team or create an issue in this repository.
