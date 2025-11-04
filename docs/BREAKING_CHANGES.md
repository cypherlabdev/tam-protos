# Breaking Changes Log

This document tracks all breaking changes made to proto definitions.

## Unreleased

(No breaking changes yet - all services on v1)

## v1.0.0 (2025-01-15)

### Initial Release

- Initial proto definitions for all 7 services
- Common types established (Money, Status, Pagination, Errors)
- Versioning strategy defined
- Non-breaking change guidelines documented

### Services Included

#### wallet-service (wallet.v1)
- GetBalance
- ReserveBalance
- CommitReservation
- CancelReservation
- CreditBalance
- GetTransactionHistory

#### user-service (user.v1)
- Login
- ValidateSession
- GetSession
- RefreshSession
- CreateSession

#### order-validator (order.v1)
- PlaceBet (saga orchestrator)

#### order-book (orderbook.v1)
- PlaceBet
- CancelBet
- SettleBet
- GetBetStatus

#### notification-service (notification.v1)
- SendNotification
- GetNotificationHistory

#### risk-analyzer (risk.v1)
- CheckBet
- GetRiskProfile

#### reporting-service (reporting.v1)
- GetDailyStats
- GetUserStats
- GenerateReport

### Design Decisions

1. **Decimal as String**: All money amounts use `string` type to avoid floating-point precision issues
2. **Idempotency Keys**: All write operations require `idempotency_key` field
3. **Saga Support**: Distributed transactions track `saga_id` for correlation
4. **2PC Reservations**: wallet-service uses Reserve → Commit/Cancel pattern
5. **Enum Prefix**: All enums use service-specific prefix (e.g., `BET_STATUS_*`)

---

## Future Breaking Changes

When a breaking change is necessary, it will be documented here before the v2 release.

### Example Format:

```
## v2.0.0 (YYYY-MM-DD)

### wallet-service

**BREAKING**: Changed `GetBalanceResponse.balance` from `string` to `Money` message

**Migration Guide**:
```go
// Before (v1)
resp.Balance  // "100.50"

// After (v2)
resp.Balance.Amount    // "100.50"
resp.Balance.Currency  // "EUR"
```

**Reason**: Provide currency information alongside balance

**Migration Timeline**:
- Week 1-2: Deploy v2, support both v1 and v2
- Week 3-4: Migrate all clients to v2
- Week 5-6: Deprecate v1 support
- Week 7+: Remove v1 implementation
```

---

## Guidelines

When documenting breaking changes:

1. **Clearly mark as BREAKING**
2. **Provide migration guide with code examples**
3. **Explain the reason for the change**
4. **Define migration timeline**
5. **List affected services**
6. **Tag the issue/PR that introduced the change**

## Deprecation Policy

- **Deprecation notice**: Minimum 4 weeks before removal
- **Dual support period**: Minimum 2 weeks running both v1 and v2
- **Migration assistance**: Platform team available for support
- **Monitoring**: Track v1 usage to ensure safe migration

## Questions?

Contact the platform team before making any breaking changes.
