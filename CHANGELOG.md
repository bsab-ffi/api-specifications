# Changelog – FFI API Specification Repository

This changelog documents updates to the public `ffi` repository, which contains synchronized API specifications from:

- FFI Core API (`api/ffi-core.yaml`)
- FFI Contract API (`api/ffi-contract.yaml`)
- FFI API Conventions (`convention/`)

Each repository release is dated (YYYY-MM-DD) and reflects the latest approved changes published by BSAB.

> Note: Individual API specification files follow [Semantic Versioning](https://semver.org/) (e.g., `1.2.0`). The repository itself uses calendar versioning (e.g., `2025-03-15`) to represent public release dates.

---

## [2025-05-13]

**Initial publication of public FFI API specifications**

- Added `api/ffi-core.yaml` (version: `3.2.0`)
- Added `api/ffi-contract.yaml` (version: `1.0.0`)
- Added `convention/README.md` – shared API usage requirements and rules
- Added `configuration/default.yaml` – default federation parameters referenced by conventions
- Added `LICENSE.md` – outlines usage restrictions and implementation terms
- Added `README.md` – describes repository structure and governance model
- Added `doc/GETTING_STARTED.md` – step-by-step usage guide with roles, flows, and conventions
- Added `SECURITY.md` – defines how to report vulnerabilities and security scope

---

## [2025-10-22]

**Incremental publication of public FFI API specifications**

- Added `api/ffi-core.yaml` (version: `3.3.0`)
- Added `convention/handling-bilateral-contracts.md` – governs the lifecycle for bilateral contracts
- Added `convention/handling-timestamps-and-lifecycle-events.md` – manage, interpret, and audit the lifecycle of message exchange
- Updated `convention/handling-api-security.md` – ensuring compliance with authentication, authorization, and encryption
- Updated `convention/handling-connectivity-check.md` – enables the verification of the connectivity
- Updated `convention/handling-parallel-major-versions.md` – handle new versions of the API in an efficient and predictable manner
- Updated `convention/handling-response-polling.md` – handle polling in a structured and scalable manner
- Updated `convention/handling-schema-validation.md` – handle payload validation in a structured and consistent manner
- Updated `convention/README.md` – referencing the FFI API conventions
- Updated `configuration/default.yaml` – default federation parameters referenced by conventions
- Updated `README.md` – describes repository structure and governance model
- Updated `doc/GETTING_STARTED.md` – step-by-step usage guide with roles, flows, and conventions
- Updated `CHANGELOG.md` – this document

---

## [2026-04-03]

**Incremental publication of public FFI API specifications**

- Updated `api/ffi-core.yaml` (version: `4.0.1`) – introduced relative paths with `/core/v4` server base path, replaced `BearerAuth` with global `OAuth2`, added required `X-FFI-ClientId` header support, added `500` responses, and clarified payload and schema validation semantics
- Updated `api/ffi-contract.yaml` (version: `2.0.1`) – introduced relative path `/contract` with `/contract/v2` server base path, replaced `BearerAuth` with global `OAuth2`, added required `X-FFI-ClientId` header support, added `500` responses, and clarified contract activation behavior
- Updated `convention/handling-api-security.md` – aligned security requirements to OAuth 2.0 Client Credentials with mutual TLS and documented usage rules for `client_id` and `X-FFI-ClientId`
- Updated `convention/handling-bilateral-contracts.md` – clarified that ongoing message exchanges remain governed by the contract active when the exchange was initiated
- Updated `convention/handling-schema-validation.md` – clarified layered validation semantics, including deferred validation of inner content and schema version interpretation for business content
- Updated `CHANGELOG.md` – this document

---

## [2026-11-27]

**Major update of the FFI API specifications and shared conventions**

### Breaking API and protocol changes

- Updated `api/ffi-core.yaml` to version `5.0.0` and `api/ffi-contract.yaml` to version `3.0.0`.
- Replaced schema-only versioning with message type versioning throughout both APIs. `X-FFI-MessageTypeSchemaVersion`, `MessageTypeSchemaVersion`, and `schemaVersions` are replaced by `X-FFI-MessageTypeVersion`, `MessageTypeVersion`, and `messageTypeVersions`. A message type version now identifies the complete contractual requirement set, including schema, interpretation, validation, and processing expectations.
- Introduced a unified `ErrorPayload` model for synchronous transport errors and asynchronous technical processing errors. The model permits only the optional `type` and `detail` properties, allows an empty object, and deliberately does not define a shared catalog of machine-readable FFI error types.
- Changed response acknowledgment from `X-FFI-AcknowledgeStatus: SUCCEEDED | FAILED` to `X-FFI-ProcessingResult: SUCCESS | ERROR`. A successful acknowledgment has no request body; when a client detects a validation error after retrieving an individual response, it reports the error through the acknowledgment endpoint with `X-FFI-ProcessingResult: ERROR` and an `ErrorPayload` body in `application/json`.
- Clarified successful and failed response retrieval. `X-FFI-ProcessingResult: SUCCESS` returns the message payload and its message type version, while `ERROR` returns an `ErrorPayload` instead of a business response payload. The JSON response models these alternatives with `anyOf`, while `X-FFI-ProcessingResult` determines which payload is expected.
- Clarified `MessagePayload` transport semantics for successful responses. The declared message type and version define whether the payload uses `application/json` or `application/xml`, while the `Content-Type` response header communicates the actual format and the Core API supports both. Generic response payload examples have been removed because the payload structure is defined by the declared message type and version and cannot be represented accurately by a Core API example. The transport description does not assume direct validation of wrapped or encrypted inner content.
- Removed the explicit `406 Not Acceptable` response from response retrieval. Added `413 Payload Too Large` when a response exceeds the configured threshold and the client has not declared gzip support.

### Traceability and error handling

- Added the optional `X-FFI-TraceId` request and response header to Core and Contract API interactions. The identifier is a UUID for one HTTP interaction and may be echoed in the response to correlate client-side and server-side technical logs.
- Explicitly prohibited using `X-FFI-TraceId` for business correlation, authorization, message identity, idempotency, or contractual validation.
- Added `convention/handling-runtime-errors.md`, covering synchronous and asynchronous technical errors, processing-result semantics, retry behavior, circuit breakers, timeouts, escalation, and operational responsibilities.
- Added runtime-error defaults for response timeout, initial retry delay, exponential backoff, retry limit, circuit-breaker threshold, and circuit-breaker cooldown period in `configuration/default.yaml`.
- Clarified operation-specific `403 Forbidden` semantics and retained bodyless authorization failures where no `ErrorPayload` is defined.
- Clarified contextual `404 Not Found` semantics so that responses do not reveal whether a protected request, response, or contract exists but is unavailable to the caller.

### Identity and bilateral contracts

- Added a reusable `EntityId` schema to the Contract API and used it for `consumerId` and `providerId`. The schema identifies the legal entities participating in a bilateral relationship, follows the FFI-defined identity naming convention rather than claiming RFC 1035 conformance, and preserves case-insensitive comparison.
- Added `convention/handling-entity-relationships.md`, defining the relationships between legal entities, technical implementations, consumer credentials, OAuth ClientIds, and bilateral contracts. Contract validation applies when a new message exchange is initiated against the active bilateral contract for the Consumer–Provider EntityId pair.
- Clarified bilateral contract date semantics: `createDate` records when the relationship was first created and remains unchanged, while `modifiedDate` identifies when an immutable contract representation became active. The active contract is the representation with the latest `modifiedDate`.
- Clarified that replacement of a bilateral contract does not invalidate an ongoing message exchange. The exchange continues under the contract and message type version that were active when its first message was received.
- Clarified federative and bilateral message type transition periods, including that the bilateral period begins when the replacement contract is published and must complete inside the federative transition window.

### Conventions, configuration, and supporting schemas

- Removed `convention/handling-parallel-major-versions.md` and the shared `transitionPeriod` and `centralizedTestingEnvironment` configuration parameters. API-major-version and FFI-release transition rules are federation-specific and are no longer prescribed by these common conventions.
- Renamed the message type transition parameters to `federativeMessageTypeTransitionPeriod` and `bilateralMessageTypeTransitionPeriod`. The federative value remains `P12M`, while the maximum bilateral value is `P3M` for an individual consumer-provider pair.
- Clarified OAuth scope handling by stating that scope parameters may be used to differentiate access between FFI APIs.
- Clarified that examples referring to connectivity verification do not imply the existence of a centrally operated Core API sandbox.
- Updated connectivity-check schemas so `connectivityCheckTimestamp` is an RFC 3339 `date-time`, and removed the separate connectivity-check error schema in favor of the unified error model.
- Clarified implicit and explicit `Content-Type` behavior and aligned compression guidance with the new `413 Payload Too Large` response.
- Updated `doc/GETTING_STARTED.md`, convention indexes, examples, and federation configuration documentation to reflect the new headers, error flow, version terminology, identity model, and transition rules.
- Updated `CHANGELOG.md` – this document.

---

© 2025 BSAB – All rights reserved.
