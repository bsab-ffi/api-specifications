# Handling Message Validation Based on Schemas

By following this section of the General FFI API Conventions, we ensure that all participants in the FFI network handle payload validation in a structured and consistent manner, ensuring compliance with bilateral contracts and maintaining interoperability.

## General Principles

- The purpose of this section is to regulate how payload validation is performed based on schemas defined in bilateral contracts.
- This section applies to all participating entities (banks, authorities, etc.) connected to the FFI network.
- Payloads must conform to the schema definitions agreed upon in bilateral contracts and must be validated accordingly.
- Contracts are bilaterally accessed through the **FFI Contract API**, ensuring that all parties rely on a unified and consistent source of truth for message validation.
- Schema access is facilitated through the **Schema Registry**, details of schema access is specified by the [Schema Distribution Mechanism](#federation-specific-parameters).

## Schema-Based Validation

- Payloads exchanged in the FFI network **MUST** be validated against the schema version specified in the MessageType Schema Version HTTP header, using the corresponding schema.
- Implementers **MUST NOT** make assumptions about schema definitions outside of what is explicitly provided in the bilateral contract.
- Validation **MUST** be performed on both the **client side** (as a requestor) and **server side** (as a respondent):
  - **Client Side:**
    - Validate outbound requests before sending them.
    - Validate received responses before processing them.
  - **Server Side:**
    - Validate received requests before processing.
    - Validate responses before publishing them.
- Validation errors **MUST** result in an appropriate API error response, with details specifying the validation failure.
- Versioning of schemas must be respected, ensuring that the **MessageTypeSchemaVersion** is correctly referenced in API interactions.
- All communication based on a request message **MUST** be validated against the schema version indicated in the Message Type Schema Version header of the original request. This applies to both the request itself and any related responses or error messages.

## Schema Complexity and Hierarchy

- Some schemas are standalone, containing all validation rules within a single file.
- Other schemas are hierarchical, meaning that a root schema references multiple sub-schemas.
- When validating hierarchical schemas, all referenced sub-schemas **MUST** be resolved before validation.

## Enforcement and Compliance

- API endpoints **MUST** reject messages that do not conform to the agreed schema with an appropriate error response.
- Failure to adhere to schema validation rules may trigger an **incident management process**.
- All entities **SHOULD** implement automated schema validation as part of their system integration tests.

## Federation-Specific Parameters

Certain parameters in this document are defined at the **federation level** and must be specified in the **FFI Federation Configuration Document**. These parameters include:

- **Schema Distribution Mechanism:** The official method of distributing schemas (Schema Registry, GitHub, email, etc.).

For details on these parameters, refer to the **FFI Federation Configuration Document**.
