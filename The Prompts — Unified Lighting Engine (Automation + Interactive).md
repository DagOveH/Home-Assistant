# The Prompts — Unified Lighting Engine Documentation

A complete specification for recreating the **Unified Lighting Engine** in Node‑RED + Home Assistant, including automation, interactive control, fallback logic, gradual stepping, and self‑healing.

---

# 1. Goal

Build a **multi‑room, self‑healing, human‑friendly lighting engine** that:

- Automates scenes based on time, workday/weekend, and per‑room policy  
- Supports rich interactive control (buttons, dashboard, voice, app)  
- Applies **gradual** brightness + kelvin changes during automation  
- Applies **instant** changes during interactive control  
- Uses **zigbee_group first**, with **automatic per‑bulb fallback**  
- Detects and corrects drift via **self‑healing**  
- Respects humans via a **grace period** after manual interaction  
- Is fully driven by `global.Scenes` and `global.Times`  
- Is deterministic, scalable, and maintainable  

---

# 2. Context

## 2.1 Core Concepts

### Room
Logical area (e.g., `Office`, `LivingRoom`, `UpstairBath`).

### Scene
Named lighting state per room (e.g., `Morgen`, `Kveld`, `Film`, `Av`).

### Devices
Each scene contains an array of device steps:

```json
{
  "type": "zigbee_group",
  "group_entity_id": "light.office_group",
  "members": ["light.office_1", "light.office_2"],
  "brightness_pct": 40,
  "kelvin": 3200
}
```

Or single entities:

```json
{
  "type": "single",
  "entity_id": "light.office_strip",
  "brightness_pct": 30,
  "kelvin": 3000
}
```

### Globals

#### `global.Scenes`
Contains all scenes and per‑room `_config`.

#### `global.Times`
Defines time‑based scene schedule per room.

#### `global.resolveRoom(name)`
Normalizes room names.

#### `global.homeassistant.homeAssistant.states`
Live HA state tree.

---

## 2.2 Per‑Room `_config` Fields

Each room has:

```json
{
  "human_grace_minutes": 30,
  "automation_policy": "A",
  "max_delta_pct": 20,
  "max_delta_kelvin": 200,
  "verification_brightness_pct_tolerance": 5,
  "verification_kelvin_tolerance": 100
}
```

---

# 3. Expectations

## 3.1 Behavioral Expectations

### Interactive
- Instant changes  
- Updates HA input_select, input_datetime, input_boolean  
- Starts human grace period  
- Self‑healing still runs  
- Fallback still works  

### Automation
- Runs every 5 minutes  
- Skipped during human grace  
- Picks scene via `Times[room]`  
- Applies automation policy A/B/C  
- Uses **Gradual Delta Calculator**  
- Uses **group-first**, fallback per-bulb  
- Sends stepped values to Scene Engine  

### Self‑Healing
- Detects drift  
- Emits `self_heal` message  
- Sets `fallback_<room> = true`  
- Next tick uses per‑bulb fallback  
- Fallback resets after one tick  

---

# 4. Automation Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                          EVERY 5 MINUTES (AUTOMATION TICK)                   │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
                    ┌────────────────────────────────────┐
                    │ Determine Rooms to Evaluate         │
                    └────────────────────────────────────┘
                                      │
                                      ▼
                    ┌────────────────────────────────────┐
                    │ Read Room State + Automation Mode  │
                    └────────────────────────────────────┘
                                      │
                                      ▼
                    ┌────────────────────────────────────┐
                    │ Respect Human Grace Period         │
                    └────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                           SELF-HEALING VERIFICATION                          │
│  - Detect drift                                                              │
│  - Emit self_heal                                                            │
│  - Set fallback_<room> = true                                                │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
                    ┌────────────────────────────────────┐
                    │ Automation Decision Engine          │
                    └────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                     GRADUAL DELTA CALCULATOR (NEW)                           │
│  - Normal mode: per device-type (group-first)                                │
│  - Fallback mode: per-bulb                                                    │
│  - Produces stepped_devices[]                                                 │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
                    ┌────────────────────────────────────┐
                    │ Format Scene Engine Message         │
                    └────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                               SCENE ENGINE                                   │
│  - Apply stepped devices                                                      │
│  - Group-first                                                                │
│  - Per-bulb fallback                                                          │
│  - Reset fallback flag                                                        │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

# 5. Interactive Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                          USER INTERACTION ENTRY POINTS                       │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
        Buttons / Dashboard / Voice / App → Normalized Interactive Message
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         HUMAN GRACE PERIOD (INTERACTIVE)                     │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         UNIFIED SCENE ENGINE (INTERACTIVE)                   │
│  - Instant changes                                                            │
│  - Updates HA state                                                           │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         SELF-HEALING VERIFICATION                            │
│  - Detect drift                                                               │
│  - Set fallback_<room> = true                                                 │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         SCENE ENGINE (SELF-HEAL MODE)                        │
│  - Per-bulb correction                                                        │
│  - Instant                                                                    │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

# 6. How to Recreate the System

## 6.1 Define Globals

### `global.Scenes`
Contains all scenes and per-room `_config`.

### `global.Times`
Contains time-based schedule.

### `global.resolveRoom(name)`
Normalizes room names.

---

## 6.2 Create HA Entities Per Room

- `input_select.<room>_scenes`  
- `input_select.<room>_automation_mode`  
- `input_boolean.<room>_light_state`  
- `input_datetime.<room>_manual_input`  
- `binary_sensor.workday_sensor`

---

## 6.3 Build Automation Flow

1. Inject (every 5 min)  
2. Determine rooms  
3. Read room state  
4. Respect human grace  
5. Self‑healing verification  
6. Automation Decision Engine  
7. Gradual Delta Calculator  
8. Format Scene Engine message  
9. Scene Engine  

---

## 6.4 Build Interactive Flow

Normalize all interactive inputs to:

```json
{
  "room": "Office",
  "scene": "Evening",
  "interactive": true,
  "gradual": false
}
```

Then send directly to Scene Engine.

---

## 6.5 Scene Engine

Handles:

- AUTO  
- NEXT/PREV  
- Explicit scenes  
- Av  
- Stepped devices  
- Fallback logic  
- HA updates  

---

# 7. Summary

This document defines:

- The **architecture**  
- The **behavior**  
- The **flow structure**  
- The **globals**  
- The **interactive logic**  
- The **automation logic**  
- The **fallback system**  
- The **gradual stepping system**  
- The **self‑healing system**  

It is the complete blueprint for recreating your lighting engine from scratch.
