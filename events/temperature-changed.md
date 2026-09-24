# `prclm_ha_blueprints_temperature_changed`

Published by a temperature controller when its source temperature changes by
at least the controller's configured change threshold.

## Event data

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `event_version` | integer | yes | Contract version. The initial version is `1`. |
| `source_entity_id` | string | yes | Home Assistant entity that produced the measurement. |
| `source_device_id` | string | no | Home Assistant device ID, when available. |
| `temperature_celsius` | number | yes | Temperature represented in degrees Celsius. |
| `temperature_fahrenheit` | number | yes | Temperature represented in degrees Fahrenheit. |

Example:

```yaml
event_type: prclm_ha_blueprints_temperature_changed
event_data:
  event_version: 1
  source_entity_id: sensor.living_room_temperature
  source_device_id: 0123456789abcdef
  temperature_celsius: 21.4
  temperature_fahrenheit: 70.52
```

The two values represent the same temperature. A controller may obtain either
value from its source and calculate the other using the corresponding formula:

```text
temperature_fahrenheit = temperature_celsius * 9 / 5 + 32
temperature_celsius = (temperature_fahrenheit - 32) * 5 / 9
```

Implementations should preserve sufficient precision during conversion and
should only round values when required by the source or destination device.

The event's Home Assistant `time_fired` metadata is the authoritative event
time; no timestamp is included in `event_data`.

Controllers should avoid publishing events for unavailable or invalid source
states. The controller's change threshold is intended to suppress
insignificant sensor noise; rate limiting is a destination concern and belongs
to the hook.

## Hook delivery behavior

Hooks that write the temperature to a destination device must use a
configurable minimum update interval. The interval and any device-specific
default are defined by the hook contract for the destination device.

Hooks use a pending (trailing-update) strategy:

1. The first valid event is delivered immediately when no interval is active.
2. While the interval is active, every valid event replaces the pending value.
3. When the interval expires, the most recently received pending value is
   delivered.
4. If no event arrived during the interval, nothing is delivered.
5. A successful delivery starts the next interval.

The pending value is local to each hook instance. A hook must not modify or
consume events intended for another destination. Invalid or unavailable events
must not start or extend the interval.
