# Changelog – FFI API Specification Repository

This changelog documents updates to the public `ffi` repository, which contains synchronized API specifications from:

- FFI Core API (`core/ffi-core.yaml`)
- FFI Contract API (`contract/ffi-contract.yaml`)
- FFI API Conventions (`conventions/`)

Each repository release is dated (YYYY-MM-DD) and reflects the latest approved changes published by BSAB.

> Note: Individual API specification files follow [Semantic Versioning](https://semver.org/) (e.g., `1.2.0`). The repository itself uses calendar versioning (e.g., `2025-03-15`) to represent public release dates.

---

## [2025-05-13]

**Initial publication of public FFI API specifications**

- Added `core/ffi-core.yaml` (version: `3.2.0`)
- Added `contract/ffi-contract.yaml` (version: `1.0.0`)
- Added `conventions/api-conventions.md` – shared API usage requirements and rules
- Added `configurations/default.yaml` – default federation parameters referenced by conventions
- Added `LICENSE.md` – outlines usage restrictions and implementation terms
- Added `README.md` – describes repository structure and governance model
- Added `GETTING_STARTED.md` – step-by-step usage guide with roles, flows, and conventions
- Added `SECURITY.md` – defines how to report vulnerabilities and security scope

---

## [2025-10-22]

**Incremental publication of public FFI API specifications**

- Added `core/ffi-core.yaml` (version: `3.3.0`)
- Added `conventions/handling-bilateral-contracts.md` – governs the lifecycle for bilateral contracts
- Added `conventions/handling-timestamps-and-lifecycle-events.md` – manage, interpret, and audit the lifecycle of message exchange
- Updated `conventions/handling-api-security.md` – ensuring compliance with authentication, authorization, and encryption
- Updated `conventions/handling-connectivity-check.md` – enables the verification of the connectivity
- Updated `conventions/handling-parallel-major-versions.md` – handle new versions of the API in an efficient and predictable manner
- Updated `conventions/handling-response-polling.md` – handle polling in a structured and scalable manner
- Updated `conventions/handling-schema-validation.md` – handle payload validation in a structured and consistent manner
- Updated `conventions/README.md` – referencing the FFI API conventions
- Updated `configurations/default.yaml` – default federation parameters referenced by conventions
- Updated `README.md` – describes repository structure and governance model
- Updated `GETTING_STARTED.md` – step-by-step usage guide with roles, flows, and conventions
- Updated `CHANGELOG.md` – this document

---

© 2025 BSAB – All rights reserved.