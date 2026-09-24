# Event contract general rules

These rules apply to every event shared between controllers and hooks in this
repository.

## Naming

- Event names use the `prclm_ha_blueprints_` prefix.
- Event data uses `snake_case`.
- Each event contract has its own Markdown file in this directory.
- Contracts that describe a measurement or state change must identify its
  source with `source_entity_id`.
- `source_entity_id` contains the Home Assistant entity ID that produced the
  value and is required unless the contract explicitly defines another source
  identifier.
- `source_device_id` contains the Home Assistant device ID when one is
  available. It is optional because not every source entity belongs to a
  device.

## Event time

Home Assistant adds event metadata, including `time_fired`, to every event.
Contracts must use that event metadata as the authoritative time at which the
event was fired. Event payloads must not duplicate it with a second timestamp.

## Values and validation

- Numeric values are represented as numbers, not formatted strings.
- Equivalent values in different units must describe the same
  measurement; no unit is implicitly authoritative unless the event contract
  explicitly says so.
- Consumers must ignore missing, non-numeric, or non-finite values.
- A consumer must not silently substitute a success-shaped default for an
  invalid required value.

## Contract versioning

Every event contract defines an integer `event_version` in its payload. The
initial version is `1`.

- An additive optional field does not require a version change.
- Changing the meaning, type, required status, units, or interpretation of an
  existing field is an incompatible change and requires incrementing
  `event_version`.
- Removing a field is an incompatible change and requires incrementing
  `event_version`.
- Consumers must explicitly support the versions they handle. They must ignore
  unsupported versions and report the reason using the repository's standard
  Home Assistant logging or notification mechanism.
- A new event name is preferred when the semantic meaning of an existing event
  changes substantially.

The contract document must be updated together with any version change and
must describe the fields and behavior for each supported version.
