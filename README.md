# ha-blueprints

Collection of Home Assistant Blueprints.

---

## Blueprints

### Circadian & Brightness-Controlled Room Lighting
**File:** [`blueprints/automation/circadian_brightness_controlled_lighting.yaml`](blueprints/automation/circadian_brightness_controlled_lighting.yaml)

**German name:** *Zirkadiane & Helligkeitsgesteuerte Raumbeleuchtung*

#### What it does

This automation keeps the artificial lighting in a room comfortable without any manual interaction. It continuously adapts two parameters:

| Parameter | Description |
|-----------|-------------|
| **Color temperature** | Follows the sun's elevation throughout the day – warm white at night, shifting to daylight white during the day |
| **Brightness** | Keeps the combined room illuminance (natural + artificial light) within a configurable target range |

#### Color temperature curve

| Sun elevation | Color temperature |
|---------------|-------------------|
| ≤ –6° (deep night) | ~2 200 K – very warm white |
| –6° … 0° (twilight) | ~6 500 K – cool white |
| > 0° (daytime) | 2 700 K → 5 500 K – scales with elevation |

#### Dynamic lux target

The target lux level rises with the sun:

- **Night / evening:** `min_target_lux` (configurable, default 150 lx)
- **Daytime:** up to `min_target_lux + 250 lx`, capped at 500 lx
- A **±30 lx tolerance band** prevents constant micro-adjustments

#### Triggers

| Trigger | Action |
|---------|--------|
| Presence detected | Turns the light ON (if room is too dark) |
| Presence gone (after delay) | Turns the light OFF |
| Sun elevation changes | Updates color temperature while light is on |
| Lux sensor changes | Adjusts brightness to stay within target range |

#### Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `presence_sensor` | `binary_sensor` | – | Occupancy / motion sensor |
| `lux_sensor` | `sensor` (illuminance) | – | Room illuminance sensor |
| `target_light` | `light` | – | Single light or light group |
| `min_target_lux` | number (20–500 lx) | 150 | Minimum desired lux at night |
| `override_switch` | `input_boolean` (optional) | none | When ON, the automation is paused (e.g. gaming mode) |
| `off_delay` | number (0–60 min) | 2 | Minutes before the light turns off after presence gone |
| `initial_brightness_pct` | number (5–100 %) | 40 | Lamp brightness when first switched on |

#### Automation logic (simplified)

```
trigger fires
  │
  ├─ override switch ON?  → do nothing
  │
  ├─ presence gone?       → turn light OFF
  │
  ├─ presence ON, light OFF, room too dark?
  │       → turn light ON at 40 % / calculated Kelvin
  │
  ├─ presence ON, light ON, room over-bright at ≤ 10 % brightness?
  │       → turn light OFF
  │
  └─ presence ON, light ON
          ├─ room too dark  → brightness +15 % (max 100 %)
          ├─ room too bright → brightness –15 % (min 5 %)
          └─ within tolerance, sun moved → update color temperature only
```

#### Import into Home Assistant

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmichigg%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fcircadian_brightness_controlled_lighting.yaml)

Or manually: **Settings → Automations & Scenes → Blueprints → Import Blueprint** and paste the raw URL of the YAML file.
