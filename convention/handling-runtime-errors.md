# Handling Runtime Errors

By following this section of the General FFI API Conventions, participants in the FFI network can ensure consistent, predictable, and interoperable behavior when technical errors occur during runtime interactions between clients and servers.

## Overview

This convention defines runtime behavior for both:

- **Synchronous transport errors** returned directly by an API endpoint using standard HTTP status codes, and
- **Asynchronous technical processing errors** reported through the Core API response flow.

It applies to APIs that use the shared FFI error model, including the Core API and the Contract API where relevant.

This convention focuses on **runtime behavior**: how errors are exposed, interpreted, acknowledged, retried, or escalated. The API specifications remain the authoritative source for endpoint-level details such as headers, request bodies, response bodies, and status codes.

## Design Principles

The FFI runtime error model follows two high-level design principles:

- For **synchronous** error reporting, `ErrorPayload` is used only where an error body is defined by the applicable API specification. Its properties are optional, and the HTTP status code and endpoint semantics remain authoritative when no additional error information is provided.
  For example, `500 Internal Server Error` does not use `ErrorPayload`, because the client cannot take any meaningful action based on additional information about internal server-side failure causes beyond following the retry and backoff strategy defined by this convention.
- For **asynchronous** error reporting, exposed technical errors are limited to errors that the client can meaningfully act on. Internal server faults, outages, dependency failures, and other non-client-actionable operational conditions are not reported through the asynchronous `ErrorPayload` model. This shields clients from non-client-actionable operational issues. Such issues **MUST** be resolved internally before a response may be created.

## Shared Error Model

The FFI specifications use a common `ErrorPayload` structure for technical error reporting where an error body is returned or exchanged.

`ErrorPayload` contains:

- `type` – an optional URI reference identifying the error type
- `detail` – an optional client-facing explanation of the specific error occurrence

All properties are optional, and an empty object is a valid `ErrorPayload`. This convention does not define a shared catalog or shared machine-readable semantics for `type`. Clients **MUST NOT** assume that `type` is present or that the same value has the same meaning across implementations.

`detail` **MAY** provide occurrence-specific information, but **MUST NOT** expose internal implementation details, secrets, stack traces, infrastructure-sensitive information, or echo original validated input values.

## Synchronous Error Handling

Synchronous errors are errors returned directly as part of the HTTP request-response interaction, for example when invoking:

- `POST /requests`
- `DELETE /requests/{requestId}`
- `GET /responses`
- `GET /responses/{responseId}`
- `POST /responses/{responseId}/acknowledge`
- `GET /contract`

### Server Behavior

- The server **MUST** use standard HTTP status codes according to the relevant API specification.
- When a synchronous error response includes a body defined by the specification, that body **MUST** use `ErrorPayload`.
- Error bodies **MUST NOT** expose internal implementation details, stack traces, infrastructure-sensitive information, or echo original validated input values.
- The server **SHOULD** log synchronous technical failures for operational monitoring, diagnostics, and incident handling.
- The server **MAY** include `Retry-After` where this is defined by the API or appropriate according to normal HTTP behavior, for example in throttling scenarios.

Typical synchronous handling includes:

- `400` for malformed input, missing required protocol elements, or schema-invalid requests where synchronous validation applies
- `401` for missing or invalid authentication credentials
- `403` for authorization or bilateral-contract related refusal
- `413` for responses that cannot be returned uncompressed because the client did not declare support for required compression
- `404` when a referenced resource does not exist
- `405` when the operation is not allowed for the addressed resource or message type
- `406` when content negotiation fails, where such behavior applies
- `429` when polling or request frequency limits are exceeded
- `500` for unexpected server-side failures and other non-client-actionable technical conditions

For `500 Internal Server Error` specifically:

- The status code may represent different internal server-side or infrastructure-related failure causes.
- These causes are **not** part of the shared client-facing `ErrorPayload` model.
- Clients **MUST NOT** expect machine-readable error classification for `500` responses.
- The only normative client behavior is to handle `500` according to the configured retry, backoff, and circuit-breaker strategy defined by this convention.

### Client Behavior

- Clients **MUST** interpret synchronous errors according to standard HTTP semantics and the relevant API specification.
- When a synchronous error body is returned, clients **MUST NOT** assume that `ErrorPayload.type` is present or has a shared machine-readable meaning across implementations.
- Clients **MUST NOT** assume that every synchronous error is retryable.
- Clients **SHOULD NOT** retry requests that fail due to client-side or contract-level issues such as `400`, `403`, `404`, `405`, or `406`, unless the underlying cause has first been corrected.
- Clients **MAY** retry `500` responses and timeouts where operational policy permits.
- Clients **MUST** respect `Retry-After` when it is present.
- Clients **SHOULD** log or otherwise retain sufficient information about repeated synchronous failures to support troubleshooting and incident handling.

### Retry and Resilience Guidance

