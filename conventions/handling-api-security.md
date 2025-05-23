# Handling API Security

By following this section of the General FFI API Conventions, we ensure that all participants in the FFI network implement security measures in a structured and consistent manner, ensuring compliance with authentication, authorization, and encryption requirements while maintaining interoperability.

## **FFI Components and Roles**  

The FFI network consists of the following key components:  

- **FFI Client** – An FFI entity acting as a client to request information from an FFI Server.  
- **FFI Server** – An FFI entity providing the FFI Server API to FFI Clients.  
- **FFI Server API** – A messaging API implemented by financial institutions to handle information request services for FFI Clients.  
- **FFI Network** – A peer-to-peer network connecting FFI Clients and FFI Servers.  
- **Notification Service** – A callback endpoint at the FFI Client where the FFI Server can send response status change notifications.  

## Security Requirements

### 1. Client Authentication and Authorization


**Required**

- FFI Servers must rely on trusted and pre-registered X.509 TLS Client Certificates for client authentication.
- FFI Clients must comply with the FFI Server’s requirements for certificate issuance and trusted signing authorities.
- FFI Servers must technically bind the client certificate to the OAuth client registration.
- Client authorization must be performed using OAuth 2.0 Client Credentials Flow with Mutual TLS Authentication ([RFC 8705](https://www.rfc-editor.org/rfc/rfc8705.html)) or Private Key JWT ([RFC 7523](https://www.rfc-editor.org/rfc/rfc7523.html)).
- The scope parameter is required in access token requests. The value must be one or more of the following: `ffi-core`, `ffi-contract`, and `ffi-notification`. Enforcement of scope values per API is optional and determined by each FFI Server implementation.

**Recommended / Optional**

- FFI Clients should use Certificate-Bound Access Tokens (CBATs) and Dynamic Client Registration, as described in ([RFC 8705](https://www.rfc-editor.org/rfc/rfc8705.html)), but these are optional for the current FFI release.
- Access tokens should have a short expiration time (recommended: ≤ 300 seconds).
- Opaque reference tokens as access tokens are recommended for enhanced security, though optional due to increased complexity and network overhead.
- Scope parameters may be used to differentiate access between FFI APIs.

### 2. Server Authentication  

**Required**

- The FFI Server must use a publicly trusted TLS Server Certificate issued by a CA recognized by the CA/Browser Forum.  
- Server certificate management (issuance, renewal, and revocation) is the responsibility of the FFI Server.  

**Recommended / Optional**

- FFI Clients should limit their trust stores to only include CA certificates necessary for secure server certificate validation.  

### 3. Network Security  

**Recommended / Optional**

- Network Access Control Lists (ACLs) (IP whitelisting) may be used to restrict which clients can connect to FFI endpoints, provided this is supported by the client.  
- ACLs must not be used as a primary authentication mechanism, as they are not a secure method of verifying client identity.  

