# Handling Compression and Large Response Messages

By following this section of the FFI API Conventions, we ensure that all participants in the FFI network can efficiently handle large response messages and compression mechanisms, contributing to improved performance and interoperability within the network.

## General Principles

- The purpose of this section is to define how the FFI API handles large response messages and the compression mechanisms available to API consumers.
- This section applies to all participating entities (banks, authorities, etc.) interacting with the FFI API.
- API clients must handle responses exceeding **[[Maximum Response Size Threshold]](#federation-specific-parameters)** according to the compression rules outlined in this document.
- Compression mechanisms ensure that responses are delivered efficiently while maintaining API performance and stability.

## Large Response Handling

- Any response payload exceeding **[[Maximum Response Size Threshold]](#federation-specific-parameters)** must be compressed using **gzip** if the client has explicitly indicated support.
- API clients that support compression should include the following HTTP header in their request:
  
  ```http
  Accept-Encoding: gzip
  ```

- If this header is included, the server will compress responses exceeding **[[Maximum Response Size Threshold]](#federation-specific-parameters)** using gzip and set the following response header:

  ```http
  Content-Encoding: gzip
  ```

- If a response payload exceeds **[[Maximum Response Size Threshold]](#federation-specific-parameters)** and the client does **not** support gzip compression (i.e., does not include `Accept-Encoding: gzip`), the request will be rejected with an HTTP **413 Payload Too Large** status code.

## Compression and Content Encoding

- The FFI Core API supports gzip compression for large responses to optimize data transfer efficiency.
- The server will return the `Content-Encoding: gzip` header only when the response is compressed.
- API clients must correctly decode gzip-compressed responses before processing the response body.

## Error Handling for Large Responses

If an API request results in a response exceeding **[[Maximum Response Size Threshold]](#federation-specific-parameters)** and compression is not accepted by the client:

- The server will return an HTTP **413 Payload Too Large** response.
- The response body will include an error message in the following format:
  
  ```json
  {
    "type": "about:blank",
    "title": "Payload too large",
    "status": 413,
    "detail": "Response payload exceeds 2kB. Enable gzip compression by adding 'Accept-Encoding: gzip' to your request headers."
  }
  ```

## Documentation and Communication

- API consumers should review the FFI API documentation and ensure their clients can handle gzip-compressed responses.
- API implementation teams must document any changes related to compression and large response handling in release notes.

## Testing and Validation

- Clients should test their ability to receive and process gzip-compressed responses before moving to production.
- The FFI Core API provides a sandbox environment where clients can validate their response handling under various payload size conditions.

## Federation-Specific Parameters

Certain parameters in this document are defined at the **federation level** and must be specified in the **FFI Federation Configuration Document**. These parameters include:

- **Maximum Response Size Threshold:** The threshold above which responses require compression.

For details on these parameters, refer to the **FFI Federation Configuration Document**.
