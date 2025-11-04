# Proto Versioning Strategy

## Overview

This document describes the versioning strategy for TAM proto definitions. We follow semantic versioning principles adapted for Protocol Buffers.

## Version Scheme

Proto packages use the format: `{service}.v{major}`

Examples:
- `wallet.v1`
- `user.v1`
- `order.v1`

## Non-Breaking Changes (Safe in v1)

The following changes are **non-breaking** and can be made within the same major version:

### ✅ Adding Optional Fields

```protobuf
message GetBalanceResponse {
    string user_id = 1;
    string balance = 2;
    string currency = 3;
    // NEW: Adding optional field is safe
    string last_updated = 4;
}
```

### ✅ Adding New RPC Methods

```protobuf
service WalletService {
    rpc GetBalance(GetBalanceRequest) returns (GetBalanceResponse);
    // NEW: Adding new method is safe
    rpc GetBalanceHistory(GetHistoryRequest) returns (GetHistoryResponse);
}
```

### ✅ Adding New Enum Values

**IMPORTANT:** Always add new enum values at the end, never in the middle.

```protobuf
enum Status {
    STATUS_UNSPECIFIED = 0;  // Required
    STATUS_PENDING = 1;
    STATUS_COMPLETED = 2;
    STATUS_FAILED = 3;
    // NEW: Adding at the end is safe
    STATUS_CANCELLED = 4;
}
```

### ✅ Adding New Messages

```protobuf
// NEW: Adding entirely new messages is safe
message RefundRequest {
    string user_id = 1;
    string amount = 2;
}
```

### ✅ Deprecating Fields (without removing)

```protobuf
message User {
    string id = 1;
    string email = 2;
    string name = 3 [deprecated = true];  // Mark as deprecated
    string first_name = 4;  // New replacement field
    string last_name = 5;   // New replacement field
}
```

## Breaking Changes (Require v2)

The following changes are **breaking** and require a new major version:

### ❌ Changing Field Types

```protobuf
// v1
message GetBalanceResponse {
    string balance = 2;  // String
}

// v2 (BREAKING)
message GetBalanceResponse {
    double balance = 2;  // Changed to double - BREAKING
}
```

### ❌ Removing Fields

```protobuf
// v1
message User {
    string id = 1;
    string email = 2;
    string name = 3;  // Field present
}

// v2 (BREAKING)
message User {
    string id = 1;
    string email = 2;
    // name removed - BREAKING
}
```

### ❌ Changing Field Numbers

```protobuf
// v1
message User {
    string id = 1;
    string email = 2;
}

// v2 (BREAKING)
message User {
    string id = 2;     // Changed from 1 - BREAKING
    string email = 1;  // Changed from 2 - BREAKING
}
```

### ❌ Renaming Fields

```protobuf
// v1
message User {
    string user_name = 1;
}

// v2 (BREAKING)
message User {
    string username = 1;  // Renamed - BREAKING
}
```

**Note:** Field names can be changed in the proto file without breaking wire compatibility, but they break generated code APIs, so they're considered breaking changes.

### ❌ Changing RPC Signatures

```protobuf
// v1
rpc GetBalance(GetBalanceRequest) returns (GetBalanceResponse);

// v2 (BREAKING)
rpc GetBalance(GetBalanceRequestV2) returns (GetBalanceResponse);  // BREAKING
```

### ❌ Removing RPC Methods

```protobuf
// v1
service WalletService {
    rpc GetBalance(GetBalanceRequest) returns (GetBalanceResponse);
    rpc GetHistory(GetHistoryRequest) returns (GetHistoryResponse);
}

// v2 (BREAKING)
service WalletService {
    rpc GetBalance(GetBalanceRequest) returns (GetBalanceResponse);
    // GetHistory removed - BREAKING
}
```

## Version Bump Process

When a breaking change is needed:

### 1. Create v2 Directory

```bash
cd proto/wallet
mkdir v2
cp v1/*.proto v2/
```

### 2. Update Package Declaration

