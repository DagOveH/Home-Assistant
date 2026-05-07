┌──────────────────────────────────────────────────────────────────────────────┐
│                          USER INTERACTION ENTRY POINTS                       │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
        ┌────────────────────────────────────────────────────────────────┐
        │  PHYSICAL BUTTONS (IKEA, Hue, Aqara, etc.)                     │
        │  - Single / Double / Hold                                      │
        │  - Brightness up/down                                          │
        │  - Color temp up/down                                          │
        └────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
        ┌────────────────────────────────────────────────────────────────┐
        │  DASHBOARD (HA UI)                                             │
        │  - Scene selection                                              │
        │  - Brightness slider                                            │
        │  - Kelvin slider                                                │
        │  - Room ON/OFF                                                  │
        └────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
        ┌────────────────────────────────────────────────────────────────┐
        │  VOICE (Google/Alexa/Siri)                                     │
        │  - “Turn on Office lights”                                     │
        │  - “Set brightness to 40%”                                     │
        │  - “Make it warmer/cooler”                                     │
        └────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
        ┌────────────────────────────────────────────────────────────────┐
        │  MOBILE APP (HA App)                                           │
        │  - Same controls as dashboard                                  │
        └────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                          INTERACTIVE INPUT NORMALIZATION                      │
│                                                                              │
│  Converts all user actions into a unified message:                           │
│                                                                              │
│      msg.room                                                                │
│      msg.scene (explicit scene OR AUTO)                                      │
│      msg.interactive = true                                                  │
│      msg.gradual = false (interactive = instant)                             │
│                                                                              │
│  Examples:                                                                    │
│      Button → “NEXT”                                                         │
│      Dashboard → “Evening”                                                   │
│      Voice → “AUTO”                                                          │
│      Slider → explicit brightness/kelvin                                     │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         HUMAN GRACE PERIOD (INTERACTIVE)                     │
│                                                                              │
│  When user interacts:                                                        │
│      → Update input_datetime.<room>_manual_input                             │
│      → Automation is suppressed for X minutes (per-room _config)             │
│                                                                              │
│  But:                                                                         │
│      → Self-healing still runs                                                │
│      → Scene Engine still applies scenes instantly                            │
└──────────────────────────────────────────────────────────────────────────────┐
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                           UNIFIED SCENE ENGINE (INTERACTIVE)                 │
│                                                                              │
│  INPUT:                                                                      │
│      msg.room                                                                │
│      msg.scene                                                               │
│      msg.interactive = true                                                  │
│                                                                              │
│  LOGIC:                                                                      │
│      - AUTO → pick time-based scene (Av not allowed)                         │
│      - NEXT/PREV → cycle through scenes                                      │
│      - Explicit scene → use directly                                         │
│      - Interactive always overrides automation                               │
│      - No gradual stepping (instant)                                         │
│                                                                              │
│  OUTPUT:                                                                     │
│      1) light.turn_on/off commands                                           │
│      2) verification message                                                 │
│      3) HA state update                                                      │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         SELF-HEALING VERIFICATION (INTERACTIVE)              │
│                                                                              │
│  Runs AFTER interactive scene is applied:                                    │
│      - Checks each bulb                                                      │
│      - If drift detected → self_heal message                                 │
│      - Sets fallback_<room> = true                                           │
│                                                                              │
│  This ensures:                                                                │
│      - Interactive changes are corrected                                     │
│      - Bulbs that missed the command are fixed                               │
│      - System stays consistent                                               │
└──────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                         SCENE ENGINE (SELF-HEAL MODE)                        │
│                                                                              │
│  If fallback_<room> = true:                                                  │
│      → Apply per-bulb stepped values                                         │
│      → Reset fallback flag                                                   │
│                                                                              │
│  Else:                                                                       │
│      → Apply group commands (zigbee_group)                                   │
│                                                                              │
│  Interactive self-heal is always INSTANT                                     │
└──────────────────────────────────────────────────────────────────────────────┘
