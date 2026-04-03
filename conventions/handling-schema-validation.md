# Handling Message Validation Based on Schemas

By following this section of the General FFI API Conventions, we ensure that all participants in the FFI network handle payload validation in a structured and consistent manner, ensuring compliance with bilateral contracts and maintaining interoperability.

## General Principles

- The purpose of this section is to regulate how payload validation is performed based on schemas defined in bilateral contracts.
- This section applies to all participating entities (banks, authorities, etc.) connected to the FFI network.
- Payloads must conform to the schema definitions agreed upon in bilateral contracts and must be validated accordingly.  
  Validation applies to the structural layer that is available at the time of receipt, while additional validation of inner content is performed during subsequent processing when applicable.
- Contracts are bilaterally accessed through the **FFI Contract API**, ensuring that all parties rely on a unified and consistent source of truth for message validation.
- Schema access is facilitated through the **Schema Registry**, with details specified by the [Schema Distribution Mechanism](#federation-specific-parameters).

## Schema-Based Validation

- Payloads exchanged in the FFI network **MUST** be validated against the schema version specified in the MessageType Schema Version HTTP header, using the corresponding schema.  
  Where the payload consists of multiple structural layers, validation occurs at the appropriate layer during processing.
- Implementers **MUST NOT** make assumptions about schema definitions outside of what is explicitly provided in the bilateral contract.
- Validation **MUST** be performed on both the **client side** (as a requestor) and **server side** (as a respondent):
  - **Client Side:**
    - Validate outbound requests before sending them.
    - Validate received responses before processing them, at the level applicable to the received structure.
  - **Server Side:**
    - Validate received requests before processing, at the level applicable to the received structure.
    - Validate responses before publishing them.
- Validation errors **MUST** result in an appropriate API error response, with details specifying the validation failure, **where synchronous validation is performed (e.g., POST /requests)**.
- If a client detects a validation error in a **polled response** (e.g., via GET /responses):
  - The client **MUST** respond with:
    ```
    X-FFI-AcknowledgeStatus: FAILED
    ```
  - Further handling of such cases is part of **operational processes** and not handled via API error responses.
- Versioning of schemas must be respected, ensuring that the **MessageTypeSchemaVersion** is correctly referenced in API interactions.  
  Where payloads contain more than one structural layer, this version refers to the schema governing the business-relevant content.
- All communication based on a request message **MUST** be validated against the schema version indicated in the MessageType Schema Version header of the original request. This applies to both the request itself and any related responses or error messages.

## Schema Complexity and Hierarchy

- Some schemas are standalone, containing all validation rules within a single file.
- Other schemas are hierarchical, meaning that a root schema references multiple sub-schemas.
- When validating hierarchical schemas, all referenced sub-schemas **MUST** be resolved before validation, at the stage where the relevant structural layer is available.

## Enforcement and Compliance

- API endpoints **MUST** reject messages that do not conform to the agreed schema with an appropriate error response, **where synchronous validation applies**.
- Failure to adhere to schema validation rules may trigger an **incident management process**.
- All entities **SHOULD** implement automated schema validation as part of their system integration tests.

## Federation-Specific Parameters

Certain parameters in this document are defined at the **federation level** and must be specified in the **FFI Federation Configuration Document**. These parameters include:

- **Schema Distribution Mechanism:** The official method of distributing schemas (Schema Registry, GitHub, email, etc.).

For details on these parameters, refer to the **FFI Federation Configuration Document**.
