# tam-protos - Ticket Tracking

## Service Overview
**Repository**: github.com/cypherlabdev/tam-protos
**Purpose**: Shared Protocol Buffer definitions for all cypherlab platform microservices
**Implementation Status**: 100% complete (proto definitions)
**Language**: Protocol Buffers v3, Go code generation
**Critical Blockers**: None - foundation service is complete

## Current Implementation

### ✅ Completed
- All proto definitions for 7 microservices (wallet, user, order-validator, order-book, notification, risk, reporting)
- Common types, pagination, and error definitions
- Buf configuration for linting and breaking change detection
- Go code generation with buf.gen.yaml
- GitHub Actions workflows for validation and publishing
- Comprehensive documentation (README.md, VERSIONING.md, BREAKING_CHANGES.md)
- Go module setup (github.com/cypherlabdev/cypherlabdev-protos)

### ⚠️ Branding Issues
- README.md references "TAM" instead of "cypherlab" in some places
- Module name inconsistency: go.mod uses "cypherlabdev-protos" but imports use "tam-protos"
- Import paths in .proto files use "github.com/tam/tam-protos" instead of "github.com/cypherlabdev/tam-protos"

### ❌ Missing
- No Asana ticket exists to track this foundational work
- Python proto code generation not configured
- No proto definitions for odds-related services (odds-adapter has custom structs instead)
- No NATS/Kafka event message schemas defined

## Existing Asana Tickets

### ⚠️ NO MAIN TICKET EXISTS

There is currently **no Asana ticket** tracking the tam-protos repository work. This is a foundational service that all other services depend on.

**Recommendation**: Create a parent epic ticket for Protocol Buffer Infrastructure

## Tickets to Create in Asana

### 1. [NEW] Protocol Buffer Infrastructure (Epic)
**Proposed Priority**: P0 (Foundation)
**Type**: feature
**Assignee**: sj@cypherlab.tech
**Labels**: Backend, Infrastructure, tam-protos
**Dependencies**: None (foundation service)

**Rationale**: tam-protos is the foundational service that all other microservices depend on for gRPC communication. It requires an Asana ticket to track ongoing maintenance, versioning, and breaking change management.

**Acceptance Criteria**:
- [x] Proto definitions for all 7 core services
- [x] Buf lint and breaking change detection
- [x] Go code generation configured
- [x] GitHub Actions CI/CD pipeline
- [ ] Python code generation configured
- [ ] Documented versioning strategy
- [ ] Proto definitions for odds-related events

**Implementation Status**:
- Core proto definitions: ✅ Complete
- Buf tooling: ✅ Complete
- CI/CD: ✅ Complete
- Multi-language support: ⚠️ Only Go supported

---

### 2. [NEW] Fix Branding and Module Path Consistency
**Proposed Priority**: P1
**Type**: bug
**Assignee**: sj@cypherlab.tech
**Labels**: Backend, tam-protos
**Dependencies**: None

**Rationale**: Import paths and module names are inconsistent, referencing both "TAM" and "cypherlab", and mixing "tam-protos" with "cypherlabdev-protos". This causes confusion and potential import issues.

**Acceptance Criteria**:
- [ ] Update all "TAM" references in README.md to "cypherlab"
- [ ] Standardize on one module path (github.com/cypherlabdev/tam-protos)
- [ ] Update go.mod to use consistent naming
- [ ] Update .proto files option go_package to use correct path
- [ ] Regenerate all Go code with corrected paths
- [ ] Update documentation

**Technical Details**:
- Current: `option go_package = "github.com/tam/tam-protos/gen/go/wallet/v1;walletv1"`
- Should be: `option go_package = "github.com/cypherlabdev/tam-protos/gen/go/wallet/v1;walletv1"`
- go.mod currently: `module github.com/cypherlabdev/cypherlabdev-protos`
- Should be: `module github.com/cypherlabdev/tam-protos`

---

### 3. [NEW] Python Proto Code Generation
**Proposed Priority**: P2
**Type**: feature
**Assignee**: sj@cypherlab.tech
**Labels**: Backend, tam-protos
**Dependencies**: [ENG-TBD] Protocol Buffer Infrastructure

**Rationale**: risk-analyzer-service and reporting-service are Python services (FastAPI) that need to communicate with Go services via gRPC. Python proto generation is required for these services.

**Acceptance Criteria**:
- [ ] Add Python plugin to buf.gen.yaml
- [ ] Configure gen/python/ output directory
- [ ] Add Python proto requirements to documentation
- [ ] Update GitHub Actions to generate Python code
- [ ] Add Python code to release artifacts
- [ ] Document Python import usage in README.md

**Technical Details**:
```yaml
# Add to buf.gen.yaml
- plugin: python
  out: gen/python
  opt:
    - paths=source_relative
- plugin: python-grpc
  out: gen/python
  opt:
    - paths=source_relative
```

---

### 4. [NEW] Event Message Schema Definitions
**Proposed Priority**: P1
**Type**: feature
**Assignee**: sj@cypherlab.tech
**Labels**: Backend, tam-protos, Events
**Dependencies**: [ENG-TBD] Protocol Buffer Infrastructure

**Rationale**: Multiple services publish events to Kafka/NATS (user login, order placed, bet settled, etc.) but currently use custom JSON structures. Standardized proto definitions ensure type safety and schema evolution.

