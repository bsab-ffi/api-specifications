# General FFI API Conventions

This repository contains shared conventions and guidelines for all participants implementing the FFI APIs. The purpose of these conventions is to ensure consistency, interoperability, and compliance across the FFI network.

Each convention outlines common requirements and best practices related to areas such as schema validation, security, headers, versioning, and more. All participants in the FFI federations are expected to follow these conventions when designing, implementing, or integrating with the FFI APIs.

## Table of Conventions

Below is a list of the available conventions.

| Convention | Description |
|------------|-------------|
| [Compression](./handling-compression-and-large-responses.md) | Specifies how clients and servers must handle large responses, gzip compression, relevant headers, and applicable error handling. |
| [Connectivity](./handling-connectivity-check.md) | Defines how a standardized `ConnectivityCheck` message-type is used to verify successful onboarding and operational readiness of FFI clients and servers in production. |
| [Polling](./handling-response-polling.md) | Defines polling behavior for the `/responses` endpoint, including frequency limits, best practices, alternatives via WebHooks, and rate limiting enforcement. |
| [Security](./handling-api-security.md) | Describes authentication, authorization, certificate requirements, OAuth flows, and network security practices for secure communication in the FFI API. |
| [Validation](./handling-schema-validation.md) | Defines how payloads must be validated using schema definitions, including versioning and enforcement rules. |
| [Versioning](./handling-parallel-major-versions.md) | Establishes rules for managing multiple major API versions in parallel, including transition periods, deprecation, and federation-specific parameters. |

## Updates and Maintenance

These conventions are maintained by BSAB.
