# Drain Switch (GF180MCU, 5 V)

Drain-switch cell for the floating-gate FPAA (Georgia Tech ICE Lab, ORS).
It selects, per row, how the drain lines of the 4x2 floating-gate tile are driven
during run / program mode.

## Circuit (per row, 4 rows: 0–3)
| Switch | Devices | Gate | Connects |
|---|---|---|---|
| switch1 | T-gate | SEL / SELN | Drainline_Direct ↔ mid |
| switch2 | T-gate | PROG (nfet) / RUN (pfet) | mid ↔ Vd_R |
| switch3 | pfet pull-up | SEL | VINJ ↔ Vd_P |
| switch4 | T-gate | SEL / SELN | Drainline_Indirect ↔ Vd_P |

28 devices total: 12 × `nfet_05v0` (W 1.2 µm / L 0.6 µm), 16 × `pfet_05v0` (W 2.4 µm / L 0.6 µm).

## Layout (`layout/`, Magic, `tech gf180mcuD`)
- `DrainSwitchTransistorsTry.mag` – top cell. Height 10.9 µm (y 860–3040 internal units),
  rows 0–1 and rows 2–3 placed side by side to fit the pitch.
  SEL/SELN inputs on the left edge, Vd_R/Vd_P outputs on the right edge.
  Shared vertical rails (M2, full height): Drainline_Direct, Drainline_Indirect, PROG, RUN, VINJ, GND;
  left/right copies strapped on M3 (GND rails tied through substrate taps).
- `nmosx4.mag`, `nmosx2.mag`, `pmosx4v2.mag` – transistor columns.
- `NmosBaseCell.mag`, `PmosBaseCell.mag` – single-finger base cells (PmosBaseCell carries its own n-well).

## Verification (`lvs/`)
- LVS vs. schematic `SWC_Drain` (netgen, gf180mcuD setup): **Circuits match uniquely**, no property errors
  (`lvs/comp.out`, extracted netlist `lvs/DrainSwitchTransistorsTry.spice`).

Run it again:
```
# Magic
extract all
ext2spice lvs
ext2spice -o DrainSwitchTransistorsTry.spice
# shell
netgen-lvs lvs "DrainSwitchTransistorsTry.spice DrainSwitchTransistorsTry" "SWC_Drain.spice SWC_Drain" $PDK_ROOT/gf180mcuD/libs.tech/netgen/gf180mcuD_setup.tcl
```

## Open items
- Confirm DRC count is 0 on the current top cell.
- Move the Vd pins to the exact y positions / metal3 of the 4x2 tile so the cells abut.
- Decide SEL pin order to match the drain decoder.
- Abutment test (drain switch + 4x2, stacked above/below) for DRC.
- Re-check T-gate R_on with pfet L = 0.6 µm (target ≤ 5 kΩ).
