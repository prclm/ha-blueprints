# Temperature changed controller

[`temperature_changed_controller.yaml`](temperature_changed_controller.yaml)
monitors one Home Assistant temperature sensor and publishes the shared
`prclm_ha_blueprints_temperature_changed` event when the temperature changes by
at least the configured threshold.

The controller is destination-independent. It does not know about the Aqara
TRV or Zigbee2MQTT; a separate hook can consume the event and update a
destination device.

## Inputs

### Source temperature entity

The sensor entity to monitor. The entity must be a Home Assistant `sensor` with
the `temperature` device class.

The controller ignores `unknown`, `unavailable`, empty, non-numeric, and
non-finite source states.

### Change threshold (°C)

The minimum absolute change between consecutive valid sensor states required to
publish an event. The default is `0.3 °C`.

For example, with the default threshold, a change from `20.0` to `20.2` does
not publish an event, while a change from `20.0` to `20.3` does.

## Published event

For a qualifying change, the controller fires:

```text
prclm_ha_blueprints_temperature_changed
```

The event data contains:

```yaml
event_version: 1
source_entity_id: sensor.living_room_temperature
source_device_id: 0123456789abcdef  # included when available
temperature_celsius: 21.3
temperature_fahrenheit: 70.34
```

`source_device_id` is omitted when Home Assistant cannot resolve a device for
the source entity. The event's Home Assistant `time_fired` metadata is the
authoritative event time.

## Install from GitHub

### Feature branch test link

Use this link to import the controller directly from the current feature
branch:

[![Import the temperature changed controller](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fprclm%2Fha-blueprints%2Ffeature%2Faqara-trv-external-temperature%2Fblueprints%2Fcontrollers%2Ftemperature_changed_controller.yaml)

The underlying raw blueprint URL is:

```text
https://raw.githubusercontent.com/prclm/ha-blueprints/feature/aqara-trv-external-temperature/blueprints/controllers/temperature_changed_controller.yaml
```

This link is intended for testing this feature branch. It should be replaced
with a `main` branch link after the controller is merged.

### Standard import

1. Open **Settings → Automations & scenes → Blueprints**.
2. Select **Import Blueprint**.
3. Paste the raw URL:

   ```text
   https://raw.githubusercontent.com/prclm/ha-blueprints/main/blueprints/controllers/temperature_changed_controller.yaml
   ```

4. Select **Preview**, then **Import**.
5. Create an automation from **Controller - Temperature changed event**.
6. Select the source temperature entity and choose a change threshold.
7. Save and enable the automation.

The import URL targets the `main` branch. To test a different branch, replace
`main` with that branch name in the URL.

## Install locally

For a local Home Assistant installation, copy the YAML file into the automation
blueprint directory. Home Assistant expects the namespace directory to be one
level below `config/blueprints/automation`:

```text
config/
└── blueprints/
    └── automation/
        └── temperature/
            └── temperature_changed_controller.yaml
```

For example, from a checkout of this repository:

```sh
cp blueprints/controllers/temperature_changed_controller.yaml \
  /config/blueprints/automation/temperature/temperature_changed_controller.yaml
```

Use the path to your Home Assistant configuration directory if it is not
`/config`. Then either restart Home Assistant or reload automations and
blueprints from the UI before creating the automation.

## Test in Home Assistant

### Verify the event

Create the controller automation, enable it, and open
**Developer tools → Events**. Listen for:

```text
prclm_ha_blueprints_temperature_changed
```

Change the selected sensor by at least the configured threshold. The event
listener should show the event data, including the source entity and both
temperature units.

Stop listening after the test so the event listener does not remain active.

### Verify the automation trace

Open the controller automation and select **Traces**. A qualifying sensor
change should show:

1. the state trigger;
2. the numeric-value validation condition;
3. the threshold check; and
4. the event action.

For an insignificant change, the trace should stop at the threshold check and
no event should be published.

### Test with a temporary event listener automation

To make the event visible as a persistent notification while testing, create a
temporary automation with this trigger:

```yaml
trigger:
  - platform: event
    event_type: prclm_ha_blueprints_temperature_changed
```

Use an action such as:

```yaml
action:
  - service: persistent_notification.create
    data:
      title: Temperature controller event
      message: "{{ trigger.event.data }}"
```

Remove or disable this temporary automation after testing.

## Troubleshooting

- Confirm the selected entity has device class `temperature`.
- Check that the source state is a plain numeric value in the entity's state,
  rather than a value embedded in text.
- Lower the threshold temporarily if the sensor changes in small increments.
- Check the controller automation trace for invalid-state or threshold
  filtering.
- Confirm that the listener uses the exact event type
  `prclm_ha_blueprints_temperature_changed`.