**Acceptance Criteria**:
- [ ] Create proto/events/v1/ directory structure
- [ ] Define user events (user_created, user_logged_in, user_logged_out, password_changed)
- [ ] Define order events (order_placed, order_validated, order_settled, order_cancelled)
- [ ] Define wallet events (balance_reserved, balance_committed, balance_credited)
- [ ] Define notification events (notification_sent, email_sent, sms_sent)
- [ ] Define odds events (odds_updated, odds_published)
- [ ] Generate code for all languages
- [ ] Document event schemas in README.md

**Technical Details**:
- Events should include: event_id, event_type, timestamp, saga_id (if applicable), payload
- Follow CloudEvents specification where applicable
- Support event versioning (v1, v2, etc.)

---

### 5. [NEW] Odds Service Proto Definitions
**Proposed Priority**: P2
**Type**: feature
**Assignee**: sj@cypherlab.tech
**Labels**: Backend, tam-protos, Odds Management
**Dependencies**: [ENG-66] Odds Adapter

**Rationale**: odds-adapter-service, data-normalizer-service, and odds-optimizer-service currently use custom Go structs instead of proto definitions. This prevents type-safe gRPC communication and makes integration harder.

**Acceptance Criteria**:
- [ ] Create proto/odds/v1/odds_service.proto
- [ ] Define OddsAdapterService with methods (GetProviderOdds, StreamOdds)
- [ ] Define NormalizerService with methods (NormalizeOdds, GetNormalizedOdds)
- [ ] Define OptimizerService with methods (OptimizeOdds, GetOptimizedOdds, CacheOdds)
- [ ] Define message types for raw odds, normalized odds, optimized odds
- [ ] Include provider metadata (source, timestamp, confidence)
- [ ] Generate code for all languages
- [ ] Update services to use proto definitions

**Technical Details**:
```protobuf
service OddsAdapterService {
    rpc GetProviderOdds(GetProviderOddsRequest) returns (GetProviderOddsResponse);
    rpc StreamOdds(StreamOddsRequest) returns (stream OddsUpdate);
}

service DataNormalizerService {
    rpc NormalizeOdds(NormalizeOddsRequest) returns (NormalizeOddsResponse);
    rpc GetNormalizedOdds(GetNormalizedOddsRequest) returns (GetNormalizedOddsResponse);
}

service OddsOptimizerService {
    rpc OptimizeOdds(OptimizeOddsRequest) returns (OptimizeOddsResponse);
    rpc GetOptimizedOdds(GetOptimizedOddsRequest) returns (GetOptimizedOddsResponse);
}
```

---

### 6. [NEW] Proto Versioning Documentation
**Proposed Priority**: P2
**Type**: documentation
**Assignee**: sj@cypherlab.tech
**Labels**: Backend, tam-protos, Documentation
**Dependencies**: [ENG-TBD] Protocol Buffer Infrastructure

**Rationale**: docs/VERSIONING.md exists but needs comprehensive guidelines for proto evolution, breaking changes, and migration strategies for a production betting platform.

**Acceptance Criteria**:
- [ ] Document backward compatibility rules
- [ ] Define breaking vs non-breaking changes
- [ ] Establish version numbering scheme (v1, v2, etc.)
- [ ] Migration strategy for breaking changes
- [ ] Deprecation policy (how long to maintain old versions)
- [ ] Testing strategy for multiple versions
- [ ] Examples of safe proto evolution

**Technical Details**:
- Non-breaking: Adding optional fields, adding new services, adding new RPCs
- Breaking: Removing fields, changing field types, changing field numbers, removing services
- Recommendation: Maintain N-1 version support for rolling deployments

## Implementation Gaps Summary

### P0 - Critical
None - core proto definitions are complete and functional

### P1 - High Priority
1. **Branding and Module Path Consistency** - Causes confusion and potential import errors
2. **Event Message Schema Definitions** - Required for proper Kafka/NATS event publishing across all services

### P2 - Medium Priority
1. **Python Proto Code Generation** - Needed for Python services (risk-analyzer, reporting)
2. **Odds Service Proto Definitions** - Would improve type safety and gRPC communication for odds pipeline
3. **Proto Versioning Documentation** - Important for long-term maintenance

## Dependency Graph

```
tam-protos (this service)
├── Blocks: ALL services (foundation)
│   ├── [ENG-90] user-service
│   ├── [ENG-86] wallet-service
│   ├── [ENG-66] odds-adapter
│   ├── [ENG-70] data-normalizer
│   ├── [ENG-74] odds-optimizer
│   ├── [ENG-82] order-book
│   ├── [ENG-78] order-validator
│   ├── [ENG-94] notification-service
│   ├── [ENG-98] risk-analyzer
│   └── [ENG-102] reporting-service
└── Blocked By: None (foundation service)
```

## Notes

- **Module Path Issue**: The current inconsistency between go.mod (`cypherlabdev-protos`) and proto imports (`tam/tam-protos`) should be resolved before creating the main Asana ticket.
- **Foundation Service**: This is a Level 1 dependency - all other services depend on it for gRPC communication.
- **Auto-generated ENG Field**: Asana will auto-assign ENG numbers when tickets are created.
- **All Services Import**: Every Go service has `import` statements referencing these proto definitions.

---

*Last Updated*: 2025-11-05
*Next Review*: When adding new services or making breaking changes