- Clients **SHOULD** wait for the **[[Initial Delay]](#federation-specific-parameters)** before the first retry attempt unless `Retry-After` is present.
- Clients **SHOULD** use the configured **[[Backoff Strategy]](#federation-specific-parameters)** for retryable failures such as `500` responses or timeouts.
- Clients **SHOULD** stop retrying when the **[[Retry Limit]](#federation-specific-parameters)** is reached.
- Clients **SHOULD** implement a circuit breaker that temporarily suspends requests to an affected API server once the **[[Circuit Breaker Threshold]](#federation-specific-parameters)** is reached.

## Asynchronous Technical Processing Errors

Asynchronous technical processing errors are conveyed through the Core API response flow rather than through the immediate HTTP response to `POST /requests`.

In this model:

- `GET /responses/{responseId}` always returns an FFI response message envelope
- `X-FFI-ProcessingResult` indicates whether processing completed as `SUCCESS` or `ERROR`
- when `X-FFI-ProcessingResult = ERROR`, the response body is an `ErrorPayload` in `application/json`

This mechanism is intended for technical processing outcomes that arise after the original request has already been accepted for asynchronous handling.

### Server Behavior for `GET /responses/{responseId}`

- The server **MUST** include `X-FFI-ProcessingResult`.
- When `X-FFI-ProcessingResult = SUCCESS`, the body **MUST** contain the response `MessagePayload` in the media type defined by the declared message type and version. The server **MUST** communicate the actual media type through the `Content-Type` response header. The Core API supports `application/json` and `application/xml` for `MessagePayload`.
- When `X-FFI-ProcessingResult = ERROR`, the body **MUST** contain an `ErrorPayload` in `application/json`.
- When `X-FFI-ProcessingResult = ERROR`, the server **MAY** omit `X-FFI-MessageType` and `X-FFI-MessageTypeVersion` if no business response payload exists for the failed technical processing outcome.
- The server **MUST NOT** expose internal infrastructure failures in a way that leaks implementation-sensitive detail through `ErrorPayload.detail`.
- The server **MUST NOT** report internal defects, dependency outages, transient infrastructure problems, or other conditions that the client cannot meaningfully act on through the asynchronous `ErrorPayload` model.

Examples of asynchronous technical processing failures may include:

- payload validation failures detected during downstream or deferred processing
- payload security or message integrity failures detected after the original request was accepted
- other technical failures attributable to the submitted message or another client-correctable condition

### Client Behavior for `GET /responses/{responseId}`

- The client **MUST** inspect `X-FFI-ProcessingResult`.
- When `X-FFI-ProcessingResult = SUCCESS`, the client **MUST** process the body as the expected business response payload.
- When `X-FFI-ProcessingResult = ERROR`, the client **MUST** process the body as an `ErrorPayload`.
- Clients **MUST NOT** treat asynchronous technical processing errors as successful business responses.
- Clients **MUST NOT** assume that the optional `ErrorPayload.type` has a shared machine-readable meaning across implementations.
- Clients **SHOULD NOT** automatically retry the original business request unless the relevant message semantics, operational agreements, or available error information clearly allow it.
- Clients **SHOULD** route repeated or unexpected asynchronous technical errors into operational follow-up and incident handling.

## Acknowledging Asynchronous Outcomes

The client acknowledges retrieved responses through `POST /responses/{responseId}/acknowledge`.

### Client Behavior for Acknowledgment

- The client **MUST** include `X-FFI-ProcessingResult` in the acknowledgment request.
- When `X-FFI-ProcessingResult = SUCCESS`, the client **MUST NOT** send an error body.
- When `X-FFI-ProcessingResult = ERROR`, the client **MUST** send an `ErrorPayload` request body. All properties in that body are optional.

### Server Behavior for Acknowledgment

- The server **MUST** validate the acknowledgment request according to the Core API specification.
- The server **MUST** reject malformed acknowledgments using the synchronous error handling model.
- The server **SHOULD** log acknowledgments and related client-reported technical errors for traceability.
- The server **MAY** trigger incident handling or operational follow-up when repeated client-side technical processing failures are reported.

## Relationship Between Synchronous and Asynchronous Errors

The two error paths serve different purposes and **MUST NOT** be conflated:

- **Synchronous errors** describe why a concrete HTTP operation could not be completed as requested.
- **Asynchronous technical processing errors** describe why previously accepted business processing later resulted in a technical failure.

In practical terms:

- `POST /requests` may fail synchronously with `400`, `403`, or `500`
- a successfully accepted `POST /requests` may later result in a response retrievable through `GET /responses/{responseId}`
- that retrieved response may itself carry `X-FFI-ProcessingResult = ERROR` together with an `ErrorPayload`

Clients and servers **MUST** therefore support both models.

## Federation-Specific Parameters

The following parameters must be defined in the **FFI Federation Configuration Document**:

- **Response Timeout** – Recommended time clients should wait for a synchronous response.
- **Initial Delay** – Time to wait before the first retry attempt begins. Applies unless a `Retry-After` header is provided by the server.
- **Backoff Strategy** – Policy for retry delay intervals, such as exponential backoff.
- **Retry Limit** – Maximum number of retries allowed for retryable synchronous failures such as `500` responses or timeouts.
- **Circuit Breaker Threshold** – Number of consecutive retryable failures after which the client must suspend further attempts to the same API server temporarily.
- **Circuit Breaker Cooldown Period** – Minimum duration the client must wait before resuming requests to the affected server after the circuit breaker is triggered.

### Exponential Backoff Explained

- **`exponential`** – Doubles the delay after each failed attempt.
  _Example_: Wait 2s -> 4s -> 8s -> 16s -> 32s for 5 attempts.
  A maximum delay cap of 60 seconds **SHOULD** be applied. Once reached, the delay **MUST NOT** increase further.
