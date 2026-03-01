# Irrigation & LED Automation — Sonoff 4CH Pro (Ogrodek)

## Overview

Automated irrigation system for a ~23 m² lawn, small trees, and flower pots, plus LED accent lighting.  
All logic runs **locally on Tasmota** — no MQTT, no Node-RED, no cloud.

**Device:** Sonoff 4CH Pro @ `192.168.20.68`  
**Firmware:** Tasmota 15.3.0  
**Location:** Wieliczka, Poland (49.9833°N, 20.0667°E)

---

## Relay Mapping

| Relay | Name | Purpose |
|-------|------|---------|
| 1 | Trawnik | Lawn watering |
| 2 | Krzaczki | Micro trees / bushes |
| 3 | Doniczki | Flower pots |
| 4 | LED ogrodek | Garden LEDs |

---

## Configuration

### 1. Auto-Off Timers (PulseTime)

Each watering relay automatically turns off after a set duration, regardless of how it was triggered (manual, scheduled, or HTTP).

PulseTime values > 111 are interpreted as `(value - 100)` seconds.

| Relay | Duration | PulseTime | Formula |
|-------|----------|-----------|---------|
| 1 (Trawnik) | 210 s (3 min 30 s) | 310 | 210 + 100 |
| 2 (Krzaczki) | 180 s (3 min) | 280 | 180 + 100 |
| 3 (Doniczki) | 120 s (2 min) | 220 | 120 + 100 |

**Console command:**
```
Backlog PulseTime1 310; PulseTime2 280; PulseTime3 220
```

### 2. Power-On State

All relays start **OFF** after a power outage or reboot. Timers will still fire at their scheduled times.

```
PowerOnState 0
```

> `PowerOnState` is a **global** setting (applies to all relays). Per-relay boot behavior is not natively supported, but can be achieved via Rules if needed.

### 3. Location (for sunrise-based scheduling)

Set to Wieliczka, Poland:

```
Backlog Latitude 49.9833; Longitude 20.0667
```

### 4. Enable Timers

```
Timers 1
```

### 5. Scheduled Tasks

#### Timer1 — LEDs OFF at 00:30

```
Timer1 {"Enable":1,"Mode":0,"Time":"00:30","Window":0,"Days":"1111111","Repeat":1,"Output":4,"Action":0}
```

#### Timer2–4 — Morning watering (sunrise-based, staggered by 5 min)

Timers use **Mode 1 (sunrise)** with negative offsets, so watering starts before sunrise and adjusts automatically by season. Staggered start times avoid overlapping water pressure on the shared supply.

| Timer | Relay | Starts | Auto-off after | Gap to next |
|-------|-------|--------|----------------|-------------|
| Timer2 | 1 (Trawnik) | sunrise − 15 min | 210 s (3 min 30 s) | ~1.5 min |
| Timer3 | 2 (Krzaczki) | sunrise − 10 min | 180 s (3 min) | ~2 min |
| Timer4 | 3 (Doniczki) | sunrise − 5 min | 120 s (2 min) | — |

```
Backlog Timer2 {"Enable":1,"Mode":1,"Time":"-00:15","Window":0,"Days":"1111111","Repeat":1,"Output":1,"Action":1}; Timer3 {"Enable":1,"Mode":1,"Time":"-00:10","Window":0,"Days":"1111111","Repeat":1,"Output":2,"Action":1}; Timer4 {"Enable":1,"Mode":1,"Time":"-00:05","Window":0,"Days":"1111111","Repeat":1,"Output":3,"Action":1}
```

> Requires `Latitude` and `Longitude` to be set (done above).

---

## Complete Setup — Copy-Paste Commands

Run these in the Tasmota console at `http://192.168.20.68/cs?`, **one block at a time**:

```
Backlog PulseTime1 310; PulseTime2 280; PulseTime3 220
```

```
PowerOnState 0
```

```
Backlog Latitude 49.9833; Longitude 20.0667
```

```
Timers 1
```

```
Timer1 {"Enable":1,"Mode":0,"Time":"00:30","Window":0,"Days":"1111111","Repeat":1,"Output":4,"Action":0}
```

```
Backlog Timer2 {"Enable":1,"Mode":1,"Time":"-00:15","Window":0,"Days":"1111111","Repeat":1,"Output":1,"Action":1}; Timer3 {"Enable":1,"Mode":1,"Time":"-00:10","Window":0,"Days":"1111111","Repeat":1,"Output":2,"Action":1}; Timer4 {"Enable":1,"Mode":1,"Time":"-00:05","Window":0,"Days":"1111111","Repeat":1,"Output":3,"Action":1}
```

---

## Optional: Fixed 5 AM Watering (instead of sunrise)

If you prefer a fixed schedule instead of sunrise-based, replace Timers 2–4 with `Mode: 0`:

```
Backlog Timer2 {"Enable":1,"Mode":0,"Time":"05:00","Window":0,"Days":"1111111","Repeat":1,"Output":1,"Action":1}; Timer3 {"Enable":1,"Mode":0,"Time":"05:05","Window":0,"Days":"1111111","Repeat":1,"Output":2,"Action":1}; Timer4 {"Enable":1,"Mode":0,"Time":"05:10","Window":0,"Days":"1111111","Repeat":1,"Output":3,"Action":1}
```

---

## Verification

After applying, verify with these console commands:

| Command | What it shows |
|---------|---------------|
| `PulseTime1` | Auto-off duration for relay 1 |
| `PulseTime2` | Auto-off duration for relay 2 |
| `PulseTime3` | Auto-off duration for relay 3 |
| `Timers` | Whether timer subsystem is enabled |
| `Timer1` | LED off schedule |
| `Timer2` / `Timer3` / `Timer4` | Watering schedules |
| `Latitude` / `Longitude` | Location for sunrise calc |
| `PowerOnState` | Boot behavior (0 = all OFF) |
