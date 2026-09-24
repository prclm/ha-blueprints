# Aqara TRV E1 external temperature

## Goal

Extend an Aqara TRV E1 connected through Zigbee2MQTT with an external
temperature measurement from a SONOFF SNZB-02B connected through Zigbee2MQTT.
The feature uses a controller/hook architecture so that the temperature source
and the destination device remain independent.

## Scope

### Included

- A temperature controller blueprint for a selected Home Assistant temperature
  entity.
- A temperature-changed event following
  [`prclm_ha_blueprints_temperature_changed`](../events/temperature-changed.md).
- An Aqara TRV E1 hook blueprint that consumes the event.
- Writing the external temperature to the Aqara TRV E1 through Zigbee2MQTT.
- Pending/trailing delivery in the hook:
  - deliver the first valid value when no interval is active;
  - replace the pending value while the interval is active;
  - deliver the most recent pending value when the interval expires.
- A configurable hook-specific minimum update interval.
- Switching the Aqara TRV E1 to external-temperature mode whenever a new
  temperature value is successfully delivered.
- Switching the Aqara TRV E1 back to internal-temperature mode after no new
  valid value has been delivered for the configured fallback period.
- A configurable hook-specific fallback period, with a default of one hour for
  the Aqara TRV E1 hook.
- Handling for unavailable and invalid source values.
- Documentation and examples for setup and supported configuration.

### Excluded from the first feature

- Temperature averaging or multiple-source prioritization.
- Temperature offsets or calibration.
- Hysteresis and heating control logic.
- Automatic discovery of compatible Zigbee2MQTT devices.
- General Better Thermostat functionality beyond external-temperature writing.
- A repository-wide default update interval.

## Architecture

```text
SONOFF SNZB-02B
    │
    │ Home Assistant temperature entity
    ▼
Temperature controller
    │
    │ prclm_ha_blueprints_temperature_changed
    ▼
Aqara TRV E1 hook
    │
    │ Zigbee2MQTT write
    ▼
Aqara TRV E1
```

### Controller responsibilities

- Monitor the configured source temperature entity.
- Ignore unavailable, unknown, non-numeric, and non-finite values.
- Apply a configurable source change threshold to suppress insignificant
  sensor noise.
- Publish the documented temperature event.
- Include both Celsius and Fahrenheit representations.
- Include the source entity ID and the source device ID when available.
- Remain independent of any specific TRV or destination device.

### Hook responsibilities

- Consume only the documented temperature event.
- Validate the event version and required payload fields.
- Filter events by the configured source when source filtering is enabled.
  Source filtering is enabled by default for safety and compatibility with
  shared temperature sources.
- Apply the destination device's configurable minimum update interval.
- Retain the newest valid value as the pending value while delivery is
  throttled.
- Write the pending value after the interval expires.
- Select external-temperature mode whenever a new temperature value is
  delivered successfully.
- Monitor the time since the last successfully delivered value.
- Select internal-temperature mode when the fallback period expires without a
  new successfully delivered value.
- Avoid repeatedly writing the internal-temperature mode while the fallback is
  already active.
- Keep pending state isolated to the individual hook instance.
- Surface delivery and validation errors using the repository's established
  Home Assistant behavior; do not silently report failed writes as successful.

## Aqara TRV E1 integration

The implementation must verify the exact Zigbee2MQTT interface of the Aqara
TRV E1 before the hook is finalized:

- the Zigbee2MQTT device `set` topic;
- the external-temperature property name;
- the property or command that selects internal versus external temperature;
- the accepted unit and numeric range;
- the required precision or rounding;
- the acknowledgment or failure behavior after a write.

The hook must use the verified device interface rather than assuming that
Home Assistant's climate target-temperature service writes the external
temperature.

## Configuration decisions

- The event namespace is `prclm_ha_blueprints_`.
- The event name is `prclm_ha_blueprints_temperature_changed`.
- The initial event contract version is `1`.
- `source_entity_id` is required for measurement events.
- `source_device_id` is optional when Home Assistant provides it.
- `temperature_celsius` and `temperature_fahrenheit` are both required and
  represent the same temperature.
- No timestamp is duplicated in the payload; Home Assistant's `time_fired`
  metadata is authoritative.
- Source filtering is a hook-level configuration and defaults to enabled.
  When enabled, the hook only accepts events whose `source_entity_id` matches
  the configured source entity.
- The hook interval is device-specific and is not defined globally by the
  event contract.
- The Aqara TRV hook selects external-temperature mode after each successful
  temperature delivery.
- The Aqara TRV hook selects internal-temperature mode after one hour without
  a successfully delivered temperature, unless configured otherwise.

## Work checklist

### Contract and design

- [x] Document general event rules.
- [x] Document the temperature-changed event contract.
- [x] Decide the event namespace and event name.
- [x] Decide that event time comes from Home Assistant `time_fired`.
- [x] Define bidirectional Celsius/Fahrenheit conversion.
- [x] Define source entity and optional source device identification.
- [x] Define versioning and unsupported-version behavior.
- [x] Define pending/trailing hook delivery.
- [x] Make the update interval hook/device specific.

### Repository implementation

- [ ] Verify the Aqara TRV E1 Zigbee2MQTT external-temperature interface.
- [ ] Define the controller blueprint inputs and defaults.
- [ ] Implement the temperature controller blueprint.
- [ ] Define the Aqara TRV E1 hook blueprint inputs and device-specific interval.
- [ ] Define the Aqara TRV E1 hook fallback-period input, defaulting to one
  hour.
- [ ] Implement event validation in the hook.
- [ ] Implement pending/trailing delivery in the hook.
- [ ] Implement the Zigbee2MQTT write action.
- [ ] Implement switching to external-temperature mode after successful
  delivery.
- [ ] Implement switching back to internal-temperature mode after the
  fallback period.
- [ ] Test that a new value resets the fallback timer and selects external
  mode.
- [ ] Test that the fallback selects internal mode after the configured period.
- [ ] Test that repeated fallback checks do not repeatedly write internal mode.
- [x] Add source filtering behavior and document its default.
- [ ] Add handling for unavailable, unknown, invalid, and unsupported values.
- [ ] Add examples for the SONOFF SNZB-02B and Aqara TRV E1.
- [ ] Validate the YAML and blueprint structure.
- [ ] Test the controller event payload.
- [ ] Test immediate hook delivery.
- [ ] Test pending-value replacement during throttling.
- [ ] Test delivery of the final pending value after the interval.
- [ ] Test failed destination writes and visible error handling.
- [ ] Update the README with import and setup instructions.

## Open decisions for implementation sessions

- [ ] Decide whether the controller change threshold is measured against the
  last published value or the last observed valid value.
- [x] Decide whether the hook should filter by `source_entity_id` by default.
  Default: yes, source filtering is enabled by default for safety.
- [ ] Decide how the hook should schedule a trailing delivery in Home
  Assistant while preserving pending state across events.
- [ ] Decide how a failed write affects the pending value and interval timer.
- [ ] Decide whether a successful mode switch is required before a temperature
  delivery is considered successful.
- [ ] Decide the exact user-facing notifications or log messages for invalid
  events and failed writes.
