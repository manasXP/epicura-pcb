# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Epicura is a kitchen appliance with a custom 4-layer PCB (160x90mm) designed in KiCad 9. The **Unified PCB** merges the former Controller and Driver boards into a single board that stacks beneath a Raspberry Pi CM5 IO Board via a 2x20 pin connector (J_CM5).

Key hardware: STM32G474RE MCU, MP1584EN buck converters (24V→12V, 24V→6.5V), TPS54531 (12V→5V), AMS1117-3.3 LDO, ISO1050 isolated CAN transceiver, DRV8876 H-bridge, TB6612FNG dual motor driver, PCF8574AT I2C GPIO expanders, Li-ion battery backup (TP4056 + DW01A + MT3608).

## Repository Structure

- `Unified_v1.3/` — Active KiCad project (`.kicad_pro`, `.kicad_pcb`, `.kicad_prl`)
  - `Schematic/` — Hierarchical schematic sheets: root `Unified_v1.3.kicad_sch`, `Controller.kicad_sch`, `Driver.kicad_sch`, `Connector.kicad_sch`
  - `Schematic_PDF/` — PDF export of full schematic
  - `BoM/` — BOM CSV export (KiCad format with Reference, Qty, Value, Footprint, MPN columns)
- `01-Unified-PCB-Design.md` — Master design document: block diagrams, pin assignments, power trees, connector pinouts, component selection rationale
- `03-PCB-Design-Rules.md` — DRC rules, stackup, track widths, clearances, net classes, JLCPCB limits
- `05-Schematic-Review-v1.1.md` — Detailed schematic review with identified issues (critical/high/moderate)
- `_archive/` — Superseded 2-board (Controller + Driver) design docs

## Board Stackup (4-layer)

| Layer | Function | Copper Weight |
|-------|----------|---------------|
| F.Cu (Top) | Signal + Components | 2 oz |
| In1.Cu | GND Plane (continuous) | 1 oz |
| In2.Cu | Split Power Plane (5V/3.3V, 12V, 24V) | 1 oz |
| B.Cu (Bottom) | Signal + Power Traces | 2 oz |

Surface finish: ENIG. Substrate: FR-4 (Tg ≥ 150°C). Board thickness: 1.6mm.

## Critical Design Constraints

- **CAN isolation slot**: ≥6mm milled cutout under ISO1050 (IEC 60664-1 reinforced isolation). No copper/traces may cross the boundary.
- **Mains relay AC traces**: ≥2.5mm clearance (IEC 60335-1).
- **High-current nets** (LACT, PUMP, SOL, FAN): 1.5mm track width, 0.5mm clearance, 1.0/0.5mm via pad/drill.
- **USB differential pair**: 90Ω impedance, length-matched within 0.5mm.
- **Fabrication target**: JLCPCB (design rules in `03-PCB-Design-Rules.md` section 9).

## Net Classes

| Net Class | Track (mm) | Clearance (mm) | Via Pad/Drill (mm) |
|-----------|-----------|----------------|---------------------|
| Default | 0.20 | 0.20 | 0.6/0.3 |
| SPI, I2C, CAN | 0.25 | 0.20 | 0.6/0.3 |
| Power_3V3, Power_5V | 0.50 | 0.30 | 0.8/0.4 |
| Power_12V | 0.80 | 0.30 | 0.8/0.4 |
| Power_24V | 1.00 | 0.30 | 1.0/0.5 |
| HighCurrent | 1.50 | 0.50 | 1.0/0.5 |

## Design History

The board evolved from a 3-board stack (CM5IO + Controller + Driver) to a 2-board stack (CM5IO + Unified). The `_archive/` docs describe the old split architecture. All current work targets the unified design in `Unified_v1.3/`.

## KiCad

- KiCad 9.0 is installed at `/Applications/KiCad/KiCad.app`
- CLI available at: `/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli`
- Use `kicad-cli` for exports, DRC checks, and file conversions

## Working with This Project

- Use the `kicad` skill for schematic/PCB analysis, DRC checks, net tracing, and design review
- Use the `bom` skill for component sourcing, pricing, and BOM management (coordinates DigiKey, LCSC, Mouser, etc.)
- Use the `jlcpcb` skill for fabrication prep (gerber export, assembly constraints)
- The schematic review (`05-Schematic-Review-v1.1.md`) documents known issues — check it before suggesting fixes that may already be identified
- BOM CSV uses KiCad's grouped format (multiple refs per row, e.g. `"C1,C2,C31"`)
