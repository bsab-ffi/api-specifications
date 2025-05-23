# Federation Configurations

This folder contains the configuration values for federation-specific parameters referenced in the FFI API conventions.

The file `default.yaml` defines baseline values that apply unless a specific federation overrides them in their own configuration. These values are based on input from contributing parties and are intended to provide a practical starting point for new or aligned federations.

## Format and Interpretation

All values are defined as key-value pairs in YAML format. Parameters follow these conventions:

* **Durations** (e.g. `pollingInterval`, `transitionPeriod`, `notificationPeriod`) use the [ISO 8601 duration format](https://en.wikipedia.org/wiki/ISO_8601#Durations):

  * `PT5M` = 5 minutes
  * `P30D` = 30 days

* **Sizes** (e.g. `maximumResponseSizeThreshold`) are expressed as plain integers and assumed to be in **bytes**.

* **Booleans** (e.g. `centralizedTestingEnvironment`) use standard YAML `true` or `false`.

* **Strings** (e.g. `schemaDistributionMechanism`) contain a free-text identifier for the distribution method (e.g. `email`, `github`, `custom`).

  * The value `custom` indicates that a federation uses its own tailored distribution approach, which may include internal tooling, secured APIs, or other mechanisms not listed in the defaults.

## Purpose

The default configuration:

* Acts as a fallback where no federation-specific values are defined.
* Supports conventions such as polling, schema validation, and version transition.
* Ensures consistent interpretation and onboarding across implementations.

## Customization

Federations may override any of these values in their own configuration. The presence of this file does not restrict federation autonomy. This includes the option to define fully customized mechanisms (e.g., for schema distribution) suited to their internal infrastructure and governance.

For details on how each parameter is used, refer to the relevant FFI API Convention documents.
