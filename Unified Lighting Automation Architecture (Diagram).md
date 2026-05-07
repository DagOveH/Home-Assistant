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
                    │ - currentScene                     │
                    │ - automation_mode (Full/Delvis/Av) │
                    │ - lastInteractive timestamp        │
                    └────────────────────────────────────┘
                                      │
                                      ▼
                    ┌────────────────────────────────────┐
                    │ Respect Human Grace Period         │
                    │ (per-room _config.human_grace)     │
                    └────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                           SELF-HEALING VERIFICATION                          │
│                                                                              │
│  Checks EACH bulb:                                                           │
│    - brightness drift > tolerance?                                           │
│    - kelvin drift > tolerance?                                               │
│                                                                              │
│  If drift detected:                                                          │
│    → emit self_heal message                                                  │
│    → set global fallback flag: fallback_<room> = true                        │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
                    ┌────────────────────────────────────┐
                    │ Automation Decision Engine          │
                    │ - AUTO scene selection              │
                    │ - policy A/B/C                      │
                    │ - skip if unchanged                 │
                    │ - produce target scene              │
                    └────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                     GRADUAL DELTA CALCULATOR (NEW)                           │
│                                                                              │
│  Reads per-room _config:                                                     │
│    - max_delta_pct                                                           │
│    - max_delta_kelvin                                                        │
│    - verification tolerances                                                 │
│                                                                              │
│  NORMAL MODE (group-first):                                                  │
│    - Compute deltas per device type (zigbee_group, wifi_strip)               │
│    - Clamp deltas                                                            │
│    - Produce stepped_devices[]                                               │
│                                                                              │
│  FALLBACK MODE (if fallback_<room> = true):                                  │
│    - Compute deltas PER BULB                                                 │
│    - Use stepped values (smooth fallback)                                    │
│    - Reset fallback flag after one tick                                      │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
                    ┌────────────────────────────────────┐
                    │ Format Scene Engine Message         │
                    │ (passes stepped_devices)            │
                    └────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                               SCENE ENGINE                                   │
│                                                                              │
│  If effectiveScene == "Av":                                                  │
│      → turn_off all devices                                                  │
│                                                                              │
│  Else:                                                                       │
│      If stepped_devices exists:                                              │
│          → use stepped values                                                │
│      Else:                                                                   │
│          → use full scene values                                             │
│                                                                              │
│  Fallback logic:                                                             │
│      If fallback_<room> == true:                                             │
│          → send per-bulb commands                                            │
│          → reset fallback flag                                               │
│      Else:                                                                   │
│          → send zigbee_group + wifi_strip commands                           │
│                                                                              │
│  Outputs:                                                                    │
│      1) light.turn_on/off commands                                           │
│      2) verification message                                                 │
│      3) HA state update (input_select, datetime, boolean)                    │
└──────────────────────────────────────────────────────────────────────────────┘
