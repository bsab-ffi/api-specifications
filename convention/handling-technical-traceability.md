# Handling Technical Traceability

By defining this section within the General FFI API Conventions, we ensure a minimal and consistent mechanism for technical observability across organizational boundaries when using the FFI APIs.

## General Principles

Technical failures may occur before a request results in a usable FFI business handle such as a request identifier or response identifier. Repeated HTTP interactions within the same message exchange may also be difficult to distinguish across client and server logs. To improve troubleshooting and correlation of log entries, the FFI APIs define `X-FFI-TraceId` as a recommended HTTP header that identifies an individual HTTP interaction.

The `X-FFI-TraceId` header identifies a single HTTP interaction. When the header is included, the client MUST generate a new UUID for each HTTP request. The value MUST NOT be reused for retries or other HTTP requests within the same FFI message exchange. It MUST NOT be used for business correlation, authorization, message identity, idempotency, or contractual validation.

## Header Definition

- Header name: `X-FFI-TraceId`
- Direction: Request header from client and optional response header from server
- Purpose: Technical observability for the current HTTP interaction only
- Value format: UUID
- Example: `4bf92f35-77b3-4da6-a3ce-929d0e0e4736`

## Client Responsibilities

- FFI Clients SHOULD send the `X-FFI-TraceId` header in requests to the FFI Core API and the FFI Contract API.
- When sent, FFI Clients MUST generate a new UUID for each HTTP request, including retries and repeated requests within the same FFI message exchange.
- When sent, the value MUST conform to the trace identifier format defined in the API specifications.
- FFI Clients MUST NOT rely on `X-FFI-TraceId` for business-level message correlation or any contractual semantics.

## Server Responsibilities

- FFI Servers SHOULD accept the `X-FFI-TraceId` header when it is present in a request.
- When a request includes `X-FFI-TraceId`, FFI Servers SHOULD return the same value in the corresponding HTTP response.
- When a request does not include `X-FFI-TraceId`, FFI Servers MAY generate a new UUID for that HTTP interaction and return it in the HTTP response.
- FFI Servers MAY omit the response header in degraded or infrastructure-limited environments, such as when gateways, proxies, legacy platforms, or framework boundaries prevent reliable handling of the header.
- FFI Servers MUST NOT use `X-FFI-TraceId` as a substitute for `requestId`, `responseId`, authentication data, authorization decisions, or schema validation behavior.

## Scope of Use

The `X-FFI-TraceId` header is intended to support troubleshooting across parties for all HTTP interactions, including successful responses and synchronous error responses such as `400`, `403`, `404`, `429`, and `500`.

Lack of echoing must not change business semantics or cause protocol-level rejection. Absence of the header must not invalidate an otherwise valid FFI request or response.

This convention does not define any persistence requirements, business workflow semantics, or linkage rules between `X-FFI-TraceId` and FFI message lifecycle identifiers.