```protobuf
// Old: proto/wallet/v1/wallet_service.proto
syntax = "proto3";
package wallet.v1;
option go_package = "github.com/tam/tam-protos/gen/go/wallet/v1;walletv1";

// New: proto/wallet/v2/wallet_service.proto
syntax = "proto3";
package wallet.v2;
option go_package = "github.com/tam/tam-protos/gen/go/wallet/v2;walletv2";
```

### 3. Make Breaking Changes

```protobuf
// wallet/v2/wallet_service.proto
service WalletService {
    rpc GetBalance(GetBalanceRequestV2) returns (GetBalanceResponseV2);
    // ... breaking changes here
}
```

### 4. Update buf.gen.yaml (if needed)

Both v1 and v2 will be generated:
```yaml
version: v1
plugins:
  - plugin: go
    out: gen/go
    opt:
      - paths=source_relative
```

### 5. Migrate Services Gradually

```
Week 1-2: wallet-service supports both v1 and v2
Week 3-4: Migrate clients to v2
Week 5-6: Deprecate v1 support
Week 7+:  Remove v1 implementation
```

## Buf Breaking Change Detection

We use Buf to automatically detect breaking changes:

```bash
# Check for breaking changes against main branch
buf breaking --against '.git#branch=main'
```

This runs in CI on every pull request.

## Best Practices

### 1. Always Use _UNSPECIFIED for Enums

```protobuf
enum Status {
    STATUS_UNSPECIFIED = 0;  // REQUIRED - default value
    STATUS_ACTIVE = 1;
    STATUS_INACTIVE = 2;
}
```

### 2. Reserve Deprecated Field Numbers

```protobuf
message User {
    reserved 3;                // Reserved field number
    reserved "old_name";       // Reserved field name

    string id = 1;
    string email = 2;
    // Field 3 was "name", now deprecated
    string first_name = 4;
    string last_name = 5;
}
```

### 3. Use Deprecated Tag Instead of Removing

```protobuf
message User {
    string id = 1;
    string name = 2 [deprecated = true];  // Don't remove, deprecate
    string first_name = 3;
    string last_name = 4;
}
```

### 4. Plan for Extensions

Leave gaps in field numbers for future additions:

```protobuf
message User {
    string id = 1;
    string email = 2;
    string name = 3;
    // Fields 4-9 reserved for future user info
    string created_at = 10;
    // Fields 11-19 reserved for future metadata
}
```

### 5. Document Breaking Changes

Update [BREAKING_CHANGES.md](BREAKING_CHANGES.md) when bumping to v2.

## Migration Example

### Before (v1)

```protobuf
// wallet/v1/wallet_service.proto
service WalletService {
    rpc GetBalance(GetBalanceRequest) returns (GetBalanceResponse);
}

message GetBalanceResponse {
    string balance = 1;  // String representation
}
```

### After (v2)

```protobuf
// wallet/v2/wallet_service.proto
service WalletService {
    rpc GetBalance(GetBalanceRequest) returns (GetBalanceResponse);
}

message GetBalanceResponse {
    Money balance = 1;  // Changed to Money type - BREAKING
}

message Money {
    string amount = 1;
    string currency = 2;
}
```

### Service Implementation During Migration

```go
// wallet-service supports both v1 and v2
func (s *Server) GetBalanceV1(ctx context.Context, req *v1.GetBalanceRequest) (*v1.GetBalanceResponse, error) {
    balance := s.getBalance(req.UserId)
    return &v1.GetBalanceResponse{
        Balance: balance.Amount,  // Return string
    }, nil
}

func (s *Server) GetBalanceV2(ctx context.Context, req *v2.GetBalanceRequest) (*v2.GetBalanceResponse, error) {
    balance := s.getBalance(req.UserId)
    return &v2.GetBalanceResponse{
        Balance: &v2.Money{
            Amount:   balance.Amount,
            Currency: balance.Currency,
        },
    }, nil
}
```

## Questions?

Contact the platform team for clarification on versioning decisions.
