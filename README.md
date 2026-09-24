# Home Assistant Blueprints

A collection of Home Assistant automation blueprints for my personal setup,
shared here as reusable examples for anyone who finds them useful.

The blueprints in this repository are intended to be:

- easy to import from Home Assistant;
- safe to copy and customize; and
- useful as starting points for new automations.

## Import a blueprint

### Directly from Home Assistant

The demo blueprint can be imported with this link:

[![Import the demo blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fprclm%2Fha-blueprints%2Fmain%2Fblueprints%2Fautomation%2Fha-blueprints%2Fdemo_entity_change.yaml)

After following the link, Home Assistant opens the blueprint import dialog. Review
the blueprint and select **Import**. You can then create an automation from it
under **Settings → Automations & scenes → Blueprints**.

### From the Home Assistant UI

1. Open **Settings → Automations & scenes → Blueprints**.
2. Select **Import Blueprint**.
3. Paste the raw URL of a blueprint, for example:

   ```text
   https://raw.githubusercontent.com/prclm/ha-blueprints/main/blueprints/automation/ha-blueprints/demo_entity_change.yaml
   ```

4. Select **Preview**, then **Import**.

### Manually

Copy a blueprint into the corresponding directory in your Home Assistant
configuration:

```text
config/blueprints/automation/<namespace>/<blueprint>.yaml
```

For example:

```text
config/blueprints/automation/ha-blueprints/demo_entity_change.yaml
```

After copying the file, reload automations or restart Home Assistant.

## Demo blueprint

[`demo_entity_change.yaml`](blueprints/automation/ha-blueprints/demo_entity_change.yaml)
is a small working example. It triggers when a selected entity changes state and
creates a persistent notification showing the old and new state. When creating a
new blueprint, this file demonstrates the basic structure:

```yaml
blueprint:
  name: ...
  description: ...
  domain: automation
  input: ...

trigger: ...
condition: []
action: ...
mode: single
```

The `condition` section is optional, but keeping it in a boilerplate makes it
clear where conditions belong when a new blueprint needs them.

## Repository layout

```text
blueprints/
└── automation/
    └── ha-blueprints/
        └── *.yaml
```

Blueprint files should include a `source_url` pointing to their location in this
repository. This lets Home Assistant show where an imported blueprint came from
and makes future updates easier to discover.

## Event contracts

Controller and hook blueprints communicate through documented Home Assistant
events. The shared event names, payloads, versioning rules,
and expected update behavior are defined in
[`events/general-rules.md`](events/general-rules.md) and
[`events/temperature-changed.md`](events/temperature-changed.md).
