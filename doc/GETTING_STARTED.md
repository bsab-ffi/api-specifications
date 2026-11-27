## Getting Started – Using the FFI APIs

This guide explains how to use the FFI APIs depending on your role in the infrastructure:

- The **Contract API** – Retrieve your bilateral contract  
- The **Core API** – Send, receive, and acknowledge messages  
- The **Conventions** – Ensure consistency, validation, security and compliance (see [Required Conventions](#required-conventions))

All usage is governed by BSAB. See [LICENSE.md](../LICENSE.md) for terms.

---

## Two Roles: Requestor (Client) and Respondent (Server)

Each participating entity acts either as:

- **FFI Requestor (Client)** – initiates message-based communication  
- **FFI Respondent (Server)** – receives and processes requests, returns responses  

Depending on the federation, an entity may need to implement **both roles**.

---

## Scenario: Acting as a Client (FFI Requestor)

As a Requestor, your system must:  
1. Retrieve the contract  
2. Submit a request  
3. Poll for the corresponding response
4. Retrieve the response   
5. Acknowledge response

See [The Happy Flow – Request to Response Acknowledgment](#the-happy-flow--request-to-response-acknowledgment) for step-by-step details including headers, payloads, and conventions.

---

## Scenario: Acting as a Server (FFI Respondent)

As a Respondent, your system must:  
1. Expose the Contract API  
2. Accept and validate incoming requests  
3. Provide responses  
4. Receive acknowledgments

See [Required Conventions](#required-conventions) for responsibilities on schema validation, security, and availability handling.

---

## About the Version Placeholders

The `<core-api-major-version>` and `<contract-api-major-version>` parts in the URLs are **placeholders** for the actual **major versions** of each respective API.  

The major version is exposed in the runtime path as `/core/vN` or `/contract/vN`, where `N` identifies a technically incompatible version of the respective API. An FFI release does not introduce a separate runtime version identifier. Each federation must establish any rules for transitioning between API major versions or FFI releases and communicate them to its participants.

---

## Required Conventions

| Area                   | Convention |
|------------------------|------------|
| Security & Auth        | [handling-api-security.md](../convention/handling-api-security.md) |
| Bilateral contracts    | [handling-bilateral-contracts.md](../convention/handling-bilateral-contracts.md) |
| Entity relationships   | [handling-entity-relationships.md](../convention/handling-entity-relationships.md) |
| Message exchange       | [handling-timestamps-and-lifecycle-events.md](../convention/handling-timestamps-and-lifecycle-events.md) |
| Schema validation      | [handling-schema-validation.md](../convention/handling-schema-validation.md) |
| Technical traceability | [handling-technical-traceability.md](../convention/handling-technical-traceability.md) |
| Polling & retry        | [handling-response-polling.md](../convention/handling-response-polling.md) |
| Runtime errors         | [handling-runtime-errors.md](../convention/handling-runtime-errors.md) |
| Compression            | [handling-compression-and-large-responses.md](../convention/handling-compression-and-large-responses.md) |
| Connectivity check     | [handling-connectivity-check.md](../convention/handling-connectivity-check.md) |

---

## The Happy Flow – Request to Response Acknowledgment

This section describes the typical end-to-end flow for the FFI Core API, from sending a request to acknowledging the response. Each step includes required headers, payload expectations, and behavioral outcomes.

FFI-specific headers are defined and governed by FFI conventions, while standard HTTP headers (like `Authorization` or `Accept-Encoding`) follow standard protocols.

### Sequence Diagram – Client Flow

```mermaid
sequenceDiagram
    participant FFI Client
    participant FFI Server

    Note over FFI Client,FFI Server: Client sends a request
    FFI Client->>FFI Server: POST /requests
    Note right of FFI Server: Server accepts and stores the request
    FFI Server-->>FFI Client: 202 Accepted

    Note over FFI Client,FFI Server: Client is polling for available responses
    loop Polling until expected response is available
        FFI Client->>FFI Server: GET /responses
        Note right of FFI Server: Server returns list of available responseIds
        FFI Server-->>FFI Client: 200 OK
    end

    Note over FFI Client,FFI Server: Client retrieves the full response message
    FFI Client->>FFI Server: GET /responses/{responseId}
    Note right of FFI Server: Server returns the response payload
    FFI Server-->>FFI Client: 200 OK

    Note over FFI Client,FFI Server: Client acknowledges valid response
    FFI Client->>FFI Server: POST /responses/{responseId}/acknowledge
    Note right of FFI Server: Server marks request and response as completed<br>and removes them from availability
    FFI Server-->>FFI Client: 200 OK
```

### POST /requests – Send a Request

- Requires: `X-FFI-MessageType`, `X-FFI-MessageTypeVersion`, `X-FFI-ClientId`
- Recommended header: `X-FFI-TraceId` for technical observability; when included, the server should return the same value in the corresponding response
- Auth: OAuth2 access token   
- Body: JSON or XML payload matching the schema derived from the declared message type and message type version
- Response: `202 Accepted` with tracking headers and, when returned, the same `X-FFI-TraceId`

See: [Schema Validation Convention](../convention/handling-schema-validation.md)

---

### GET /responses – Poll for Responses

- Requires: `X-FFI-ClientId`  
- Recommended header: `X-FFI-TraceId` for technical observability; when included, the server should return the same value in the corresponding response
- Auth: OAuth2 access token   
- Response: List of available responseIds with metadata  
- Constrained by polling rules

See: [Polling Convention](../convention/handling-response-polling.md)

---

### GET /responses/{responseId} – Retrieve the Response

- Requires: `X-FFI-ClientId`  
- Recommended header: `X-FFI-TraceId` for technical observability; when included, the server should return the same value in the corresponding response
- Auth: OAuth2 access token   
- Optional header: `Accept-Encoding: gzip`  
- Response body depends on `X-FFI-ProcessingResult`
    - If `SUCCESS`, the payload conforms to the response schema associated with the message type and message type version for the exchange.
    - If `ERROR`, the payload is an `ErrorPayload` in `application/json`.

See: [Schema Validation Convention](../convention/handling-schema-validation.md)

---

### POST /responses/{responseId}/acknowledge – Confirm Receipt

- Header: `X-FFI-ProcessingResult: SUCCESS | ERROR`, `X-FFI-ClientId`
- Recommended header: `X-FFI-TraceId` for technical observability; when included, the server should return the same value in the corresponding response
- Auth: OAuth2 access token   
- Request body: omitted for `SUCCESS`, required as `ErrorPayload` for `ERROR`
- Marks message and request as completed and no longer retrievable

See: [Timestamps and Lifecycle Events Convention](../convention/handling-timestamps-and-lifecycle-events.md)

---

## Implementation Guidance

- Use the `ffi-core.yaml` and `ffi-contract.yaml` OpenAPI files  
- Follow conventions under `/convention/`  
- Implement security according to [handling-api-security.md](../convention/handling-api-security.md)  
- Monitor releases and version updates

---

© 2025 BSAB – All rights reserved.
