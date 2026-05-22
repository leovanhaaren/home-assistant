# Home Assistant Configuration

## Project Overview

This repository contains Home Assistant configuration files managed via git. The configuration runs on a Home Assistant server at `192.168.1.134`.

## Deployment Workflow

**IMPORTANT: All changes require user approval before committing.**

### Change Process

1. **Make changes** - Edit configuration files as needed
2. **Show changes** - Present the diff to the user for review
3. **Get approval** - Wait for explicit user approval before proceeding
4. **Commit** - Auto-commit with a generated descriptive commit message
5. **Push** - Push to the remote repository
6. **Deploy** - Update the server by running:
   ```bash
   ssh root@192.168.1.134 "cd /config && git checkout -- . && git pull"
   ```

### Server Commands

```bash
# Pull latest changes to server (resets HA editor formatting first)
ssh root@192.168.1.134 "cd /config && git checkout -- . && git pull"

# Reload automations (no restart needed)
ssh root@192.168.1.134 'curl -s -X POST http://supervisor/core/api/services/automation/reload -H "Authorization: Bearer $SUPERVISOR_TOKEN"'

# Restart Home Assistant (if reload isn't sufficient)
ssh root@192.168.1.134 "ha core restart"

# Check configuration validity
ssh root@192.168.1.134 "ha core check"

# View logs
ssh root@192.168.1.134 "ha core logs"
```

## File Structure

- `configuration.yaml` - Main configuration file
- `automations.yaml` - All automation definitions
- `automation_toggles.yaml` - Input booleans to enable/disable automations
- `scripts.yaml` - Script definitions
- `scenes.yaml` - Scene definitions
- `secrets.yaml` - Sensitive data (not committed)
- `.storage/` - Lovelace dashboard configs

## Configuration Guidelines

### Automations

- Each automation should have a unique `id` and descriptive `alias`
- Use `automation_toggles.yaml` to create enable/disable switches for automations
- Each automation should have a toggle on the automation dashboard
- Test automations after deployment using Developer Tools > Services, this can take up to a minute to reflect changes.
- **Always make trigger times configurable** — never hardcode times in automations. Define an `input_datetime` helper in `configuration.yaml` and reference it in the trigger (`at: input_datetime.your_helper`). Add the helper to the relevant dashboard section and document it in the Entities Reference at the bottom of this file.

### Dashboard Updates

When adding new automations or configuration, update the relevant dashboards in `.storage/`:

- **`lovelace.automations`** — Add a toggle tile (`type: tile`) for the new `input_boolean` toggle. Group it under an existing section heading if it fits (e.g. "Lighting Automations", "Special Modes"), or add a new section. Each automation should be represented by its toggle here.
- **`lovelace.devices`** — Add tiles for new physical devices (lights, sensors, switches, etc.) to the appropriate section in the "Devices" view (e.g. "Lights", "Motion & Presence"). New views or sections may be added for device categories that don't fit existing ones.

Always present dashboard changes alongside the automation/config diff for user review before committing.

### Guest Mode

`input_boolean.guest_mode` is available for automations that may be disruptive or unwanted when guests are present. **Not every automation needs to check guest mode** — use judgement:

- **Good candidates to disable in guest mode:** presence-triggered automations (welcome/goodbye lights, away detection), personal notifications, automations tied to specific people's routines (bedtime, vitamins), vacuum scheduling.
- **Leave unaffected:** safety automations, ambient/comfort automations (blinds, auto lights), TV mode, outdoor lights.

To add guest mode awareness, include a condition:
```yaml
conditions:
  - condition: state
    entity_id: input_boolean.guest_mode
    state: "off"
```

### YAML Syntax

- Use 2-space indentation
- Validate YAML before committing
- Use `!include` for organizing large configurations

## Entities Reference

### People
- `person.leo`
- `person.kardi`

### Presence Tracking
- `input_datetime.leo_arrived_home`
- `input_datetime.kardi_arrived_home`
- `sensor.leo_presence_duration`
- `sensor.kardi_presence_duration`

### Blinds
- East side covers (morning sun): `cover.rs100_solar_io_palace`, `cover.rs100_solar_io_bathroom`, `cover.sunea40_solar_io_kitchen`
- West side covers (afternoon sun): `cover.rs100_solar_io_bedroom`, `cover.rs100_solar_io_office`, `cover.rs100_solar_io_attic`
- Per-blind luminance sensors follow `sensor.<cover_slug>_<cover_slug>_luminance` (doubly-prefixed, e.g. `sensor.rs100_solar_io_palace_rs100_solar_io_palace_luminance`). Kitchen (SUNEA40) has no luminance sensor.
- Sun-tracking toggle: `input_boolean.sun_blinds`
- Sun-tracking config: `input_number.blinds_luminance_threshold`, `input_number.blinds_east_sunny_position`, `input_number.blinds_east_shaded_position`, `input_number.blinds_west_sunny_position`, `input_number.blinds_west_shaded_position`
- Computed "side is sunny" sensors (5 min on / 10 min off debounce): `binary_sensor.east_side_sunny`, `binary_sensor.west_side_sunny`