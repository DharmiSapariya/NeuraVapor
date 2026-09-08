# NeuraVapor

**A Closed-Loop, Waste-Heat-to-Water Recovery and Noise-Adaptive Cooling System for Data Centers, with a Live 3D Digital Twin**

Engineering Clinics Capstone — Hardware Engineering, 3rd Year
Budget: ₹6,000–7,000 | Timeline: 10 Weeks

---

## Table of Contents

- [Overview](#overview)
- [The Problem](#the-problem)
- [System Architecture](#system-architecture)
- [How It Works](#how-it-works)
- [Subsystems](#subsystems)
- [Bill of Materials](#bill-of-materials)
- [Wiring Reference](#wiring-reference)
- [Build Guide](#build-guide)
- [Software Stack](#software-stack)
- [Budget Breakdown](#budget-breakdown)
- [Project Timeline](#project-timeline)
- [Regulatory Context](#regulatory-context)
- [Repository Structure](#repository-structure)
- [Status](#status)
- [License](#license)

---

## Overview

**NeuraVapor** is a low-cost, closed-loop hardware prototype that tackles three failure modes of conventional data center cooling simultaneously:

| Failure Mode | NeuraVapor's Answer |
|---|---|
| **Freshwater depletion** | Membrane distillation recovers water from waste heat instead of evaporating it away |
| **Acoustic nuisance** | Adaptive control + passive damping minimize pump/fan noise in real time |
| **Operational opacity** | A live Three.js 3D digital twin visualizes temperature, flow, and noise as they happen |

The system draws heat from a simulated data center load, pushes it through a sealed membrane distillation (MD) chamber, and condenses distilled water on the cold side via a Peltier module — all inside a single recirculating loop with no external water inlet or outlet.

> No external inlet/outlet. This is a closed recirculating system.

---

## The Problem

**Freshwater Depletion.** Evaporative cooling consumes 15–50% of a data center's total water intake. In 2026, a $20.5M settlement in Oregon linked groundwater nitrate contamination directly to data center operations — a landmark case signaling escalating environmental liability for operators.

**Noise Pollution.** Pumps, fans, and generators create persistent community nuisance. 2026 saw lawsuits filed in New Jersey, Michigan, and Mississippi, while a Georgia family's well-water complaints are drawing regulatory and congressional attention.

**Regulatory Momentum is accelerating:**

| Jurisdiction | Regulation | Requirement |
|---|---|---|
| Germany | EnEfG (July 2026) | Mandates 10% energy reuse for new data centers, rising to 20% by 2028 |
| Virginia, USA | HB323 | First US law mandating data center waste-heat reuse |
| Oregon, USA | 2026 Settlement | $20.5M — establishes direct financial liability for water externalities |

NeuraVapor is designed as a feasibility prototype against exactly this regulatory backdrop.

---

## System Architecture

The build separates into two physically independent modules connected by a single umbilical cable:

```
┌─────────────────────────────┐         ┌──────────────────────────────┐
│   WET BOX (sealed)          │         │   ELECTRONICS TRAY (open)     │
│                              │  1x PG7 │                                │
│  • MD Chamber (PTFE)        │  gland, │  • Raspberry Pi Zero          │
│  • Water Block               │◄──────► │  • 2x L298D motor drivers    │
│  • Peltier (TEC1-12706)     │ umbilical│ • ADS1115 ADC                │
│  • Resistor heat bank       │  cable  │                                │
│  • 12V pump + reservoir     │         │  Handles: control logic,      │
│  • Cooling fan               │         │  sensor reads, PWM outputs    │
│  • Distillate collector      │         │                                │
└─────────────────────────────┘         └──────────────────────────────┘
```

**Design rationale:** keeping all wet components in one sealed, foam-lined box and all electronics on an open, dry tray means a single point of failure (a leak) can't take out the control hardware, and the tray can be serviced without touching the sealed loop.

---

## How It Works

The recirculation loop, closed and continuous:

```
        ┌─────────────┐
        │  RESERVOIR   │◄───────────────────┐
        │ (Buffer Tank)│                     │
        └──────┬───────┘                     │
               │                              │
               ▼                              │
          ┌─────────┐                         │
          │  PUMP    │                         │
          └────┬─────┘                        │
               │                              │
               ▼                              │
      ┌──────────────────┐                    │
      │   WATER BLOCK      │                    │
      │   (Heated by       │                    │
      │   resistor bank)   │                    │
      └────────┬───────────┘                   │
               │ Hot Side                       │
               ▼                                │
      ┌──────────────────────┐                  │
      │     MD CHAMBER         │                  │
      │  (PTFE membrane,       │                  │
      │   0.45µm pore)         │                  │
      └──────────┬─────────────┘                 │
                 │ Cold Side (Peltier-cooled)     │
                 ▼                                │
        ┌──────────────────┐                      │
        │     PELTIER        │                      │
        │   (TEC1-12706)      │                      │
        └────────┬─────────┘                       │
                 │                                 │
       ┌─────────┴──────────┐                      │
       ▼                    ▼                      │
┌─────────────┐    ┌─────────────────┐             │
│ DISTILLATE   │    │  Cooled water    │─────────────┘
│  COLLECTOR    │    │  returns to      │
│  (Output)     │    │  reservoir       │
└─────────────┘    └─────────────────┘
```

1. The pump circulates water from the reservoir through the water block.
2. The resistor bank heats the water block, simulating a data center thermal load.
3. Hot water flows across the MD chamber's membrane face. Vapor passes through the PTFE membrane while liquid water cannot (hydrophobic membrane distillation).
4. The Peltier module cools the opposite face of the membrane, causing the vapor to condense.
5. Condensed distillate drips into the collector cup — this is the recovered water output.
6. Cooled water on the hot side returns to the reservoir, and the cycle repeats.

---

## Subsystems

### 1. Thermal Recovery
Membrane distillation converts waste heat into recovered water in a closed loop. The MD chamber is built from two acrylic panels bolted around a 25mm PTFE membrane (0.45µm pore size), sealed with silicone.

### 2. Noise Adaptation
Adaptive control minimizes acoustic noise from pumps, fans, and cooling components. A KY-038 sound sensor feeds real-time acoustic data back into the control loop, which can throttle pump/fan PWM in response. Passive damping (acoustic foam lining, rubber vibration mounts under the pump) handles the baseline noise floor.

### 3. Digital Twin
A live Three.js 3D dashboard visualizes real-time temperature, flow, and noise metrics, rendering an animated 3D model of the physical rig with live sensor overlays.

---

## Bill of Materials

<details>
<summary><strong>Wet box structure</strong></summary>

| Item | Spec |
|---|---|
| ABS junction box | 280×190×130mm, clear PC cover |
| Acrylic sheet | 3mm, 6×6" (4 pcs — 2 working + 2 spare) |
| PTFE membrane | 0.45µm pore, 25mm diameter (pack of 10) |
| M3 nut/bolt/washer kit | — |
| Silicone sealant | Water-safe (e.g. Fevicol Feviseal) |
| Silicone tubing | 6mm ID / 8mm OD |
| PG7 cable gland | ×1 |

</details>

<details>
<summary><strong>Cooling / heat assembly</strong></summary>

| Item | Spec |
|---|---|
| Aluminum water block | 40×40mm |
| Aluminum heat sink | 40×40×20mm + thermal paste |
| Peltier module | TEC1-12706 |
| Cooling fan | 12V, 4010 |
| Resistor heat bank | — |
| Submersible water pump | 12V DC |
| Vibration pad | For pump isolation |

</details>

<details>
<summary><strong>Sensors</strong></summary>

| Item | Qty |
|---|---|
| DS18B20 waterproof temp probe | ×2 |
| Water flow sensor | ×1 |
| Sound sensor module (KY-038) | ×1 |
| Water level sensor | ×1 |

</details>

<details>
<summary><strong>Electronics (tray side)</strong></summary>

| Item | Qty / Spec |
|---|---|
| Raspberry Pi Zero | ×1 |
| L298D motor driver | ×2 |
| ADS1115 ADC module | ×1 |
| Pull-up resistor (DS18B20) | 4.7kΩ |
| Voltage divider (flow sensor, if 5V) | 1kΩ + 2kΩ |
| Female-to-female jumper wires | — |
| Breadboard | For bench testing |

</details>

<details>
<summary><strong>Power & misc.</strong></summary>

| Item | Notes |
|---|---|
| Pi 5V adapter | Or USB power bank for untethered runs |
| 12V wall adapter | Or 12V battery pack |
| Acoustic foam panels | + spray adhesive (not silicone) |
| Heat shrink tubing | Optional |
| Distilled water | For initial fill/testing |

</details>

> **Note on the project proposal deck:** the original capstone submission specifies an **Arduino Uno**–based control board with a Python (`pyserial` + Flask) serial bridge feeding the Three.js dashboard over WebSocket. The build guide above reflects the **as-built** electronics, which moved to a **Raspberry Pi Zero** running the control loop directly. If you're following this repo end-to-end, treat the Pi Zero wiring as authoritative.

---

## Wiring Reference

### Shared Ground
Pi GND, 12V supply GND, and both L298D driver GNDs join on one common rail.

### Motor Drivers (2× L298D)

Each channel: `IN1 → 3.3V (HIGH)`, `IN2 → GND (LOW)`, `EN → GPIO (PWM control)`

| Load | GPIO (BCM) | Physical Pin |
|---|---|---|
| Water pump | GPIO18 | Pin 12 |
| Peltier | GPIO13 | Pin 33 |
| Resistor bank | GPIO12 | Pin 32 |
| Cooling fan | GPIO19 | Pin 35 |

### DS18B20 ×2 (1-Wire, shared bus)

| Wire | Connection |
|---|---|
| Red | Pi 3.3V |
| Black | Pi GND |
| Yellow | GPIO4 (Pin 7) + 4.7kΩ pull-up to 3.3V |

### Water Flow Sensor

| Wire | Connection |
|---|---|
| Red (VCC) | 5V |
| Black | Pi GND |
| Yellow (Signal) | GPIO17 (Pin 11), via 1kΩ+2kΩ divider if 5V output |

### ADS1115

| Pin | Connection |
|---|---|
| VDD/VREF | Pi 3.3V |
| GND | Pi GND |
| SCL | GPIO3 (Pin 5) |
| SDA | GPIO2 (Pin 3) |
| A0 | Sound sensor analog out |
| A1 | Water level sensor analog out |

---

## Build Guide

The full build proceeds in 10 stages — each bench-tested before moving to the next:

1. **Parts checklist** — confirm every component before starting
2. **Build the MD chamber** — sandwich PTFE membrane between acrylic panels, star-pattern bolt tightening, 24hr sealant cure
3. **Build the cooling assembly** — heat sink, water block, Peltier polarity check *before* final mounting
4. **Build the electronics tray** — mount Pi + drivers + ADS1115 on a rigid open base
5. **Wire the umbilical cable** — single PG7 gland, quick-disconnect terminal block for tray/box separation
6. **Electronics wiring** — breadboard first, then transfer to permanent connections
7. **Bench test** — `i2cdetect -y 1` (expect `48`), check `/sys/bus/w1/devices/` for two `28-xxxxxxxxxxxx` entries
8. **Leak test** — 15+ minutes minimum, fully dry before proceeding
9. **Final wet box assembly** — foam lining, vibration pad, gland seal, multimeter short-check
10. **Power-on checklist** — full pre-flight before first live run

**Full power-on checklist:**

- [ ] DS18B20 ×2 confirmed wired and reading
- [ ] Multimeter: no short between 12V and GND on either driver
- [ ] Peltier polarity re-confirmed after final mounting
- [ ] All GND wires joined at one common point
- [ ] Pull-up resistor on DS18B20 data line
- [ ] Voltage divider on flow sensor signal line (if 5V)
- [ ] ADS1115 wired and detected via `i2cdetect`
- [ ] Cooling loop leak-tested 15+ min, fully dry
- [ ] Umbilical cable connector seated firmly, gland tightened
- [ ] Electronics tray secured (nothing sliding, ports accessible)
- [ ] Nothing electrical sits in a splash zone

---

## Software Stack

The digital twin layer is fully open-source (₹0 contribution to budget):

| Component | Stack | Role |
|---|---|---|
| **Python Serial Bridge** | `pyserial` + Flask | Reads sensor data over USB serial, exposes it as a WebSocket stream |
| **Digital Twin Dashboard** | Three.js + WebSocket | Renders a live 3D model with animated water flow, temperature gradients, and noise indicators |
| **Firmware / Control Loop** | C/C++ (Arduino IDE) *or Python (Pi)* | Adaptive control — adjusts pump speed and Peltier power based on temperature/noise feedback |
| **Sensor Libraries** | OneWire (DS18B20), Flask, Three.js, WebSocket | All open-source, no licensing cost, fully reproducible |

**Live data streams:** 2× temperature channels + 1× acoustic noise channel, streamed to the dashboard in real time.

---

## Budget Breakdown

| Category | Estimated Cost (₹) |
|---|---|
| Brain & sensing (controller, sensors) | 1,230 |
| Heat source (resistors, MOSFETs, heatsink) | 480 |
| Cooling loop (pump, tubing, water block) | 870 |
| MD chamber (acrylic/polycarbonate, membrane, Peltier) | 1,250 |
| Cold side (heatsink, fan) | 430 |
| Acoustic treatment & enclosure (foam, mounts, box) | 1,180 |
| Power & safety (adapter, fuse, wiring) | 440 |
| Measurement (cylinder, misc.) | 100 |
| **Hardware Total** | **≈ ₹5,980–6,900** |
| Digital twin software | ₹0 (open-source) |
| **Grand Total** | **≈ ₹6,000–7,000** |

**Cost optimizations applied:** single pump instead of dual, lab-borrowed instruments (multimeter) where available.

---

## Project Timeline

10-week single-semester build, tracked week by week:

| Week | Phase | Key Deliverable |
|---|---|---|
| 1 | Procurement & setup | Controller configured, serial comms tested |
| 2 | Heat source build | Working adjustable heat load via PWM |
| 2–3 | Liquid cooling loop | Leak-free closed loop with pump circulation |
| 3–4 | MD chamber construction | Sealed chamber with Peltier installed |
| 4–5 | Sensor integration | Calibrated temperature and noise sensors |
| 5–6 | Control software | Adaptive control loop working, structured serial output |
| 6–7 | Digital twin dashboard | Live 3D dashboard end-to-end |
| 7 | Enclosure & acoustic treatment | Assembled enclosure with foam and isolators |
| 8 | Full system integration | End-to-end working prototype, all subsystems live |
| 9 | Data collection | Water recovery, noise, and thermal efficiency data |
| 10 | Documentation & demo prep | Final report + rehearsed live demo |

**Milestone grouping:**
- **Weeks 1–4:** Hardware foundation (heat source, cooling loop, MD chamber)
- **Weeks 4–7:** Software & integration (sensors, control logic, digital twin)
- **Weeks 7–8:** Enclosure & acoustic treatment
- **Weeks 8–10:** Validation & delivery

---

## Regulatory Context

NeuraVapor is explicitly positioned as a feasibility prototype against real, active regulation — not a hypothetical use case:

- **EnEfG (Germany, July 2026):** mandates 10% waste-heat reuse for new data centers, rising to 20% by 2028.
- **HB323 (Virginia, USA):** the first US law requiring data center waste-heat reuse.
- **Oregon 2026 settlement:** $20.5M paid out over groundwater nitrate contamination linked to data center operations — establishing that water externalities now carry direct financial risk for operators.

---

## Repository Structure

*(suggested — adapt to your actual repo layout)*

```
neuravapor/
├── README.md
├── docs/
│   ├── build-guide.md
│   ├── wet-part-layout.png
│   └── project-deck.pptx
├── firmware/
│   └── control-loop/
├── software/
│   ├── serial-bridge/        # pyserial + Flask
│   └── digital-twin/         # Three.js dashboard
├── hardware/
│   ├── bom.csv
│   ├── wiring-diagram.png
│   └── tubing-cut-plan.csv
└── data/
    └── test-logs/
```

---

## Status

🔧 **Active build** — hardware assembly in progress. Wet box design finalized; electronics tray wiring and bench testing underway per the build guide above.

---

## License

*Add your chosen license here (e.g. MIT for code, CERN-OHL for hardware designs).*
