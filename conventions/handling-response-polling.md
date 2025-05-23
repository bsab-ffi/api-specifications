# Handling Response Polling

By following this section of the General FFI API Conventions, we ensure that all participants in the FFI network handle polling in a structured and scalable manner, preventing unnecessary load on the system while ensuring timely retrieval of responses.

## General Principles

- The purpose of this section is to regulate how frequently clients can poll for responses using the `GET /[MAJOR-API-VERSION]/v3/responses` endpoint.
- This section applies to all participating entities (banks, authorities, etc.) connected to the FFI network.
- Polling limits are established to balance system efficiency with the need for timely response retrieval.
- Clients **SHOULD** use the **FFI Notification API** as a WebHook-based alternative to polling where supported.

## Polling Frequency Limits

- A client must adhere to the maximum allowed polling frequency defined as **[[Polling Interval]](#federation-specific-parameters)**.
- If a client exceeds the allowed polling frequency, the API server **MAY** return an HTTP `429 Too Many Requests` response with a `Retry-After` header indicating the required wait time before retrying.
- Clients **SHOULD** implement exponential backoff strategies to avoid excessive retries when receiving a `429 Too Many Requests` response.
- Clients **MUST NOT** use parallel polling mechanisms to bypass the frequency limitations.

## Recommended Polling Strategy

- Clients should prioritize receiving responses through asynchronous WebHook notifications via the **FFI Notification API**, where supported, rather than relying on polling.
- If polling is necessary, clients should:
  - Only poll when they have outstanding requests that require responses.
  - Implement a **grace period** after receiving a response before polling again.

## Enforcement and Rate Limiting

- The API provider **MAY** enforce rate limiting by rejecting requests that exceed the allowed polling frequency.
- Violations of the polling convention may trigger an **incident management process** if a participant systematically ignores the limits.
- Persistent non-compliance may result in temporary or permanent restrictions on API access.

## Federation-Specific Parameters

Certain parameters in this document are defined at the **federation level** and must be specified in the **FFI Federation Configuration Document**. These parameters include:

- **Polling Interval:** The minimum time between consecutive polling requests from the same client.

For details on these parameters, refer to the **FFI Federation Configuration Document**.
