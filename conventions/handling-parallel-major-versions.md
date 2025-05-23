# Handling Parallel Major Versions

By following this section of the General FFI API Conventions, we ensure that all participants in the FFI network can handle API versions in a structured and scalable way, contributing to long-term stability and interoperability within the network.

## General Principles

- The purpose of this section is to ensure that all participants in all FFI federations can handle new versions of the API in an efficient and predictable manner.
- This section applies to all participating entities (banks, authorities, etc.) connected to the FFI network.
- Each major version of the API should be reflected in the URLs to clearly distinguish between different versions (e.g., `/v1/`, `/v2/`, etc.).
- Certain parameters, such as the **transition period** and **notification period**, are **federation-specific values** and are referenced internally in this document (see [Federation-Specific Parameters](#federation-specific-parameters)).

## Maximum Number of Supported Parallel Versions

- Each entity is required to support a maximum of two parallel major versions of the API:
  - The current version (latest major version).
  - The previous version (the prior major version).
- When a new major version (e.g., `v3`) is introduced, a transition period begins to phase out the oldest version (e.g., `v1`).

## Transition Period and Decommissioning

- Upon the release of a new major version, there will be a transition period of **[[Transition Period]](#federation-specific-parameters)** during which both the new and previous versions are supported in parallel.
- After the transition period, it is permissible to stop supporting the oldest version according to the following schedule:

  | Major Version | Status        | Transition Period |
  |--------------|--------------|-------------------|
  | v1           | Phasing out  | **[[Transition Period]](#federation-specific-parameters)** |
  | v2           | Current version | Fully supported |
  | v3           | New version  | In introduction |

## Documentation and Communication

- Prior to the planned release of a new major version, all participants must be notified at least **[Notification Period]** in advance (see [Federation-Specific Parameters](#federation-specific-parameters)).
- Any changes to the API must be documented and made available to all relevant parties through:
  - Version-specific documentation (release notes).
  - Migration guides to facilitate the transition to new versions.

## Handling of Minor Changes

- Minor changes (minor and patch versions) must always be backward-compatible and should not require consumers to migrate to a new major version.
- Examples of minor changes:
  - Adding new fields (without making existing fields mandatory).
  - Enhancements to existing functionality without altering existing contracts.

## Exceptions and Exemptions

- In special circumstances, an entity may apply for an exemption from the requirement to support two parallel versions, e.g., due to technical difficulties or resource constraints.
- Exemption requests will be reviewed and decided upon by the FFI Federation Owner.

## Incident Management

- If an entity fails to meet the requirements for supporting parallel versions according to this section, an incident management process will be initiated, and a remediation plan will be developed.
- Repeated violations may result in sanctions.

## Federation-Specific Parameters

Certain parameters in this document are defined at the **federation level** and must be specified in the **FFI Federation Configuration Document**. These parameters include:

- **Transition Period:** The duration during which the previous major version remains supported after a new major version is released.
- **Notification Period:** The minimum time before a new major version release that all participants must be notified.
- **Centralized Testing Environment:** Whether the federation provides a shared testing environment or if each implementor is responsible for their own.

For details on these parameters, refer to the **FFI Federation Configuration Document**.
