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
   ssh root@192.168.1.134 "cd /config && git pull"
   ```

### Server Commands

```bash
# Pull latest changes to server
ssh root@192.168.1.134 "cd /config && git pull"

# Reload automations (no restart needed)
ssh root@192.168.1.134 "ha core reload"

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
- Test automations after deployment using Developer Tools > Services

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
