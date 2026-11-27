# FFI API Specifications

This repository contains the specifications for the Financial Flow Infrastructure (FFI) APIs governed by BSAB.

It includes:

- **FFI Core API** – For secure message transport
- **FFI Contract API** – For the exchange and referencing of FFI Contracts
- **API Conventions** – Common rules and guidelines applicable across all APIs

## Folder Structure

- `api/` – Contains the OpenAPI specifications for the Core API (`ffi-core.yaml`) and Contract API (`ffi-contract.yaml`)
- `convention/` – Contains shared conventions such as naming, structures, and common rules
- `configuration/` – Contains federation-specific configuration values used by the conventions
- `schema/` - Contains common message-type-specific schema definitions

## Licensing & Usage

The specifications are made public for transparency. Implementation is restricted to authorized participants with a valid agreement with BSAB.  

See [LICENSE.md](./LICENSE.md) for terms of use.

## Versioning

This repository is synchronized from internal source repositories.  
A new release is made in this repository every time a change is published in the Core API, Contract API or in the API conventions.  

See [CHANGELOG.md](./CHANGELOG.md) for details.

## Getting Started

To understand how to use the FFI APIs, including the correct order of calls, roles (Client vs Respondent), and conventions to follow:

See [GETTING_STARTED.md](./doc/GETTING_STARTED.md) for details.

## References

Some FFI API conventions rely on federation-specific configuration values. These are stored in:

- [`/configuration/default.yaml`](./configuration/default.yaml) – Default parameters used unless overridden by a federation
- Convention documents will refer to these values where applicable (e.g. polling intervals, transition periods)

See [Federation Configurations](./configuration/) for format, examples, and interpretation guidelines.

## Contact

For questions, contact ffisecretariat@bankinfrastruktur.se.

---

© 2025 BSAB – All rights reserved.
