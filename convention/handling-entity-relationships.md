# Handling Entity Relationships

This section of the FFI API Conventions establishes common principles for how participants manage and interpret relationships between legal entities, technical implementations, and identity artifacts such as ClientIds and bilateral contracts.

It establishes a common understanding of how accountability and traceability are maintained across all FFI communication scenarios, from simple one-to-one integrations to complex multi-entity infrastructures.

## General Principles

- Every API transaction **MUST** be traceable to a single **acting legal entity**, as determined through the binding between the issued access token, its associated client certificate, and the registered OAuth 2.0 **ClientId** within the relevant bilateral relationship.
- Technical implementations **MAY** serve one or more legal entities, provided that each transaction remains traceable to the correct acting legal entity.
- Identity relationships between entities, client certificates, and bilateral contracts **MUST** be deterministic and verifiable.
- Each participant’s identity infrastructure **MUST** integrate according to the FFI’s conventions for trust establishment and entity resolution.

## Core Identity Relationships

| **Component** | **Purpose** | **Relationship to Other Components** |
|----------------|-------------|--------------------------------------|
| **EntityId** | Unique identifier representing a legal entity on the consumer side or provider side of a bilateral relationship. | Acts as the legal anchor for requests, responses, contracts, and audits. In the Contract API, these identifiers are represented by the `consumerId` and `providerId` fields. |
| **Consumer Credential** | The consumer-side client authentication artifact used when obtaining an access token. | In FFI, this is the pre-registered TLS client certificate defined by the security convention. It is bound to the OAuth 2.0 ClientId during client registration and token issuance. |
| **OAuth2 ClientId** | OAuth 2.0 registration used by the consumer side for a specific consumer-provider pair. | **MUST** be bilaterally registered between a consumer EntityId and a provider EntityId; used by the consumer to request tokens within that relationship. In FFI, this identifier is conveyed in API traffic through the `X-FFI-ClientId` header, as defined by the Security Convention, and remains supplementary request metadata. |
| **Bilateral Contract** | Immutable legal and technical agreement between the consumer side and provider side. | **MUST** define which message types may be exchanged within a specific consumer-provider pair; new contract versions replace earlier ones, and existing contracts **MUST NOT** be modified. Each contract references the EntityIds of both provider and consumer, and applies only within that bilateral relationship. |

Technical implementations may serve the consumer side, the provider side, or both, and may operate on behalf of one or more legal entities. Regardless of deployment model, technical implementation boundaries **MUST NOT** replace or obscure the bilateral identity model defined by EntityId, Consumer Credential, OAuth 2.0 ClientId, and Bilateral Contract.

The **EntityId** is issued during central or federation onboarding and uniquely identifies a legal participant. The EntityId is globally unique within the entire FFI network and does not depend on any bilateral relationship.
In the Contract API, the legal entities participating in a bilateral relationship are identified by `consumerId` and `providerId`, both of which are EntityIds.
The **Consumer Credential**, in contrast, is the consumer-side client authentication artifact used in the token request flow. Within FFI, this means the pre-registered TLS client certificate required by the security convention.
The OAuth 2.0 **ClientId**, in contrast, is established bilaterally between two onboarded entities as part of their mutual onboarding process and is used by the consumer side. When FFI API calls are made, this value is typically reflected in the `X-FFI-ClientId` header, but the header itself remains supplementary metadata and does not by itself determine authentication or authorization.

> **Note:**
> Authentication methods are defined by the FFI security convention and, for FFI, client authorization is performed using OAuth 2.0 Client Credentials Flow with Mutual TLS Authentication.

## Example: One Consumer, Multiple Providers

A single public authority (`authority1.se`) initiates information requests toward two separate bank groups:
- **Bank Group A** represents one legal entity (`bank-a.com`).
- **Bank Group B** represents five legal entities (`entity1.bank-b.com` – `entity5.bank-b.com`).

Each relationship between the authority and a bank entity **MUST** be governed by its own bilateral contract and ClientId.
The same client certificate **MAY** be reused across multiple bilateral relationships as long as traceability and authorization remain deterministic.

| Consumer EntityId | Provider EntityId | Base URL | OAuth ClientId | Client Certificate | ContractId |
|--------------------|---------------------|-----------|-----------|-------------|-------------|
| authority1.se | bank-a.com | `https://api.bank-a.com` | client-auth1-bank-a | cred-auth1 | e5ebde5e-... |
| authority1.se | entity1.bank-b.com | `https://api.bank-b.com/entity1` | client-auth1-b1 | cred-auth1 | 45e65787-... |
| authority1.se | entity2.bank-b.com | `https://api.bank-b.com/entity2` | client-auth1-b2 | cred-auth1 | 133f8800-... |
| authority1.se | entity3.bank-b.com | `https://api.bank-b.com/entity3` | client-auth1-b3 | cred-auth1 | 4b70c2ce-... |
| authority1.se | entity4.bank-b.com | `https://api.bank-b.com/entity4` | client-auth1-b4 | cred-auth1 | 5ffbd080-... |
| authority1.se | entity5.bank-b.com | `https://api.bank-b.com/entity5` | client-auth1-b5 | cred-auth1 | e3efd6f0-... |

**Explanation:**

- The authority uses a single client certificate (`cred-auth1`) across multiple bilateral relationships.
- Each Consumer–Provider EntityId pair **MUST** have its own active **ClientId** and **ContractId** at any given point in time.
- The Contract API **MUST** identify the legal entities participating in each bilateral relationship using the corresponding `consumerId` and `providerId` EntityIds.
- The base URL **MUST** clearly identify the serving entity, as part of the server’s contract resolution logic, but **MAY** do so by any deterministic means visible to the client, for example through distinct hostnames or path segments, as long as the client can unambiguously address the correct legal entity.
- The authorization server (token endpoint) **MAY** be shared across entities belonging to the same organization, provided that token issuance remains entity-specific and traceable.
- When a new message exchange is initiated, the request **MUST** be validated against the active bilateral contract for the Consumer–Provider EntityId pair.

In practice, the `EntityId` values identify the legal consumer and provider participating in a bilateral relationship, while the OAuth 2.0 `ClientId` identifies the specific bilateral client registration used by the consumer when interacting with that provider. Together, these identifiers establish the relationship context in which requests are made and received. The applicable bilateral contract is therefore the one bound to that consumer-provider pair, and it defines which message types and behaviors are valid within that relationship.

## Federation-Specific Parameters

This convention does not have any federation-specific parameters.
